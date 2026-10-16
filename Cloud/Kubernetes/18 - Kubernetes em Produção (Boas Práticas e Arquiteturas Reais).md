---
tags:
  - kubernetes
  - production
  - gitops
  - helm
  - kustomize
  - argocd
  - autoscaling
  - sre
  - platform-engineering
---
> [!INFO]
> Objetivo
>
> Entender como clusters Kubernetes são operados em ambientes reais de produção, conhecendo padrões arquiteturais, GitOps, gerenciamento de configurações, estratégias de deploy, escalabilidade, backup e boas práticas utilizadas por grandes empresas.

---

# Objetivos de aprendizagem

Ao concluir este capítulo você deverá ser capaz de:

- [ ] Entender como empresas utilizam Kubernetes em produção.
- [ ] Explicar GitOps.
- [ ] Compreender Helm e Kustomize.
- [ ] Conhecer ArgoCD.
- [ ] Entender estratégias de Deploy.
- [ ] Configurar Autoscaling.
- [ ] Compreender Backup e Disaster Recovery.
- [ ] Conhecer práticas de Platform Engineering.

---

# Pré-requisitos

- Todos os capítulos anteriores.

---

# Mudança de Mentalidade

Até agora aprendemos Kubernetes.

Agora aprenderemos **Operação de Plataformas Kubernetes**.

A pergunta deixa de ser:

> Como criar um Deployment?

E passa a ser:

> Como administrar centenas de Deployments em dezenas de clusters?

---

# O ciclo de vida de uma aplicação em produção

Uma aplicação moderna normalmente percorre este fluxo.

```text
Developer

↓

Git

↓

CI

↓

Imagem Docker

↓

Registry

↓

GitOps

↓

Cluster Kubernetes

↓

Observabilidade

↓

Usuário
```

Observe.

O desenvolvedor normalmente nunca executa.

```bash
kubectl apply
```

---

# GitOps

GitOps é um dos conceitos mais importantes do Kubernetes moderno.

Sua premissa é simples.

> O Git é a fonte única da verdade (Single Source of Truth).

Toda alteração acontece através do Git.

---

# Fluxo GitOps

```text
Developer

↓

Git Push

↓

CI

↓

Atualiza Manifestos

↓

ArgoCD

↓

Cluster
```

Não existe acesso manual ao cluster.

---

# Benefícios do GitOps

- Auditoria completa.
- Versionamento.
- Rollback simples.
- Padronização.
- Reprodutibilidade.
- Menor risco operacional.

---

# Argo CD

O Argo CD é um controlador GitOps.

Ele monitora um repositório Git.

Sempre que detecta mudanças.

Sincroniza automaticamente com o cluster.

```text
Git Repository

↓

ArgoCD

↓

Kubernetes API
```

---

# Drift Detection

Imagine.

Alguém executa.

```bash
kubectl edit deployment api
```

Agora.

O cluster ficou diferente do Git.

Isso é chamado de.

```text
Configuration Drift
```

O ArgoCD detecta isso.

Pode inclusive corrigir automaticamente.

---

# Helm

Imagine que você precisa instalar.

- Prometheus
- Grafana
- Loki
- PostgreSQL

Você escreveria centenas de arquivos YAML?

Não.

Utilizamos Helm.

---

# O que é Helm?

Helm é o gerenciador de pacotes do Kubernetes.

Assim como.

```text
apt

yum

npm

pip
```

Helm instala aplicações Kubernetes.

---

# Helm Chart

Um Chart contém.

```text
Templates

↓

Values.yaml

↓

Manifestos Kubernetes
```

---

# Vantagens

Sem Helm.

Você mantém dezenas de YAMLs.

Com Helm.

Basta alterar.

```yaml
replicas: 5

image:

  tag: 2.0.0
```

---

# Kustomize

Outra ferramenta extremamente importante.

Ela não gera templates.

Ela aplica modificações.

---

Imagine.

Base.

```text
Deployment

Service

Ingress
```

Depois.

Produção.

```text
Replicas = 10
```

Homologação.

```text
Replicas = 2
```

Sem duplicar arquivos.

---

# Helm vs Kustomize

| Helm | Kustomize |
|--------|-----------|
| Templates | Overlays |
| values.yaml | patches |
| Mais flexível | Mais simples |
| Excelente para distribuição | Excelente para customização |

Na prática.

É comum utilizar ambos.

---

# Estratégias de Deploy

Atualizar aplicações parece simples.

Mas.

Como evitar downtime?

---

## Rolling Update

Padrão do Kubernetes.

```text
v1

↓

v1 + v2

↓

v2
```

Nunca derruba todas as réplicas ao mesmo tempo.

---

## Blue/Green

Mantém dois ambientes.

```text
Blue

Green
```

Troca apenas o roteamento.

Rollback quase instantâneo.

---

## Canary

Envia.

```text
5%
```

dos usuários para a nova versão.

Depois.

20%.

50%.

100%.

Muito utilizado em sistemas críticos.

---

# Autoscaling

Existem três tipos.

---

## HPA

Horizontal Pod Autoscaler.

Escala Pods.

```text
3 Pods

↓

10 Pods
```

---

## VPA

Vertical Pod Autoscaler.

Aumenta recursos.

```text
500Mi

↓

2Gi
```

---

## Cluster Autoscaler

Escala Nodes.

```text
3 Nodes

↓

6 Nodes
```

Muito utilizado em Cloud.

---

# Requests e Limits

Sem eles.

O Scheduler não consegue tomar boas decisões.

```yaml
resources:

  requests:

    cpu: 250m

    memory: 512Mi

  limits:

    cpu: 1000m

    memory: 2Gi
```

Boa prática.

Sempre definir.

---

# Alta Disponibilidade

Produção exige redundância.

---

## Control Plane

Nunca possuir apenas um.

Arquitetura comum.

```text
3 Masters
```

---

## etcd

Sempre em número ímpar.

```text
3

ou

5
```

---

## Worker Nodes

Distribuir Pods entre Nodes.

Utilizando.

- Affinity
- Anti-Affinity
- Topology Spread Constraints

---

# Backup

Persistência é crítica.

Ferramenta padrão.

```text
Velero
```

Ele realiza backup de.

- Recursos Kubernetes.
- Volumes Persistentes.
- Metadados.

---

# Disaster Recovery

Planeje sempre.

- RPO (Recovery Point Objective).
- RTO (Recovery Time Objective).

Esses conceitos definem quanto dado você pode perder e quanto tempo sua aplicação pode ficar indisponível.

---

# Multi-tenancy

Imagine.

Mesmo cluster.

```text
Equipe A

Equipe B

Equipe C
```

Separação utilizando.

- Namespaces.
- RBAC.
- ResourceQuota.
- NetworkPolicies.

---

# ResourceQuota

Evita que uma equipe consuma todos os recursos.

Exemplo.

```text
CPU

Memória

Pods

PVCs
```

---

# LimitRange

Define valores padrão.

```text
Requests

Limits
```

Para novos Pods.

---

# Custos (FinOps)

Clusters mal configurados custam caro.

Boas práticas.

- Requests realistas.
- Autoscaling.
- Spot Instances.
- Escalonamento automático de Nodes.
- Monitoramento contínuo.

---

# Platform Engineering

Empresas modernas possuem equipes dedicadas à plataforma.

Objetivo.

Permitir que desenvolvedores façam deploy sem conhecer profundamente Kubernetes.

---

# Internal Developer Platform (IDP)

Fluxo moderno.

```text
Developer

↓

Git Push

↓

CI

↓

ArgoCD

↓

Cluster

↓

Observabilidade
```

Toda a complexidade da plataforma fica escondida.

---

# Ferramentas comuns

| Categoria | Ferramentas |
|------------|-------------|
| GitOps | ArgoCD, Flux |
| Pacotes | Helm |
| Customização | Kustomize |
| Observabilidade | Prometheus, Grafana, Loki |
| Backup | Velero |
| Autoscaling | HPA, VPA, Cluster Autoscaler |
| Segurança | Kyverno, Gatekeeper, Trivy |

---

# Hands-on

Instalar um Chart.

```bash
helm install prometheus prometheus-community/kube-prometheus-stack
```

---

Ver releases.

```bash
helm list
```

---

Atualizar.

```bash
helm upgrade
```

---

Rollback.

```bash
helm rollback
```

---

Ver HPA.

```bash
kubectl get hpa
```

---

Ver ResourceQuota.

```bash
kubectl get resourcequota
```

---

# Arquitetura moderna

```text
Git

↓

CI

↓

Docker Registry

↓

ArgoCD

↓

Kubernetes

↓

Prometheus

↓

Grafana

↓

Alertmanager
```

---

# Como grandes empresas trabalham

Normalmente.

- GitOps.
- Múltiplos clusters.
- Helm.
- ArgoCD.
- Observabilidade centralizada.
- Segurança automatizada.
- Políticas como código.
- Deploy automatizado.
- Rollback automático.
- Autoscaling.

Pouquíssimas alterações são realizadas manualmente no cluster.

---

# Relação com os capítulos anteriores

Agora temos a visão completa da plataforma.

```text
Developer
      │
      ▼
Git Repository
      │
      ▼
CI/CD
      │
      ▼
Docker Registry
      │
      ▼
ArgoCD (GitOps)
      │
      ▼
Kubernetes Cluster
      │
 ┌────┼─────────────────────┐
 │    │                     │
Pods Services         Observabilidade
 │    │                     │
 └────┴──────────────► Grafana
```

---

# Boas práticas

- Utilize GitOps para todas as mudanças.
- Nunca faça alterações manuais em produção.
- Utilize Helm para aplicações distribuídas.
- Utilize Kustomize para customizações por ambiente.
- Configure Requests e Limits.
- Utilize HPA e Cluster Autoscaler.
- Faça backups frequentes com Velero.
- Planeje Disaster Recovery.
- Automatize o máximo possível.

---

# Erros comuns

> [!WARNING]

Executar `kubectl apply` diretamente em produção.

Isso dificulta auditoria e pode gerar Configuration Drift.

---

> [!WARNING]

Não definir Requests e Limits.

Isso prejudica o Scheduler e pode causar instabilidade.

---

> [!WARNING]

Executar um único nó de Control Plane em produção.

Isso cria um ponto único de falha.

---

> [!WARNING]

Não testar procedimentos de restauração.

Backup sem teste de restore não garante recuperação.

---

# Resumo

Neste capítulo aprendemos que:

- GitOps transforma o Git na fonte única da verdade.
- ArgoCD sincroniza o estado desejado com o cluster.
- Helm gerencia pacotes Kubernetes.
- Kustomize permite customizações sem duplicação.
- Rolling Update, Blue/Green e Canary possuem objetivos distintos.
- HPA, VPA e Cluster Autoscaler atuam em camadas diferentes de escalabilidade.
- Velero é a principal ferramenta para backup e recuperação.
- Platform Engineering abstrai a complexidade do Kubernetes para os desenvolvedores.

---

# Checklist

Antes de continuar, verifique se você consegue responder:

- [ ] O que é GitOps?
- [ ] Qual o papel do ArgoCD?
- [ ] Quando utilizar Helm e Kustomize?
- [ ] Qual a diferença entre Rolling Update, Blue/Green e Canary?
- [ ] Quando utilizar HPA, VPA e Cluster Autoscaler?
- [ ] Por que Requests e Limits são importantes?
- [ ] Como funciona o Velero?
- [ ] O que é Configuration Drift?
- [ ] Como implementar Multi-tenancy?
- [ ] O que caracteriza uma plataforma Kubernetes pronta para produção?

---

# Diagrama consolidado

```text
                     Developer
                         │
                    Git Repository
                         │
                    CI / Build Image
                         │
                  Container Registry
                         │
                 GitOps (ArgoCD/Flux)
                         │
                  Kubernetes Cluster
                         │
      ┌──────────────────┼──────────────────┐
      │                  │                  │
   Helm/Kustomize    Autoscaling      Observabilidade
      │                  │                  │
      └──────────────────┼──────────────────┘
                         ▼
                    Aplicações
                         │
                    Usuários Finais
```
