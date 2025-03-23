# GitOps Ansible K3s Homelab Deployment

This repository contains Ansible playbooks and roles for deploying a highly available K3s Kubernetes cluster with various applications and services, following GitOps principles.

## What is GitOps?

GitOps is a modern approach to managing infrastructure and applications where:

- **Git is the single source of truth**: All configuration is stored in Git repositories
- **Declarative configuration**: The desired state of the system is explicitly defined
- **Continuous reconciliation**: Automated processes ensure the actual state matches the desired state
- **Pull-based deployment**: Agents in the cluster pull changes from Git repositories
- **Observability**: Drift between the desired and actual state is detected and reported

In this project, we use ArgoCD as our GitOps engine to automatically deploy and manage applications from Git repositories, ensuring your Kubernetes cluster always reflects the state defined in your code.

## Overview

This project automates the deployment of a k3s cluster on multiple nodes, along with the following applications:

| Category | Applications |
|----------|-------------|
| Infrastructure | - MetalLB (LoadBalancer)<br>- Traefik (Ingress Controller)<br>- Longhorn (Distributed Block Storage) |
| GitOps & Management | - ArgoCD (GitOps CD)<br>- Rancher (Kubernetes Management)<br>- GitLab (Source Control) |
| Security | - Authentik (SSO/IAM) |
| Monitoring Stack | - Prometheus (Metrics)<br>- Grafana (Visualization)<br>- Loki (Log Aggregation) |
| Additional | - Homepage (Dashboard) |

## Features

- 🚀 Single command cluster deployment
- 🔄 GitOps-based application deployment and lifecycle management
- 📊 Comprehensive monitoring and logging stack
- 🔐 Integrated SSO with Authentik
- 💾 Distributed storage with Longhorn
- 📝 Centralized logging with Loki
- 📈 Pre-configured Grafana dashboards for:
  - Application logs and metrics
  - Error monitoring
  - System performance
  - Storage metrics
- 🛡️ Configuration drift detection and automated remediation
- 📜 Audit trail of all configuration changes
- 🔄 High availability cluster configuration
- 🔒 TLS encryption for all services with automatic certificate management

## Prerequisites

### Hardware Requirements
- **Control Node:**
  - Linux system/VM or WSL2 on Windows
  - With Git, Ansible and kubectl installed

- **Cluster Nodes:**
  - Ubuntu Server 24.04
  - Minimum 16GB RAM per node
  - Recommended: 3 nodes for HA
  - [Low-cost mini PC options](https://www.lowcostminipcs.com/)

- **My Setup:**
  - HP ProDesk 600 G3 mini - i3 7300t/16GB RAM/500GB SSD cost 65 €
  - GenMachine Mini PC - AMD 5300U/16GB RAM/500GB SSD cost 150 €

### Network Requirements
- SSH access to all nodes (root SSH keys)
- Domain name ([Porkbun](https://porkbun.com/) or [DuckDNS](https://www.duckdns.org/))
- [Tailscale account](https://login.tailscale.com/admin/settings/keys)

### Optional
- Cloudflare account and API token (for TLS certificates)
- NFS share for media storage and backups

## Quick Start

1. **Clone and prepare configuration**
   ```bash
   git clone https://github.com/rtomik/k3s_ansible.git && cd k3s_ansible\
   mv inventory.yml_ex inventory.yml && \
   mv group_vars/all/main.yml_ex group_vars/all/main.yml
   ```

2. **Configure nodes**
   - Update `inventory.yml` with node IPs
   - Modify `group_vars/all/main.yml` with your configurations
   - Set required variables in `Required variables` section

3. **Configure Tailscale**
   ```bash
   ansible-playbook playbooks/main.yml --tags local,config
   ```
   ⚠️ Important: Set node IPs to 100.64.0.1, 100.64.0.2, etc. in [Tailscale admin](https://login.tailscale.com/admin/machines)  

4. **Initial Deployment**
   NOTE: If you enabled cloudflare it can take some time to propagate new TLS certificate
   First deploy k3s and cert-manager, 
   ```
   ansible-playbook playbooks/main.yml --tags local,config,k3s,infra:traefik,infra:certs
   ```
   Kubeconfig will be stored in playbook dir
   Verify certificate (wait 5-10 minutes):
   ```
   k get certificate -A
   ```
   Check cert-manager pod logs for any error
   ```
   CERT_MANAGER_POD=$(kubectl get pods -n cert-manager -l app.kubernetes.io/name=cert-manager -o jsonpath='{.items[0].metadata.name}')
   k logs $CERT_MANAGER_POD -n cert-manager
   ```
   On error
   ```
   Error cleaning up challenge: while querying the Cloudflare API for DELETE
   ```
   Delete TXT _acme-challenge records in cloudflare

   After the certificate is created rerun the main playbook.

5. **Complete Deployment**
   ```bash
   ansible-playbook playbooks/main.yml
   ```

6. To deploy specific components, use tags
   ```
   ansible-playbook playbooks/main.yml --tags "k3s"
   ```

7. To destroy the cluster and remove everything:
   ```
   ansible-playbook playbooks/destroy.yml
   ```

## Access Services

| Service | URL | Description |
|---------|-----|-------------|
| Rancher | `https://rancher.<domain>` | Kubernetes Management UI |
| ArgoCD | `https://argocd.<domain>` | GitOps Control Panel |
| Grafana | `https://grafana.<domain>` | Metrics & Logs Visualization |
| Authentik | `https://authentik.<domain>` | SSO/IAM Portal |
| Prometheus | `https://prometheus.<domain>` | Metrics Storage |
| GitLab | `https://gitlab.<domain>` | Source Control |
| Homepage | `https://home.<domain>` | System Dashboard |


## Troubleshooting

Common issues and solutions:

1. **Certificate Issues**
   - Check cert-manager logs
   - Clear Cloudflare DNS TXT records
   - Verify API token permissions

2. **Node Connectivity**
   - Verify Tailscale connectivity
   - Check firewall rules
   - Ensure correct SSH keys

3. **GitOps Sync Issues**
   - Check ArgoCD logs for sync errors
   - Verify repository access
   - Validate YAML syntax in manifests

## Contributing

Contributions are welcome! Please:
1. Fork the repository
2. Create a feature branch
3. Submit a Pull Request

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Support

- 🌟 Star this repo if you find it helpful!
- 🐛 Report issues in the [Issue Tracker](https://github.com/rtomik/k3s_ansible/issues)
- 📝 Submit improvements via Pull Requests