---
tags:
  - kubernetes
  - daemonset
  - scheduler
  - nodes
  - observability
  - networking
---

# 12 - DaemonSets

> [!INFO] Objetivo
> Entender como executar um Pod em todos (ou em um subconjunto) dos Nodes do cluster, compreender o funcionamento dos DaemonSets e aprofundar o conhecimento sobre agendamento de Pods no Kubernetes.

---

# Objetivos de aprendizagem

Ao concluir este capítulo você deverá ser capaz de:

- [ ] Explicar o que é um DaemonSet.
- [ ] Diferenciar Deployment, StatefulSet e DaemonSet.
- [ ] Entender como o Scheduler decide onde executar Pods.
- [ ] Compreender Node Selector, Node Affinity, Taints e Tolerations.
- [ ] Identificar casos reais de uso para DaemonSets.

---

# Pré-requisitos

- [[02 - Arquitetura do Kubernetes]]
- [[07 - Deployments]]
- [[11 - StatefulSets]]

---

# O problema que este capítulo resolve

Imagine um cluster com cinco Worker Nodes.

```text
Worker 1

Worker 2

Worker 3

Worker 4

Worker 5
```

Agora imagine que você deseja instalar um agente de logs.

Ele precisa capturar os logs de todos os Nodes.

Como garantir isso?

Criando cinco Deployments?

Não.

Criando cinco Pods manualmente?

Também não.

A resposta é um DaemonSet.

---

# Modelo Mental

Imagine uma empresa com cinco filiais.

Cada filial precisa obrigatoriamente de:

- um vigilante;
- um roteador;
- um gerador;
- uma câmera de segurança.

Não importa quantas filiais existam.

Cada uma deve possuir exatamente um equipamento.

É exatamente isso que um DaemonSet faz.

---

# O que é um DaemonSet?

Um DaemonSet é um Controller que garante que exista uma instância de um Pod em cada Node selecionado.

Visualmente.

```text
Worker 1

↓

Pod
```

```text
Worker 2

↓

Pod
```

```text
Worker 3

↓

Pod
```

Sempre.

---

# Como funciona?

Deployment.

```text
3 réplicas

↓

Scheduler decide
```

Pode acontecer.

```text
Node A

2 Pods
```

```text
Node B

1 Pod
```

```text
Node C

0 Pods
```

Tudo certo.

Deployment não se importa.

---

DaemonSet.

```text
Node A

↓

1 Pod
```

```text
Node B

↓

1 Pod
```

```text
Node C

↓

1 Pod
```

Sempre.

---

# O que acontece quando um novo Node entra?

Imagine.

Cluster.

```text
Node A

Node B
```

DaemonSet.

```text
Agente
```

Resultado.

```text
Node A

↓

Agente
```

```text
Node B

↓

Agente
```

Agora um novo Node entra.

```text
Node C
```

Automaticamente.

```text
Node C

↓

Agente
```

Nenhuma ação manual é necessária.

---

# O que acontece quando um Node sai?

Se um Node for removido.

O Pod correspondente também desaparece.

Como o Node não existe mais.

Não há motivo para manter aquele Pod.

---

# Anatomia de um DaemonSet

```yaml
apiVersion: apps/v1

kind: DaemonSet

metadata:

  name: fluent-bit

spec:

  selector:

    matchLabels:

      app: fluent-bit

  template:

    metadata:

      labels:

        app: fluent-bit

    spec:

      containers:

      - name: fluent-bit

        image: fluent/fluent-bit
```

Observe.

Não existe.

```yaml
replicas:
```

Porque o número de Pods depende da quantidade de Nodes.

---

# Deployment x DaemonSet

Deployment.

```text
Desejo 5 Pods.
```

DaemonSet.

```text
Desejo 1 Pod por Node.
```

Essa diferença é extremamente importante.

---

# Como o Scheduler funciona?

Até agora falamos pouco sobre ele.

Fluxo.

```text
kubectl apply

↓

API Server

↓

etcd

↓

Scheduler

↓

Worker Node

↓

Kubelet

↓

Container Runtime
```

O Scheduler decide onde executar um Pod.

---

# Critérios utilizados pelo Scheduler

Ele considera diversos fatores.

- Recursos disponíveis
- Afinidade
- Anti-affinity
- Node Selector
- Taints
- Tolerations
- Topologia
- Prioridade
- Afinidade de volumes

Não é uma decisão aleatória.

---

# Node Selector

Imagine.

Nodes.

```text
Node A

ssd=true
```

```text
Node B

ssd=false
```

Deployment.

```yaml
nodeSelector:

  ssd: "true"
```

Resultado.

O Pod só poderá executar no Node A.

---

# Node Affinity

É uma evolução do Node Selector.

Permite regras muito mais sofisticadas.

Exemplo.

```text
Região = us-east

OU

Disco = SSD
```

Também suporta preferências.

Não apenas regras obrigatórias.

---

# Taints

Agora imagine o contrário.

Você quer impedir que Pods sejam executados em determinado Node.

Aplicamos um Taint.

```text
Node

↓

NoSchedule
```

Agora.

Nenhum Pod será executado nele.

---

# Tolerations

Alguns Pods precisam ignorar esse bloqueio.

Eles recebem uma Toleration.

Fluxo.

```text
Node

↓

Taint

↓

Pod possui Toleration

↓

Pode executar
```

Muito comum em DaemonSets.

---

# Exemplo real

Node de GPU.

```text
gpu=true
```

Aplicamos.

```text
NoSchedule
```

Apenas workloads de IA devem utilizar esse Node.

Todos os outros Pods serão impedidos.

---

# DaemonSets normalmente possuem Tolerations

Por quê?

Porque alguns Nodes possuem Taints.

Mesmo assim.

Queremos instalar.

- CNI
- Agente de logs
- Agente de monitoramento

Inclusive em Nodes especiais.

---

# Casos reais

## Observabilidade

Node Exporter.

```text
Cada Node

↓

Node Exporter
```

---

## Logs

Fluent Bit.

```text
Cada Node

↓

Coletar logs
```

---

## Segurança

Falco.

```text
Cada Node

↓

Monitorar chamadas de sistema
```

---

## Rede

Cilium.

```text
Cada Node

↓

Plugin de Rede
```

---

## Cloud

AWS VPC CNI.

```text
Cada Node

↓

Gerenciar interfaces de rede
```

---

# Hands-on

Criar DaemonSet.

```bash
kubectl apply -f daemonset.yaml
```

Verificar.

```bash
kubectl get daemonsets
```

Listar Pods.

```bash
kubectl get pods -o wide
```

Observe.

Cada Pod estará em um Node diferente.

---

Adicionar novo Node.

```text
Novo Worker
```

Verificar novamente.

```bash
kubectl get pods -o wide
```

Novo Pod criado automaticamente.

---

# Comparando os Controllers

| Controller | Objetivo |
|------------|----------|
| Deployment | Aplicações stateless |
| StatefulSet | Aplicações stateful |
| DaemonSet | Um Pod por Node |
| Job | Executar uma tarefa até terminar |
| CronJob | Executar tarefas agendadas |

---

# Relação com os capítulos anteriores

Agora nosso modelo mental fica assim.

```text
                    Scheduler
                        │
        ┌───────────────┼───────────────┐
        │               │               │
   Deployment      StatefulSet     DaemonSet
        │               │               │
   ReplicaSet        Pods         Um Pod por Node
        │               │               │
       Pods       PVC por Pod     Worker Nodes
```

---

# Boas práticas

- Utilize DaemonSets apenas para componentes de infraestrutura.
- Evite executar aplicações de negócio como DaemonSets.
- Utilize Node Affinity para restringir onde o DaemonSet pode ser executado.
- Adicione Tolerations quando o DaemonSet precisar rodar em Nodes com Taints.

---

# Erros comuns

> [!WARNING]

Esperar que um DaemonSet distribua carga como um Deployment.

DaemonSets não fazem balanceamento.

---

> [!WARNING]

Executar APIs REST como DaemonSets.

Isso normalmente desperdiça recursos.

---

> [!WARNING]

Ignorar Taints dos Nodes.

Em muitos clusters gerenciados, alguns Nodes possuem Taints por padrão.

---

# Resumo

Neste capítulo aprendemos que:

- DaemonSets garantem um Pod por Node.
- Eles são amplamente utilizados para infraestrutura.
- O Scheduler decide onde os Pods serão executados.
- Node Selector e Node Affinity restringem o agendamento.
- Taints bloqueiam Nodes.
- Tolerations permitem que Pods ignorem esses bloqueios.

---

# Checklist

Antes de continuar, verifique se você consegue responder:

- [ ] Qual problema um DaemonSet resolve?
- [ ] Qual a diferença entre Deployment e DaemonSet?
- [ ] O que acontece quando um novo Node entra no cluster?
- [ ] O que faz o Scheduler?
- [ ] Qual a diferença entre Node Selector e Node Affinity?
- [ ] O que são Taints e Tolerations?

---

# Diagrama consolidado

```text
                         Scheduler
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
   Deployment           StatefulSet          DaemonSet
        │                     │                     │
   ReplicaSet          Pods (Identidade)   Um Pod por Node
        │                     │                     │
       Pods              PVC por Pod        Worker Nodes
   ┌────┼────┐                │
   │    │    │                │
Config Secret Volumes         │
         │                    │
         └──────────────┬─────┘
                        ▼
                   Containers
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
```
