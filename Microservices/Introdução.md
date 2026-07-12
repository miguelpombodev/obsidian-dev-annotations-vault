## Problemas Clássicos de Microservices e suas Resoluções

### 1. Distributed Data Management

**O problema em profundidade:** Em um monolito, você tem um banco de dados único e pode fazer JOINs, transações ACID e consultas complexas trivialmente. Em microservices, cada serviço tem seu próprio banco (Database per Service pattern). Isso significa que uma operação de negócio que antes era uma transação agora envolve múltiplos bancos independentes. O pedido está no Order Service, o inventário no Inventory Service, o pagamento no Payment Service. Como garantir consistência quando o pagamento confirma mas o inventário falha?

**A resolução — Saga Pattern:**

Existem duas implementações: Coreografia e Orquestração.

Na Coreografia cada serviço publica eventos e reage a eventos de outros serviços sem um coordenador central. O Order Service publica `OrderPlaced`, o Inventory Service escuta, reserva o estoque e publica `StockReserved`, o Payment Service escuta e processa o pagamento. Se o pagamento falha, o Payment Service publica `PaymentFailed`, o Inventory Service escuta e libera o estoque. É descentralizado, mas o fluxo de negócio fica espalhado e difícil de visualizar. Debugging de falhas requer correlacionar eventos de múltiplos serviços.

Na Orquestração um Saga Orchestrator central coordena os passos. Ele chama cada serviço explicitamente e decide o próximo passo. Se algo falha, o orquestrador invoca as transações compensatórias na ordem inversa. O fluxo fica centralizado e visível, mas o orquestrador se torna um ponto de acoplamento. Ferramentas: [Temporal.io](http://Temporal.io), Conductor (Netflix), MassTransit Saga (que você já usa).

A chave conceitual é que Sagas não são transações ACID — elas são ACD (sem Isolation). Durante a execução, outros serviços podem ler dados em estado intermediário. Isso é chamado de "dirty read" e precisa ser tratado no design de negócio com compensações e estados temporários explícitos.

### 2. Service-to-Service Communication e Cascading Failures

**O problema em profundidade:** Serviço A chama B que chama C que chama D. Se D está lento (não down, só lento), C fica aguardando, suas threads ficam ocupadas, B fica aguardando, suas threads ficam ocupadas, A fica aguardando. Em minutos, um serviço lento na ponta da cadeia derruba toda a cadeia por esgotamento de recursos. Isso é cascade failure e é o modo de falha mais comum em microservices.

**A resolução — Camadas de defesa:**

Timeout é a primeira linha. Toda chamada externa precisa de timeout explícito e agressivo. Um timeout de 30s em uma cadeia de 5 serviços significa que o usuário espera potencialmente 150s antes de ver um erro. Defina timeouts por budget: se o SLO do endpoint é 500ms e você tem 3 calls, cada uma tem budget de ~150ms.

Circuit Breaker é a segunda linha. Quando o timeout está estourando consistentemente, o circuit breaker abre e as chamadas falham imediatamente sem tentar o serviço. Isso protege os recursos do caller e dá tempo para o serviço downstream se recuperar sem ser martelado. Polly no .NET, Resilience4j no Java.

Bulkhead é a terceira linha. Isola os recursos por serviço downstream. Em vez de um pool de threads compartilhado, cada integração tem seu próprio pool limitado. Se o serviço D esgota seu pool, não afeta as chamadas para o serviço E.

Async communication é a resolução estrutural. Quando possível, substitua chamadas síncronas por mensagens assíncronas. O produtor publica e segue em frente sem esperar. Isso elimina o cascade failure por definição — se o consumer está lento, a fila cresce, mas o produtor não é afetado.

### 3. Distributed Tracing e Observability

**O problema em profundidade:** Uma requisição do usuário atravessa 8 serviços. O usuário reporta lentidão. Qual serviço é o culpado? Com logs distribuídos sem correlação, você tem que vasculhar logs de 8 serviços manualmente tentando correlacionar por timestamp. Com logs correlacionados mas sem tracing, você sabe a sequência mas não sabe onde o tempo foi gasto. Com métricas mas sem contexto, você vê que a latência aumentou mas não sabe qual request específico está lento.

**A resolução — OpenTelemetry + Distributed Tracing:**

Cada request ganha um `trace_id` único na entrada do sistema (API Gateway ou primeiro serviço). Esse trace_id é propagado em todos os headers subsequentes (W3C `traceparent` header). Cada serviço cria spans filhos com o mesmo trace_id. No final, você tem uma árvore de spans que mostra exatamente onde cada milissegundo foi gasto, em qual serviço, em qual operação.

O Jaeger ou Grafana Tempo armazena e visualiza esses traces. Você abre um trace e vê: 400ms total, sendo 10ms no API Gateway, 5ms no Auth Service, 380ms no Order Service (sendo 350ms na query ao banco), 5ms no Notification Service. O gargalo é óbvio.

### 4. Service Discovery e Configuration Management

**O problema em profundidade:** Com dezenas de serviços, como o serviço A sabe o endereço do serviço B? IPs mudam com escalonamento. Ambientes diferentes têm endpoints diferentes. Configurações sensíveis (strings de conexão, secrets) não podem estar no código.

**A resolução:**

No Kubernetes, Service Discovery é nativo via DNS. Cada Service tem um DNS name estável (`payment-service.default.svc.cluster.local`) independente de quantos pods existam ou quais IPs tenham. Fora do Kubernetes, Consul fornece service registry com health checking integrado.

Para configuração, o padrão é externalizar via variáveis de ambiente (12-factor app) combinado com um config server (Spring Cloud Config, Azure App Configuration) ou secrets manager (Vault, AWS Secrets Manager). A configuração muda sem redeployar o serviço.

### 5. API Versioning e Backward Compatibility

**O problema em profundidade:** O Order Service v2 muda o formato do evento `OrderPlaced`. O Notification Service ainda está na versão que espera o formato v1. O deploy do Order Service v2 quebra o Notification Service silenciosamente — mensagens chegam, o consumer tenta desserializar e falha.

**A resolução:**

Contract Testing com Pact. O Notification Service define um contrato: "eu espero um evento com esses campos nessa estrutura". O Order Service verifica seu output contra esse contrato no CI antes do deploy. Se o Order Service mudou o formato de forma incompatível, o pipeline falha antes de chegar em produção.

Schema Registry (Confluent, Apicurio) para sistemas baseados em mensageria. Todo evento tem um schema registrado. Producers validam o output contra o schema antes de publicar. Consumers sabem qual versão do schema usar para desserializar. Compatibility modes (BACKWARD, FORWARD, FULL) definem quais mudanças são permitidas automaticamente.

Tolerant Reader Pattern: consumers devem ignorar campos desconhecidos em vez de falhar. Isso permite que producers adicionem campos sem quebrar consumers existentes.

### 6. Eventual Consistency e User Experience

**O problema em profundidade:** O usuário faz um pedido. O Order Service salva e retorna 200. Internamente, um evento é publicado, o Inventory Service processa e atualiza o estoque, o Notification Service envia o email. Tudo isso é eventual. O usuário atualiza a página imediatamente e o pedido aparece como "pending" no histórico porque a read model ainda não foi atualizada. O usuário pensa que falhou e faz o pedido de novo.

**A resolução:**

Read-your-own-writes consistency. Após uma escrita, o sistema garante que o mesmo usuário que escreveu vai ler seu próprio dado atualizado. Implementações: rotear reads do mesmo usuário para o mesmo nó por um período pós-write, ou usar um cache local do write que é retornado nas reads subsequentes por um tempo.

Optimistic UI com estado temporário. O frontend exibe o estado esperado imediatamente (o pedido como "placed") sem esperar a confirmação do backend. Se o backend eventualmente confirmar o estado diferente, reconcilia. É o que Amazon, Netflix e a maioria dos sistemas modernos fazem.

Causal consistency tokens. O write retorna um token de consistência (um timestamp ou versão). O frontend inclui esse token nas reads subsequentes, e o sistema garante que a read vê pelo menos o estado no momento do write.

---

## Problemas de Persistência de Dados em Microservices

### 1. Database per Service vs Shared Database

**O problema:** Times iniciantes em microservices frequentemente mantêm um banco de dados compartilhado entre serviços para "facilitar". O resultado é acoplamento no nível de banco: qualquer alteração de schema afeta múltiplos serviços, qualquer serviço com query pesada afeta todos os outros, e o banco vira o ponto central de falha que a arquitetura deveria eliminar.

**A resolução:** Database per Service é inegociável para microservices reais. Cada serviço possui seu schema completamente. Outros serviços nunca acessam diretamente — apenas via API ou eventos. Se dois serviços precisam dos mesmos dados frequentemente, é um sinal de que os bounded contexts estão errados e talvez devessem ser o mesmo serviço, ou que um dos serviços deveria manter uma cópia local (projeção) dos dados que precisa do outro.

### 2. Distributed Transactions e o Problema do Two-Phase Commit (2PC)

**O problema em profundidade:** 2PC é o protocolo clássico para transações distribuídas. Um coordenador pergunta a todos os participantes se podem commitar (fase 1), espera todos confirmarem, então manda commitar (fase 2). O problema: se o coordenador cai após a fase 1 mas antes da fase 2, os participantes ficam em estado bloqueado indefinidamente segurando locks. 2PC também é síncrono e lento — todos os participantes ficam bloqueados esperando o coordenador.

**A resolução:** Evitar 2PC. As alternativas são Saga (descrita acima) para consistência eventual, ou Outbox Pattern para garantia de entrega sem transação distribuída.

O Outbox Pattern funciona assim: a aplicação escreve no seu próprio banco (tabela de dados + tabela outbox) em uma única transação local ACID. Um processo separado (ou CDC via Debezium) lê a tabela outbox e publica os eventos no broker. Se a publicação falha, o processo tenta novamente — a mensagem permanece na outbox até ser confirmada. A atomicidade entre "salvar o dado" e "publicar o evento" é garantida sem 2PC.

### 3. Cross-Service Queries e Reporting

**O problema em profundidade:** O manager quer um relatório que combina dados de Order Service, Customer Service, Product Service e Payment Service. No monolito seria um JOIN. Em microservices, você não pode fazer JOIN entre bancos diferentes de serviços diferentes.

**A resolução — CQRS com Read Models:**

Cada serviço publica eventos quando seu estado muda. Um serviço dedicado de reporting (ou uma projeção) consome esses eventos e mantém um banco desnormalizado, otimizado para as queries que o relatório precisa. O schema desse banco é desenhado para leitura, não para escrita — pode ter dados duplicados, denormalizados, exatamente no formato que a query precisa.

Essa projeção pode estar em um banco relacional (para queries SQL), Elasticsearch (para full-text search e aggregations), Redis (para acesso rápido), ou um Data Warehouse (para analytics complexos). O ponto é que a read model é derivada dos eventos e pode ser reconstruída a qualquer momento fazendo replay do event log.

### 4. Data Consistency em Falhas Parciais

**O problema em profundidade:** O Order Service salva o pedido com sucesso. Tenta publicar o evento `OrderPlaced` no broker. O broker está temporariamente indisponível. O evento nunca é publicado. O Inventory Service nunca reserva o estoque. O Payment Service nunca é acionado. O pedido existe no banco do Order Service mas o restante do sistema nunca soube que ele existe.

**A resolução — Outbox Pattern (novamente, mas agora sob a perspectiva de falha):**

Com Outbox, o evento é gravado na tabela outbox na mesma transação que o pedido. Se o broker está down, o evento permanece na outbox. Quando o broker volta, o processo de relay lê os eventos pendentes e publica. A ordem de entrega pode ser afetada (at-least-once), então os consumers precisam ser idempotentes.

O Inbox Pattern é o complemento do Outbox no lado do consumer. Antes de processar uma mensagem, o consumer grava o `message_id` em uma tabela inbox. Se o mesmo `message_id` chegar novamente (reentrega), o consumer detecta e ignora. Isso garante exatamente-uma vez no processamento, mesmo que a entrega seja at-least-once.

### 5. Schema Evolution e Backward Compatibility no Banco

**O problema em profundidade:** O Order Service precisa adicionar um novo campo ao banco. A migration altera a tabela. Se a migration for incompatível com o código antigo (remover uma coluna que o código ainda usa, renomear uma coluna), o deploy causa falha imediatamente ou dados corrompidos.

**A resolução — Expand-Contract com Blue-Green:**

A sequência correta foi detalhada no guia anterior, mas vale reforçar o ponto crítico: nunca faça a migration e o deploy do novo código simultaneamente. A migration deve ser compatível com o código atual antes do deploy da nova versão.

Para colunas NOT NULL sem default, a sequência é: adicione como nullable (expand), deploye o código que popula o campo, execute backfill para dados históricos, adicione o NOT NULL constraint apenas quando 100% dos dados estiverem populados (contract). Fazer tudo de uma vez bloqueia a tabela e causa downtime.

### 6. Polyglot Persistence e Gestão de Múltiplos Datastores

**O problema em profundidade:** Com Database per Service e liberdade tecnológica, diferentes serviços escolhem diferentes bancos: PostgreSQL, MongoDB, Redis, Cassandra, Elasticsearch. Isso é polyglot persistence — cada serviço usa o banco mais adequado ao seu modelo de dados e padrão de acesso. O problema é operacional: mais tecnologias = mais expertise necessária, mais runbooks, mais pontos de falha, mais custos de licença e infra.

**A resolução — Governed Polyglot Persistence:**

Defina um conjunto aprovado de tecnologias de dados (o "short list") que o time domina e tem capacidade operacional de manter. Times podem escolher dentro desse conjunto. Desvios precisam de justificativa técnica e aprovação arquitetural — não por burocracia, mas porque o custo de adicionar uma nova tecnologia de dados ao stack é maior do que parece.

Para cada tecnologia do short list, mantenha: runbooks operacionais, backup e recovery testados, monitoring e alerting configurados, expertise distribuída em pelo menos dois engenheiros. Uma tecnologia que só uma pessoa entende é um risco arquitetural.

### 7. Event Sourcing e Reconstrução de Estado

**O problema em profundidade:** Em Event Sourcing, o estado atual é reconstruído fazendo replay de todos os eventos. Com o tempo, o número de eventos cresce e o replay fica cada vez mais lento. Um Aggregate com 50.000 eventos precisa processar 50.000 eventos para chegar ao estado atual.

**A resolução — Snapshots:**

Periodicamente, o estado atual do Aggregate é serializado e salvo como um snapshot. Na reconstrução, em vez de fazer replay desde o início, carrega o snapshot mais recente e faz replay apenas dos eventos posteriores ao snapshot. O custo de um snapshot a cada 100 eventos significa no máximo 100 eventos de replay em vez de 50.000.

A frequência de snapshot é um trade-off: frequente demais aumenta o custo de escrita, raro demais mantém o problema de replay lento. Uma estratégia comum é criar um snapshot quando o número de eventos desde o último snapshot ultrapassa um threshold.