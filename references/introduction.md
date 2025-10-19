# Juicefs-Csi - Introduction

**Pages:** 8

---

## Introduction

**URL:** https://juicefs.com/docs/csi/introduction#other-mount-modes

**Contents:**
- Introduction
- Architecture​
- Usage​
  - Static provisioning​
  - Dynamic provisioning​
- Other mount modes​
  - Sidecar mode​
  - Mount by process​

JuiceFS CSI Driver implements the CSI specification, allowing JuiceFS to be integrated with container orchestration systems. Under Kubernetes, JuiceFS can provide storage service to Pods via PersistentVolume.

JuiceFS CSI Driver consists of JuiceFS CSI Controller (StatefulSet) and JuiceFS CSI Node Service (DaemonSet), they can be viewed using kubectl:

By default, CSI Driver runs in Mount Pod mode, in which JuiceFS Client runs in a dedicated Mount Pod, like the architecture shown below:

A dedicated Mount Pod, managed by CSI Node Service, such architecture proves several advantages:

On the same node, a PVC corresponds to a Mount Pod, while Pods using the same PV may share a single Mount Pod. The relationship between different resources:

If Mount Pod mode doesn't suit you, check out other mount modes provided by JuiceFS CSI Driver.

To use JuiceFS CSI Driver, you can create and manage a PersistentVolume (PV) via "Static Provisioning" or "Dynamic Provisioning".

Static provisioning is the simpler approach, which by default mounts the whole JuiceFS volume root into application Pod (also supports mounting subdirectories), the Kubernetes administrator is in charge of creating the PersistentVolume (PV) and JuiceFS Volume Credentials (stored as Kubernetes secret). After that, user will create a PVC binding that PV, and then finally use this PVC in application Pod definition. The relationship between different resources:

Use static provisioning when:

Managing PVs can be wearisome, when using CSI Driver at scale, it's recommended to create PV dynamically via dynamic provisioning, relieving the administrator from managing the PVs, while also achieving application data isolation. Under dynamic provisioning, the Kubernetes administrator will create and manage one or more StorageClass, the user only need to create a PVC and reference it in Pod definition, and JuiceFS CSI Driver will create the corresponding PV for you, with each PV corresponding to a subdirectory inside JuiceFS.

The relationship between different resources:

Taking Mount Pod mode for example, this is the overall process:

By default, CSI Driver runs in Mount Pod mode, which isn't allowed in certain scenarios, other mount mode may come in handy when that happens.

Mount Pod is created by CSI Node, due to CSI Node being a DaemonSet component, if your Kubernetes cluster does not allow DaemonSets (like some Serverless Kubernetes platform), CSI Node will not be able to install, and JuiceFS CSI Driver c

*[Content truncated]*

**Examples:**

Example 1 (shell):
```shell
$ kubectl -n kube-system get pod -l app.kubernetes.io/name=juicefs-csi-driverNAME                       READY   STATUS        RESTARTS   AGEjuicefs-csi-controller-0   2/2     Running       0          141djuicefs-csi-node-8rd96     3/3     Running       0          141d
```

---

## Introduction

**URL:** https://juicefs.com/docs/csi/introduction#static-provisioning

**Contents:**
- Introduction
- Architecture​
- Usage​
  - Static provisioning​
  - Dynamic provisioning​
- Other mount modes​
  - Sidecar mode​
  - Mount by process​

JuiceFS CSI Driver implements the CSI specification, allowing JuiceFS to be integrated with container orchestration systems. Under Kubernetes, JuiceFS can provide storage service to Pods via PersistentVolume.

JuiceFS CSI Driver consists of JuiceFS CSI Controller (StatefulSet) and JuiceFS CSI Node Service (DaemonSet), they can be viewed using kubectl:

By default, CSI Driver runs in Mount Pod mode, in which JuiceFS Client runs in a dedicated Mount Pod, like the architecture shown below:

A dedicated Mount Pod, managed by CSI Node Service, such architecture proves several advantages:

On the same node, a PVC corresponds to a Mount Pod, while Pods using the same PV may share a single Mount Pod. The relationship between different resources:

If Mount Pod mode doesn't suit you, check out other mount modes provided by JuiceFS CSI Driver.

To use JuiceFS CSI Driver, you can create and manage a PersistentVolume (PV) via "Static Provisioning" or "Dynamic Provisioning".

Static provisioning is the simpler approach, which by default mounts the whole JuiceFS volume root into application Pod (also supports mounting subdirectories), the Kubernetes administrator is in charge of creating the PersistentVolume (PV) and JuiceFS Volume Credentials (stored as Kubernetes secret). After that, user will create a PVC binding that PV, and then finally use this PVC in application Pod definition. The relationship between different resources:

Use static provisioning when:

Managing PVs can be wearisome, when using CSI Driver at scale, it's recommended to create PV dynamically via dynamic provisioning, relieving the administrator from managing the PVs, while also achieving application data isolation. Under dynamic provisioning, the Kubernetes administrator will create and manage one or more StorageClass, the user only need to create a PVC and reference it in Pod definition, and JuiceFS CSI Driver will create the corresponding PV for you, with each PV corresponding to a subdirectory inside JuiceFS.

The relationship between different resources:

Taking Mount Pod mode for example, this is the overall process:

By default, CSI Driver runs in Mount Pod mode, which isn't allowed in certain scenarios, other mount mode may come in handy when that happens.

Mount Pod is created by CSI Node, due to CSI Node being a DaemonSet component, if your Kubernetes cluster does not allow DaemonSets (like some Serverless Kubernetes platform), CSI Node will not be able to install, and JuiceFS CSI Driver c

*[Content truncated]*

**Examples:**

Example 1 (shell):
```shell
$ kubectl -n kube-system get pod -l app.kubernetes.io/name=juicefs-csi-driverNAME                       READY   STATUS        RESTARTS   AGEjuicefs-csi-controller-0   2/2     Running       0          141djuicefs-csi-node-8rd96     3/3     Running       0          141d
```

---

## Introduction

**URL:** https://juicefs.com/docs/csi/introduction#dynamic-provisioning

**Contents:**
- Introduction
- Architecture​
- Usage​
  - Static provisioning​
  - Dynamic provisioning​
- Other mount modes​
  - Sidecar mode​
  - Mount by process​

JuiceFS CSI Driver implements the CSI specification, allowing JuiceFS to be integrated with container orchestration systems. Under Kubernetes, JuiceFS can provide storage service to Pods via PersistentVolume.

JuiceFS CSI Driver consists of JuiceFS CSI Controller (StatefulSet) and JuiceFS CSI Node Service (DaemonSet), they can be viewed using kubectl:

By default, CSI Driver runs in Mount Pod mode, in which JuiceFS Client runs in a dedicated Mount Pod, like the architecture shown below:

A dedicated Mount Pod, managed by CSI Node Service, such architecture proves several advantages:

On the same node, a PVC corresponds to a Mount Pod, while Pods using the same PV may share a single Mount Pod. The relationship between different resources:

If Mount Pod mode doesn't suit you, check out other mount modes provided by JuiceFS CSI Driver.

To use JuiceFS CSI Driver, you can create and manage a PersistentVolume (PV) via "Static Provisioning" or "Dynamic Provisioning".

Static provisioning is the simpler approach, which by default mounts the whole JuiceFS volume root into application Pod (also supports mounting subdirectories), the Kubernetes administrator is in charge of creating the PersistentVolume (PV) and JuiceFS Volume Credentials (stored as Kubernetes secret). After that, user will create a PVC binding that PV, and then finally use this PVC in application Pod definition. The relationship between different resources:

Use static provisioning when:

Managing PVs can be wearisome, when using CSI Driver at scale, it's recommended to create PV dynamically via dynamic provisioning, relieving the administrator from managing the PVs, while also achieving application data isolation. Under dynamic provisioning, the Kubernetes administrator will create and manage one or more StorageClass, the user only need to create a PVC and reference it in Pod definition, and JuiceFS CSI Driver will create the corresponding PV for you, with each PV corresponding to a subdirectory inside JuiceFS.

The relationship between different resources:

Taking Mount Pod mode for example, this is the overall process:

By default, CSI Driver runs in Mount Pod mode, which isn't allowed in certain scenarios, other mount mode may come in handy when that happens.

Mount Pod is created by CSI Node, due to CSI Node being a DaemonSet component, if your Kubernetes cluster does not allow DaemonSets (like some Serverless Kubernetes platform), CSI Node will not be able to install, and JuiceFS CSI Driver c

*[Content truncated]*

**Examples:**

Example 1 (shell):
```shell
$ kubectl -n kube-system get pod -l app.kubernetes.io/name=juicefs-csi-driverNAME                       READY   STATUS        RESTARTS   AGEjuicefs-csi-controller-0   2/2     Running       0          141djuicefs-csi-node-8rd96     3/3     Running       0          141d
```

---

## Introduction

**URL:** https://juicefs.com/docs/csi/introduction#usage

**Contents:**
- Introduction
- Architecture​
- Usage​
  - Static provisioning​
  - Dynamic provisioning​
- Other mount modes​
  - Sidecar mode​
  - Mount by process​

JuiceFS CSI Driver implements the CSI specification, allowing JuiceFS to be integrated with container orchestration systems. Under Kubernetes, JuiceFS can provide storage service to Pods via PersistentVolume.

JuiceFS CSI Driver consists of JuiceFS CSI Controller (StatefulSet) and JuiceFS CSI Node Service (DaemonSet), they can be viewed using kubectl:

By default, CSI Driver runs in Mount Pod mode, in which JuiceFS Client runs in a dedicated Mount Pod, like the architecture shown below:

A dedicated Mount Pod, managed by CSI Node Service, such architecture proves several advantages:

On the same node, a PVC corresponds to a Mount Pod, while Pods using the same PV may share a single Mount Pod. The relationship between different resources:

If Mount Pod mode doesn't suit you, check out other mount modes provided by JuiceFS CSI Driver.

To use JuiceFS CSI Driver, you can create and manage a PersistentVolume (PV) via "Static Provisioning" or "Dynamic Provisioning".

Static provisioning is the simpler approach, which by default mounts the whole JuiceFS volume root into application Pod (also supports mounting subdirectories), the Kubernetes administrator is in charge of creating the PersistentVolume (PV) and JuiceFS Volume Credentials (stored as Kubernetes secret). After that, user will create a PVC binding that PV, and then finally use this PVC in application Pod definition. The relationship between different resources:

Use static provisioning when:

Managing PVs can be wearisome, when using CSI Driver at scale, it's recommended to create PV dynamically via dynamic provisioning, relieving the administrator from managing the PVs, while also achieving application data isolation. Under dynamic provisioning, the Kubernetes administrator will create and manage one or more StorageClass, the user only need to create a PVC and reference it in Pod definition, and JuiceFS CSI Driver will create the corresponding PV for you, with each PV corresponding to a subdirectory inside JuiceFS.

The relationship between different resources:

Taking Mount Pod mode for example, this is the overall process:

By default, CSI Driver runs in Mount Pod mode, which isn't allowed in certain scenarios, other mount mode may come in handy when that happens.

Mount Pod is created by CSI Node, due to CSI Node being a DaemonSet component, if your Kubernetes cluster does not allow DaemonSets (like some Serverless Kubernetes platform), CSI Node will not be able to install, and JuiceFS CSI Driver c

*[Content truncated]*

**Examples:**

Example 1 (shell):
```shell
$ kubectl -n kube-system get pod -l app.kubernetes.io/name=juicefs-csi-driverNAME                       READY   STATUS        RESTARTS   AGEjuicefs-csi-controller-0   2/2     Running       0          141djuicefs-csi-node-8rd96     3/3     Running       0          141d
```

---

## Introduction

**URL:** https://juicefs.com/docs/csi/introduction#by-process

**Contents:**
- Introduction
- Architecture​
- Usage​
  - Static provisioning​
  - Dynamic provisioning​
- Other mount modes​
  - Sidecar mode​
  - Mount by process​

JuiceFS CSI Driver implements the CSI specification, allowing JuiceFS to be integrated with container orchestration systems. Under Kubernetes, JuiceFS can provide storage service to Pods via PersistentVolume.

JuiceFS CSI Driver consists of JuiceFS CSI Controller (StatefulSet) and JuiceFS CSI Node Service (DaemonSet), they can be viewed using kubectl:

By default, CSI Driver runs in Mount Pod mode, in which JuiceFS Client runs in a dedicated Mount Pod, like the architecture shown below:

A dedicated Mount Pod, managed by CSI Node Service, such architecture proves several advantages:

On the same node, a PVC corresponds to a Mount Pod, while Pods using the same PV may share a single Mount Pod. The relationship between different resources:

If Mount Pod mode doesn't suit you, check out other mount modes provided by JuiceFS CSI Driver.

To use JuiceFS CSI Driver, you can create and manage a PersistentVolume (PV) via "Static Provisioning" or "Dynamic Provisioning".

Static provisioning is the simpler approach, which by default mounts the whole JuiceFS volume root into application Pod (also supports mounting subdirectories), the Kubernetes administrator is in charge of creating the PersistentVolume (PV) and JuiceFS Volume Credentials (stored as Kubernetes secret). After that, user will create a PVC binding that PV, and then finally use this PVC in application Pod definition. The relationship between different resources:

Use static provisioning when:

Managing PVs can be wearisome, when using CSI Driver at scale, it's recommended to create PV dynamically via dynamic provisioning, relieving the administrator from managing the PVs, while also achieving application data isolation. Under dynamic provisioning, the Kubernetes administrator will create and manage one or more StorageClass, the user only need to create a PVC and reference it in Pod definition, and JuiceFS CSI Driver will create the corresponding PV for you, with each PV corresponding to a subdirectory inside JuiceFS.

The relationship between different resources:

Taking Mount Pod mode for example, this is the overall process:

By default, CSI Driver runs in Mount Pod mode, which isn't allowed in certain scenarios, other mount mode may come in handy when that happens.

Mount Pod is created by CSI Node, due to CSI Node being a DaemonSet component, if your Kubernetes cluster does not allow DaemonSets (like some Serverless Kubernetes platform), CSI Node will not be able to install, and JuiceFS CSI Driver c

*[Content truncated]*

**Examples:**

Example 1 (shell):
```shell
$ kubectl -n kube-system get pod -l app.kubernetes.io/name=juicefs-csi-driverNAME                       READY   STATUS        RESTARTS   AGEjuicefs-csi-controller-0   2/2     Running       0          141djuicefs-csi-node-8rd96     3/3     Running       0          141d
```

---

## Introduction

**URL:** https://juicefs.com/docs/csi/introduction

**Contents:**
- Introduction
- Architecture​
- Usage​
  - Static provisioning​
  - Dynamic provisioning​
- Other mount modes​
  - Sidecar mode​
  - Mount by process​

JuiceFS CSI Driver implements the CSI specification, allowing JuiceFS to be integrated with container orchestration systems. Under Kubernetes, JuiceFS can provide storage service to Pods via PersistentVolume.

JuiceFS CSI Driver consists of JuiceFS CSI Controller (StatefulSet) and JuiceFS CSI Node Service (DaemonSet), they can be viewed using kubectl:

By default, CSI Driver runs in Mount Pod mode, in which JuiceFS Client runs in a dedicated Mount Pod, like the architecture shown below:

A dedicated Mount Pod, managed by CSI Node Service, such architecture proves several advantages:

On the same node, a PVC corresponds to a Mount Pod, while Pods using the same PV may share a single Mount Pod. The relationship between different resources:

If Mount Pod mode doesn't suit you, check out other mount modes provided by JuiceFS CSI Driver.

To use JuiceFS CSI Driver, you can create and manage a PersistentVolume (PV) via "Static Provisioning" or "Dynamic Provisioning".

Static provisioning is the simpler approach, which by default mounts the whole JuiceFS volume root into application Pod (also supports mounting subdirectories), the Kubernetes administrator is in charge of creating the PersistentVolume (PV) and JuiceFS Volume Credentials (stored as Kubernetes secret). After that, user will create a PVC binding that PV, and then finally use this PVC in application Pod definition. The relationship between different resources:

Use static provisioning when:

Managing PVs can be wearisome, when using CSI Driver at scale, it's recommended to create PV dynamically via dynamic provisioning, relieving the administrator from managing the PVs, while also achieving application data isolation. Under dynamic provisioning, the Kubernetes administrator will create and manage one or more StorageClass, the user only need to create a PVC and reference it in Pod definition, and JuiceFS CSI Driver will create the corresponding PV for you, with each PV corresponding to a subdirectory inside JuiceFS.

The relationship between different resources:

Taking Mount Pod mode for example, this is the overall process:

By default, CSI Driver runs in Mount Pod mode, which isn't allowed in certain scenarios, other mount mode may come in handy when that happens.

Mount Pod is created by CSI Node, due to CSI Node being a DaemonSet component, if your Kubernetes cluster does not allow DaemonSets (like some Serverless Kubernetes platform), CSI Node will not be able to install, and JuiceFS CSI Driver c

*[Content truncated]*

**Examples:**

Example 1 (shell):
```shell
$ kubectl -n kube-system get pod -l app.kubernetes.io/name=juicefs-csi-driverNAME                       READY   STATUS        RESTARTS   AGEjuicefs-csi-controller-0   2/2     Running       0          141djuicefs-csi-node-8rd96     3/3     Running       0          141d
```

---

## Introduction

**URL:** https://juicefs.com/docs/csi/introduction#architecture

**Contents:**
- Introduction
- Architecture​
- Usage​
  - Static provisioning​
  - Dynamic provisioning​
- Other mount modes​
  - Sidecar mode​
  - Mount by process​

JuiceFS CSI Driver implements the CSI specification, allowing JuiceFS to be integrated with container orchestration systems. Under Kubernetes, JuiceFS can provide storage service to Pods via PersistentVolume.

JuiceFS CSI Driver consists of JuiceFS CSI Controller (StatefulSet) and JuiceFS CSI Node Service (DaemonSet), they can be viewed using kubectl:

By default, CSI Driver runs in Mount Pod mode, in which JuiceFS Client runs in a dedicated Mount Pod, like the architecture shown below:

A dedicated Mount Pod, managed by CSI Node Service, such architecture proves several advantages:

On the same node, a PVC corresponds to a Mount Pod, while Pods using the same PV may share a single Mount Pod. The relationship between different resources:

If Mount Pod mode doesn't suit you, check out other mount modes provided by JuiceFS CSI Driver.

To use JuiceFS CSI Driver, you can create and manage a PersistentVolume (PV) via "Static Provisioning" or "Dynamic Provisioning".

Static provisioning is the simpler approach, which by default mounts the whole JuiceFS volume root into application Pod (also supports mounting subdirectories), the Kubernetes administrator is in charge of creating the PersistentVolume (PV) and JuiceFS Volume Credentials (stored as Kubernetes secret). After that, user will create a PVC binding that PV, and then finally use this PVC in application Pod definition. The relationship between different resources:

Use static provisioning when:

Managing PVs can be wearisome, when using CSI Driver at scale, it's recommended to create PV dynamically via dynamic provisioning, relieving the administrator from managing the PVs, while also achieving application data isolation. Under dynamic provisioning, the Kubernetes administrator will create and manage one or more StorageClass, the user only need to create a PVC and reference it in Pod definition, and JuiceFS CSI Driver will create the corresponding PV for you, with each PV corresponding to a subdirectory inside JuiceFS.

The relationship between different resources:

Taking Mount Pod mode for example, this is the overall process:

By default, CSI Driver runs in Mount Pod mode, which isn't allowed in certain scenarios, other mount mode may come in handy when that happens.

Mount Pod is created by CSI Node, due to CSI Node being a DaemonSet component, if your Kubernetes cluster does not allow DaemonSets (like some Serverless Kubernetes platform), CSI Node will not be able to install, and JuiceFS CSI Driver c

*[Content truncated]*

**Examples:**

Example 1 (shell):
```shell
$ kubectl -n kube-system get pod -l app.kubernetes.io/name=juicefs-csi-driverNAME                       READY   STATUS        RESTARTS   AGEjuicefs-csi-controller-0   2/2     Running       0          141djuicefs-csi-node-8rd96     3/3     Running       0          141d
```

---

## Introduction

**URL:** https://juicefs.com/docs/csi/introduction#sidecar

**Contents:**
- Introduction
- Architecture​
- Usage​
  - Static provisioning​
  - Dynamic provisioning​
- Other mount modes​
  - Sidecar mode​
  - Mount by process​

JuiceFS CSI Driver implements the CSI specification, allowing JuiceFS to be integrated with container orchestration systems. Under Kubernetes, JuiceFS can provide storage service to Pods via PersistentVolume.

JuiceFS CSI Driver consists of JuiceFS CSI Controller (StatefulSet) and JuiceFS CSI Node Service (DaemonSet), they can be viewed using kubectl:

By default, CSI Driver runs in Mount Pod mode, in which JuiceFS Client runs in a dedicated Mount Pod, like the architecture shown below:

A dedicated Mount Pod, managed by CSI Node Service, such architecture proves several advantages:

On the same node, a PVC corresponds to a Mount Pod, while Pods using the same PV may share a single Mount Pod. The relationship between different resources:

If Mount Pod mode doesn't suit you, check out other mount modes provided by JuiceFS CSI Driver.

To use JuiceFS CSI Driver, you can create and manage a PersistentVolume (PV) via "Static Provisioning" or "Dynamic Provisioning".

Static provisioning is the simpler approach, which by default mounts the whole JuiceFS volume root into application Pod (also supports mounting subdirectories), the Kubernetes administrator is in charge of creating the PersistentVolume (PV) and JuiceFS Volume Credentials (stored as Kubernetes secret). After that, user will create a PVC binding that PV, and then finally use this PVC in application Pod definition. The relationship between different resources:

Use static provisioning when:

Managing PVs can be wearisome, when using CSI Driver at scale, it's recommended to create PV dynamically via dynamic provisioning, relieving the administrator from managing the PVs, while also achieving application data isolation. Under dynamic provisioning, the Kubernetes administrator will create and manage one or more StorageClass, the user only need to create a PVC and reference it in Pod definition, and JuiceFS CSI Driver will create the corresponding PV for you, with each PV corresponding to a subdirectory inside JuiceFS.

The relationship between different resources:

Taking Mount Pod mode for example, this is the overall process:

By default, CSI Driver runs in Mount Pod mode, which isn't allowed in certain scenarios, other mount mode may come in handy when that happens.

Mount Pod is created by CSI Node, due to CSI Node being a DaemonSet component, if your Kubernetes cluster does not allow DaemonSets (like some Serverless Kubernetes platform), CSI Node will not be able to install, and JuiceFS CSI Driver c

*[Content truncated]*

**Examples:**

Example 1 (shell):
```shell
$ kubectl -n kube-system get pod -l app.kubernetes.io/name=juicefs-csi-driverNAME                       READY   STATUS        RESTARTS   AGEjuicefs-csi-controller-0   2/2     Running       0          141djuicefs-csi-node-8rd96     3/3     Running       0          141d
```

---
