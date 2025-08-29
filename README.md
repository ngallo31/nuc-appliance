# OpenShift Single Node Appliance - Configuration as Code

Automated workspace setup for deploying OpenShift as a single-node appliance using configuration-as-code principles.

## Why Use This?

If you need **OpenShift running on a single node** with a **repeatable, code-driven deployment process**, this is for you. Perfect for:

- **Edge computing** deployments where high availability isn't needed
- **Development/testing** environments that need full OpenShift functionality  
- **Resource-constrained** environments (labs, demos, proof-of-concepts)
- **Standardized deployments** across multiple single-node installations
- **GitOps workflows** where infrastructure is managed as code

## What You Get

A **single-node OpenShift cluster** deployed as an appliance with:
- ✅ **Full OpenShift functionality** on one machine
- ✅ **Configuration as code** - everything defined in YAML
- ✅ **Repeatable deployments** - same result every time
- ✅ **Operator availability** - your operators included in repository for installation
- ✅ **Disconnected operation** - works without internet access
- ✅ **Custom image pre-loading** - container images embedded in appliance

## Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│                Single Node OpenShift                   │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐    │
│  │   Control   │  │   Worker    │  │  Operators  │    │
│  │    Plane    │  │  Workloads  │  │ (Pre-built) │    │
│  └─────────────┘  └─────────────┘  └─────────────┘    │
└─────────────────────────────────────────────────────────┘
                         │
              ┌─────────────────────┐
              │  Appliance Image    │
              │  (ISO or Raw Disk)  │
              └─────────────────────┘
                         │
              ┌─────────────────────┐
              │ Configuration Code  │
              │ (YAML Definitions)  │
              └─────────────────────┘
```

## Quick Start

### 1. Prepare Your Configuration
Create your appliance configuration as code:

```yaml
# artifacts/appliance-config.yaml
# NOTE: This is an example configuration - customize for your environment
apiVersion: v1beta1
kind: ApplianceConfig
ocpRelease:
  version: 4.19.6
  channel: stable
  cpuArchitecture: x86_64
pullSecret: 'YOUR_RED_HAT_PULL_SECRET'
userCorePass: YourSecurePassword123
sshKey: ssh-rsa AAAAB3NzaC1yc2E... your-key-here
enableDefaultSources: false
stopLocalRegistry: false

# Pre-include operators (makes them available for installation)
operators:
  - catalog: registry.redhat.io/redhat/redhat-operator-index:v4.19
    packages:
      - name: compliance-operator
        channels:
          - name: stable
      - name: ansible-automation-platform-operator  
        channels:
          - name: stable-2.5

# Pre-load container images
additionalImages:
  - name: registry.redhat.io/ubi9/ubi:latest
  - name: registry.redhat.io/rhel9/support-tools
```

> **⚠️ Important**: The configuration above is an example. You must customize it with your actual pull secret, SSH key, password, and desired operators before building your appliance.

### 2. Set Up Build Environment
```bash
# Run the workspace preparation
sudo ansible-playbook -K prep-appliance-workspace.yml
```

### 3. Build Your Appliance
```bash
# Build the single-node appliance ISO
sudo podman run --rm -it --pull newer --privileged --net=host \
  -v /mnt/appliance_assets:/assets:Z \
  quay.io/edge-infrastructure/openshift-appliance build live-iso
```

### 4. Deploy to Hardware
```bash
# Boot target machine from appliance.iso
# OpenShift installs automatically using your configuration
```

## Configuration as Code Examples

### Minimal Single Node Setup
```yaml
apiVersion: v1beta1
kind: ApplianceConfig
ocpRelease:
  version: 4.19.6
  channel: stable
pullSecret: 'YOUR_PULL_SECRET'
userCorePass: SecurePassword123
sshKey: ssh-rsa AAAAB... your-ssh-key
```

### Edge Computing with Compliance
```yaml
apiVersion: v1beta1
kind: ApplianceConfig
ocpRelease:
  version: 4.19.6
  channel: stable
pullSecret: 'YOUR_PULL_SECRET'
userCorePass: EdgePassword123
sshKey: ssh-rsa AAAAB... your-ssh-key
enableDefaultSources: false

operators:
  - catalog: registry.redhat.io/redhat/redhat-operator-index:v4.19
    packages:
      - name: compliance-operator
      - name: file-integrity-operator
      - name: kubernetes-nmstate-operator

additionalImages:
  - name: registry.redhat.io/rhel9/support-tools
  - name: registry.redhat.io/ubi9/ubi:latest
```

### Development Environment with Virtualization
```yaml
apiVersion: v1beta1
kind: ApplianceConfig
ocpRelease:
  version: 4.19.6
  channel: stable
pullSecret: 'YOUR_PULL_SECRET'
userCorePass: DevPassword123
sshKey: ssh-rsa AAAAB... your-ssh-key

operators:
  - catalog: registry.redhat.io/redhat/redhat-operator-index:v4.19
    packages:
      - name: kubevirt-hyperconverged
      - name: kubernetes-nmstate-operator
      - name: local-storage-operator
      - name: web-terminal

additionalImages:
  - name: registry.redhat.io/rhel9/rhel-guest-image:latest
  - name: registry.redhat.io/ubi8/ubi:latest
  - name: registry.redhat.io/openshift4/ose-must-gather:latest
```

## Hardware Requirements

### Minimum Requirements
- **CPU**: 8 vCPU cores
- **RAM**: 32GB (16GB absolute minimum)
- **Storage**: 120GB available space
- **Network**: Single network interface

### Recommended for Production Edge
- **CPU**: 16+ vCPU cores  
- **RAM**: 64GB+
- **Storage**: 500GB+ SSD
- **Network**: Dual interfaces (management + workload)

### Recommended Hardware Platforms
- **Intel NUC** (12th gen or newer)
- **Dell Edge Gateways** (3000/5000 series)
- **HPE Edgeline** converged systems
- **Supermicro** compact servers
- **Custom x86_64** hardware meeting specs

## Deployment Scenarios

### 1. **Remote Edge Sites**
```bash
# Build appliance at headquarters
sudo ansible-playbook -K prep-appliance-workspace.yml
# Ship ISO to remote site
# Remote team boots from ISO - no technical expertise needed
```

### 2. **Multi-Site Consistency**  
```bash
# Same configuration code deployed across 50 retail locations
# Guaranteed identical OpenShift setup everywhere
```

### 3. **Development Pipeline**
```bash
# Version control your appliance configs
# Automated testing of configuration changes
# Promote configs through dev → staging → production
```

### 4. **Disconnected Environments**
```bash
# Build appliance in connected environment
# Deploy in air-gapped networks
# All container images pre-loaded
```

## GitOps Integration

### Repository Structure
```
openshift-appliances/
├── configs/
│   ├── edge-retail/
│   │   └── appliance-config.yaml
│   ├── development/  
│   │   └── appliance-config.yaml
│   └── factory-floor/
│       └── appliance-config.yaml
├── cluster-configs/
│   ├── install-config.yaml
│   └── agent-config.yaml
└── automation/
    ├── prep-appliance-workspace.yml
    └── build-pipeline.yml
```

### CI/CD Pipeline Example
```yaml
# .github/workflows/build-appliance.yml
name: Build OpenShift Appliance
on:
  push:
    paths: ['configs/**']

jobs:
  build:
    runs-on: self-hosted
    steps:
      - uses: actions/checkout@v3
      - name: Prepare workspace
        run: sudo ansible-playbook -K automation/prep-appliance-workspace.yml
      - name: Build appliance
        run: |
          sudo podman run --rm -it --privileged --net=host \
            -v /mnt/appliance_assets:/assets:Z \
            quay.io/edge-infrastructure/openshift-appliance build live-iso
      - name: Upload ISO artifact
        uses: actions/upload-artifact@v3
        with:
          name: openshift-appliance.iso
          path: /mnt/appliance_assets/appliance.iso
```

## Troubleshooting Single Node Deployments

### Common Single Node Issues

**Insufficient Resources**
```bash
# Check available resources
free -h
df -h /mnt
lscpu
```

**Network Configuration**  
```bash
# Single node needs proper DNS resolution
nslookup api.cluster-name.domain.com
nslookup *.apps.cluster-name.domain.com
```

**Storage Requirements**
```bash
# Single node needs adequate storage for etcd + workloads
# Monitor disk usage during deployment
watch df -h
```

### Validation Commands
```bash
# After deployment, verify single node health
oc get nodes
oc get co  # cluster operators
oc get pods -A | grep -v Running
```

## Migration from Multi-Node

If you're moving from multi-node to single-node OpenShift:

1. **Extract configurations** from existing cluster
2. **Adapt resource limits** for single node constraints  
3. **Review storage classes** - single node typically uses local storage
4. **Adjust networking** - simplified network topology
5. **Test workloads** - ensure they run on single node architecture

## Best Practices

### Configuration Management
- ✅ **Version control** all YAML configurations
- ✅ **Environment-specific** configs (dev/staging/prod)  
- ✅ **Automated testing** of configuration changes
- ✅ **Documentation** of custom settings and rationale

### Security
- ✅ **Rotate passwords** and SSH keys regularly
- ✅ **Minimal operator set** - only install what you need
- ✅ **Regular updates** - rebuild appliances with latest OpenShift versions
- ✅ **Network segmentation** - proper firewall rules for single node

### Operations  
- ✅ **Monitoring strategy** - single node has no redundancy
- ✅ **Backup procedures** - regular etcd and persistent volume backups
- ✅ **Update process** - plan for OpenShift version updates
- ✅ **Disaster recovery** - ability to rebuild from configuration code

## Requirements

- **Ansible**: 2.9 or higher
- **Container Runtime**: Podman (recommended) or Docker  
- **System Access**: Sudo privileges
- **Storage**: 200GB+ available (for build environment)
- **Red Hat Account**: For pull secret and operator access

## Support

This approach is fully supported by Red Hat for:
- **OpenShift Single Node** deployments
- **OpenShift Appliance Builder** tooling
- **Individual operators** installed via configuration

Standard Red Hat support channels apply for single-node OpenShift issues.

## Getting Started

Ready to deploy OpenShift as a single-node appliance? 

1. **Get your Red Hat pull secret** from console.redhat.com
2. **Define your configuration** in `appliance-config.yaml`
3. **Run the workspace setup**: `sudo ansible-playbook -K prep-appliance-workspace.yml`
4. **Build your appliance** using the generated workspace
5. **Deploy to target hardware** by booting from ISO

Your OpenShift single-node appliance will be running with your exact configuration, ready for workloads.