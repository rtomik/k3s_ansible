# Ansible k3s Cluster Deployment

This repository contains Ansible playbooks and roles for deploying a highly available k3s Kubernetes cluster with various applications and services.

## Overview

This project automates the deployment of a k3s cluster on multiple nodes, along with the following applications:

- Traefik (Ingress Controller)
- Longhorn (Distributed Block Storage)
- ArgoCD (GitOps Continuous Delivery)
- Rancher (Kubernetes Management Platform)
- Authentik (Identity Provider)
- Prometheus and Grafana (Monitoring Stack)

## Prerequisites

- Ansible 2.9 or higher
- At least three nodes (virtual or physical machines) running a compatible Linux distribution (e.g., Ubuntu 20.04)
- SSH access to all nodes
- `kubectl` installed on your local machine

## Directory Structure

```
k3s-cluster/
├── ansible.cfg
├── inventory.yaml
├── group_vars/
│   └── all.yaml
├── roles/
│   ├── common/
│   ├── k3s/
│   └── apps/
├── playbooks/
│   ├── main.yaml
│   ├── prepare.yaml
│   ├── k3s.yaml
│   ├── apps.yaml
│   └── destroy.yaml
└── requirements.yaml
```

## Configuration

1. Update `inventory.yaml` with your node IP addresses and SSH user.
2. Modify `group_vars/all.yaml` to set your desired versions and configurations.
3. Adjust any role-specific variables in the respective `roles/*/defaults/main.yaml` files.

## Usage

1. Install required Ansible collections:
   ```
   ansible-galaxy collection install -r requirements.yaml
   ```

2. Run the main playbook to deploy everything:
   ```
   ansible-playbook playbooks/main.yaml
   ```

3. To deploy specific components, use tags:
   ```
   ansible-playbook playbooks/main.yaml --tags k3s,apps:traefik,apps:argocd
   ```

4. To destroy the cluster and remove everything:
   ```
   ansible-playbook playbooks/destroy.yaml
   ```

## Post-Deployment

After successful deployment:

1. Access Rancher at `https://rancher.<your-domain>`
2. Access ArgoCD at `https://argocd.<your-domain>`
3. Access Grafana at `https://grafana.<your-domain>`
4. Use Authentik for Single Sign-On (SSO) at `https://auth.<your-domain>`

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
