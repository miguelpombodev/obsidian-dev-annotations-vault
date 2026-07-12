
## O que é um API Gateway?

É uma ferramenta de gerenciamento, geralmente adicionada entre o **cliente** e um **grupo de sistemas** de um determinado contexto, atuando como ponto único de entrada das APIs

### Funcionalidades de um API Gateway

Geralmente o API Gateway atua na camada de rede, provendo funcionalidade ortogonais, as que não necessariamente são responsabilidade das aplicações.

### Quais são essas funcionalidades?

- Controle de Abuso (Rate Limiting;
- Autenticação / Autorização de maneira padronizada
- Controle de Logs
- Gerenciamento de APIs (routing)
- Métricas Padronizadas (OPS Team)
- Tracing Distribuido

## Tipos de API Gateway

Hoje temos dois tipos de Gateways para usar:

- Enterprise Gateway;
- MiniGateway

### Enterprise Gateway

Foco deste tipo de solução é na grande maioria das vezes realizar exposição e gerenciamento de deployment de APIs voltadas ao negócio, em geral ele também permite controlar o ciclo de vida de uma API. Em geral, é uma oferta de algum vendor (agente comerical), com estratégias comerciais suportando a solução. Este também tende a direcionar a aplicação da solução e implicar no design dos serviços.

### Propósito Principal

Exposição, composição e gerenciamento de API externas/internas

### Manutenção das APIs

Time de APIs em geral faz administração via Portal do API Gateway

### Suporte a ambientes

Suporta múltiplos ambientes DEV, HMG, PRD

---

# 🦍 Sobre o Kong API Gateway

Pontos de Vantagem do Kong:

- API Gateway Open Source, ou seja, podemos usar ele em PRD sem termos um custo sobre o serviço;
- Caracteristicas de Micro Gateway;
- Deployment flexível, podemos tanto usar ele dentro da arquitetura fazendo uma segregação ou usando a camada de borda (**egde**)
- Pronto para o Kubernetes
- Extensivel via plugins, a parte customizada do Kong, criando um comportamento específico dentro das linguagens disponiveis