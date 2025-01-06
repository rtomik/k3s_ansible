
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
- SSH access to all nodes (ssh keys to root)
- `kubectl` installed on your local machine
- A custom domain that you control and can configure DNS records for (recommended: https://porkbun.com/)
- Basic VPS (recommended: https://www.hetzner.com/cloud/ CAX11)
- Login to Tailscale https://login.tailscale.com/admin/settings/keys create new auth keys


## Usage

1. Clone the repo
   ```
   git clone https://github.com/rtomik/k3s_ansible.git && \
   cd k3s_ansible && \ 
   mv inventory.yml_ex inventory.yml && \
   mv group_vars/all/main.yml_ex group_vars/all/main.yml
   ```

2. Update `inventory.yml` with your node IP addresses and SSH user.

   Modify `group_vars/all.yml` to set your desired versions, configurations, and variables

3. Create vault 
   ```
   echo "vault_k3s_token: $(openssl rand -base64 48)" | ansible-vault create group_vars/all/vault.yml
   ```
   Add tailscale key to: vault_tailscale_key
   Save the password to .vault

4. Install required Ansible collections:
   ```
   ansible-galaxy collection install -r requirements.yml
   ```

5. Run the main playbook
   ```
   ansible-playbook playbooks/main.yml
   ```

6. To deploy specific components, use tags:
   ```
   ansible-playbook playbooks/main.yml --tags "k3s"
   ```

7. To destroy the cluster and remove everything:
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

## Diagram

``` mermaid
graph TD
    Internet((Internet)) --> Firewall
    Firewall --> VPS[VPS with Traefik]
    VPS <--> |Tailscale| K3s
    subgraph "Physical Infrastructure"
        MiniPC1[Mini PC 1]
        MiniPC2[Mini PC 2]
        MiniPC3[Mini PC 3]
    end
    subgraph "Kubernetes Cluster"
        MiniPC1 & MiniPC2 & MiniPC3 --> K3s
        K3s --> ArgoCD
        K3s --> Authentik
        K3s --> Apps
        ArgoCD --> GitRepo[(Git Repository)]
        ArgoCD --> Apps
        subgraph "Monitoring"
            style Monitoring fill:#ffffff,stroke:#333,stroke-width:2px,color:#333
            Prometheus --> Grafana
        end
        subgraph "Logging"
            Graylog
        end
        subgraph "Apps"
            Homepage
            Immich
            OtherApps[Other Apps]
        end
        Apps --> Prometheus
        Apps --> Graylog
        Authentik --> Apps
        Authentik --> ArgoCD
        Authentik --> Grafana
        Velero --> HetznerBackup[(Hetzner Backup)]
    end
    classDef default fill:#f0f0f0,stroke:#333,stroke-width:2px,color:#333;
    classDef k8s fill:#326ce5,stroke:#333,stroke-width:2px,color:#fff;
    classDef ext fill:#ffd966,stroke:#333,stroke-width:2px;
    classDef hardware fill:#d5e8d4,stroke:#82b366,stroke-width:2px;
    classDef vps fill:#ff9999,stroke:#333,stroke-width:2px;
    class K3s,K3sTraefik,ArgoCD,Authentik,Apps,Prometheus,Grafana,Graylog,K8sCluster,Velero k8s;
    class GitRepo,HetznerBackup ext;
    class MiniPC1,MiniPC2,MiniPC3 hardware;
    class VPS vps;
```

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

