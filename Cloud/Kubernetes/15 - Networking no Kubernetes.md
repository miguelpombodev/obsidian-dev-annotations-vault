---
tags:
  - kubernetes
  - networking
  - cni
  - kube-proxy
  - coredns
  - networkpolicy
  - ebpf
  - iptables
---
> [!INFO]
> Objetivo
>
> Entender profundamente como funciona a rede do Kubernetes, desde a comunicação entre Pods até o acesso vindo da Internet, compreendendo o papel do CNI, kube-proxy, CoreDNS, Services, EndpointSlices e NetworkPolicies.

---

# Objetivos de aprendizagem

Ao concluir este capítulo você deverá ser capaz de:

- [ ] Explicar o modelo de rede do Kubernetes.
- [ ] Entender como Pods se comunicam.
- [ ] Compreender o funcionamento dos Services.
- [ ] Entender kube-proxy.
- [ ] Entender CoreDNS.
- [ ] Explicar CNI.
- [ ] Explicar como funciona uma requisição ponta a ponta.
- [ ] Entender NetworkPolicies.
- [ ] Diferenciar iptables, IPVS e eBPF.

---

# Pré-requisitos

- [[02 - Arquitetura do Kubernetes]]
- [[04 - Pods]]
- [[08 - Services]]
- [[14 - Ingress]]

---

# O grande objetivo da rede do Kubernetes

Existe apenas uma regra.

> Todo Pod deve conseguir conversar com qualquer outro Pod sem NAT.

Essa é a base de toda a arquitetura.

---

# Antes do Kubernetes

Imagine três containers Docker.

```text
Docker Bridge

Container A

↓

172.17.0.2

Container B

↓

172.17.0.3
```

Tudo acontece dentro da mesma máquina.

Agora imagine três Nodes.

```text
Node A

↓

Container
```

```text
Node B

↓

Container
```

Como eles conversam?

---

# O Modelo de Rede do Kubernetes

Cada Pod recebe seu próprio endereço IP.

Não o container.

O Pod.

```text
Pod

↓

10.244.1.15
```

Outro.

```text
10.244.2.7
```

Outro.

```text
10.244.3.10
```

Todos são roteáveis.

---

# Modelo Mental

Imagine uma empresa.

Cada funcionário possui um telefone próprio.

Não importa em qual andar trabalha.

Você liga diretamente.

É exatamente isso.

Cada Pod possui um IP próprio.

---

# As quatro regras fundamentais

O Kubernetes define quatro regras.

## 1

Todo Pod possui um IP único.

---

## 2

Pods conseguem conversar diretamente.

Sem NAT.

---

## 3

Nodes conseguem conversar entre si.

---

## 4

Containers dentro do mesmo Pod compartilham a mesma interface de rede.

---

# Por que Containers compartilham a rede?

Lembre-se.

Um Pod pode possuir vários containers.

```text
Pod

├── API

└── Sidecar
```

Ambos utilizam.

```text
localhost
```

Porque compartilham.

- IP
- Namespace de rede
- Interface

---

# Como um Pod recebe um IP?

Entra o CNI.

Container Network Interface.

---

# O que é um CNI?

O Kubernetes não implementa rede.

Ele delega essa responsabilidade.

Quem faz isso é o CNI.

Exemplos.

- Calico
- Cilium
- Flannel
- Weave
- Canal

---

# Fluxo

Novo Pod.

↓

Scheduler escolhe Node.

↓

Kubelet solicita.

↓

CNI cria interface.

↓

Atribui IP.

↓

Atualiza rotas.

↓

Pod Running.

---

# O que o CNI faz?

Muito mais do que atribuir IP.

Ele.

- cria interfaces virtuais;
- configura rotas;
- cria regras de firewall;
- conecta Nodes;
- aplica NetworkPolicies;
- pode implementar balanceamento.

---

# Comunicação Pod → Pod

Imagine.

```text
Pod A

↓

10.244.1.10
```

```text
Pod B

↓

10.244.2.7
```

Fluxo.

```text
Pod

↓

veth

↓

Bridge

↓

Node

↓

Rede

↓

Outro Node

↓

Bridge

↓

veth

↓

Pod
```

Sem NAT.

---

# O que é uma veth?

Virtual Ethernet Pair.

É como um cabo de rede virtual.

Uma ponta.

Container.

Outra.

Node.

---

# Como um Service funciona?

Aqui existe um conceito importante.

Service não é um proxy.

Service não é um container.

Service não possui processo.

Ele é apenas.

```text
IP Virtual
```

---

# ClusterIP

Imagine.

```text
Service

↓

10.96.0.20
```

Esse IP não pertence a nenhuma interface física.

É um IP virtual.

---

# Quem responde esse IP?

kube-proxy.

---

# kube-proxy

Apesar do nome.

Ele normalmente não faz proxy.

Ele configura regras.

Dependendo do modo.

- iptables
- IPVS
- eBPF (quando substituído)

---

# Fluxo

Cliente.

↓

Service IP.

↓

iptables.

↓

EndpointSlice.

↓

Pod.

---

# EndpointSlice

Lembra do capítulo de Services?

Quando um Service possui.

```text
3 Pods
```

O EndpointSlice guarda.

```text
10.244.1.10

10.244.2.15

10.244.3.8
```

O kube-proxy consulta essa lista.

---

# Balanceamento

Cada nova conexão.

↓

iptables.

↓

Escolhe um Endpoint.

↓

Encaminha.

---

# IPVS

Mais moderno.

Mais eficiente.

Mantém tabelas em memória.

Escala melhor.

Muito utilizado em clusters grandes.

---

# eBPF

Aqui começa a nova geração.

Em vez de.

```text
iptables
```

Ou.

```text
IPVS
```

Executamos programas diretamente no Kernel.

Benefícios.

- menor latência;
- menos cópias;
- maior throughput;
- observabilidade;
- segurança.

É exatamente isso que o Cilium faz.

---

# CoreDNS

Imagine.

API quer acessar.

```text
postgres.default.svc.cluster.local
```

Quem resolve?

CoreDNS.

---

# Fluxo DNS

Aplicação.

↓

Resolver.

↓

CoreDNS.

↓

ClusterIP.

↓

Service.

---

# DNS interno

Todo Service recebe automaticamente.

```text
service.namespace.svc.cluster.local
```

Exemplo.

```text
redis.default.svc.cluster.local
```

A aplicação normalmente usa apenas.

```text
redis
```

Porque o namespace é inferido.

---

# Fluxo completo

Imagine.

API acessando PostgreSQL.

```text
API

↓

postgres

↓

CoreDNS

↓

ClusterIP

↓

kube-proxy

↓

EndpointSlice

↓

PostgreSQL Pod
```

---

# NetworkPolicy

Por padrão.

Todo mundo conversa com todo mundo.

Isso nem sempre é desejável.

---

Imagine.

```text
Frontend

↓

API

↓

Banco
```

Queremos impedir.

```text
Frontend

↓

Banco
```

Diretamente.

Criamos uma NetworkPolicy.

---

# O que ela faz?

Ela define.

Quem pode conversar com quem.

Exemplo.

```text
Frontend

↓

API

↓

Banco
```

Permitido.

```text
Frontend

↓

Banco
```

Negado.

---

# Atenção

Uma dúvida extremamente comum.

> NetworkPolicy funciona sempre?

Não.

Ela depende do CNI.

Plugins simples como Flannel, por exemplo, não implementam NetworkPolicies sozinhos.

Já Calico e Cilium implementam.

---

# Comunicação completa

Agora temos.

```text
Browser

↓

DNS Público

↓

Load Balancer

↓

Ingress Controller

↓

Ingress

↓

Service

↓

kube-proxy

↓

EndpointSlice

↓

Pod

↓

Aplicação

↓

CoreDNS

↓

Service PostgreSQL

↓

kube-proxy

↓

Banco
```

Todo esse fluxo acontece em poucos milissegundos.

---

# Hands-on

Verificar Pods.

```bash
kubectl get pods -o wide
```

Ver IPs.

---

Consultar Services.

```bash
kubectl get svc
```

---

Consultar Endpoints.

```bash
kubectl get endpointslices
```

---

Consultar DNS.

```bash
kubectl exec -it pod-api -- nslookup postgres
```

---

Verificar NetworkPolicies.

```bash
kubectl get networkpolicies
```

---

# Comparando as tecnologias

| Tecnologia | Função |
|------------|--------|
| CNI | Implementa a rede |
| CoreDNS | Resolve nomes |
| kube-proxy | Encaminha tráfego para Services |
| Service | IP Virtual |
| EndpointSlice | Lista de Pods |
| Ingress | Regras HTTP |
| Ingress Controller | Proxy reverso |

---

# Relação com os capítulos anteriores

Agora conseguimos enxergar toda a arquitetura.

```text
Internet
      │
DNS Público
      │
Load Balancer
      │
Ingress Controller
      │
Ingress
      │
Service
      │
kube-proxy
      │
EndpointSlice
      │
Pods
      │
CNI
      │
Worker Nodes
      │
CoreDNS
      │
Outros Services
```

---

# Boas práticas

- Utilize um CNI moderno (Cilium ou Calico).
- Utilize NetworkPolicies desde o início do projeto.
- Evite comunicação por IP; utilize sempre Services.
- Monitore CoreDNS.
- Utilize eBPF quando disponível para reduzir latência e melhorar observabilidade.

---

# Erros comuns

> [!WARNING]

Achar que o Service possui um processo executando.

Ele é apenas um IP virtual.

---

> [!WARNING]

Pensar que kube-proxy sempre atua como um proxy tradicional.

Na maioria das implementações, ele apenas programa regras do kernel.

---

> [!WARNING]

Achar que toda CNI suporta NetworkPolicies.

Verifique as capacidades do plugin utilizado.

---

> [!WARNING]

Conectar aplicações diretamente pelo IP dos Pods.

Esses IPs podem mudar quando os Pods são recriados.

Sempre utilize Services.

---

# Resumo

Neste capítulo aprendemos que:

- Cada Pod possui um IP próprio.
- O CNI implementa a rede do cluster.
- Services fornecem IPs virtuais.
- kube-proxy encaminha conexões para os Pods.
- CoreDNS resolve nomes internos.
- EndpointSlices armazenam os endpoints dos Services.
- NetworkPolicies controlam a comunicação entre Pods.
- eBPF representa a evolução do modelo tradicional baseado em iptables.

---

# Checklist

Antes de continuar, verifique se você consegue responder:

- [ ] Por que cada Pod possui um IP próprio?
- [ ] Qual é a responsabilidade do CNI?
- [ ] O que é um ClusterIP?
- [ ] Qual o papel do kube-proxy?
- [ ] Como o CoreDNS participa da comunicação?
- [ ] O que é um EndpointSlice?
- [ ] Quando uma NetworkPolicy entra em ação?
- [ ] Quais as vantagens do eBPF?

---

# Diagrama consolidado

```text
                     Internet
                         │
                   DNS Público
                         │
                  Load Balancer
                         │
               Ingress Controller
                         │
                     Ingress
                         │
                     Service
                         │
                   kube-proxy
                         │
                  EndpointSlice
                         │
                  Pod (Cluster IP)
                         │
             ┌───────────┼───────────┐
             │           │           │
         ConfigMap    Secret     Volumes
                                      │
                                     PVC
                                      │
                                      PV
                                      │
                                StorageClass
                                      │
                                 CSI Driver
                                      │
                                 Cloud Storage

                  ▲
                  │
              CoreDNS
                  │
             CNI Plugin
                  │
          Rede entre Nodes
```
