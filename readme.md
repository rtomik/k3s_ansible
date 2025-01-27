
# Ansible k3s Cluster Deployment

This repository contains Ansible playbooks and roles for deploying a highly available k3s Kubernetes cluster with various applications and services.

## Overview

This project automates the deployment of a k3s cluster on multiple nodes, along with the following applications:

- Traefik (Ingress Controller)
- Longhorn (Distributed Block Storage)
- ArgoCD (GitOps Continuous Delivery)
- Rancher (Kubernetes Management Platform)
- Authentik (SSO)
- Prometheus Stack (Monitoring)
- GitLab
- Metalb (K3s LoadBalancer)


## Prerequisites

- Linux system/vm/or WSL on Windows where you can clone the repo. With packages Ansible 2.9 or higher and kubectl
- Linux bare metal servers or VMs with installed Ubuntu Server 24.04 (Recommended 3)
- SSH access to all nodes (ssh keys to root)
- Domain (recommended: https://porkbun.com/ or https://www.duckdns.org/)
- Tailscale account https://login.tailscale.com/admin/settings/keys and create new auth keys


## Usage

1. Clone the repo
   ```
   git clone https://github.com/rtomik/k3s_ansible.git && cd k3s_ansible\
   mv inventory.yml_ex inventory.yml && \
   mv group_vars/all/main.yml_ex group_vars/all/main.yml
   ```

2. Modify `inventory.yml` with your node IP addresses.
   
3. Modify `group_vars/all/main.yml` to set your desired versions, configurations, and variables 
   Make sure to define variables under `Required variables` section

4. Run the main playbook
   ```
   ansible-playbook playbooks/main.yml
   ```

5. To deploy specific components, use tags
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

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

