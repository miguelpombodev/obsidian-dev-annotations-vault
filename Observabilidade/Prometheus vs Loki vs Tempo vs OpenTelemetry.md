# Prometheus

Para uma compreensão maior sobre o Prometheus, veja o artigo abaixo:

[[Prometheus]]

Basicamente, o Prometheus é uma aplicação de monitoramento e alertas de outras aplicações nas quais precisemos dessa estratégia.

# Grafana Loki

O Grafana Loki é um sistema multi-tenant de agregação de logs criado pela Grafana Labs, que é horizontalmente escalável e com alta disponibilidade. Resumidamente, o Loki vai coletar os logs de uma aplicação, sejam eles gerados em um arquivo ou por console, e todos esses logs podem ser visto no Grafana.

# Grafana Tempo

O Grafana Tempo é um tracer distribuído também desenvolvido pela Grafana Labs, que basicamente, é uma aplicação que verificar os traces de uma outra aplicação, podendo ela ser uma API, ou qualquer outro tipo de aplicação.

# OpenTelemetry

Do mesmo jeito que o Prometheus, para uma maior compreensão, veja o artigo abaixo:

[[Open Telemetry]]

O OpenTelemetry é um framework de observabilidade para softwares cloud native, criado pela CNCF (**Cloud Native Computing Foundation**), ou seja, o OpenTelemetry cuida de tudo o que precisamos para trabalhar com observabilidade, que são logs, métricas e tracing, além de trabalhar com **instrumentação, geração, coleta e exportação de dados de telemetria.**