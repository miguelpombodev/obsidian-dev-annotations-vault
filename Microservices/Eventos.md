### O Que é um Evento — O Conceito Mais Fundamental

Um evento é um **fato que aconteceu no passado e não pode ser desfeito**. A palavra "aconteceu" é chave — não é uma intenção, não é um comando, não é uma pergunta. É um registro imutável de algo que ocorreu.

Compare:

```
Comando:  "RegisterUser"     → uma intenção, pode falhar, pode ser rejeitada
Evento:   "UserRegistered"   → um fato, já aconteceu, é imutável
```

Essa distinção muda tudo sobre como você modela e processa cada um. Um comando pode ser recusado. Um evento nunca é recusado — ele é o relato de algo que já ocorreu.

---

### Domain Events — Eventos Que Têm Significado no Domínio

Um Domain Event é um evento que **tem significado para o negócio dentro de um Bounded Context específico**. Não é um evento técnico — é um evento de negócio.

No Identity Service do CloudMart:

```
UserRegistered      → um usuário completou o cadastro
UserLoggedIn        → um usuário autenticou com sucesso
RefreshTokenRotated → um token foi rotacionado
TokenReuseDetected  → um token revogado foi usado (possível comprometimento)
UserRevoked         → todos os tokens de um usuário foram revogados
```

Cada um desses é um fato de negócio — algo que aconteceu no domínio de identidade e que **pode interessar a outras partes do sistema**.

A característica fundamental de um Domain Event em DDD é que ele é gerado **dentro da entidade ou aggregate**, não fora. Você não gera um `UserRegistered` no Handler — você gera dentro do `User.Create()`. O Handler apenas orquestra; a entidade decide quando o evento acontece.

```
// ❌ Errado — Handler decide quando o evento existe
public async Task Handle(RegisterUserCommand command, ...)
{
    var user = User.Create(...);
    await _repository.AddAsync(user);
    // Handler gerando o evento — não é responsabilidade dele
    _eventBus.Publish(new UserRegisteredEvent(user.Id, user.Email));
}

// ✅ Correto — a entidade registra o fato
public static Result<User> Create(CompleteName name, Email email, Password password, Role role)
{
    var user = new User(name, email, password, role) { Id = Guid.NewGuid() };
    // A entidade sabe que o fato "User foi criado" aconteceu
    user.AddDomainEvent(new UserRegisteredEvent(user.Id, email.Address));
    return user;
}
```

---

### Domain Events vs Integration Events — A Distinção Mais Importante

Aqui é onde a maioria das implementações se confunde.

**Domain Event** — acontece **dentro** de um único Bounded Context, na mesma transação, de forma síncrona. É uma comunicação interna do domínio consigo mesmo. Outros aggregates do mesmo serviço podem reagir a ele.

**Integration Event** — acontece **entre** Bounded Contexts, de forma assíncrona, via mensageria. É a versão "publicada para o mundo" de um Domain Event.

```
Identity Service
  │
  ├── User.Create() dispara UserRegisteredDomainEvent   ← Domain Event (interno, síncrono)
  │
  ├── Handler reage ao DomainEvent:
  │     → envia email de boas-vindas (mesmo serviço)
  │     → publica UserRegisteredIntegrationEvent       ← Integration Event (externo, assíncrono)
  │
  └── RabbitMQ/Kafka recebe o Integration Event
        │
        ├── Catalog Service reage → cria lista de desejos padrão
        └── Notification Service reage → envia push notification
```

A transformação de Domain Event em Integration Event é **deliberada e explícita**. Você decide quais fatos internos merecem ser comunicados ao mundo externo. Nem todo Domain Event vira Integration Event.

---

### Onde o Outbox Pattern Entra

O Outbox Pattern resolve o problema de garantir que o Integration Event **sempre** seja publicado quando o dado é salvo, sem possibilidade de inconsistência.

O problema sem Outbox:

```
1. Salva o User no banco           ✅
2. Publica no RabbitMQ             💥 falha aqui

Resultado: User existe no banco, mas o Catalog nunca soube
```

Você já tem `OutboxMessage` implementado no BuildingBlocks. O fluxo correto com Outbox:

```
Dentro da mesma transação (atômica):
  1. INSERT INTO identity.users (...)
  2. INSERT INTO identity.outbox_messages (
       type:    "UserRegisteredIntegrationEvent",
       payload: { "userId": "...", "email": "...", "occurredOn": "..." }
     )
  COMMIT — ou os dois acontecem ou nenhum

Background Worker (separado):
  3. SELECT * FROM outbox_messages WHERE processed_on IS NULL
  4. Publica no RabbitMQ/Kafka
  5. UPDATE outbox_messages SET processed_on = now()
```

O banco e a mensageria nunca são escritos na mesma operação — a mensagem fica no banco como "pendente" e o worker publica depois. Se o worker cair, na próxima execução ele encontra a mensagem pendente e tenta de novo. **Sem perda de eventos**.

---

### Onde o Kafka Entra Nessa Lógica

O Kafka não substitui o Outbox Pattern — ele é uma opção de **transporte** para os Integration Events, no lugar do RabbitMQ.

A diferença que o Kafka muda na arquitetura é o que acontece **depois** da publicação:

```
Com RabbitMQ:
  Outbox Worker → publica → RabbitMQ Queue → Consumer A processa → mensagem some
                                           → Consumer B nunca viu

Com Kafka:
  Outbox Worker → publica → Kafka Topic → Consumer A lê do offset 100
                                        → Consumer B lê do offset 78 (atrasado)
                                        → Consumer C lê do offset 1 (replay de ontem)
```

O Kafka faz sentido quando:

**Múltiplos consumidores independentes** precisam do mesmo evento — `UserRegistered` interessa ao Catalog (criar wishlist), ao Notification Service (boas-vindas), ao Analytics (métricas de crescimento), ao Data Warehouse (ETL). Com RabbitMQ, você precisaria de múltiplas filas ou fanout. Com Kafka, todos leem do mesmo topic no próprio ritmo.

**Replay de eventos** — se o Catalog Service ficou fora por 2 horas, quando voltar pode ler os eventos que perdeu. Com RabbitMQ sem DLQ configurada corretamente, esses eventos teriam sumido.

**Event Sourcing** — se você quiser reconstruir o estado do sistema a partir dos eventos históricos, o Kafka serve como o log imutável de tudo que aconteceu.

Para o CloudMart no estágio atual, o **RabbitMQ com Outbox é suficiente e correto**. Kafka seria a evolução natural quando você precisar de analytics em tempo real ou Event Sourcing.

---

### Como Encaixar um Domain Event no Identity Service

Vou mostrar o fluxo completo de `UserRegistered` no seu código atual.

#### Passo 1 — Definir o Domain Event

csharp

```csharp
// Identity.Domain/Events/UserRegisteredEvent.cs
namespace Identity.Domain.Events;

public sealed record UserRegisteredEvent : DomainEvent
{
    public Guid   UserId   { get; }
    public string Email    { get; }
    public string Provider { get; }   // "manual", "google", etc.

    public UserRegisteredEvent(Guid userId, string email, string provider)
    {
        UserId   = userId;
        Email    = email;
        Provider = provider;
    }
}
```

#### Passo 2 — A Entidade Dispara o Evento

csharp

```csharp
// Identity.Domain/Entities/User.cs
public static Result<User> Create(
    CompleteName name,
    Email        email,
    Password     password,
    Role         role)
{
    var user = new User(name, email, password, role) { Id = Guid.NewGuid() };

    // A entidade registra que o fato "User foi criado" aconteceu
    // O Handler não sabe disso — é responsabilidade do domínio
    user.AddDomainEvent(new UserRegisteredEvent(
        userId:   user.Id,
        email:    email.Address,
        provider: "manual"));

    return user;
}
```

#### Passo 3 — O EF Core Intercepta e Converte para Outbox

Esse é o mecanismo que conecta Domain Events ao Outbox Pattern automaticamente. Você cria um `SaveChangesInterceptor` que, antes de commitar, pega os domain events das entidades e os converte em `OutboxMessage`:

csharp

```csharp
// Identity.Infrastructure/Persistence/Interceptors/OutboxDomainEventInterceptor.cs
namespace Identity.Infrastructure.Persistence.Interceptors;

public sealed class OutboxDomainEventInterceptor : SaveChangesInterceptor
{
    public override async ValueTask<InterceptionResult<int>> SavingChangesAsync(
        DbContextEventData      eventData,
        InterceptionResult<int> result,
        CancellationToken       cancellationToken = default)
    {
        var context = eventData.Context;

        if (context is null)
            return await base.SavingChangesAsync(eventData, result, cancellationToken);

        // Coleta todos os domain events de todas as entidades rastreadas
        var outboxMessages = context.ChangeTracker
            .Entries<Aggregate<Guid>>()             // só Aggregates têm domain events
            .SelectMany(entry => entry.Entity.DomainEvents)
            .Select(OutboxMessage.CreateFromEvent)   // converte para OutboxMessage
            .ToList();

        // Limpa os eventos das entidades — já foram "coletados"
        context.ChangeTracker
            .Entries<Aggregate<Guid>>()
            .ToList()
            .ForEach(entry => entry.Entity.ClearDomainEvents());

        // Adiciona as OutboxMessages ao contexto — serão salvas na mesma transação
        await context.Set<OutboxMessage>().AddRangeAsync(outboxMessages, cancellationToken);

        return await base.SavingChangesAsync(eventData, result, cancellationToken);
    }
}
```

Registra o interceptor no DbContext:

csharp

```csharp
// Em AddInfrastructureServices
services.AddSingleton<OutboxDomainEventInterceptor>();

services.AddDbContext<ApplicationDbContext>((sp, options) =>
{
    options.UseNpgsql(connectionString);
    options.AddInterceptors(sp.GetRequiredService<OutboxDomainEventInterceptor>());
});
```

#### Passo 4 — O Background Worker Publica no RabbitMQ

csharp

```csharp
// Identity.Infrastructure/BackgroundJobs/OutboxProcessorJob.cs
namespace Identity.Infrastructure.BackgroundJobs;

public sealed class OutboxProcessorJob : BackgroundService
{
    private readonly IServiceScopeFactory _scopeFactory;
    private readonly ILogger<OutboxProcessorJob> _logger;

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            await ProcessPendingMessagesAsync(stoppingToken);

            // Intervalo entre execuções — ajuste conforme necessidade
            await Task.Delay(TimeSpan.FromSeconds(10), stoppingToken);
        }
    }

    private async Task ProcessPendingMessagesAsync(CancellationToken ct)
    {
        // Cria escopo porque DbContext é Scoped, e BackgroundService é Singleton
        using var scope   = _scopeFactory.CreateScope();
        var       context = scope.ServiceProvider
            .GetRequiredService<ApplicationDbContext>();

        var pendingMessages = await context.Set<OutboxMessage>()
            .Where(m => m.ProcessedOn == null)
            .OrderBy(m => m.OccurredOn)
            .Take(20)                       // processa em lotes
            .ToListAsync(ct);

        foreach (var message in pendingMessages)
        {
            try
            {
                // Desserializa para o tipo correto
                var eventType = Type.GetType(message.Type)!;
                var domainEvent = JsonSerializer.Deserialize(message.Payload, eventType)
                    as IDomainEvent;

                // Publica via MediatR (handlers locais) ou via bus (outros serviços)
                // Para outros serviços: publique como Integration Event no RabbitMQ
                _logger.LogInformation(
                    "Publishing outbox message {MessageId} of type {Type}",
                    message.Id, message.Type);

                message.MarkAsProcessed();
            }
            catch (Exception ex)
            {
                _logger.LogError(ex,
                    "Failed to process outbox message {MessageId}", message.Id);

                message.MarkAsFailed(ex.Message);
            }
        }

        await context.SaveChangesAsync(ct);
    }
}
```

---

### O Fluxo Completo no Identity — Do Registro ao Evento

```
POST /register
  │
  ▼
RegisterUserHandler.Handle()
  │
  ▼
User.Create()
  ├── cria a entidade User
  └── AddDomainEvent(new UserRegisteredEvent(...))   ← domínio registra o fato
  │
  ▼
_uow.SaveChangesAsync()
  │
  ▼
OutboxDomainEventInterceptor (roda automaticamente antes do COMMIT)
  ├── coleta UserRegisteredEvent da entidade
  ├── cria OutboxMessage { type: "UserRegisteredEvent", payload: {...} }
  └── INSERT INTO outbox_messages (...)
  │
  COMMIT (atômico: User + OutboxMessage)
  │
  ▼
[10 segundos depois]
OutboxProcessorJob
  ├── SELECT * FROM outbox_messages WHERE processed_on IS NULL
  ├── publica UserRegisteredIntegrationEvent no RabbitMQ
  └── UPDATE outbox_messages SET processed_on = now()
  │
  ▼
email-sender-service consome a fila
  └── envia email de boas-vindas ao usuário
```

---

### Resumo das Camadas

```
Domain Event     → fato interno do domínio, gerado pela entidade
                   síncrono, mesma transação
                   ex: UserRegisteredEvent dentro de User.Create()

Outbox Pattern   → ponte entre o domínio e a mensageria
                   garante que o evento não se perde se a mensageria cair
                   converte DomainEvent → OutboxMessage na mesma transação

Integration Event → versão do evento publicada para outros serviços
                    assíncrona, via RabbitMQ ou Kafka
                    ex: UserRegisteredIntegrationEvent consumido pelo email-sender

Kafka            → transporte alternativo ao RabbitMQ para Integration Events
                   adiciona: replay, múltiplos consumers independentes, histórico
                   não muda o conceito de Domain Event nem de Outbox
```

Os quatro conceitos resolvem problemas diferentes e em camadas diferentes — entender onde cada um começa e termina é o que permite combinar todos corretamente.