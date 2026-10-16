---
tags:
  - kubernetes
  - jobs
  - cronjobs
  - batch
  - scheduler
---
> [!INFO] Objetivo
> Entender como o Kubernetes executa tarefas temporárias, compreender a diferença entre aplicações contínuas e workloads batch, além de aprender como agendar execuções recorrentes utilizando CronJobs.

---

# Objetivos de aprendizagem

Ao concluir este capítulo você deverá ser capaz de:

- [ ] Explicar a diferença entre um Deployment e um Job.
- [ ] Entender o ciclo de vida de um Job.
- [ ] Criar Jobs únicos.
- [ ] Criar CronJobs.
- [ ] Compreender políticas de retry e paralelismo.
- [ ] Escolher corretamente entre Deployment, Job e CronJob.

---

# Pré-requisitos

- [[07 - Deployments]]
- [[12 - DaemonSets]]

---

# O problema que este capítulo resolve

Imagine uma migração de banco.

Você possui um script Python.

```python
python migrate.py
```

Ele executa.

Atualiza o banco.

Finaliza.

Faz sentido manter esse container rodando para sempre?

Não.

Agora imagine um backup diário.

```bash
02:00

↓

Executar backup

↓

Finalizar
```

Também não faz sentido um Deployment.

---

# Modelo Mental

Imagine uma cafeteria.

Deployment.

O funcionário trabalha o dia inteiro.

```text
08:00

↓

17:00
```

Agora imagine um eletricista.

Ele chega.

Executa o serviço.

Vai embora.

Isso é um Job.

---

# O que é um Job?

Um Job é um Controller responsável por executar uma tarefa até sua conclusão.

Fluxo.

```text
Criar Pod

↓

Executar

↓

Finalizar

↓

Sucesso
```

O Kubernetes considera o Job concluído.

---

# Deployment x Job

Deployment.

```text
Pod terminou

↓

Criar outro
```

Job.

```text
Pod terminou

↓

Missão cumprida
```

Essa diferença é o conceito central.

---

# Anatomia de um Job

```yaml
apiVersion: batch/v1

kind: Job

metadata:

  name: migrate-db

spec:

  template:

    spec:

      restartPolicy: Never

      containers:

      - name: migrate

        image: minha-api

        command:

        - python

        - migrate.py
```

Observe.

O Pod não ficará executando.

---

# restartPolicy

Para Jobs normalmente utilizamos.

```yaml
restartPolicy: Never
```

Ou.

```yaml
restartPolicy: OnFailure
```

Nunca.

```yaml
Always
```

---

# Como um Job funciona?

Fluxo.

```text
kubectl apply

↓

Job

↓

Pod

↓

Executar

↓

Exit Code 0

↓

Completed
```

---

# Exit Codes

O Kubernetes utiliza o código de saída do processo.

```text
0

↓

Sucesso
```

Qualquer valor diferente.

```text
1

↓

Erro
```

```text
137

↓

OOMKilled
```

```text
143

↓

SIGTERM
```

Esses códigos determinam se o Job foi bem-sucedido.

---

# Retries

Imagine.

```text
Erro
```

O Kubernetes pode tentar novamente.

Exemplo.

```yaml
backoffLimit: 3
```

Fluxo.

```text
Falha

↓

Retry

↓

Falha

↓

Retry

↓

Falha

↓

Retry

↓

Falhou definitivamente
```

---

# Parallelism

Alguns Jobs podem executar em paralelo.

```yaml
parallelism: 5
```

Resultado.

```text
5 Pods

↓

Executando simultaneamente
```

Muito útil para ETLs e processamento de arquivos.

---

# Completions

Também podemos definir.

```yaml
completions: 10
```

Significa.

O Job será considerado concluído apenas após dez execuções bem-sucedidas.

---

# Parallelism x Completions

Imagine.

```yaml
parallelism: 2

completions: 10
```

O Kubernetes executará.

```text
2 Jobs

↓

Concluem

↓

Mais 2

↓

Mais 2

↓

...

↓

10 concluídos
```

---

# CronJobs

Agora imagine.

Todo dia.

À meia-noite.

Executar backup.

Não queremos fazer.

```bash
kubectl apply
```

Manualmente.

Criamos um CronJob.

---

# O que é um CronJob?

Um CronJob cria Jobs automaticamente em horários programados.

Fluxo.

```text
CronJob

↓

Criar Job

↓

Criar Pod

↓

Executar

↓

Finalizar
```

---

# Anatomia de um CronJob

```yaml
apiVersion: batch/v1

kind: CronJob

metadata:

  name: backup

spec:

  schedule: "0 2 * * *"

  jobTemplate:

    spec:

      template:

        spec:

          restartPolicy: Never

          containers:

          - name: backup

            image: backup-tool
```

---

# Cron Expression

Formato.

```text
MIN HORA DIA MÊS DIA_SEMANA
```

Exemplos.

Todos os dias.

```text
0 2 * * *
```

Toda hora.

```text
0 * * * *
```

A cada cinco minutos.

```text
*/5 * * * *
```

Segunda-feira às 08:00.

```text
0 8 * * 1
```

---

# concurrencyPolicy

Imagine.

Backup demora.

40 minutos.

Mas roda a cada.

30 minutos.

Teremos dois backups simultâneos.

Podemos controlar isso.

```yaml
concurrencyPolicy: Forbid
```

Novo Job não inicia.

Até o anterior terminar.

Também existe.

```yaml
Replace
```

Novo Job cancela o anterior.

---

# startingDeadlineSeconds

Imagine.

Cluster indisponível.

Cron deveria executar.

02:00.

Mas voltou às.

02:03.

Podemos definir.

```yaml
startingDeadlineSeconds: 300
```

O Job ainda será executado.

Depois disso.

É descartado.

---

# Suspend

Também podemos pausar.

```yaml
suspend: true
```

Cron continua existindo.

Mas não executa novos Jobs.

---

# Hands-on

Criar Job.

```bash
kubectl apply -f job.yaml
```

Consultar.

```bash
kubectl get jobs
```

Ver Pods.

```bash
kubectl get pods
```

Detalhes.

```bash
kubectl describe job migrate-db
```

Logs.

```bash
kubectl logs job/migrate-db
```

---

Criar CronJob.

```bash
kubectl apply -f cronjob.yaml
```

Consultar.

```bash
kubectl get cronjobs
```

Executar manualmente.

```bash
kubectl create job backup-manual --from=cronjob/backup
```

Muito útil para testes.

---

# Casos reais

## Migrações de banco

```text
Deploy

↓

Executar Job

↓

Migrar Schema

↓

Subir API
```

---

## Backup

```text
02:00

↓

Backup PostgreSQL

↓

S3
```

---

## ETL

```text
CSV

↓

Job

↓

Transformação

↓

Banco
```

---

## Relatórios

```text
CronJob

↓

Gerar PDF

↓

Enviar e-mail
```

---

## Limpeza

```text
CronJob

↓

Excluir arquivos temporários
```

---

# Comparando os Controllers

| Controller | Executa continuamente? | Principal uso |
|------------|-----------------------|---------------|
| Deployment | Sim | APIs e aplicações stateless |
| StatefulSet | Sim | Bancos e aplicações stateful |
| DaemonSet | Sim | Infraestrutura por Node |
| Job | Não | Tarefas únicas |
| CronJob | Não | Tarefas agendadas |

---

# Relação com os capítulos anteriores

Nosso modelo mental evolui.

```text
                     Scheduler
                          │
      ┌───────────────────┼───────────────────┐
      │                   │                   │
 Deployment         StatefulSet         DaemonSet
      │                   │                   │
 ReplicaSet       Pods com identidade   Um por Node
      │
     Pods
      │
      ├──────────────┐
      │              │
     Job         CronJob
      │              │
      ▼              ▼
   Pod único     Cria Jobs
```

---

# Boas práticas

- Utilize Jobs para tarefas finitas.
- Utilize CronJobs para automações recorrentes.
- Configure `backoffLimit` de acordo com a criticidade da tarefa.
- Utilize `concurrencyPolicy` para evitar execuções concorrentes indesejadas.
- Monitore Jobs finalizados e configure políticas de limpeza quando necessário.

---

# Erros comuns

> [!WARNING]

Executar um Job utilizando um Deployment.

O Pod permanecerá ativo desnecessariamente.

---

> [!WARNING]

Criar CronJobs sem controlar concorrência.

Pode gerar múltiplos backups ou múltiplas execuções simultâneas.

---

> [!WARNING]

Ignorar logs de Jobs.

Após a conclusão, os Pods podem ser removidos e os logs podem deixar de estar disponíveis, dependendo da política de retenção.

---

# Resumo

Neste capítulo aprendemos que:

- Jobs executam tarefas temporárias.
- CronJobs executam Jobs em horários programados.
- `parallelism` controla quantos Pods executam simultaneamente.
- `completions` define quantas execuções bem-sucedidas são necessárias.
- `backoffLimit` controla tentativas de repetição.
- `concurrencyPolicy` evita conflitos entre execuções agendadas.

---

# Checklist

Antes de continuar, verifique se você consegue responder:

- [ ] Quando utilizar um Job em vez de um Deployment?
- [ ] Qual a diferença entre Job e CronJob?
- [ ] O que faz `backoffLimit`?
- [ ] Qual a diferença entre `parallelism` e `completions`?
- [ ] O que faz `concurrencyPolicy`?
- [ ] Como executar manualmente um CronJob?

---

# Diagrama consolidado

```text
                           Scheduler
                                │
      ┌─────────────────────────┼─────────────────────────┐
      │                         │                         │
 Deployment               StatefulSet               DaemonSet
      │                         │                         │
 ReplicaSet            Pods (Identidade)         Um Pod por Node
      │
     Pods
      │
 ┌────┴────┐
 │         │
Job    CronJob
 │         │
 ▼         ▼
Pod     Cria Jobs
 │
Finaliza
```
