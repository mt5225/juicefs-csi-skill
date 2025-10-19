---
name: juicefs-csi
description: JuiceFS CSI Driver documentation. Use for Kubernetes Container Storage Interface (CSI) driver, persistent volumes, dynamic provisioning, resource optimization, and container storage management.
---

# JuiceFS CSI Driver Skill

Comprehensive assistance with JuiceFS CSI Driver development and deployment in Kubernetes environments, generated from official documentation.

## When to Use This Skill

This skill should be triggered when:
- **Installing or configuring** JuiceFS CSI Driver in Kubernetes clusters
- **Creating PersistentVolumes** (PV) or PersistentVolumeClaims (PVC) with JuiceFS
- **Setting up StorageClasses** for dynamic provisioning
- **Implementing static or dynamic provisioning** workflows
- **Troubleshooting** CSI Driver issues, Mount Pod problems, or storage connectivity
- **Working with different mount modes** (Mount Pod, Sidecar, or Process mode)
- **Optimizing storage performance** and resource utilization in containerized environments
- **Managing JuiceFS volumes** as persistent storage for Kubernetes workloads
- **Debugging** CSI Controller or CSI Node Service components
- **Integrating JuiceFS** with container orchestration systems

## Key Concepts

### Architecture Components
- **JuiceFS CSI Controller**: StatefulSet that manages volume provisioning and lifecycle
- **JuiceFS CSI Node Service**: DaemonSet that handles mounting volumes on each node
- **Mount Pod**: Dedicated pod where JuiceFS Client runs (default mode)
- **PersistentVolume (PV)**: Kubernetes resource representing storage volume
- **PersistentVolumeClaim (PVC)**: User request for storage resources
- **StorageClass**: Template for dynamically provisioning storage

### Provisioning Modes
1. **Static Provisioning**: Administrator manually creates PVs, users create PVCs to bind them
   - Simpler approach
   - Mounts entire JuiceFS volume or subdirectories
   - Best for simpler deployments or when sharing full volumes

2. **Dynamic Provisioning**: PVs are automatically created from PVCs using StorageClass
   - Scales better for large deployments
   - Each PV gets its own subdirectory in JuiceFS
   - Provides better data isolation between applications

### Mount Modes
- **Mount Pod Mode** (default): JuiceFS runs in dedicated pods managed by CSI Node
  - Mount pod is created per unique volume mount on a node
  - Multiple app pods using same volume share one mount pod
  - Mount pod has hardcoded nodeSelector/nodeName for co-location with app pods
- **Sidecar Mode**: JuiceFS runs as a sidecar container in application pods
  - Each app pod has its own JuiceFS client sidecar
  - Simpler scheduling (single pod decision)
  - Higher resource overhead but better isolation
- **Process Mode**: JuiceFS runs as a process directly on the node
  - JuiceFS runs as systemd service on each node
  - Lowest overhead but requires node-level setup

### CSI Sidecar Containers
Standard Kubernetes CSI components that work with JuiceFS CSI Driver:

1. **external-provisioner**: Watches PVCs and triggers CreateVolume/DeleteVolume calls for dynamic provisioning
2. **external-attacher**: Manages VolumeAttachment objects and calls ControllerPublishVolume/ControllerUnpublishVolume
3. **external-snapshotter**: Handles VolumeSnapshot creation and deletion
4. **external-resizer**: Enables dynamic volume expansion for PVCs
5. **node-driver-registrar**: Registers the CSI driver with kubelet on each node
6. **livenessprobe**: Monitors CSI driver health and exposes liveness endpoint

These sidecars communicate with the JuiceFS CSI driver via Unix domain sockets and follow the CSI specification.

### Mount Pod Resource Management

**Critical Concept:** Mount pods and application pods are **logically coupled** (must be co-located) but **not formally related** in Kubernetes, which creates unique scheduling challenges.

#### The Mount Pod Scheduling Problem

When an application pod is scheduled to a node:
1. Application pod schedules to a node (e.g., node near capacity)
2. CSI Driver creates mount pod with `nodeSelector` for that specific node
3. If node lacks resources, mount pod stays **Pending**
4. Application pod stuck in **ContainerCreating** (waiting for volume)
5. Cluster autoscaler doesn't trigger (mount pod targets specific node)

**Root cause:** Kubernetes scheduler doesn't account for mount pod resource needs when scheduling the application pod.

#### Solution 1: Always Set Resource Requests on Application Pods

**Most important solution** - prevents the problem entirely:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: app
    resources:
      requests:
        memory: "1Gi"     # App needs (512Mi) + Mount pod overhead (512Mi)
        cpu: "500m"       # App needs (300m) + Mount pod overhead (200m)
      limits:
        memory: "2Gi"
        cpu: "1000m"
```

**Why this works:** Scheduler considers total resource needs and won't place app pod on nodes that can't also fit the mount pod.

#### Solution 2: Configure Mount Pod Resources

Set mount pod resource requests via StorageClass:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: juicefs-sc
provisioner: csi.juicefs.com
parameters:
  csi.storage.k8s.io/provisioner-secret-name: juicefs-secret
  csi.storage.k8s.io/provisioner-secret-namespace: default
  csi.storage.k8s.io/node-publish-secret-name: juicefs-secret
  csi.storage.k8s.io/node-publish-secret-namespace: default
  juicefs/mount-cpu-request: "200m"
  juicefs/mount-memory-request: "512Mi"
  juicefs/mount-cpu-limit: "2000m"
  juicefs/mount-memory-limit: "5Gi"
mountOptions:
  - cache-size=2048  # Size affects memory requirements
```

#### Solution 3: Priority Classes (Advanced)

**Use non-preempting priority classes** for mount pods:

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: juicefs-mount-priority
value: 1000
preemptionPolicy: Never  # Critical: prevents eviction cascades
globalDefault: false
description: "Non-preempting priority for JuiceFS mount pods"
```

#### Priority Class Behavior Explained

JuiceFS CSI Driver behavior depends on scheduling mechanism and priority class:

**Scenario A: nodeSelector-based scheduling (default)**
```yaml
spec:
  nodeSelector:
    kubernetes.io/hostname: worker-node-1
```
- Mount pod goes through Kubernetes scheduler
- If node is full, mount pod stays **Pending**
- **CSI Driver cannot intervene** (scheduler owns the pod)
- Only fix: proper resource requests on app pods

**Scenario B: nodeName with non-preempting priority (recommended)**
```yaml
spec:
  nodeName: worker-node-1  # Bypasses scheduler
  priorityClassName: juicefs-mount-priority  # PreemptionPolicy: Never
```
- Mount pod bypasses scheduler, goes directly to node
- Kubelet detects OutOfResource condition
- Priority class prevents evicting other pods
- **CSI Driver detects OutOfResource and removes resource requests**
- Mount pod retries and runs as best-effort (no guarantees)
- **Result:** Mount succeeds without causing evictions

**Scenario C: nodeName with preempting priority (dangerous)**
```yaml
spec:
  nodeName: worker-node-1
  priorityClassName: system-cluster-critical  # PreemptionPolicy: PreemptLowerPriority
```
- Mount pod bypasses scheduler, goes directly to node
- Kubelet detects OutOfResource condition
- **Kubelet evicts lower-priority pods** (possibly the app pod itself!)
- Creates **"creating-deleting" loop**:
  1. App pod scheduled → mount pod created
  2. Mount pod evicts app pod to get resources
  3. App pod deleted → mount pod no longer needed → mount pod deleted
  4. App pod rescheduled → back to step 1 (infinite loop)
- **Result:** Cluster instability, no progress

**Why non-preempting is recommended:**
- Prevents eviction cascades and creating-deleting loops
- Enables CSI Driver to gracefully degrade (remove resource requests)
- Better to run without guarantees than cause cluster-wide instability
- Exposes capacity issues rather than hiding them with evictions

#### Solution 4: Use Sidecar Mode for Resource-Constrained Clusters

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-sidecar
  annotations:
    juicefs/mount-mode: sidecar
spec:
  containers:
  - name: app
    image: myapp:latest
    resources:
      requests:
        memory: "1Gi"
        cpu: "500m"
    volumeMounts:
    - name: data
      mountPath: /data
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: juicefs-pvc
```

**Advantages:** Single scheduling decision (no coupling issues), complete resource accounting
**Disadvantages:** Higher resource overhead (one JuiceFS client per pod)

## Quick Reference

### 1. Check CSI Driver Status

Verify that JuiceFS CSI Driver components are running:

```shell
kubectl -n kube-system get pod -l app.kubernetes.io/name=juicefs-csi-driver
```

Expected output:
```
NAME                       READY   STATUS    RESTARTS   AGE
juicefs-csi-controller-0   2/2     Running   0          141d
juicefs-csi-node-8rd96     3/3     Running   0          141d
```

### 2. Create Secret for JuiceFS Credentials (Static Provisioning)

Store JuiceFS volume credentials as a Kubernetes secret:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: juicefs-secret
  namespace: default
type: Opaque
stringData:
  name: <JUICEFS_NAME>
  metaurl: redis://[:<PASSWORD>]@<HOST>:6379[/<DB>]
  storage: s3
  bucket: https://<BUCKET>.s3.<REGION>.amazonaws.com
  access-key: <ACCESS_KEY>
  secret-key: <SECRET_KEY>
```

### 3. Create PersistentVolume (Static Provisioning)

Define a PV that references the JuiceFS volume:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: juicefs-pv
spec:
  capacity:
    storage: 10Pi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  csi:
    driver: csi.juicefs.com
    volumeHandle: juicefs-pv
    fsType: juicefs
    nodePublishSecretRef:
      name: juicefs-secret
      namespace: default
```

### 4. Create PersistentVolumeClaim

Request storage by creating a PVC that binds to a PV:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: juicefs-pvc
  namespace: default
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 10Pi
  volumeName: juicefs-pv
```

### 5. Use PVC in Application Pod

Mount the JuiceFS volume in your application:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: juicefs-app
  namespace: default
spec:
  containers:
  - name: app
    image: centos
    command: ["/bin/sh"]
    args: ["-c", "while true; do echo $(date -u) >> /data/out.txt; sleep 5; done"]
    volumeMounts:
    - name: data
      mountPath: /data
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: juicefs-pvc
```

### 6. Create StorageClass (Dynamic Provisioning)

Define a StorageClass for automatic PV provisioning:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: juicefs-sc
provisioner: csi.juicefs.com
parameters:
  csi.storage.k8s.io/provisioner-secret-name: juicefs-secret
  csi.storage.k8s.io/provisioner-secret-namespace: default
  csi.storage.k8s.io/node-publish-secret-name: juicefs-secret
  csi.storage.k8s.io/node-publish-secret-namespace: default
reclaimPolicy: Delete
volumeBindingMode: Immediate
```

### 7. Create PVC with Dynamic Provisioning

Request storage using StorageClass (PV created automatically):

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: juicefs-dynamic-pvc
  namespace: default
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 100Gi
  storageClassName: juicefs-sc
```

### 8. Mount Subdirectory (Static Provisioning)

Mount only a specific subdirectory instead of the entire volume:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: juicefs-subdir-pv
spec:
  capacity:
    storage: 100Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  csi:
    driver: csi.juicefs.com
    volumeHandle: juicefs-subdir-pv
    fsType: juicefs
    nodePublishSecretRef:
      name: juicefs-secret
      namespace: default
    volumeAttributes:
      subPath: /my-app-data
```

### 9. Configure Mount Options

Add custom mount options for performance tuning:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: juicefs-sc-optimized
provisioner: csi.juicefs.com
parameters:
  csi.storage.k8s.io/provisioner-secret-name: juicefs-secret
  csi.storage.k8s.io/provisioner-secret-namespace: default
  csi.storage.k8s.io/node-publish-secret-name: juicefs-secret
  csi.storage.k8s.io/node-publish-secret-namespace: default
mountOptions:
  - cache-size=20480
  - buffer-size=2048
  - max-uploads=50
```

### 10. Enable Sidecar Mode

Use sidecar mode when DaemonSets are not allowed:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: juicefs-sidecar-app
spec:
  containers:
  - name: app
    image: centos
    command: ["/bin/sh"]
    args: ["-c", "while true; do echo $(date) >> /data/out.txt; sleep 5; done"]
    volumeMounts:
    - name: data
      mountPath: /data
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: juicefs-pvc
  annotations:
    juicefs/mount-mode: sidecar
```

### 11. Production-Ready Configuration (Complete Example)

Complete production setup incorporating all best practices:

```yaml
---
# 1. Non-preempting priority class for mount pods
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: juicefs-mount-priority
value: 1000
preemptionPolicy: Never  # Critical: prevents eviction cascades
globalDefault: false
description: "Priority for JuiceFS mount pods - non-preempting for stability"

---
# 2. StorageClass with mount pod resource configuration
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: juicefs-production
provisioner: csi.juicefs.com
parameters:
  csi.storage.k8s.io/provisioner-secret-name: juicefs-secret
  csi.storage.k8s.io/provisioner-secret-namespace: kube-system
  csi.storage.k8s.io/node-publish-secret-name: juicefs-secret
  csi.storage.k8s.io/node-publish-secret-namespace: kube-system
  # Mount pod resource requests
  juicefs/mount-cpu-request: "200m"
  juicefs/mount-memory-request: "512Mi"
  juicefs/mount-cpu-limit: "2000m"
  juicefs/mount-memory-limit: "5Gi"
reclaimPolicy: Retain
volumeBindingMode: WaitForFirstConsumer  # Better for topology-aware scheduling
mountOptions:
  - cache-size=2048
  - buffer-size=1024
  - max-uploads=50

---
# 3. Application deployment with proper resource requests
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: app
        image: myapp:v1.0.0
        resources:
          requests:
            # Application actual needs + mount pod overhead
            memory: "1Gi"    # 512Mi (app) + 512Mi (mount pod)
            cpu: "500m"      # 300m (app) + 200m (mount pod)
          limits:
            memory: "2Gi"
            cpu: "1000m"
        volumeMounts:
        - name: data
          mountPath: /data
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
      volumes:
      - name: data
        persistentVolumeClaim:
          claimName: myapp-data

---
# 4. PersistentVolumeClaim using production StorageClass
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myapp-data
  namespace: production
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: juicefs-production
  resources:
    requests:
      storage: 100Gi
```

**This configuration ensures:**
- ✅ Mount pods use non-preempting priority (prevents loops)
- ✅ Mount pod resources explicitly configured
- ✅ Application pods have sufficient resource requests
- ✅ WaitForFirstConsumer enables better scheduling
- ✅ Proper health checks for application stability
- ✅ Production-grade mount options

## Reference Files

This skill includes comprehensive documentation in `references/`:

### Core CSI Concepts

#### introduction.md
Complete introduction to JuiceFS CSI Driver covering:
- **Architecture overview**: CSI Controller, CSI Node Service, and Mount Pod design
- **Component relationships**: How PVs, PVCs, Pods, and Mount Pods interact
- **Usage patterns**: Static vs. dynamic provisioning workflows
- **Mount modes**: Mount Pod (default), Sidecar, and Process modes
- **When to use each approach**: Decision guide for different deployment scenarios

### Kubernetes CSI Standards

#### developing.md
Guide for developing CSI drivers for Kubernetes:
- **CSI specification compliance** and Kubernetes integration requirements
- **Controller Service** implementation (CreateVolume, DeleteVolume, ControllerPublish)
- **Node Service** implementation (NodeStageVolume, NodePublishVolume)
- **Identity Service** requirements (GetPluginInfo, Probe)
- **CSI feature gates** and version compatibility matrix
- **Best practices** for production-ready CSI driver development

#### deploying.md
Deployment patterns for CSI drivers in Kubernetes:
- **Recommended deployment architecture** using StatefulSet + DaemonSet
- **RBAC configuration** for CSI components (ServiceAccounts, Roles, RoleBindings)
- **Container images** and version requirements
- **Installation methods** (kubectl, Helm, operators)
- **Multi-cluster** and **multi-tenant** deployment strategies
- **Upgrade procedures** and compatibility considerations

#### csi-driver-object.md
CSIDriver custom resource specification:
- **CSIDriver object** fields and their purposes
- **attachRequired** flag for skip attach feature
- **podInfoOnMount** for passing Pod information to mount
- **volumeLifecycleModes** supporting persistent and ephemeral volumes
- **fsGroupPolicy** for volume ownership management
- **tokenRequests** for projected service account tokens

#### csi-node-object.md
CSINode object and node registration:
- **CSINode object** structure and automatic registration
- **Node topology** labels and their propagation
- **Driver allocatable resources** and capacity tracking
- **Node registration** process via node-driver-registrar
- **Troubleshooting** node registration issues

### Sidecar Containers

#### sidecar-containers.md
Overview of CSI sidecar containers:
- **Purpose and responsibilities** of each sidecar
- **Communication patterns** between sidecars and CSI driver
- **Deployment models** (which sidecars go with Controller vs Node)
- **Version compatibility** and upgrade considerations

#### external-provisioner.md
Dynamic volume provisioning sidecar:
- **CreateVolume** and **DeleteVolume** RPC calls
- **StorageClass parameters** handling
- **PVC annotations** for customization
- **Topology support** for zone-aware provisioning
- **Snapshot restoration** from VolumeSnapshot sources
- **Command-line flags** and configuration options

#### external-attacher.md
Volume attach/detach operations:
- **VolumeAttachment** resource management
- **ControllerPublishVolume** and **ControllerUnpublishVolume** RPCs
- **Multi-attach** prevention and conflict resolution
- **Attach timeout** and retry logic
- **Leader election** for high availability

#### external-snapshotter.md
Volume snapshot management:
- **VolumeSnapshot** and **VolumeSnapshotContent** resources
- **CreateSnapshot** and **DeleteSnapshot** operations
- **Snapshot classes** and default snapshot class
- **Pre-provisioned snapshots** vs dynamic snapshots
- **Snapshot restoration** workflow

#### external-resizer.md
Volume expansion capability:
- **ExpandVolume** RPC implementation
- **Online vs offline expansion** support
- **File system resize** operations
- **PVC expansion** workflow and status tracking
- **Configuration flags** and feature gates

#### node-driver-registrar.md
Node registration sidecar:
- **Kubelet plugin registration** mechanism
- **Unix domain socket** management
- **Node label management** for driver availability
- **Health monitoring** and automatic re-registration
- **Troubleshooting** registration failures

#### livenessprobe.md
Health monitoring for CSI driver:
- **Probe** RPC health checks
- **HTTP endpoint** for Kubernetes liveness probes
- **Timeout configuration** and retry policies
- **Integration** with CSI Node and Controller pods
- **Alert setup** for production monitoring

### Advanced Features

#### secrets-and-credentials.md
Secure credential management:
- **Secret references** in StorageClass and PV specifications
- **Node publish secrets** vs provisioner secrets
- **Credential rotation** strategies
- **Security best practices** for multi-tenant environments
- **Service account tokens** for driver authentication
- **KMS integration** for encryption key management

#### volume-expansion.md
Dynamic volume resizing:
- **Online expansion** without pod restarts
- **Offline expansion** requiring pod recreation
- **File system resize** (ext4, xfs support)
- **Allowable usage limits** and expansion policies
- **PVC expansion workflow** step-by-step
- **Limitations** and error handling

#### topology.md
Zone and region awareness:
- **Topology keys** and label requirements
- **Allowed topologies** in StorageClass
- **Scheduler integration** for topology-aware scheduling
- **Volume binding modes** (Immediate vs WaitForFirstConsumer)
- **Multi-zone** deployment patterns
- **Affinity rules** and failure domain configuration

#### troubleshooting.md
Common CSI driver issues and solutions:
- **Pod stuck in ContainerCreating** - volume mount failures
- **Volume not attaching** - controller plugin issues
- **Permission denied** - secret or RBAC configuration problems
- **Performance issues** - caching and mount option tuning
- **Log collection** from CSI components
- **Debugging techniques** using kubectl and driver logs

Use the `view` command to read any reference file when you need detailed information about CSI architecture, sidecar containers, advanced features, or troubleshooting.

## Working with This Skill

### For Beginners
1. **Start with `introduction.md`** to understand the architecture and components
2. **Learn the difference** between static and dynamic provisioning
3. **Begin with static provisioning** for simpler initial setup
4. **Practice creating** secrets, PVs, PVCs, and pods using the Quick Reference examples
5. **Verify installation** using the CSI Driver status check command

### For Intermediate Users
1. **Implement dynamic provisioning** using StorageClass for scalable deployments
2. **Configure mount options** for performance optimization
3. **Use subdirectory mounting** to organize data and implement multi-tenancy
4. **Explore different mount modes** based on cluster constraints
5. **Set up monitoring** for Mount Pods and CSI components

### For Advanced Users
1. **Optimize resource allocation** for Mount Pods and CSI components
2. **Implement custom StorageClasses** with specific performance profiles
3. **Troubleshoot complex issues** involving CSI Controller and Node Service
4. **Design multi-tenant architectures** with proper isolation and quotas
5. **Integrate with CI/CD pipelines** for automated storage provisioning

### Navigation Tips
- Use the **Quick Reference** section for common code patterns and configurations
- Check **Key Concepts** to understand terminology and architecture
- Read `references/introduction.md` for in-depth explanations and diagrams
- Start with static provisioning examples before moving to dynamic provisioning
- Always verify CSI Driver status before troubleshooting storage issues

## Common Workflows

### Initial Setup
1. Install JuiceFS CSI Driver in your Kubernetes cluster
2. Verify CSI components are running (use Quick Reference example #1)
3. Create Kubernetes secret with JuiceFS credentials (example #2)

### Static Provisioning Workflow
1. Create secret with JuiceFS credentials
2. Create PersistentVolume referencing the secret
3. Create PersistentVolumeClaim binding to the PV
4. Use PVC in application Pod definition
5. Deploy the application

### Dynamic Provisioning Workflow
1. Create secret with JuiceFS credentials
2. Create StorageClass with secret references
3. Create PersistentVolumeClaim referencing the StorageClass
4. Use PVC in application Pod definition
5. Deploy the application (PV created automatically)

### Troubleshooting Mount Pod Issues

#### Mount Pod Stuck in Pending State

**Symptoms:**
- Mount pod in Pending state
- Application pod stuck in ContainerCreating
- Event: `Unable to attach or mount volumes`

**Diagnosis:**
```bash
# Check mount pod status
kubectl get pods -A | grep juicefs-.*-mount

# Describe mount pod to see events
kubectl describe pod <mount-pod-name> -n <namespace>

# Check for resource constraints
kubectl describe node <node-name> | grep -A 10 "Allocated resources"
```

**Common Causes & Solutions:**

1. **Insufficient node resources**
   - **Cause:** Node doesn't have resources for mount pod
   - **Solution:** Add resource requests to application pods (see Mount Pod Resource Management)
   - **Quick fix:** Scale up cluster or free resources on the node

2. **Missing priority class**
   - **Cause:** Mount pod using default priority
   - **Solution:** Configure non-preempting priority class for mount pods

3. **Node selector mismatch**
   - **Cause:** Mount pod nodeSelector doesn't match any node
   - **Solution:** Verify node labels match mount pod's nodeSelector

#### Mount Pod in Creating-Deleting Loop

**Symptoms:**
- Mount pod repeatedly created and deleted
- Application pod cycling between Pending/ContainerCreating/Running/Terminating
- Cluster appears unstable with constant pod churn

**Cause:** Preempting priority class causing eviction cascades

**Solution:**
```bash
# Check mount pod priority class
kubectl get pod <mount-pod-name> -o jsonpath='{.spec.priorityClassName}'

# Check if it's preempting
kubectl get priorityclass <priority-class-name> -o yaml | grep preemptionPolicy
```

If `preemptionPolicy: PreemptLowerPriority` (or not set), change to non-preempting:
```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: juicefs-mount-priority
value: 1000
preemptionPolicy: Never  # Fix: prevent evictions
```

#### General Troubleshooting Steps

1. **Check CSI Driver pod status**
   ```bash
   kubectl -n kube-system get pod -l app.kubernetes.io/name=juicefs-csi-driver
   ```

2. **Verify secret credentials are correct**
   ```bash
   kubectl get secret <secret-name> -n <namespace> -o yaml
   ```

3. **Examine Mount Pod logs**
   ```bash
   kubectl logs <mount-pod-name> -n <namespace>
   ```

4. **Check PVC binding status and events**
   ```bash
   kubectl describe pvc <pvc-name> -n <namespace>
   ```

5. **Review CSI Node Service logs**
   ```bash
   kubectl logs -n kube-system juicefs-csi-node-<pod-id> -c juicefs-plugin
   ```

6. **Check mount pod resource allocation**
   ```bash
   kubectl get pod <mount-pod-name> -o yaml | grep -A 10 resources:
   ```

## Resources

### references/
Organized documentation extracted from official JuiceFS CSI Driver sources:
- Detailed architectural explanations with diagrams
- Complete configuration examples for all scenarios
- Links to original documentation at juicefs.com
- Comprehensive troubleshooting guides
- Performance tuning recommendations

### scripts/
Add helper scripts here for common automation tasks:
- Secret generation scripts
- Volume provisioning automation
- Health check scripts
- Monitoring and alerting helpers

### assets/
Add templates, boilerplate, or example projects here:
- Complete application deployment examples
- Multi-tier application configurations
- Production-ready YAML templates
- Helm chart examples

## Best Practices

### Critical Resource Management (Must Do)

1. **Always set resource requests on application pods**
   - Include overhead for mount pod resources
   - Example: App needs 512Mi → Request 1Gi (app + mount pod overhead)
   - Prevents mount pod scheduling deadlocks
   - Most important best practice for production stability

2. **Configure mount pod resource requests explicitly**
   - Set via StorageClass parameters (`juicefs/mount-cpu-request`, `juicefs/mount-memory-request`)
   - Account for cache size in memory calculations
   - Monitor actual usage and adjust accordingly

3. **Use non-preempting priority classes for mount pods**
   - Set `preemptionPolicy: Never` to prevent eviction cascades
   - Prevents creating-deleting loops
   - Better to expose capacity issues than hide them with evictions

### Operational Best Practices

4. **Use dynamic provisioning** for large-scale deployments to reduce management overhead

5. **Monitor mount pod health** and scheduling behavior
   - Alert on mount pods stuck in Pending state
   - Track mount pod creation latency
   - Monitor resource utilization on mount pods

6. **Use subdirectories** instead of separate volumes for better resource utilization
   - One mount pod can serve multiple app pods using same volume
   - Reduces overall resource consumption

7. **Configure mount options** based on workload characteristics
   - Set appropriate cache-size, buffer-size based on I/O patterns
   - Tune max-uploads for write-heavy workloads
   - Consider read-ahead settings for sequential access

8. **Choose appropriate mount mode for your environment**
   - Mount Pod mode: Best for high pod density, shared caching
   - Sidecar mode: Better for resource-constrained nodes, simpler scheduling
   - Process mode: Lowest overhead but requires node-level setup

### Reliability Best Practices

9. **Reserve node capacity for mount pods**
   - Configure kubelet reserved resources
   - Reserve 10-20% capacity for system and mount pods
   - Prevents nodes from being completely full

10. **Test backup and restore** procedures regularly

11. **Implement proper RBAC** for StorageClass and PVC access control

12. **Use meaningful naming** for PVs, PVCs, and secrets to simplify operations

13. **Document custom configurations** and mount options for team reference

14. **Keep CSI Driver updated** to benefit from bug fixes and performance improvements

### Troubleshooting Best Practices

15. **Establish monitoring for mount pod issues**
   - Alert when mount pods are Pending > 2 minutes
   - Alert on repeated mount pod creation/deletion cycles
   - Monitor OutOfResource events on nodes

16. **Use non-preempting priority classes in production**
   - Avoids cluster instability from eviction cascades
   - Makes capacity issues visible instead of hidden

## Notes

- This skill was automatically generated from official JuiceFS CSI Driver documentation
- Reference files preserve the structure and examples from source docs at juicefs.com
- Code examples include language detection for better syntax highlighting
- Quick reference patterns are extracted from common usage examples in the docs
- JuiceFS CSI Driver implements the Kubernetes CSI specification for container storage
- The driver supports both Community Edition and Enterprise Edition of JuiceFS

## Updating

To refresh this skill with updated documentation:
1. Re-run the documentation scraper with the same configuration
2. The skill will be rebuilt with the latest information from juicefs.com
3. Review changelog for any breaking changes or new features
4. Update your deployments to use new recommended configurations
