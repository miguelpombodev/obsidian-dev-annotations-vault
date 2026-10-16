---
tags:
  - kubernetes
  - cloud-native
  - cncf
  - ecosystem
  - operators
  - crd
  - service-mesh
  - gateway-api
  - kubeflow
  - backstage
---
> [!INFO]
> Objetivo
>
> Entender como Kubernetes se tornou o núcleo do ecossistema Cloud Native e conhecer as principais tecnologias que complementam a plataforma em ambientes modernos de produção.

---

# Objetivos de aprendizagem

Ao concluir este capítulo você deverá ser capaz de:

- [ ] Entender o conceito de Cloud Native.
- [ ] Conhecer a CNCF.
- [ ] Compreender CRI, CNI e CSI.
- [ ] Entender CRDs e Operators.
- [ ] Conhecer Service Mesh.
- [ ] Entender API Gateway.
- [ ] Conhecer ferramentas do ecossistema.
- [ ] Entender tendências como eBPF, WASM e Gateway API.

---

# Pré-requisitos

- Todos os capítulos anteriores.

---

# Mudança de Mentalidade

Kubernetes não é um produto.

Kubernetes é uma plataforma extensível.

Seu maior diferencial é justamente permitir que novas funcionalidades sejam adicionadas sem alterar seu núcleo.

---

# O que significa Cloud Native?

Cloud Native não significa "rodando na Cloud".

Cloud Native é uma abordagem arquitetural baseada em:

- Containers
- Microsserviços
- Automação
- Infraestrutura como Código
- APIs
- Observabilidade
- Escalabilidade
- Resiliência

O Kubernetes é uma das tecnologias que viabilizam esse modelo.

---

# A CNCF

A **Cloud Native Computing Foundation (CNCF)** é a organização responsável por hospedar e governar diversos projetos do ecossistema Cloud Native.

Alguns dos principais projetos são:

- Kubernetes
- Prometheus
- containerd
- Helm
- Argo
- Fluent Bit
- OpenTelemetry
- Cilium
- Harbor
- KEDA
- Backstage (sandbox)
- Envoy
- Linkerd
- Istio (em parceria)

---

# CNCF Landscape

O CNCF Landscape é um mapa do ecossistema Cloud Native.

Ele organiza centenas de projetos em categorias como:

```text
Provisioning

Runtime

Storage

Networking

Security

Observability

CI/CD

Database

Service Mesh

Application Definition

Platform Engineering

AI/ML
```

O objetivo não é conhecer todas as ferramentas, mas saber qual problema cada categoria resolve.

---

# O Modelo de Extensibilidade do Kubernetes

Grande parte da flexibilidade do Kubernetes vem de três interfaces fundamentais:

```text
CRI

CNI

CSI
```

---

# CRI — Container Runtime Interface

Define como o Kubernetes conversa com o runtime de containers.

Exemplos:

- containerd
- CRI-O

Historicamente existia suporte ao Docker, mas ele foi removido a partir da versão 1.24.

---

# CNI — Container Network Interface

Já estudamos este conceito.

Responsável por implementar a rede do cluster.

Exemplos:

- Cilium
- Calico
- Flannel
- Weave

Além da conectividade, alguns CNIs implementam:

- NetworkPolicies
- Load Balancing
- eBPF
- Observabilidade

---

# CSI — Container Storage Interface

Responsável pela integração com sistemas de armazenamento.

Exemplos:

- Amazon EBS CSI
- Azure Disk CSI
- GCE PD CSI
- Ceph CSI
- Longhorn
- NFS CSI

Graças ao CSI, o Kubernetes consegue provisionar volumes dinamicamente.

---

# CRDs (Custom Resource Definitions)

Uma das funcionalidades mais poderosas do Kubernetes.

Permitem criar novos tipos de recursos.

Exemplo:

```text
Deployment
```

é um recurso nativo.

Mas podemos criar:

```text
Database

KafkaCluster

RedisCluster

Certificate
```

Todos como recursos da API do Kubernetes.

---

# Operator Pattern

Se CRDs adicionam novos tipos de objetos...

Quem implementa a lógica?

Os Operators.

Um Operator é um controlador especializado.

Modelo:

```text
CRD

↓

Operator

↓

Recursos Kubernetes
```

---

# Exemplo

Criamos:

```yaml
kind: PostgreSQLCluster
```

O Operator automaticamente cria:

- StatefulSets
- Services
- PVCs
- Backups
- Failover

Tudo de forma automatizada.

---

# Exemplos de Operators

- PostgreSQL Operator
- RabbitMQ Operator
- Kafka Operator
- Prometheus Operator
- Elastic Operator
- Strimzi
- Cert-Manager

---

# Service Mesh

Até agora vimos comunicação entre Pods.

Mas...

Como controlar:

- retries;
- circuit breakers;
- mTLS;
- observabilidade;
- políticas de tráfego?

Entra o Service Mesh.

---

# Modelo Mental

Sem Service Mesh.

```text
Aplicação

↓

Aplicação
```

Com Service Mesh.

```text
Aplicação

↓

Sidecar Proxy

↓

Rede

↓

Sidecar Proxy

↓

Aplicação
```

---

# Sidecar Proxy

O proxy intercepta todo o tráfego.

Isso permite:

- autenticação mútua;
- criptografia;
- métricas;
- tracing;
- políticas de tráfego.

---

# Principais Service Meshes

- Istio
- Linkerd
- Cilium Service Mesh
- Kuma

---

# API Gateway

Muitas pessoas confundem API Gateway com Ingress.

Eles possuem responsabilidades diferentes.

Ingress:

- roteamento HTTP;
- TLS;
- balanceamento.

API Gateway adiciona funcionalidades como:

- autenticação;
- rate limiting;
- quotas;
- transformação de requisições;
- versionamento de APIs;
- monetização.

---

# Exemplos

- Kong
- Ambassador
- Apache APISIX
- NGINX Plus
- Envoy Gateway

---

# Gateway API

É a evolução do Ingress.

Fornece:

- melhor separação de responsabilidades;
- suporte a TCP, UDP e HTTP;
- extensibilidade;
- padronização.

Ela tende a substituir gradualmente o modelo tradicional de Ingress.

---

# KEDA

Kubernetes escala normalmente baseado em CPU e memória.

Mas e se quisermos escalar baseado em:

- mensagens no Kafka?
- tamanho de uma fila RabbitMQ?
- eventos do Azure Service Bus?

Utilizamos o KEDA.

---

# Harbor

Registry corporativo de imagens.

Funcionalidades:

- controle de acesso;
- replicação;
- scan de vulnerabilidades;
- assinatura de imagens.

---

# Crossplane

Permite gerenciar infraestrutura usando Kubernetes.

Exemplo.

Criar um banco RDS.

```yaml
kind: RDSInstance
```

Aplicar.

```bash
kubectl apply
```

O Crossplane cria o recurso na AWS.

É Kubernetes atuando como plano de controle para infraestrutura.

---

# Knative

Leva Serverless ao Kubernetes.

Permite:

- scale to zero;
- eventos;
- funções sob demanda.

---

# Kubeflow

Plataforma para Machine Learning.

Inclui:

- pipelines;
- notebooks;
- treinamento distribuído;
- serving de modelos.

Muito utilizada em ambientes de IA.

---

# Backstage

Projeto criado pelo Spotify.

Objetivo.

Construir um portal interno para desenvolvedores.

Funcionalidades:

- catálogo de serviços;
- documentação;
- templates;
- scaffolding;
- integração com GitOps.

É uma das principais ferramentas de Platform Engineering.

---

# eBPF

Uma das maiores tendências do ecossistema.

Permite executar programas diretamente no Kernel Linux.

Benefícios:

- observabilidade;
- networking;
- segurança;
- baixa latência.

Projetos que utilizam eBPF:

- Cilium
- Tetragon
- Pixie

---

# WebAssembly (WASM)

Outra tendência.

Permite executar cargas de trabalho leves, isoladas e portáveis.

Pode complementar containers em cenários específicos, como plugins e edge computing.

---

# Plataforma Cloud Native Moderna

Uma plataforma típica pode ser representada assim:

```text
                Backstage
                     │
                 Git / GitOps
                     │
                 Argo CD
                     │
              Kubernetes Cluster
                     │
 ┌───────────┬────────────┬────────────┐
 │           │            │            │
Helm      Operators   Service Mesh  Observability
 │           │            │            │
 └───────────┴────────────┴────────────┘
                     │
             Aplicações e Dados
```

---

# Como tudo se conecta

```text
CRI
 │
 ▼
Container Runtime

CNI
 │
 ▼
Networking

CSI
 │
 ▼
Storage

CRDs
 │
 ▼
Operators

GitOps
 │
 ▼
Entrega Contínua

Observabilidade
 │
 ▼
Operação

Service Mesh
 │
 ▼
Comunicação Segura
```

---

# Hands-on

Verificar CRDs instaladas:

```bash
kubectl get crds
```

---

Listar APIs disponíveis:

```bash
kubectl api-resources
```

---

Ver API Versions:

```bash
kubectl api-versions
```

---

Listar Operators instalados:

```bash
kubectl get deployment -A
```

(identificando operadores como `cert-manager`, `prometheus-operator`, etc.)

---

# Boas práticas

- Utilize CRDs para representar recursos de negócio quando fizer sentido.
- Prefira Operators maduros para softwares complexos.
- Avalie Service Mesh apenas quando houver necessidade real.
- Adote Gateway API em novos projetos sempre que suportado.
- Centralize infraestrutura declarativa utilizando GitOps.
- Evite adicionar ferramentas apenas porque são populares; cada componente aumenta a complexidade operacional.

---

# Erros comuns

> [!WARNING]

Instalar dezenas de projetos CNCF sem necessidade.

Cada componente adiciona custo operacional e curva de aprendizado.

---

> [!WARNING]

Utilizar Service Mesh para aplicações simples.

Nem toda aplicação precisa de um proxy sidecar.

---

> [!WARNING]

Criar CRDs sem uma necessidade clara.

Nem todo problema deve ser resolvido estendendo a API do Kubernetes.

---

> [!WARNING]

Confundir API Gateway com Ingress.

Eles atuam em camadas diferentes e possuem responsabilidades distintas.

---

# Resumo

Neste capítulo aprendemos que:

- Kubernetes é o núcleo de um ecossistema Cloud Native muito maior.
- A CNCF organiza e mantém diversos projetos do ecossistema.
- CRI, CNI e CSI são interfaces fundamentais de extensibilidade.
- CRDs permitem criar novos recursos para a API do Kubernetes.
- Operators automatizam o ciclo de vida de aplicações complexas.
- Service Mesh adiciona capacidades avançadas de comunicação.
- API Gateways complementam o Ingress com funcionalidades de gestão de APIs.
- Tecnologias como Crossplane, Knative, Kubeflow, Backstage, eBPF e WASM ampliam o papel do Kubernetes como plataforma.

---

# Checklist

Antes de continuar, verifique se você consegue responder:

- [ ] O que caracteriza uma aplicação Cloud Native?
- [ ] Qual o papel da CNCF?
- [ ] Qual a diferença entre CRI, CNI e CSI?
- [ ] O que é uma CRD?
- [ ] Como funciona um Operator?
- [ ] Quando utilizar um Service Mesh?
- [ ] Qual a diferença entre Ingress e API Gateway?
- [ ] O que faz o KEDA?
- [ ] Como o Crossplane utiliza o Kubernetes como plano de controle?
- [ ] Quais tendências estão moldando o futuro do ecossistema Cloud Native?

---

# Diagrama consolidado

```text
                     Backstage
                         │
                      GitOps
                         │
                     Argo CD
                         │
                  Kubernetes API
                         │
      ┌──────────────────┼──────────────────┐
      │                  │                  │
     CRDs              Helm           Gateway API
      │                  │                  │
 Operators         Aplicações        Ingress/Envoy
      │                  │                  │
      ├──────────┬────────┴─────────┐
      │          │                  │
     CNI        CSI               CRI
      │          │                  │
 Networking   Storage       Container Runtime
      │          │                  │
      └──────────┴──────────┬───────┘
                             ▼
                      Cluster Kubernetes
```
