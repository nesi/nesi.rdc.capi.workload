# NeSI CAPI Workload

This repository provides Ansible playbooks and configurations for deploying Kubernetes workload clusters on NeSI's Research Data Centre (RDC) using Cluster API (CAPI).

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [Setup](#setup)
- [Configuration](#configuration)
- [Installation](#installation)
- [Deployment](#deployment)
- [Advanced Features](#advanced-features)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)

## Overview

This project enables the deployment of Kubernetes workload clusters on NeSI's RDC infrastructure. It leverages Cluster API to manage the lifecycle of clusters, providing a standardized way to deploy, scale, and manage Kubernetes clusters.

### Key Features

- Automated deployment of Kubernetes workload clusters
- Support for multiple Kubernetes versions
- Cluster autoscaling capabilities
- GPU node integration
- Security group management
- Integration with OpenStack infrastructure

## Prerequisites

Before using this repository, ensure you have:

1. **Access to CAPI Management Cluster**: You need access to a Cluster API management cluster. To bootstrap one, refer to the [NeSI RDC CAPI Management Cluster](https://github.com/nesi/nesi.rdc.kind-bootstrap-capi) repository.

2. **CAPI Images**: The Cluster API images are built from the [CAPI Images](https://github.com/lbrick/image-builder/tree/2023-nesi_images) repository.

3. **OpenStack Access**: Valid credentials and access to NeSI RDC OpenStack environment.

4. **Ansible**: Installed on your local machine or in a virtual environment.

5. **Python Dependencies**: Required Python packages for OpenStack SDK.

### Version Compatibility

The Management version matrix shows recommended versions:

| Management Version | Workload Version |
|--------------------|------------------|
| v0.2.X            | v0.3.X          |
| v0.4.X            | v0.4.X          |

## Quick Start

1. Clone this repository
2. Copy and configure your environment
3. Install dependencies
4. Deploy the workload cluster

```bash
git clone <repository-url>
cp -r environments/example environments/my-test
# Edit environments/my-test/variables.yml
ansible-galaxy role install -r requirements.yml -p ansible/roles
ansible-galaxy collection install -r requirements.yml -p ansible/collections
ansible-playbook ansible/setup-workload.yml
```

## Setup

### Creating Your Environment

1. Copy the example environment folder:

```bash
cp -r environments/example environments/my-test
```

2. Replace `my-test` with a descriptive name for your environment.

### Environment Structure

Your environment folder should contain:
- `variables.yml`: Configuration variables
- `ansible.cfg`: Ansible configuration
- `activate`: Environment activation script

## Configuration

Edit the `variables.yml` file in your environment directory with your specific settings.

### Required Variables

```yaml
# Kubernetes and CAPI Configuration
kubernetes_version: v1.32.7
capi_image_name: rocky-9-containerd-v1.32.7

# Cluster Details
cluster_rdc_project: NeSI_RDC_PROJECT_NAME
cluster_name: CAPI_CLUSTER_NAME
cluster_namespace: default

# Management Cluster
capi_management_cluster: MANAGEMENT_CLUSTER_NAME

# OpenStack SSH Key
openstack_ssh_key: NeSI_RDC_KEYPAIR_NAME

# Control Plane Configuration
cluster_control_plane_count: 1
control_plane_flavor: balanced1.2cpu4ram
control_plane_volume_size: 0

# Worker Nodes Configuration
cluster_worker_count: 2
cluster_max_worker_count: 3
worker_flavor: balanced1.2cpu4ram
worker_volume_size: 0

# Network Configuration
cluster_node_cidr: 10.10.0.0/24
cluster_pod_cidr: 192.168.0.0/16
cluster_route_id: 3c0cb930-2bbe-4c9c-ac61-6dbc9410c3e9
cluster_external_network_id: 3f405cc9-28a3-4973-b5a1-7f50f112e5d5

# OpenStack Configuration
clouds_yaml_location: ~/.config/openstack/clouds.yaml
clouds_yaml_cloud: openstack

# Kubeconfig Configuration
kube_config_local_location: ~/.kube/config
kube_config_location: "~/.kube"

# GPU Configuration
enable_gpu_nodes: false

# Security Groups
capi_managed_secgroups: false
yaml_openstack_cloud: openstack

# Source IPs for Security Groups
source_ips:
  - 163.7.144.0/21
  - "{{ cluster_node_cidr }}"

# OIDC Authentication (Optional)
kube_oidc_auth: false

# Additional Configuration
additional_cluster_secgroups: []
additional_cluster_networks: []

# Cluster Settle Timeout
cluster_settle_timeout_base: 3

# Autoscaler Configuration
autoscaler_overprovision: false
```

### Variable Descriptions

- **NeSI_RDC_PROJECT_NAME**: Your NeSI RDC project name (e.g., NeSI-Training-Test)
- **CAPI_CLUSTER_NAME**: Desired name for your cluster
- **MANAGEMENT_CLUSTER_NAME**: Name of your CAPI management cluster
- **NeSI_RDC_KEYPAIR_NAME**: Name of your SSH keypair in NeSI RDC
- **cluster_max_worker_count**: Maximum number of worker nodes for autoscaling (must be > cluster_worker_count)
- **cluster_route_id**: Route ID for network routing (alternative to cluster_external_network_id)
- **cluster_external_network_id**: External network ID for OpenStack networking
- **additional_cluster_secgroups**: List of additional security groups to attach to the cluster
- **additional_cluster_networks**: List of additional networks for the cluster
- **cluster_settle_timeout_base**: Base timeout (in minutes) for cluster to settle during deployment
- **autoscaler_overprovision**: Enable overprovisioning in cluster autoscaler

### Available CAPI Images

```bash
rocky-9-containerd-v1.30.5
rocky-9-containerd-v1.31.1
rocky-9-containerd-v1.31.6
rocky-9-containerd-v1.32.2
rocky-9-containerd-v1.32.7
rocky-9-containerd-v1.33.3
```

**Recommendation**: Use Kubernetes version 1.32+ for workload clusters.

**Important**: Ensure the `kubernetes_version` matches the CAPI image version. For example, using `rocky-9-containerd-v1.33.3` requires `kubernetes_version: v1.33.3`.

## Installation

### Ansible Dependencies

Install required Ansible roles and collections:

```bash
ansible-galaxy role install -r requirements.yml -p ansible/roles
ansible-galaxy collection install -r requirements.yml -p ansible/collections
```

### Python Virtual Environment (Recommended for Security Groups)

If managing security groups with Ansible, use a Python virtual environment:

```bash
# Create virtual environment
python3 -m venv ~/nesi-capi

# Activate virtual environment
source ~/nesi-capi/bin/activate

# Install dependencies
pip install ansible ansible-core
pip install "openstacksdk>=1.0.0"
```

## Deployment

### Running the Playbook

Deploy your workload cluster:

```bash
ansible-playbook ansible/setup-workload.yml
```

### Retrieving Kubeconfig

After deployment, retrieve the kubeconfig for your cluster:

```bash
kubectl get secret CLUSTER_NAME-kubeconfig -o jsonpath='{.data.value}' --decode
```

Replace `CLUSTER_NAME` with your cluster name.

## Advanced Features

### Cluster Autoscaling

Enable autoscaling by setting `cluster_max_worker_count` greater than `cluster_worker_count`. The Cluster Autoscaler will be deployed automatically.

**Note**: GPU nodes are not scaled by the autoscaler at this time.

### GPU Nodes

To enable GPU access:

1. Set `enable_gpu_nodes: true` in your `variables.yml`
2. Ensure your RDC project has access to GPU flavors
3. The system will deploy a GPU node with flavor `gpu1.44cpu240ram.a40.1g.48gb`

#### Available GPU Images

```bash
# Rocky 9 with NVIDIA support
rocky-9-containerd-nvidia-v1.32.7
rocky-9-containerd-nvidia-v1.33.3
```

## Troubleshooting

### Common Issues

1. **Authentication Errors**: Verify your OpenStack credentials and project access
2. **Image Not Found**: Ensure the CAPI image exists and matches your Kubernetes version
3. **Network Issues**: Check CIDR configurations and route IDs
4. **Ansible Failures**: Confirm all dependencies are installed and paths are correct

### Logs and Debugging

- Check Ansible output for detailed error messages
- Review OpenStack logs in your RDC project
- Verify network connectivity and security group rules

### Getting Help

- Check the [NeSI RDC CAPI Management Cluster](https://github.com/nesi/nesi.rdc.kind-bootstrap-capi) documentation
- Review [CAPI Images](https://github.com/lbrick/image-builder/tree/2023-nesi_images) repository
- Contact NeSI support for RDC-specific issues

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

## License

This project is licensed under the [Apache License 2.0](LICENSE).

---

For more information about NeSI and the Research Data Centre, visit [nesi.org.nz](https://nesi.org.nz).
