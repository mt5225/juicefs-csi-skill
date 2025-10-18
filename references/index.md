# JuiceFS CSI Driver References

This directory contains comprehensive documentation for JuiceFS CSI Driver and Kubernetes CSI standards.

## Core Documentation

### JuiceFS CSI Specific
- **introduction.md** (8 pages) - JuiceFS CSI Driver architecture, components, and usage patterns

## Kubernetes CSI Standards

### Development & Deployment
- **developing.md** - Guide for developing CSI drivers for Kubernetes
- **deploying.md** - Deployment patterns and best practices for CSI drivers

### CSI Objects
- **csi-driver-object.md** - CSIDriver custom resource specification
- **csi-node-object.md** - CSINode object and node registration

### Sidecar Containers
- **sidecar-containers.md** - Overview of CSI sidecar containers and their roles
- **external-provisioner.md** - Dynamic volume provisioning sidecar
- **external-attacher.md** - Volume attach/detach operations
- **external-snapshotter.md** - Volume snapshot management
- **external-resizer.md** - Volume expansion capability
- **node-driver-registrar.md** - Node registration sidecar
- **livenessprobe.md** - Health monitoring for CSI drivers

### Advanced Features
- **secrets-and-credentials.md** - Secure credential management in CSI drivers
- **volume-expansion.md** - Dynamic volume resizing capabilities
- **topology.md** - Zone and region awareness for volume scheduling

### Operations
- **troubleshooting.md** - Common CSI driver issues and solutions

## Quick Navigation

### For Installation & Setup
Start with `deploying.md` for deployment architecture and `introduction.md` for JuiceFS-specific setup.

### For Development
Read `developing.md` for CSI specification compliance and `sidecar-containers.md` to understand component interactions.

### For Troubleshooting
Check `troubleshooting.md` for common issues and solutions.

### For Advanced Features
Explore `topology.md` for multi-zone deployments, `volume-expansion.md` for resizing, and `secrets-and-credentials.md` for security.
