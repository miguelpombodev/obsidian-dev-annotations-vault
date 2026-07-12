Esta é uma explicação sobre como estas quatro ferramentas funcionam em conjunto para criar um sistema de observabilidade completo, sem redundâncias.

A melhor analogia é um **diagnóstico médico completo**:

- **Prometheus + Grafana (Métricas):** São os *sinais vitais* do seu paciente. Mostram gráficos da temperatura corporal, ritmo cardíaco e pressão arterial ao longo do tempo. (Visão Agregada).
- **Jaeger (Traces):** É a *ressonância magnética*. Quando o paciente diz "o meu braço dói", você faz um exame detalhado que mostra todo o percurso dos nervos e ossos *apenas* desse braço. (Visão Específica).
- **OpenTelemetry (OTel):** É o *equipamento médico* (sensores, agulhas, a própria máquina de ressonância) e o *protocolo* usado pelo hospital para recolher e enviar estes dados de forma padronizada.

### 1. A Proposta de Cada Ferramenta

### A. OpenTelemetry (OTel): O Canalizador (A "Cola")

- **O que é?** O OpenTelemetry não é um produto que você "vê"; é um **padrão** e um conjunto de ferramentas (SDKs, APIs) para *instrumentar* o seu código. É a "canalização" da sua observabilidade.
- **Proposta:** A sua proposta é **padronizar a recolha e exportação** dos três pilares: Métricas, Traces e Logs. Antes do OTel, cada fornecedor (Jaeger, Prometheus) tinha a sua própria forma de recolha, o que o "prendia" a essa ferramenta.
- **No seu projeto:** Você usa o OTel para:
    1. **Gerar Traces:** (Ex: `@opentelemetry/instrumentation-pg` que vê as suas *queries* SQL).
    2. **Enviar Traces:** (Ex: `OTLPTraceExporter` que envia os dados para o Jaeger).
    3. **Gerar Métricas:** (Ex: OTel pode gerar métricas como `http_requests_total`).
    4. **Enviar Métricas:** (OTel pode expor um *endpoint* que o Prometheus irá ler).

### B. Jaeger: O Detetive de Traces (Pilar 1: Rastreamento)

- **O que é?** Um sistema de *visualização* de *traces* (rastreamentos) distribuídos.
- **Proposta:** A sua proposta é responder à pergunta: **"Onde é que esta requisição específica passou e porquê é que ela foi lenta?"**
- **Como funciona:** Ele recebe os *traces* do OpenTelemetry e desenha aquele "gráfico em cascata" (waterfall) que você viu. Você pode ver:
    - `GET /users` demorou 500ms.
    - ...porque a chamada ao `UsersService` demorou 490ms.
    - ...porque a *query* SQL `SELECT *` demorou 480ms.
- **Foco:** A história de **uma** única requisição, em detalhe.

### C. Prometheus: A Base de Dados de Métricas (Pilar 2: Métricas)

- **O que é?** Uma base de dados de *séries temporais* (Time-Series Database) otimizada para armazenar **métricas** agregadas.
- **Proposta:** A sua proposta é responder à pergunta: **"Qual é a saúde geral do meu sistema?"**
- **Como funciona:** O Prometheus "raspa" (scrapes) *endpoints* `/metrics` da sua API em intervalos regulares (ex: a cada 15 segundos). Ele armazena dados como:
    - `http_requests_total{metodo="GET", rota="/users"}`: 1502 chamadas
    - `http_request_duration_seconds{percentil="0.95"}`: 0.8s
    - `database_connections_active`: 12
- **Foco:** Dados **agregados** e numéricos ao longo do tempo. Ele não sabe *porque é que* uma chamada específica foi lenta, apenas que a *média* de chamadas está a aumentar.

### D. Grafana: O Dashboard (A Visualização)

- **O que é?** Uma ferramenta de *visualização* e *dashboarding*.
- **Proposta:** A sua proposta é **mostrar os dados de forma bonita e útil**. O Grafana não armazena (quase) nada; ele apenas lê de outras fontes.
- **Como funciona:** Você "liga" o Grafana ao Prometheus. O Grafana "pergunta" ao Prometheus: "Dá-me a métrica `http_requests_total` das últimas 6 horas", e depois desenha um gráfico.
- **Foco:** Ser o seu "painel de controlo" central.

### 2. Como Eles se Completam (Sem Redundância)

Eles são perfeitamente complementares. Vamos usar um cenário real:

**O Cenário: O seu chefe diz "O site está lento!"**

1. **O Alerta (Grafana):** Você abre o seu *dashboard* no Grafana. Você vê imediatamente um gráfico (alimentado pelo Prometheus) que mostra que a "Latência p95 das Requisições" subiu de 100ms para 1500ms nos últimos 30 minutos.
    - **Pergunta respondida:** *O quê* está a acontecer? (O site está lento).
2. **A Investigação (Jaeger):** No Grafana, você tem um *link* que o leva ao Jaeger, filtrando por esse período de tempo. Você encontra um *trace* de uma requisição `GET /users` que demorou 1.5 segundos.
    - Ao abrir o *trace*, você vê que a chamada ao Redis foi rápida (5ms), mas a chamada à base de dados (a *query* SQL) demorou 1.4 segundos.
    - **Pergunta respondida:** *Onde* está a lentidão? (É na *query* SQL do *endpoint* `/users`).
3. **A Causa Raiz (Logs):** No seu *trace* do Jaeger, você vê um *log* (do Pino, correlacionado pelo OTel) que diz: `WARN - query took too long, index 'idx_profile_id' not used`.
    - **Pergunta respondida:** *Exatamente porquê* a *query* foi lenta? (Faltava um índice na base de dados).
4. **A Confirmação (Prometheus):** Você volta ao Prometheus (via Grafana) e consulta uma métrica `database_query_time_avg`. Você vê que, de facto, o tempo médio de todas as *queries* (não só daquela) subiu.

### Veredito: Não há Redundância

- **Prometheus** diz-lhe que a *febre* (latência) do seu paciente (aplicação) está alta.
- **Jaeger** é a *ressonância magnética* que lhe diz que a febre é causada por uma infeção no joelho.
- **Logs** são as *notas* que dizem que o paciente caiu e cortou o joelho às 10h.
- **OTel** é o *protocolo* que garante que a amostra de sangue (trace) e a medição da temperatura (métrica) chegam ao laboratório de forma correta.

Você não está a ser redundante; está a construir um sistema de diagnóstico completo.