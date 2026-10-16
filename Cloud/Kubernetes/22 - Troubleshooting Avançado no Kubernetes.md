---
tags:
  - kubernetes
  - troubleshooting
  - sre
  - debugging
  - observability
  - production
---
> [!INFO]
> Objetivo
>
> Desenvolver uma metodologia sistemática para diagnosticar problemas em clusters Kubernetes, desde a camada de infraestrutura até a aplicação.

---

# Objetivos de aprendizagem

Ao concluir este capítulo você deverá ser capaz de:

- [ ] Diagnosticar problemas em Pods.
- [ ] Resolver falhas de Deployments.
- [ ] Investigar problemas de rede.
- [ ] Identificar erros de Storage.
- [ ] Diagnosticar problemas no Scheduler.
- [ ] Utilizar corretamente logs, eventos e métricas.
- [ ] Criar uma metodologia consistente de troubleshooting.

---

# Pré-requisitos

- Todos os capítulos anteriores.

---

# A Regra Mais Importante

Nunca comece pelo Pod.

Comece pelo usuário.

---

# Modelo Mental

Sempre investigue nesta ordem.

```text
Usuário

↓

DNS

↓

Load Balancer

↓

Ingress

↓

Service

↓

EndpointSlice

↓

Pod

↓

Container

↓

Aplicação
```

Se todas essas camadas estiverem corretas.

Continue.

```text
Node

↓

Kubelet

↓

Container Runtime

↓

Control Plane
```

---

# A Pirâmide de Troubleshooting

```text
Aplicação

↓

Container

↓

Pod

↓

Service

↓

Ingress

↓

Networking

↓

Node

↓

Cluster
```

Nunca pule etapas.

---

# Passo 1 — O problema realmente existe?

Antes de qualquer comando.

Pergunte.

- Qual serviço está afetado?
- Desde quando?
- Apenas produção?
- Apenas um usuário?
- Todos os usuários?
- Alguma mudança recente?

---

# Passo 2 — O Deployment existe?

```bash
kubectl get deploy
```

Verifique.

- READY
- AVAILABLE
- UP-TO-DATE

---

# Passo 3 — Os Pods existem?

```bash
kubectl get pods
```

Estados comuns.

```text
Running

Pending

CrashLoopBackOff

ImagePullBackOff

Completed

Terminating
```

Cada estado indica um tipo diferente de problema.

---

# Running

O Pod está executando.

Isso NÃO significa que a aplicação está funcionando.

Ainda precisamos verificar.

- Readiness
- Logs
- Service

---

# Pending

O Scheduler ainda não conseguiu posicionar o Pod.

Possíveis causas.

- CPU insuficiente
- Memória insuficiente
- PVC pendente
- Node Selector
- Taints
- Affinity

---

# CrashLoopBackOff

O container inicia.

Depois encerra.

Depois reinicia.

Em loop.

Causas comuns.

- Erro da aplicação
- Configuração incorreta
- Secret ausente
- Banco indisponível
- Porta errada

---

# ImagePullBackOff

O Kubernetes não conseguiu baixar a imagem.

Verifique.

- Nome da imagem
- Tag
- Registry
- Credenciais
- Conectividade

---

# Describe é seu melhor amigo

```bash
kubectl describe pod api
```

Leia principalmente.

```text
Events
```

Muitas vezes a resposta está ali.

---

# Logs

Primeiro comando.

```bash
kubectl logs pod
```

Container específico.

```bash
kubectl logs pod -c api
```

Container anterior.

```bash
kubectl logs pod --previous
```

Muito útil para CrashLoopBackOff.

---

# Eventos

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```

Eventos costumam explicar:

- FailedScheduling
- FailedMount
- Pull Errors
- Probe Failures

---

# Readiness Probe

Aplicação inicia.

Mas não recebe tráfego.

Verifique.

```bash
kubectl describe pod
```

Procure.

```text
Readiness Probe Failed
```

---

# Liveness Probe

Aplicação reiniciando constantemente?

Verifique.

```text
Liveness Probe Failed
```

---

# Startup Probe

Aplicações Java ou .NET.

Podem demorar.

Sem Startup Probe.

O Kubernetes pode matar o processo antes da inicialização.

---

# Service

Pod funcionando.

Mas aplicação inacessível.

Verifique.

```bash
kubectl get svc
```

Depois.

```bash
kubectl describe svc
```

---

# EndpointSlice

Service sem endpoints.

Normalmente significa.

Selector incorreto.

```bash
kubectl get endpointslices
```

---

# Labels

Confira.

```bash
kubectl get pods --show-labels
```

Compare com.

```yaml
selector:
```

---

# DNS

Teste.

```bash
kubectl exec -it pod -- nslookup api
```

Ou.

```bash
kubectl exec -it pod -- dig api
```

---

# Conectividade

Teste.

```bash
kubectl exec -it pod -- curl http://api
```

---

# Ingress

Confira.

```bash
kubectl get ingress
```

Depois.

```bash
kubectl describe ingress
```

---

# TLS

Verifique certificados.

```bash
kubectl get certificate
```

Se utilizar cert-manager.

---

# NetworkPolicy

Pod existe.

Service existe.

Mas conexão falha.

Confira.

```bash
kubectl get networkpolicy
```

---

# Storage

PVC.

```bash
kubectl get pvc
```

PV.

```bash
kubectl get pv
```

---

# PVC Pending

Possíveis causas.

- StorageClass inexistente
- CSI Driver
- Disco indisponível

---

# Node

Verifique.

```bash
kubectl get nodes
```

Depois.

```bash
kubectl describe node
```

---

# Node NotReady

Possíveis causas.

- kubelet
- Disco cheio
- Rede
- Certificados
- Runtime

---

# Recursos

Veja consumo.

```bash
kubectl top nodes
```

```bash
kubectl top pods
```

---

# OOMKilled

Container morto por falta de memória.

Verifique.

```text
Exit Code 137
```

Depois ajuste.

```yaml
resources:
```

---

# Evicted

Pod removido pelo Node.

Normalmente.

- Memória
- Disco

---

# API Server

Verifique.

```bash
kubectl cluster-info
```

---

# Scheduler

Pods presos em Pending.

Veja eventos.

```text
FailedScheduling
```

---

# etcd

Problemas comuns.

- Alta latência
- Banco muito grande
- Disco lento

---

# CoreDNS

Teste.

```bash
kubectl logs -n kube-system deployment/coredns
```

---

# kube-proxy

Confira.

```bash
kubectl get pods -n kube-system
```

---

# CNI

Verifique Pods do plugin.

Exemplo.

```bash
kubectl get pods -n kube-system
```

---

# Container Runtime

No Node.

```bash
crictl ps
```

Ou.

```bash
ctr containers ls
```

---

# Fluxo de diagnóstico

```text
Usuário

↓

Ingress

↓

Service

↓

EndpointSlice

↓

Pod

↓

Logs

↓

Eventos

↓

Node

↓

Control Plane
```

---

# Ferramentas úteis

| Ferramenta | Objetivo |
|------------|----------|
| kubectl logs | Logs |
| kubectl describe | Diagnóstico |
| kubectl get events | Eventos |
| kubectl top | Recursos |
| kubectl exec | Testes |
| kubectl port-forward | Acesso local |
| kubectl debug | Container temporário |
| stern | Logs de múltiplos Pods |
| k9s | Interface interativa |
| kubectx | Alternar clusters |
| kubens | Alternar Namespaces |

---

# Comandos indispensáveis

```bash
kubectl get all
```

---

```bash
kubectl describe pod
```

---

```bash
kubectl logs
```

---

```bash
kubectl get events
```

---

```bash
kubectl top pods
```

---

```bash
kubectl exec
```

---

# Checklist de produção

Sempre responda.

- O Pod existe?
- Está Running?
- Está Ready?
- O Service possui endpoints?
- O DNS resolve?
- O Ingress aponta corretamente?
- Há NetworkPolicies?
- O Node está saudável?
- Há recursos suficientes?
- O problema é da aplicação?

---

# Fluxograma Mental

```text
Aplicação fora do ar

↓

Pod existe?

↓

Não

↓

Deployment

↓

Sim

↓

Running?

↓

Não

↓

Describe

↓

Eventos

↓

Sim

↓

Logs

↓

Readiness

↓

Service

↓

Endpoint

↓

Ingress

↓

DNS

↓

Usuário
```

---

# Boas práticas

- Nunca faça troubleshooting por tentativa e erro.
- Leia eventos antes de alterar configurações.
- Utilize observabilidade para validar hipóteses.
- Faça mudanças pequenas e reversíveis.
- Documente incidentes recorrentes.
- Automatize verificações sempre que possível.

---

# Erros comuns

> [!WARNING]

Executar comandos aleatórios sem hipótese.

---

> [!WARNING]

Ignorar os eventos (`kubectl get events`).

---

> [!WARNING]

Assumir que um Pod `Running` significa aplicação saudável.

---

> [!WARNING]

Alterar recursos em produção antes de identificar a causa raiz.

---

# Resumo

Neste capítulo aprendemos que:

- Troubleshooting eficiente segue uma metodologia.
- O diagnóstico deve partir da borda do sistema em direção ao cluster.
- `kubectl describe`, `logs` e `events` são as principais ferramentas de investigação.
- Estados como `Pending`, `CrashLoopBackOff` e `ImagePullBackOff` possuem causas típicas.
- Problemas de rede, armazenamento, scheduler e infraestrutura exigem abordagens específicas.
- Observabilidade é essencial para validar hipóteses e reduzir o tempo de resolução.

---

# Checklist

Antes de continuar, verifique se você consegue responder:

- [ ] Qual a ordem correta para investigar uma aplicação indisponível?
- [ ] Como diferenciar `CrashLoopBackOff` de `ImagePullBackOff`?
- [ ] Quando utilizar `kubectl describe` em vez de `kubectl logs`?
- [ ] Como verificar se um Service possui endpoints?
- [ ] Como identificar problemas de DNS?
- [ ] Quais são as causas mais comuns de um Pod `Pending`?
- [ ] Como diagnosticar `OOMKilled`?
- [ ] Como verificar se uma NetworkPolicy está bloqueando tráfego?
- [ ] Como investigar problemas relacionados ao Scheduler?
- [ ] Qual é a importância de formular hipóteses antes de executar comandos?

---

# Diagrama consolidado

```text
                 Usuário
                     │
                  DNS/LB
                     │
                  Ingress
                     │
                  Service
                     │
               EndpointSlice
                     │
                    Pod
                     │
                Container
                     │
                Aplicação
                     │
────────────────────────────────────
                     │
                  Node
                     │
                 kubelet
                     │
           Container Runtime
                     │
              Control Plane
                     │
                    etcd
```
