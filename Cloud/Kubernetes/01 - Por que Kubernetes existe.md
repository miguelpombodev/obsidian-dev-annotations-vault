---
tags:
  - kubernetes
  - docker
  - containers
  - arquitetura
---
> [!INFO] Objetivo
> Compreender quais problemas o Kubernetes resolve, por que ele foi criado e qual o modelo mental correto para estudá-lo.

---

# Objetivos de aprendizagem

Ao finalizar este capítulo você deverá ser capaz de:

- [ ] Explicar por que o Kubernetes foi criado.
- [ ] Diferenciar Docker de Kubernetes.
- [ ] Entender os limites do Docker Compose.
- [ ] Compreender o conceito de **Desired State**.
- [ ] Entender quando Kubernetes faz (e quando não faz) sentido.

---

# Pré-requisitos

- Docker
- Docker Compose
- Conceitos básicos de Linux
- Conceitos básicos de redes
- Conhecimento básico sobre deploy de aplicações

---

# O problema

É muito comum encontrar pessoas que aprendem Kubernetes decorando comandos como:

```bash
kubectl get pods

kubectl logs

kubectl apply
```

Mas poucos conseguem responder uma pergunta muito mais importante:

> **Por que o Kubernetes foi criado?**

Antes de aprender qualquer comando precisamos entender o problema que ele resolve.

Toda tecnologia surge para resolver limitações de outra tecnologia.

Por exemplo:

| Tecnologia | Problema que resolveu |
|------------|-----------------------|
| Git | Controle de versão distribuído |
| Docker | Empacotamento e isolamento de aplicações |
| Kubernetes | Orquestração de aplicações distribuídas |

Perceba que Kubernetes **não substitui Docker**.

Ele resolve um problema completamente diferente.

---

# Cenário 1 — Uma aplicação simples

Imagine uma API escrita em Python utilizando FastAPI.

Sua arquitetura é bastante simples.

```text
Cliente
    │
    ▼
 FastAPI
    │
    ▼
PostgreSQL
```

Durante o desenvolvimento basta executar:

```bash
uvicorn app:app --reload
```

Tudo funciona perfeitamente.

---

# Primeiro deploy

Agora você cria um Dockerfile.

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY . .

CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "8000"]
```

Depois:

```bash
docker build -t minha-api:v1 .
```

E finalmente:

```bash
docker run -p 8000:8000 minha-api:v1
```

Sua aplicação já está em produção.

Até este ponto, Kubernetes seria completamente desnecessário.

> [!TIP]
> Uma aplicação pequena **não precisa de Kubernetes**. Ele não deve ser adotado apenas porque está "na moda".

---

# O crescimento da aplicação

Agora imagine que sua API cresce.

Antes:

- 50 usuários por dia

Agora:

- 50.000 usuários por dia

O servidor começa a apresentar problemas.

- CPU alta
- Memória elevada
- Latência crescente
- Timeouts

A primeira solução normalmente é aumentar o servidor.

---

# Escalabilidade Vertical (Scale Up)

Escalar verticalmente significa adicionar mais recursos à mesma máquina.

Antes:

```text
Servidor

2 vCPU
4 GB RAM
```

Depois:

```text
Servidor

16 vCPU
64 GB RAM
```

## Vantagens

- Implementação simples.
- Não exige mudanças na aplicação.
- Rápida de executar.

## Desvantagens

- Existe um limite físico para crescimento.
- O custo cresce rapidamente.
- Geralmente exige downtime.
- Continua existindo apenas um servidor.

> [!WARNING]
> Escalar verticalmente melhora a capacidade de processamento, mas não melhora a disponibilidade da aplicação.

---

# Escalabilidade Horizontal (Scale Out)

Ao invés de aumentar um único servidor, criamos vários servidores menores.

```text
                 Cliente
                     │
                     ▼
             Load Balancer
          ┌──────┼──────┐
          ▼      ▼      ▼
       API A   API B   API C
```

Agora cada servidor atende parte das requisições.

Esse modelo é muito mais escalável.

Mas ele introduz novos desafios.

---

# Problema 1 — Balanceamento de carga

Agora existem três servidores.

Como o cliente sabe para qual deles enviar uma requisição?

Precisamos de um balanceador de carga.

Exemplos:

- Nginx
- HAProxy
- AWS ELB
- AWS ALB
- Azure Load Balancer

---

# Problema 2 — Atualizações

Imagine que todos os servidores executam a versão:

```text
v1
```

Você publica:

```text
v2
```

Agora é necessário atualizar todos os servidores.

Com três máquinas isso é simples.

Com cinquenta já começa a ficar complicado.

Com centenas torna-se inviável manualmente.

---

# Problema 3 — Downtime

Uma atualização tradicional normalmente segue este fluxo:

```text
Parar aplicação

↓

Atualizar

↓

Iniciar novamente
```

Durante esse intervalo os usuários recebem erros.

```text
HTTP 502

HTTP 503
```

Em produção queremos atualizar sem indisponibilidade.

Mais adiante veremos que esse processo recebe o nome de **Rolling Update**.

---

# Problema 4 — Recuperação automática

Imagine que o processo da API termina inesperadamente.

```text
API

↓

Erro fatal

↓

Container encerrado
```

Quem recria esse container?

Docker consegue reiniciar containers utilizando políticas de restart.

Mas isso funciona apenas naquela máquina.

Agora imagine que o servidor inteiro falhou.

Quem cria a aplicação em outro servidor?

---

# Problema 5 — Descoberta de serviços

Agora sua aplicação virou uma arquitetura de microsserviços.

```text
Gateway

↓

Usuários

↓

Pedidos

↓

Pagamentos
```

A API de Pedidos precisa localizar a API de Usuários.

Hoje ela está em:

```text
10.0.1.10
```

Amanhã:

```text
10.0.8.20
```

Depois:

```text
10.0.5.42
```

Os endereços IP mudam constantemente.

Precisamos de uma forma estável de localizar serviços.

---

# Problema 6 — Escalabilidade automática

Às 3h da manhã existem apenas alguns usuários.

Às 14h existem milhares.

O ideal seria:

```text
Poucas instâncias

↓

Aumento de carga

↓

Novas instâncias

↓

Redução da carga

↓

Remoção das instâncias excedentes
```

Quem toma essa decisão?

Precisamos de um sistema capaz de observar métricas e reagir automaticamente.

---

# Problema 7 — Configuração

Imagine três ambientes.

- Desenvolvimento
- Homologação
- Produção

Todos utilizam a mesma imagem Docker.

O que muda é apenas a configuração.

Exemplo:

```text
DATABASE_URL

↓

Dev

↓

Homologação

↓

Produção
```

A configuração não deve estar embutida na imagem.

Ela precisa ser separada da aplicação.

---

# Problema 8 — Persistência

Considere um banco PostgreSQL executando em Docker.

Se o container for recriado sem um volume persistente:

```text
Banco de dados

↓

Container removido

↓

Todos os dados são perdidos
```

Em um ambiente distribuído esse desafio é ainda maior.

Precisamos garantir que os dados permaneçam disponíveis independentemente do servidor onde a aplicação esteja executando.

---

# Problema 9 — Agendamento

Imagine um cluster com três servidores.

```text
Node A

32 CPUs
```

```text
Node B

2 CPUs
```

```text
Node C

64 CPUs
```

Uma aplicação que necessita de 16 CPUs jamais poderá ser executada no Node B.

Alguém precisa decidir onde cada aplicação será executada.

---

# O conceito de Desired State

Este é provavelmente o conceito mais importante do Kubernetes.

Docker trabalha de forma imperativa.

Você diz:

```text
Execute este container.
```

Kubernetes trabalha de forma declarativa.

Você diz:

```text
Quero cinco instâncias desta aplicação.
```

O Kubernetes observa continuamente o estado do cluster.

Se existirem apenas quatro instâncias:

```text
Desejado: 5

Atual: 4

↓

Criar mais uma instância.
```

Se existirem seis:

```text
Desejado: 5

Atual: 6

↓

Remover uma instância.
```

Esse processo é chamado de **Reconciliação** (*Reconciliation Loop*).

> [!IMPORTANT]
> O Kubernetes não executa containers.
>
> O Kubernetes mantém continuamente o estado desejado da aplicação.

---

# Docker vs Kubernetes

| Docker | Kubernetes |
|---------|------------|
| Executa containers | Orquestra containers |
| Máquina única | Cluster |
| Imperativo | Declarativo |
| Escalabilidade manual | Escalabilidade automática |
| Restart local | Auto-recuperação distribuída |
| Sem scheduler | Scheduler integrado |
| Sem service discovery | Service Discovery nativo |

---

# Mental Model

> [!TIP]
> Não pense em "executar containers".
>
> Pense em "descrever sistemas".

Essa mudança de mentalidade facilita o aprendizado de todos os capítulos seguintes.

---

# Resumo

Neste capítulo aprendemos que:

- Docker e Kubernetes possuem objetivos diferentes.
- Docker resolve o problema de empacotamento e isolamento.
- Kubernetes resolve problemas operacionais de aplicações distribuídas.
- O conceito central do Kubernetes é o **Desired State**.
- Escalar aplicações envolve muito mais do que executar múltiplos containers.

---

# Checklist

Antes de continuar, verifique se você consegue responder:

- [ ] Por que Kubernetes foi criado?
- [ ] O que é Escalabilidade Horizontal?
- [ ] O que é Escalabilidade Vertical?
- [ ] O que significa Desired State?
- [ ] Qual a diferença entre Docker e Kubernetes?

---

# Próximo capítulo

➡️ [[Capítulo 2 - Arquitetura do Kubernetes]]