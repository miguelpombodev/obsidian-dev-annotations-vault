---
tags:
  - kubernetes
  - observability
  - prometheus
  - grafana
  - loki
  - fluentbit
  - opentelemetry
  - jaeger
  - metrics
  - logs
  - tracing
---
> [!INFO]
> Objetivo
>
> Compreender como monitorar aplicações e o próprio cluster Kubernetes utilizando métricas, logs, tracing distribuído e estado do cluster, formando uma estratégia completa de observabilidade.

---

# Objetivos de aprendizagem

Ao concluir este capítulo você deverá ser capaz de:

- [ ] Entender os quatro pilares da observabilidade.
- [ ] Compreender o funcionamento do Prometheus.
- [ ] Entender Grafana.
- [ ] Compreender coleta de logs.
- [ ] Entender tracing distribuído.
- [ ] Monitorar componentes internos do Kubernetes.
- [ ] Construir um modelo de troubleshooting baseado em observabilidade.

---

# Pré-requisitos

- [[12 - DaemonSets]]
- [[15 - Networking]]
- [[16 - Segurança]]

---

# O problema que este capítulo resolve

Imagine.

Usuário informa.

```text
A API está lenta.
```

Você verifica.

O Deployment está saudável.

```bash
kubectl get deploy
```

Tudo certo.

Os Pods.

```bash
kubectl get pods
```

Também.

Então.

Onde está o problema?

Sem observabilidade.

Você apenas tenta adivinhar.

---

# Modelo Mental

Imagine um hospital.

Para diagnosticar um paciente.

O médico utiliza.

- temperatura;
- pressão;
- exames;
- raio-x;
- histórico.

Nenhum exame sozinho responde tudo.

Observabilidade funciona exatamente da mesma forma.

---

# Os quatro pilares

Neste guia utilizaremos.

```text
Métricas

↓

Logs

↓

Tracing

↓

Estado do Cluster
```

Todos trabalham juntos.

---

# Pilar 1 — Métricas

Métricas respondem.

> O que está acontecendo?

Exemplos.

```text
CPU

Memória

Latência

Requests

Erros

Disco

Rede
```

São dados numéricos ao longo do tempo.

---

# Prometheus

É o padrão de mercado.

Praticamente todo cluster Kubernetes utiliza.

---

# Como funciona?

Ao contrário de muitos sistemas.

Prometheus não recebe métricas.

Ele busca.

Modelo Pull.

```text
Prometheus

↓

HTTP GET

↓

/metrics

↓

Aplicação
```

---

# Exposição de métricas

Aplicações normalmente disponibilizam.

```text
http://app:8080/metrics
```

Formato.

```text
http_requests_total

cpu_usage

memory_usage
```

---

# ServiceMonitor

Quando utilizamos Prometheus Operator.

Criamos.

```text
ServiceMonitor
```

Ele informa ao Prometheus.

```text
Colete métricas deste Service.
```

---

# Principais Exporters

Prometheus coleta métricas através de Exporters.

Exemplos.

- Node Exporter
- kube-state-metrics
- cAdvisor
- PostgreSQL Exporter
- Redis Exporter
- NGINX Exporter

---

# Node Exporter

Executa como.

```text
DaemonSet
```

Cada Node possui um.

Coleta.

- CPU
- Memória
- Disco
- Rede
- Sistema Operacional

---

# kube-state-metrics

Muito importante.

Ele não coleta CPU.

Nem memória.

Ele consulta a API do Kubernetes.

E gera métricas como.

```text
Quantidade de Pods

Deployments

Namespaces

ReplicaSets

PVCs
```

---

# cAdvisor

Já vem integrado ao Kubelet.

Responsável por métricas dos containers.

---

# Pilar 2 — Logs

Métricas dizem.

```text
Algo está errado.
```

Logs respondem.

```text
O que aconteceu?
```

---

# Modelo Mental

Imagine.

Métricas.

```text
Febre.
```

Logs.

```text
Exame laboratorial.
```

---

# Logs no Kubernetes

Boa prática.

Aplicações escrevem.

```text
stdout

stderr
```

Nunca diretamente em arquivos.

---

# Coleta

Quem coleta?

Normalmente.

```text
Fluent Bit
```

Executando como.

```text
DaemonSet
```

Ele lê.

```text
/var/log/containers
```

Depois envia.

- Loki
- Elasticsearch
- Splunk
- Datadog

---

# Loki

Muito utilizado com Grafana.

Fluxo.

```text
Pod

↓

stdout

↓

Fluent Bit

↓

Loki

↓

Grafana
```

---

# Elasticsearch

Outra arquitetura bastante comum.

```text
Pod

↓

Fluent Bit

↓

Elasticsearch

↓

Kibana
```

---

# Pilar 3 — Tracing

Logs respondem.

```text
O que aconteceu?
```

Tracing responde.

```text
Onde aconteceu?
```

---

Imagine.

Frontend.

↓

API.

↓

Auth.

↓

Redis.

↓

PostgreSQL.

Qual serviço ficou lento?

Tracing responde.

---

# Span

Cada operação gera.

```text
Span
```

---

# Trace

Conjunto de Spans.

```text
Browser

↓

API

↓

Redis

↓

Banco
```

---

# OpenTelemetry

Hoje é o padrão da indústria.

Substituiu diversas implementações proprietárias.

---

# Componentes

```text
SDK

↓

Collector

↓

Backend
```

---

# Collector

Recebe dados.

Processa.

Exporta.

Muito utilizado em Kubernetes.

---

# Backends

Exemplos.

- Jaeger
- Grafana Tempo
- Zipkin

---

# Pilar 4 — Estado do Cluster

Este é o quarto pilar.

Específico do Kubernetes.

---

Imagine.

Deployment.

```text
Desired

5
```

Atual.

```text
3
```

CPU está baixa.

Logs limpos.

Tracing normal.

Mesmo assim.

Existe um problema.

O estado do cluster está inconsistente.

---

# kube-state-metrics

É justamente ele que fornece essas informações.

---

# Componentes do Cluster

Também precisamos monitorar.

- API Server
- Scheduler
- Controller Manager
- etcd
- CoreDNS
- kube-proxy
- CNI

---

# API Server

Monitorar.

- latência;
- requisições;
- erros;
- autenticação.

---

# etcd

Muito importante.

Monitorar.

- latência;
- tamanho;
- IOPS;
- compactação.

---

# CoreDNS

Monitorar.

- consultas DNS;
- falhas;
- tempo de resposta.

---

# Scheduler

Monitorar.

- tempo de agendamento;
- fila;
- Pods pendentes.

---

# Grafana

Grafana não coleta métricas.

Ele apenas consulta fontes de dados.

Pode visualizar.

- Prometheus
- Loki
- Tempo
- Elasticsearch

---

# Dashboard

Exemplo.

```text
CPU

Memória

Pods

Requests

Latência

Erros
```

Tudo em um único painel.

---

# Alertmanager

Prometheus detecta.

```text
CPU > 90%
```

Alertmanager envia.

- Slack
- Teams
- E-mail
- PagerDuty

---

# Golden Signals

Google SRE define quatro métricas fundamentais.

- Latência
- Tráfego
- Erros
- Saturação

Todo sistema deveria monitorá-las.

---

# RED Method

Muito utilizada para APIs.

- Rate
- Errors
- Duration

---

# USE Method

Muito utilizada para infraestrutura.

- Utilization
- Saturation
- Errors

---

# Hands-on

Verificar métricas.

```bash
kubectl top nodes
```

---

```bash
kubectl top pods
```

---

Logs.

```bash
kubectl logs pod-api
```

---

Eventos.

```bash
kubectl get events
```

---

Estado.

```bash
kubectl get deployments
```

---

Dashboard.

```bash
kubectl port-forward svc/grafana 3000:80
```

---

# Stack de observabilidade moderna

Uma stack extremamente comum.

```text
Prometheus

Grafana

Loki

Fluent Bit

OpenTelemetry Collector

Tempo
```

---

# Arquitetura completa

```text
Aplicação

↓

stdout

↓

Fluent Bit

↓

Loki

↓

Grafana
```

---

```text
Aplicação

↓

/metrics

↓

Prometheus

↓

Grafana
```

---

```text
Aplicação

↓

OpenTelemetry SDK

↓

Collector

↓

Tempo

↓

Grafana
```

---

# Troubleshooting

Imagine.

Usuário.

```text
API lenta
```

Fluxo recomendado.

```text
Grafana

↓

CPU?

↓

Não

↓

Latência?

↓

Sim

↓

Tracing

↓

Banco lento

↓

Logs

↓

Timeout SQL
```

Sem observabilidade.

Esse processo poderia levar horas.

---

# Comparando as ferramentas

| Ferramenta | Função |
|------------|--------|
| Prometheus | Métricas |
| Grafana | Dashboards |
| Fluent Bit | Coleta de Logs |
| Loki | Armazena Logs |
| OpenTelemetry | Instrumentação |
| Tempo | Tracing |
| Alertmanager | Alertas |
| kube-state-metrics | Estado do Cluster |

---

# Relação com os capítulos anteriores

Agora nossa arquitetura está completa.

```text
Internet
      │
Ingress
      │
Service
      │
Pods
      │
stdout ───────► Fluent Bit ─────► Loki
      │
/metrics ─────► Prometheus ─────► Grafana
      │
Tracing ──────► OTel Collector ─► Tempo
      │
API Server ───► kube-state-metrics
```

---

# Boas práticas

- Instrumente aplicações desde o início.
- Utilize métricas, logs e tracing em conjunto.
- Monitore também os componentes do Kubernetes.
- Centralize logs.
- Configure alertas para eventos críticos.
- Utilize dashboards padronizados para toda a organização.

---

# Erros comuns

> [!WARNING]

Confiar apenas em logs.

Logs não substituem métricas nem tracing.

---

> [!WARNING]

Monitorar apenas aplicações.

O cluster também precisa ser monitorado.

---

> [!WARNING]

Não configurar retenção para métricas e logs.

Isso pode consumir rapidamente armazenamento.

---

> [!WARNING]

Criar dashboards sem objetivos claros.

Dashboards devem responder perguntas operacionais.

---

# Resumo

Neste capítulo aprendemos que:

- Observabilidade moderna é composta por quatro pilares.
- Prometheus coleta métricas.
- Grafana visualiza dados.
- Fluent Bit coleta logs.
- Loki e Elasticsearch armazenam logs.
- OpenTelemetry é o padrão para tracing distribuído.
- kube-state-metrics monitora o estado do cluster.
- Alertmanager envia notificações baseadas em regras.

---

# Checklist

Antes de continuar, verifique se você consegue responder:

- [ ] Quais são os quatro pilares da observabilidade?
- [ ] Como o Prometheus coleta métricas?
- [ ] Qual a diferença entre Node Exporter e kube-state-metrics?
- [ ] Por que aplicações devem escrever logs em stdout?
- [ ] O que é um Trace?
- [ ] Qual o papel do OpenTelemetry Collector?
- [ ] O que faz o Grafana?
- [ ] Para que serve o Alertmanager?
- [ ] O que são os Golden Signals?
- [ ] Qual a diferença entre RED e USE?

---

# Diagrama consolidado

```text
                      Internet
                          │
                       Ingress
                          │
                       Service
                          │
                         Pods
           ┌──────────────┼──────────────┐
           │              │              │
      stdout         /metrics       OpenTelemetry
           │              │              │
      Fluent Bit     Prometheus     OTel Collector
           │              │              │
         Loki        Alertmanager      Tempo
           └──────────────┼──────────────┘
                          ▼
                       Grafana

               kube-state-metrics
                       │
                 API Server
                       │
                  Estado do Cluster
```
