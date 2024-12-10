
# Ansible k3s Cluster Deployment

This repository contains Ansible playbooks and roles for deploying a highly available k3s Kubernetes cluster with various applications and services.

## Overview

This project automates the deployment of a k3s cluster on multiple nodes, along with the following applications:

- Traefik (Ingress Controller)
- Longhorn (Distributed Block Storage)
- ArgoCD (GitOps Continuous Delivery)
- Rancher (Kubernetes Management Platform)
- Authentik (Identity Provider)
- Prometheus Stack (Monitoring, including Grafana)

## Directory Structure

```
k3s-ansible/
├── ansible.cfg
├── inventory.yml
├── group_vars/
│   └── all.yml
├── playbooks/
│   ├── destroy.yml
│   └── main.yml
├── roles/
│   ├── 01_base_config/
│   │   ├── handlers/
│   │   │   └── main.yml
│   │   └── tasks/
│   │       └── main.yml
│   ├── 02_k3s/
│   │   ├── handlers/
│   │   │   └── main.yml
│   │   └── tasks/
│   │       └── main.yml
│   └── 03_apps/
│       ├── tasks/
│       │   ├── argocd.yml
│       │   ├── authentik.yml
│       │   ├── longhorn.yml
│       │   ├── main.yml
│       │   ├── monitoring.yml
│       │   ├── rancher.yml
│       │   └── traefik.yml
│       └── templates/
│           ├── values-authentik.yml.j2
│           └── values-rancher.yml.j2
└── requirements.yml
```

## Prerequisites

- Ansible 2.9 or higher
- At least three nodes (virtual or physical machines) running a compatible Linux distribution (e.g., Ubuntu 20.04)
- SSH access to all nodes
- `kubectl` installed on your local machine
- A custom domain that you control and can configure DNS records for

## Configuration

1. Update `inventory.yml` with your node IP addresses and SSH user.
2. Modify `group_vars/all.yml` to set your desired versions, configurations, and variables

## Usage

1. Install required Ansible collections:
   ```
   ansible-galaxy collection install -r requirements.yml
   ```

2. Run the main playbook to deploy everything:
   ```
   ansible-playbook playbooks/main.yml
   ```

3. To deploy specific components, use tags:
   ```
   ansible-playbook playbooks/main.yml --tags "k3s"
   ```

5. To destroy the cluster and remove everything:
   ```
   ansible-playbook playbooks/destroy.yml
   ```

## Post-Deployment

After successful deployment:

1. Access Rancher at `https://rancher.<your-domain>`
2. Access ArgoCD at `https://argocd.<your-domain>`
3. Access Grafana at `https://grafana.<your-domain>`
4. Use Authentik for Single Sign-On (SSO) at `https://auth.<your-domain>`
5. Access Prometheus at `https://prometheus.<your-domain>`


## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

