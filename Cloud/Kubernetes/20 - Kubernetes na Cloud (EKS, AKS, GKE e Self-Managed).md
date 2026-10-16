---
tags:
  - kubernetes
  - cloud
  - eks
  - aks
  - gke
  - kubeadm
  - k3s
  - openshift
  - rancher
  - aws
  - azure
  - gcp
---
> [!INFO]
> Objetivo
>
> Entender as diferenças entre Kubernetes gerenciado e autogerenciado, compreender como os principais provedores de nuvem integram Kubernetes aos seus serviços e conhecer boas práticas para ambientes corporativos.

---

# Objetivos de aprendizagem

Ao concluir este capítulo você deverá ser capaz de:

- [ ] Diferenciar Kubernetes gerenciado e self-managed.
- [ ] Comparar EKS, AKS e GKE.
- [ ] Entender integrações com IAM, VPC e Storage.
- [ ] Compreender Auto Scaling de infraestrutura.
- [ ] Conhecer distribuições Kubernetes.
- [ ] Escolher a melhor solução para diferentes cenários.

---

# Pré-requisitos

- Todos os capítulos anteriores.

---

# Kubernetes não é apenas Kubernetes

Quando dizemos:

> "Tenho um cluster Kubernetes"

Isso pode significar diversas coisas.

Exemplos:

- Amazon EKS
- Azure AKS
- Google GKE
- kubeadm
- k3s
- OpenShift
- Rancher RKE2
- Talos Linux

Todos executam Kubernetes.

Mas possuem responsabilidades operacionais diferentes.

---

# Modelo Mental

Imagine um carro.

Você pode:

```text
Comprar um carro

↓

Dirigir
```

Ou.

```text
Construir o carro

↓

Fazer manutenção

↓

Dirigir
```

kubeadm é construir o carro.

EKS é comprar o carro pronto.

---

# Self-Managed

Você administra:

- Control Plane
- etcd
- Workers
- Upgrades
- Backup
- Segurança
- Certificados
- Networking

Você possui controle total.

Mas também responsabilidade total.

---

# Managed Kubernetes

O provedor administra:

- API Server
- Scheduler
- Controller Manager
- etcd
- Atualizações do Control Plane
- Alta disponibilidade do Control Plane

Você normalmente administra apenas os Worker Nodes e as aplicações.

---

# Modelo de responsabilidade

## Self-Managed

```text
Você

↓

Tudo
```

---

## Managed

```text
Cloud Provider

↓

Control Plane

↓

Você

↓

Aplicações
```

---

# Amazon EKS

Serviço gerenciado da AWS.

A AWS administra:

- API Server
- Scheduler
- Controller Manager
- etcd

Você administra:

- Nodes (EC2 ou Fargate)
- Deployments
- Storage
- Aplicações

---

# Arquitetura EKS

```text
Internet

↓

ALB

↓

Ingress

↓

Service

↓

Pods

↓

EC2 Worker Nodes

↓

VPC
```

---

# Integração com IAM

Uma das maiores vantagens.

O Kubernetes consegue utilizar identidades da AWS.

Exemplo.

Uma aplicação acessando S3.

Sem armazenar chaves.

Utilizando IAM Roles for Service Accounts (IRSA).

---

# Storage no EKS

Normalmente utilizamos.

Amazon EBS CSI Driver.

Fluxo.

```text
PVC

↓

StorageClass

↓

EBS

↓

Volume criado automaticamente
```

---

# Load Balancer

Ao criar.

```yaml
type: LoadBalancer
```

A AWS automaticamente cria.

- Classic Load Balancer
- Network Load Balancer
- Application Load Balancer (via AWS Load Balancer Controller)

---

# Auto Scaling

No EKS temos.

- Cluster Autoscaler
- Karpenter

O Karpenter é a solução moderna da AWS para provisionamento automático de Nodes.

---

# Amazon Fargate

Permite executar Pods.

Sem administrar servidores.

Modelo.

```text
Pod

↓

Fargate

↓

Infraestrutura invisível
```

---

# Azure AKS

Serviço Kubernetes da Microsoft.

Muito integrado ao Azure.

---

# Integrações importantes

- Azure AD (Entra ID)
- Managed Identity
- Azure Disk CSI
- Azure Files CSI
- Azure Monitor
- Azure Policy

---

# Networking no AKS

Normalmente utiliza.

Azure CNI.

Cada Pod recebe um IP da VNet.

Isso facilita integrações corporativas.

---

# Google GKE

Historicamente.

É considerado a implementação Kubernetes mais madura.

Não por acaso.

O Kubernetes nasceu dentro do Google.

---

# Recursos interessantes

- Autopilot Mode
- Workload Identity
- Cloud Logging
- Cloud Monitoring
- Cloud DNS

---

# Autopilot

Nesse modo.

O Google administra praticamente toda infraestrutura.

Você paga pelos recursos consumidos.

---

# Comparando os serviços gerenciados

| Serviço | Destaques |
|----------|-----------|
| EKS | Forte integração com AWS |
| AKS | Excelente integração com Azure |
| GKE | Kubernetes mais maduro e recursos avançados |

---

# kubeadm

Ferramenta oficial para criação de clusters.

Muito utilizada para:

- Laboratórios
- On-premises
- Ambientes controlados

Você administra tudo.

---

# k3s

Distribuição extremamente leve.

Ideal para:

- Edge Computing
- IoT
- Raspberry Pi
- Ambientes pequenos

Consome poucos recursos.

---

# RKE2

Distribuição da SUSE/Rancher.

Muito utilizada em empresas.

Possui foco em:

- Segurança
- Compliance
- Ambientes corporativos

---

# OpenShift

Plataforma da Red Hat.

É mais do que Kubernetes.

Inclui.

- Registry
- CI/CD
- Segurança
- Operators
- Console Web
- GitOps
- Observabilidade

Muito utilizada em grandes empresas.

---

# Talos Linux

Sistema operacional minimalista.

Projetado exclusivamente para Kubernetes.

Não possui.

- SSH
- Shell tradicional
- Gerenciador de pacotes

Toda administração ocorre via API.

---

# Networking na Cloud

Todo cluster está dentro de uma rede.

Exemplo AWS.

```text
VPC

↓

Subnets

↓

Worker Nodes

↓

Pods
```

---

# Segurança

Nunca exponha diretamente os Nodes.

Boa prática.

```text
Internet

↓

Load Balancer

↓

Ingress

↓

Service

↓

Pods
```

---

# DNS

Normalmente utilizamos.

- Route53
- Azure DNS
- Cloud DNS

Integrados ao Ingress.

---

# Certificados

Ferramenta mais utilizada.

```text
cert-manager
```

Integra automaticamente com Let's Encrypt.

---

# Auto Scaling de Infraestrutura

Existem duas escalas diferentes.

Pods.

```text
HPA
```

Nodes.

```text
Cluster Autoscaler

ou

Karpenter
```

---

# Custos

Itens que mais impactam.

- Nodes
- Storage
- Load Balancers
- Tráfego
- NAT Gateway
- Logs

Sempre monitore utilização.

---

# Multi-AZ

Produção normalmente distribui Nodes.

```text
AZ A

AZ B

AZ C
```

Isso aumenta disponibilidade.

---

# Multi-Cluster

Empresas grandes normalmente possuem.

```text
Produção

Homologação

Desenvolvimento

DR
```

Ou.

Clusters separados por região.

---

# Hands-on

Verificar Nodes.

```bash
kubectl get nodes -o wide
```

---

Ver StorageClasses.

```bash
kubectl get storageclass
```

---

Ver LoadBalancer.

```bash
kubectl get svc
```

---

Ver eventos.

```bash
kubectl get events
```

---

Ver informações do cluster.

```bash
kubectl cluster-info
```

---

# Comparando distribuições

| Distribuição | Melhor uso |
|--------------|------------|
| EKS | AWS |
| AKS | Azure |
| GKE | GCP |
| kubeadm | On-premises/Laboratório |
| k3s | Edge/IoT |
| RKE2 | Enterprise |
| OpenShift | Grandes organizações |
| Talos | Ambientes imutáveis |

---

# Arquitetura de produção

```text
Internet
      │
Load Balancer
      │
Ingress
      │
Services
      │
Pods
      │
Worker Nodes
      │
VPC / VNet
      │
Control Plane Gerenciado
```

---

# Boas práticas

- Utilize serviços gerenciados quando possível.
- Distribua Nodes entre múltiplas zonas de disponibilidade.
- Utilize IAM/Managed Identity em vez de credenciais estáticas.
- Automatize upgrades do cluster.
- Monitore custos continuamente.
- Prefira CSI Drivers oficiais.
- Utilize Cluster Autoscaler ou Karpenter para otimizar recursos.

---

# Erros comuns

> [!WARNING]

Executar clusters self-managed sem necessidade operacional clara.

---

> [!WARNING]

Armazenar chaves de acesso em Secrets quando o provedor oferece identidade gerenciada.

---

> [!WARNING]

Concentrar todos os Nodes em uma única zona de disponibilidade.

---

> [!WARNING]

Não considerar custos de componentes auxiliares como NAT Gateways e Load Balancers.

---

# Resumo

Neste capítulo aprendemos que:

- Kubernetes pode ser gerenciado ou autogerenciado.
- EKS, AKS e GKE oferecem integração profunda com seus respectivos provedores.
- IAM, Managed Identity e Workload Identity eliminam a necessidade de credenciais estáticas.
- CSI Drivers permitem provisionamento dinâmico de armazenamento.
- Auto Scaling ocorre em diferentes camadas (Pods e Nodes).
- Distribuições como k3s, RKE2, OpenShift e Talos atendem cenários específicos.

---

# Checklist

Antes de continuar, verifique se você consegue responder:

- [ ] Qual a diferença entre Kubernetes gerenciado e self-managed?
- [ ] Quando escolher EKS, AKS ou GKE?
- [ ] Como funciona o provisionamento de volumes na cloud?
- [ ] Qual o papel de IAM Roles for Service Accounts (IRSA)?
- [ ] Quando utilizar Karpenter em vez de Cluster Autoscaler?
- [ ] O que caracteriza uma distribuição como k3s ou Talos?
- [ ] Como distribuir um cluster entre múltiplas zonas de disponibilidade?
- [ ] Quais são os principais fatores de custo de um cluster Kubernetes?

---

# Diagrama consolidado

```text
                    Internet
                        │
                  DNS (Route53/Azure DNS)
                        │
                 Load Balancer (ALB/NLB)
                        │
                     Ingress
                        │
                     Services
                        │
                       Pods
                        │
             Worker Nodes (EC2/VMSS/GCE)
                        │
        ┌───────────────┼────────────────┐
        │               │                │
      CSI            CNI/IPAM        IAM/Identity
        │               │                │
     Storage        Networking      Credenciais
        └───────────────┼────────────────┘
                        ▼
            Control Plane Gerenciado
        (EKS / AKS / GKE ou kubeadm)
```
