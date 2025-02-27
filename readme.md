
# Ansible k3s Cluster Deployment

This repository contains Ansible playbooks and roles for deploying a highly available k3s Kubernetes cluster with various applications and services.

## Overview

This project automates the deployment of a k3s cluster on multiple nodes, along with the following applications:

- Metalb (LoadBalancer)
- Traefik (Ingress Controller)
- Longhorn (Distributed Block Storage)
- ArgoCD (GitOps Continuous Delivery)
- Rancher (Kubernetes Management Platform)
- Authentik (SSO)
- Prometheus (Monitoring)
- Grafana (Visualization)
- Loki (Log Aggregation)
- GitLab
- Homepage
  
## Features

- Automated cluster deployment
- Integrated monitoring and logging stack
- Single Sign-On (SSO) integration
- Persistent storage with Longhorn
- Log aggregation with Loki
- Customizable Grafana dashboards for:
  - Application logs
  - Error monitoring
  - System metrics
  - Storage metrics
  
## Requirements

- Linux system/vm or WSL on Windows where you can clone the repo. With Ansible and kubectl
- Servers/PCs/VMs with installed Ubuntu Server 24.04 min 16 GB RAM (recommended 3) (for testing it can also run on single node) you can find used cheap pcs https://www.lowcostminipcs.com/
- SSH access to all nodes (ssh keys to root)
- Domain (recommended: https://porkbun.com/ or https://www.duckdns.org/)
- Tailscale account https://login.tailscale.com/admin/settings/keys and create new auth keys
- Optional: If you want to have TLS certificate on the services create cloudflare account and create api token
- Optional: NFS share for media storage and backups


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

4. NOTE: If you enabled cloudflare it can take some time to propagate new TLS certificate
   First deploy k3s and cert-manager, 
   ```
   ansible-playbook playbooks/main.yml --tags local,config,k3s,infra:metallb,infra:traefik,infra:certs
   ```
   Check if certificate is ready it may take up to 5-10 minutes.
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

5. Run the main playbook
   ```
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

## Post-Deployment

After successful deployment:

1. Access Rancher at `https://rancher.<your-domain>`
2. Access ArgoCD at `https://argocd.<your-domain>`
3. Access Grafana at `https://grafana.<your-domain>`
   - View application logs dashboard
   - Monitor system metrics
   - Track error rates
4. Use Authentik for Single Sign-On (SSO) at `https://authentik.<your-domain>`
5. Access Prometheus at `https://prometheus.<your-domain>`

## Diagram

``` mermaid
flowchart TD
    subgraph ExternalAccess["External Access"]
        Internet((Internet))
        Internet --> |Port 41641| TailscaleMesh[Tailscale Mesh Network]
    end

    subgraph Infrastructure["Kubernetes Infrastructure"]
        subgraph Physical["Physical Infrastructure"]
            Node1[Node 1]
            Node2[Node 2]
            Node3[Node 3]
            TailscaleMesh --> |Tailscale| Node1
            TailscaleMesh --> |Tailscale| Node2
            TailscaleMesh --> |Tailscale| Node3
        end

        subgraph Cluster["Kubernetes Cluster"]
            subgraph Storage["Storage Layer"]
                Longhorn[Longhorn Storage]
                Longhorn --> |Volume 1| Node1
                Longhorn --> |Volume 2| Node2
                Longhorn --> |Volume 3| Node3
            end

            subgraph Apps["Applications"]
                Homepage[Homepage]
                GitLab[GitLab]
                OtherApps[Other Apps]
            end

            subgraph Observe["Observability Stack"]
                subgraph LogCol["Log Collection"]
                    Promtail --> Loki
                end
                subgraph MetCol["Metrics Collection"]
                    ServiceMonitor --> Prometheus
                end
                Loki --> Grafana
                Prometheus --> Grafana
            end

            subgraph Core["Core Services"]
                K3s[K3s Control Plane]
                Authentik
                ArgoCD
            end
        end

        GitRepo[(Git Repository)] --- ArgoCD

        %% Connections
        Node1 & Node2 & Node3 --> K3s
        K3s --> Authentik
        Authentik --> ArgoCD
        Authentik --> Grafana
        Apps --> Promtail
        Apps --> ServiceMonitor
    end

    %% Styling
    classDef external fill:#f9f,stroke:#333,stroke-width:2px;
    classDef infrastructure fill:#beb,stroke:#333,stroke-width:2px;
    classDef core fill:#58f,stroke:#333,stroke-width:2px;
    classDef observability fill:#9cf,stroke:#333,stroke-width:2px;
    classDef storage fill:#fc9,stroke:#333,stroke-width:2px;
    classDef apps fill:#333,stroke:#fff,stroke-width:2px,color:#fff;
    
    class Internet,TailscaleMesh external;
    class Node1,Node2,Node3 infrastructure;
    class K3s,Authentik,ArgoCD core;
    class Prometheus,Grafana,Loki,Promtail,ServiceMonitor observability;
    class Longhorn storage;
    class Homepage,GitLab,OtherApps apps;

    %% Layout
    direction TB
```

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

