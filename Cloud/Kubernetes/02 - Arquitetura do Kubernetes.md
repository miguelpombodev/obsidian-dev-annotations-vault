---
tags:
  - kubernetes
  - arquitetura
  - control-plane
  - worker-node
---
> [!INFO] Objetivo
> Entender como o Kubernetes é organizado internamente, como seus componentes colaboram para manter o estado desejado e qual é o fluxo completo desde um `kubectl apply` até a criação de um Pod.

---

# Objetivos de aprendizagem

Ao concluir este capítulo você deverá ser capaz de:

- [ ] Explicar o que é um Cluster Kubernetes.
- [ ] Diferenciar Control Plane de Worker Nodes.
- [ ] Entender como os componentes do cluster se comunicam.
- [ ] Explicar o conceito de **Control Loop** (Reconciliation Loop).
- [ ] Entender o fluxo completo de um `kubectl apply`.
- [ ] Conhecer a responsabilidade de cada componente do Control Plane.

---

# Pré-requisitos

- [[03 - Objetos da API]]
- Docker
- Conceitos básicos de Linux
- Conceitos básicos de Redes

---

# O problema que este capítulo resolve

No capítulo anterior entendemos **por que** o Kubernetes foi criado.

Agora precisamos responder outra pergunta:

> **Como o Kubernetes consegue manter uma aplicação funcionando mesmo quando containers morrem, máquinas falham ou a carga aumenta?**

Para responder isso, precisamos entender como o cluster funciona internamente.

---

# Modelo Mental

Antes de estudar cada componente, é importante entender como o Kubernetes "pensa".

O Docker funciona de forma imperativa.

Você executa um comando.

```bash
docker run nginx
```

O Docker apenas executa o container.

O Kubernetes funciona de maneira completamente diferente.

Você declara um estado desejado.

```yaml
replicas: 3
```

A partir desse momento, o cluster trabalha continuamente para manter esse estado.

Nunca dizemos:

> Crie três containers.

Nós dizemos:

> Quero três réplicas desta aplicação.

Essa diferença muda completamente a forma de operar um ambiente distribuído.

---

# O Kubernetes é um sistema declarativo

O Kubernetes trabalha comparando dois estados.

```text
Estado Desejado

↓

Estado Atual

↓

Existe diferença?

↓

Sim

↓

Corrigir diferença
```

Esse processo acontece continuamente.

O cluster nunca "termina" seu trabalho.

Ele está constantemente observando e reconciliando o ambiente.

Esse mecanismo é chamado de **Reconciliation Loop**.

---

# O Reconciliation Loop

Imagine que você possui um Deployment.

```yaml
spec:
  replicas: 3
```

O Controller verifica continuamente:

```text
Quantas réplicas deveriam existir?

↓

3
```

Depois verifica:

```text
Quantas existem agora?

↓

2
```

Existe diferença?

Sim.

Então ele cria mais uma.

```text
Desejado

3

↓

Atual

2

↓

Criar 1 Pod
```

Alguns segundos depois:

```text
Desejado

3

↓

Atual

3
```

Agora ele apenas continua observando.

> [!IMPORTANT]
> O Kubernetes não executa tarefas apenas uma vez.
>
> Ele executa milhares de ciclos de reconciliação continuamente.

---

# O que é um Cluster?

Um Cluster Kubernetes é um conjunto de máquinas trabalhando em conjunto.

Essas máquinas possuem responsabilidades diferentes.

```text
                Kubernetes Cluster

        ┌────────────────────────────┐

        │      Control Plane         │

        └────────────────────────────┘

                │

────────────────┼────────────────────────────

                │

     Worker 1   Worker 2   Worker 3
```

O cluster é dividido em dois grandes grupos:

- Control Plane
- Worker Nodes

---

# Control Plane

O Control Plane pode ser entendido como o cérebro do cluster.

Ele **não executa aplicações**.

Sua responsabilidade é tomar decisões.

Por exemplo:

- Onde um Pod deve executar?
- Quantas réplicas devem existir?
- Algum Node falhou?
- Algum Deployment precisa ser atualizado?
- Existe algum Pod que deve ser recriado?

> [!TIP]
> Pense no Control Plane como um sistema operacional distribuído.

---

# Worker Nodes

Os Worker Nodes são responsáveis por executar as aplicações.

Eles apenas seguem as instruções recebidas do Control Plane.

```text
Control Plane

↓

Criar um Pod

↓

Worker Node

↓

Container Runtime

↓

Container
```

Enquanto o Control Plane decide.

Os Workers executam.

---

# Arquitetura Geral

```text
                    kubectl

                       │

                       ▼

                 kube-apiserver

        ┌──────────────┼──────────────┐

        ▼              ▼              ▼

      etcd        Scheduler     Controller Manager

──────────────────────────────────────────────────────

                    Worker Node

         kubelet

         kube-proxy

         containerd

              │

             Pods
```

---

# Como os componentes se comunicam?

Uma das características mais importantes do Kubernetes é:

> [!IMPORTANT]
> Quase nenhum componente conversa diretamente com outro.

Todos conversam com o **API Server**.

Exemplo:

```text
kubectl

↓

API Server
```

---

```text
Scheduler

↓

API Server
```

---

```text
Controller Manager

↓

API Server
```

---

```text
kubelet

↓

API Server
```

O API Server é a única porta de entrada do cluster.

Essa arquitetura traz diversas vantagens:

- autenticação centralizada;
- autorização;
- auditoria;
- versionamento da API;
- consistência dos dados.

---

# Componentes do Control Plane

## kube-apiserver

É o componente central do Kubernetes.

Toda comunicação passa por ele.

Quando executamos:

```bash
kubectl apply -f deployment.yaml
```

O kubectl envia uma requisição HTTPS para o API Server.

O API Server:

- autentica o usuário;
- autoriza a operação;
- valida o manifesto YAML;
- grava o objeto no etcd;
- disponibiliza a alteração para os demais componentes.

---

## etcd

O etcd é um banco de dados distribuído do tipo Key-Value.

Ele armazena todo o estado do cluster.

Exemplos:

- Pods
- Deployments
- Services
- ConfigMaps
- Secrets
- Nodes
- Namespaces

> [!WARNING]
> Se o etcd for perdido e não existir backup, praticamente todo o estado do cluster será perdido.

---

## Scheduler

O Scheduler decide onde um Pod será executado.

Ele analisa fatores como:

- CPU disponível;
- memória disponível;
- afinidade;
- anti-affinity;
- taints;
- tolerations;
- políticas de agendamento.

O Scheduler **não cria Pods**.

Ele apenas escolhe o Node mais adequado.

---

## Controller Manager

O Controller Manager implementa o conceito de Reconciliation Loop.

Ele verifica continuamente se o estado atual corresponde ao estado desejado.

Exemplo:

```text
Desejado: 5 Pods

↓

Atual: 4 Pods

↓

Criar mais 1
```

Existem diversos Controllers.

Alguns exemplos:

- Deployment Controller
- ReplicaSet Controller
- Job Controller
- Node Controller
- Namespace Controller

Cada Controller possui uma responsabilidade específica.

---

# Componentes do Worker Node

Cada Worker possui alguns componentes fundamentais.

## kubelet

O kubelet é o agente instalado em cada Worker.

Ele recebe instruções do API Server.

Depois solicita ao runtime a criação dos containers.

Também monitora continuamente os Pods.

---

## Container Runtime

O Kubernetes não executa containers diretamente.

Ele utiliza um runtime.

Hoje o mais comum é:

- containerd

Outros exemplos:

- CRI-O

O runtime é responsável por:

- baixar imagens;
- criar containers;
- iniciar processos;
- remover containers.

---

## kube-proxy

O kube-proxy implementa parte da rede do Kubernetes.

Suas principais responsabilidades são:

- regras de iptables/IPVS;
- balanceamento interno;
- roteamento para Services;
- comunicação entre Pods.

Mais adiante veremos esse componente em detalhes.

---

# Fluxo completo de um kubectl apply

Imagine o seguinte comando:

```bash
kubectl apply -f deployment.yaml
```

O fluxo interno é:

```text
kubectl

↓

API Server

↓

Validação

↓

Persistência no etcd

↓

Controller detecta novo Deployment

↓

ReplicaSet

↓

Novo Pod

↓

Scheduler escolhe Node

↓

API Server registra decisão

↓

kubelet recebe atualização

↓

containerd baixa imagem

↓

Container inicia

↓

Pod Running
```

Esse fluxo representa o ciclo básico de criação de praticamente qualquer recurso no Kubernetes.

---

# Responsabilidade de cada componente

| Componente | Responsabilidade |
|------------|------------------|
| API Server | Porta de entrada do cluster |
| etcd | Persistência do estado |
| Scheduler | Escolha do Node |
| Controller Manager | Reconciliação do estado |
| kubelet | Execução dos Pods |
| containerd | Execução dos containers |
| kube-proxy | Rede e roteamento interno |

---

# Como isso aparece na prática

Imagine que um desenvolvedor executa:

```bash
kubectl apply -f deployment.yaml
```

Ele vê apenas um comando.

Mas internamente o Kubernetes:

1. Valida o manifesto.
2. Persiste o Deployment.
3. Detecta que um novo Deployment existe.
4. Cria um ReplicaSet.
5. Cria novos Pods.
6. Escolhe os Nodes.
7. Cria os containers.
8. Atualiza as regras de rede.

Tudo isso acontece automaticamente.

---

# Boas práticas

- Nunca altere diretamente objetos nos Worker Nodes.
- Sempre utilize o API Server.
- Nunca acesse o etcd para modificar recursos.
- Entenda que os Workers apenas executam decisões.
- Trate o cluster como um sistema distribuído.

---

# Erros comuns

> [!WARNING]
> Acreditar que o Scheduler cria Pods.

O Scheduler apenas escolhe o Node.

---

> [!WARNING]
> Acreditar que o kubelet toma decisões.

Quem toma decisões é o Control Plane.

---

> [!WARNING]
> Acreditar que o kubectl conversa diretamente com os Nodes.

Toda comunicação passa pelo API Server.

---

# Resumo

Neste capítulo aprendemos que:

- Um Cluster é composto por um Control Plane e um ou mais Worker Nodes.
- O Kubernetes trabalha de forma declarativa.
- O Reconciliation Loop mantém continuamente o estado desejado.
- O API Server é o ponto central de comunicação.
- O etcd armazena o estado do cluster.
- O Scheduler escolhe onde os Pods serão executados.
- O Controller Manager garante que o estado desejado seja mantido.
- O kubelet executa Pods nos Workers.
- O container runtime executa os containers.

---

# Checklist

Antes de continuar, verifique se você consegue responder:

- [ ] O que é um Cluster Kubernetes?
- [ ] Qual a diferença entre Control Plane e Worker Node?
- [ ] O que é o Reconciliation Loop?
- [ ] Por que o Kubernetes é considerado declarativo?
- [ ] Quem escolhe o Node de um Pod?
- [ ] Qual é a função do kubelet?
- [ ] O que acontece após executar `kubectl apply`?
