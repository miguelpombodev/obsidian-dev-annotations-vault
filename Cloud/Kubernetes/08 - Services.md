---
tags:
  - kubernetes
  - networking
  - services
  - clusterip
  - kube-proxy
  - coredns
---
> [!INFO] Objetivo
> Entender como aplicações se comunicam dentro de um cluster Kubernetes, compreender por que Pods não devem ser acessados diretamente e como os Services fornecem descoberta, balanceamento e estabilidade para aplicações distribuídas.

---

# Objetivos de aprendizagem

Ao concluir este capítulo você deverá ser capaz de:

- [ ] Explicar por que Pods não devem ser acessados diretamente.
- [ ] Entender o que é um Service.
- [ ] Compreender como o kube-proxy implementa um Service.
- [ ] Explicar o funcionamento do ClusterIP.
- [ ] Diferenciar ClusterIP, NodePort, LoadBalancer e ExternalName.
- [ ] Entender como o CoreDNS participa da comunicação.
- [ ] Rastrear uma requisição desde o cliente até um container.

---

# Pré-requisitos

- [[04 - Pods]]
- [[05 - Labels, Selectors e Annotations]]
- [[06 - ReplicaSets]]
- [[07 - Deployments]]

---

# O problema que este capítulo resolve

Imagine um Deployment.

```text
Deployment

↓

ReplicaSet

↓

3 Pods
```

Cada Pod recebe um IP.

```text
Pod A

10.244.1.20
```

```text
Pod B

10.244.2.13
```

```text
Pod C

10.244.3.8
```

Tudo funciona.

Até que ocorre um Rolling Update.

Agora temos:

```text
Pod A

10.244.8.15
```

```text
Pod B

10.244.9.7
```

```text
Pod C

10.244.4.90
```

Todos os IPs mudaram.

Como outra aplicação encontraria essa API?

A resposta é:

**Ela nunca conversa com Pods.**

Ela conversa com um **Service**.

---

# Modelo Mental

Pense em um Service como o recepcionista de um escritório.

O cliente nunca sabe em qual sala o funcionário está.

Ele conversa com a recepção.

A recepção encaminha a ligação.

Service faz exatamente isso.

---

# O que é um Service?

Um Service é um objeto da API responsável por fornecer um ponto de acesso estável para um conjunto de Pods.

Ele resolve três problemas:

- descoberta de serviços;
- balanceamento de carga;
- abstração dos Pods.

---

# Arquitetura

```text
Cliente

↓

Service

↓

Selector

↓

Pods

↓

Containers
```

Perceba.

O cliente nunca conversa diretamente com os Pods.

---

# Como um Service encontra os Pods?

Exatamente da mesma forma que vimos nos capítulos anteriores.

Utilizando Labels.

Pods.

```yaml
labels:

  app: backend
```

Service.

```yaml
selector:

  app: backend
```

Resultado.

```text
Service

↓

Pod 1

Pod 2

Pod 3
```

---

# Anatomia de um Service

Exemplo.

```yaml
apiVersion: v1

kind: Service

metadata:

  name: backend

spec:

  selector:

    app: backend

  ports:

    - port: 80

      targetPort: 8080

  type: ClusterIP
```

---

# Explicando o YAML

## selector

Indica quais Pods pertencem ao Service.

---

## port

Porta exposta pelo Service.

```yaml
port: 80
```

---

## targetPort

Porta do container.

```yaml
targetPort: 8080
```

Fluxo.

```text
Cliente

↓

80

↓

Service

↓

8080

↓

Container
```

---

# Internals

Quando um Service é criado.

Ele **não cria containers**.

Ele **não cria Pods**.

Ele apenas registra uma regra.

Fluxo.

```text
Service

↓

API Server

↓

Endpoints

↓

kube-proxy

↓

iptables/IPVS

↓

Pods
```

---

# O papel do kube-proxy

O kube-proxy roda em todos os Worker Nodes.

Sua responsabilidade é implementar a lógica de encaminhamento.

Quando criamos um Service.

O kube-proxy instala regras no Linux.

Normalmente utilizando.

- iptables

ou

- IPVS

Essas regras encaminham as conexões para um dos Pods.

---

# Balanceamento de carga

Imagine três Pods.

```text
Pod A
```

```text
Pod B
```

```text
Pod C
```

Chega uma requisição.

```text
Cliente

↓

Service
```

O kube-proxy escolhe um Pod.

```text
↓

Pod B
```

Próxima requisição.

```text
↓

Pod A
```

Depois.

```text
↓

Pod C
```

Esse balanceamento ocorre automaticamente.

---

# Endpoints

Quando um Service é criado.

O Kubernetes cria automaticamente um objeto chamado **Endpoints** (ou **EndpointSlice** nas versões mais recentes).

Ele contém a lista de IPs dos Pods.

Exemplo.

```text
Service

↓

EndpointSlice

↓

10.244.1.5

10.244.1.8

10.244.2.3
```

Sempre que Pods entram ou saem.

O EndpointSlice é atualizado.

---

# O papel do CoreDNS

Até agora sabemos como um Service encontra Pods.

Mas...

Como uma aplicação encontra o Service?

Utilizando DNS.

Exemplo.

```text
backend.default.svc.cluster.local
```

Esse nome é resolvido pelo CoreDNS.

Fluxo.

```text
Aplicação

↓

DNS

↓

CoreDNS

↓

ClusterIP

↓

Service

↓

Pod
```

Na prática.

Dentro do mesmo Namespace.

Basta utilizar.

```text
http://backend
```

---

# Fluxo completo de uma requisição

Imagine uma API chamando outra API.

```text
API Cliente

↓

DNS

↓

CoreDNS

↓

ClusterIP

↓

Service

↓

EndpointSlice

↓

kube-proxy

↓

Pod

↓

Container
```

Esse fluxo acontece em poucos milissegundos.

---

# Tipos de Service

Existem quatro principais.

---

# ClusterIP

É o padrão.

O Service fica acessível apenas dentro do cluster.

```text
Cluster

↓

ClusterIP

↓

Pods
```

É o tipo mais utilizado.

---

# NodePort

Além do ClusterIP.

Abre uma porta em todos os Nodes.

```text
Cliente

↓

NodeIP:30080

↓

Service

↓

Pods
```

Muito utilizado em laboratórios.

Pouco utilizado em produção.

---

# LoadBalancer

Solicita um Load Balancer ao provedor de cloud.

Exemplo.

```text
Internet

↓

Load Balancer

↓

Service

↓

Pods
```

Na AWS.

Normalmente cria um ELB/NLB.

Na Azure.

Um Azure Load Balancer.

Na GCP.

Cloud Load Balancer.

---

# ExternalName

Não encaminha tráfego.

Ele apenas cria um alias DNS.

Exemplo.

```yaml
externalName:

  api.externa.com
```

Quando alguém consulta.

```text
backend
```

Recebe.

```text
api.externa.com
```

---

# Hands-on

Deployment.

```bash
kubectl apply -f deployment.yaml
```

Criar Service.

```bash
kubectl apply -f service.yaml
```

Listar.

```bash
kubectl get svc
```

Ver detalhes.

```bash
kubectl describe svc backend
```

Ver EndpointSlices.

```bash
kubectl get endpointslices
```

Entrar em outro Pod.

```bash
kubectl exec -it busybox -- sh
```

Testar.

```bash
wget http://backend
```

Ou.

```bash
curl http://backend
```

---

# Como isso aparece na prática

Imagine um sistema composto por microsserviços.

```text
Frontend

↓

API

↓

Auth

↓

Payments

↓

Orders

↓

Database
```

Nenhum deles conhece IPs.

Todos utilizam Services.

Por exemplo.

```text
http://orders
```

```text
http://payments
```

```text
http://auth
```

Os Pods podem ser recriados centenas de vezes.

Nada muda para quem consome o serviço.

---

# Relação com os capítulos anteriores

Agora nossa arquitetura evolui para:

```text
Deployment
      │
      ▼
ReplicaSet
      │
Selector
      │
      ▼
Labels
      │
      ▼
Pods
      ▲
      │
Service
      ▲
      │
ClusterIP
      ▲
      │
CoreDNS
      ▲
      │
Cliente
```

---

# Boas práticas

- Nunca acesse Pods diretamente.
- Sempre utilize Services.
- Utilize ClusterIP para comunicação interna.
- Prefira Ingress ou Gateway API para exposição HTTP externa.
- Evite NodePort em ambientes de produção.

---

# Erros comuns

> [!WARNING]

Criar um Service sem Labels compatíveis.

O Service existirá.

Mas não encaminhará tráfego para nenhum Pod.

---

> [!WARNING]

Acessar Pods pelo IP.

Os IPs mudam constantemente.

---

> [!WARNING]

Expor aplicações utilizando NodePort em produção.

Normalmente utiliza-se LoadBalancer + Ingress.

---

# Resumo

Neste capítulo aprendemos que:

- Services fornecem um ponto de acesso estável para Pods.
- Services utilizam Selectors para localizar Pods.
- O kube-proxy implementa o balanceamento de carga.
- O CoreDNS resolve nomes de Services.
- O ClusterIP é o tipo padrão.
- NodePort e LoadBalancer expõem aplicações para fora do cluster.
- ExternalName cria aliases DNS.

---

# Checklist

Antes de continuar, verifique se você consegue responder:

- [ ] Por que não devemos acessar Pods diretamente?
- [ ] Como um Service encontra seus Pods?
- [ ] Qual o papel do kube-proxy?
- [ ] O que é um EndpointSlice?
- [ ] Qual a função do CoreDNS?
- [ ] Quando utilizar ClusterIP, NodePort e LoadBalancer?

---

# Diagrama consolidado

Até aqui, nosso modelo mental é:

```text
                 Cliente
                    │
               DNS (CoreDNS)
                    │
                    ▼
               ClusterIP (Service)
                    │
              EndpointSlice
                    │
             kube-proxy (iptables/IPVS)
                    │
                    ▼
              Pods (Labels)
                    ▲
                    │
              ReplicaSet
                    ▲
                    │
              Deployment
```
