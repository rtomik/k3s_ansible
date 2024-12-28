
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

## Prerequisites

- Ansible 2.9 or higher
- Nodes with installed Ubuntu 24.04
- SSH access to all nodes
- `kubectl` installed on your local machine
- A custom domain that you control and can configure DNS records for (recommended: https://porkbun.com/)
- Basic VPS (recommended: https://www.hetzner.com/cloud/ CAX11)

## Configuration

1. Update `inventory.yml` with your node IP addresses and SSH user.
2. Modify `group_vars/all.yml` to set your desired versions, configurations, and variables

## Usage

1. Clone the repo
   ```
   git clone https://github.com/rtomik/k3s_ansible.git && \
   cd k3s_ansible && \ 
   mv inventory.yml_ex inventory.yml && \
   mv group_vars/all/main.yml_ex group_vars/all/main.yml
   ```

2. Create vault 
   ```
   echo "vault_k3s_token: $(openssl rand -base64 48)" | ansible-vault create group_vars/all/vault.yml
   ```
   Save the password to .vault

3. Install required Ansible collections:
   ```
   ansible-galaxy collection install -r requirements.yml
   ```

4. Run the main playbook to deploy everything:
   ```
   ansible-playbook playbooks/main.yml
   ```

5. To deploy specific components, use tags:
   ```
   ansible-playbook playbooks/main.yml --tags "k3s"
   ```

6. To destroy the cluster and remove everything:
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

