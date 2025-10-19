# JuiceFS CSI Driver Skill

A Claude skill providing comprehensive documentation and guidance for the JuiceFS CSI Driver in Kubernetes environments.

## Overview

This skill enables Claude to assist with JuiceFS CSI Driver development, deployment, configuration, and troubleshooting. It contains official documentation covering all aspects of using JuiceFS as a Container Storage Interface (CSI) driver in Kubernetes.

## Purpose

The JuiceFS CSI Driver skill helps with:

- **Installation & Configuration**: Deploy and configure JuiceFS CSI Driver in Kubernetes clusters
- **Volume Management**: Work with PersistentVolumes (PV) and PersistentVolumeClaims (PVC)
- **Storage Provisioning**: Implement both static and dynamic provisioning workflows
- **Mount Modes**: Configure Mount Pod, Sidecar, or Process mount modes
- **Troubleshooting**: Debug CSI Controller, CSI Node Service, and mount pod issues
- **Performance Optimization**: Tune resource allocation and storage performance
- **Advanced Features**: Implement volume expansion, snapshots, topology, and credential management

## Documentation Structure

The skill includes comprehensive reference documentation in the `references/` directory:

### Core Concepts
- `introduction.md` - Overview of CSI specification and JuiceFS CSI Driver
- `deploying.md` - Installation and deployment guides
- `csi-driver-object.md` - CSIDriver object configuration
- `csi-node-object.md` - CSINode object details

### Sidecar Components
- `external-provisioner.md` - Dynamic volume provisioning
- `external-attacher.md` - Volume attachment management
- `external-resizer.md` - Volume expansion capabilities
- `external-snapshotter.md` - Volume snapshot features
- `node-driver-registrar.md` - Node plugin registration
- `livenessprobe.md` - Health monitoring

### Advanced Topics
- `sidecar-containers.md` - Sidecar deployment patterns
- `volume-expansion.md` - Volume resizing workflows
- `topology.md` - Topology-aware scheduling
- `secrets-and-credentials.md` - Credential management
- `troubleshooting.md` - Common issues and solutions
- `developing.md` - Development guidelines

## Key Features

### Architecture Components
- **CSI Controller**: StatefulSet managing volume lifecycle
- **CSI Node Service**: DaemonSet handling volume mounting
- **Mount Pods**: Dedicated pods running JuiceFS Client

### Provisioning Modes
1. **Static Provisioning**: Manual PV creation for simpler deployments
2. **Dynamic Provisioning**: Automatic PV creation using StorageClass

### Mount Modes
- **Mount Pod Mode**: Default, dedicated pods for mounting (best isolation)
- **Sidecar Mode**: Sidecar containers in application pods (tighter coupling)
- **Process Mode**: JuiceFS runs as node service process (lowest overhead)

## Usage

This skill is automatically triggered when discussing topics related to:
- JuiceFS CSI Driver installation or configuration
- Kubernetes persistent storage with JuiceFS
- CSI driver troubleshooting and debugging
- Container storage performance optimization
- Dynamic or static volume provisioning

## Repository Structure

```
.
├── README.md           # This file
├── SKILL.md           # Skill definition and complete documentation
├── references/        # Markdown documentation files
├── scripts/          # Utility scripts
└── assets/           # Supporting resources
```

## Contributing

To update this skill's documentation:

1. Update relevant files in `references/` directory
2. Regenerate `SKILL.md` if structure changes
3. Update `README.md` to reflect content changes
4. Commit and push changes

## Related Skills

- **juicefs**: Core JuiceFS distributed file system skill
- For general JuiceFS questions, use the `juicefs` skill
- For CSI-specific topics, use this `juicefs-csi` skill

## License

Documentation derived from official JuiceFS CSI Driver documentation.
