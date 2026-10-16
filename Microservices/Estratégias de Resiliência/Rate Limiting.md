Rate Limiting é um ponto chave para o controle de acesso de clientes a uma certa API dentro de um dado espaço de tempo.
Quando este limite é aplicado, algumas requisições são permitidas chegar ao servidor enquanto outras são rejeitadas uma vez que o *threshold* é alcançado.

![[Pasted image 20260918121605.png]]

Podemos aplicar o rate limiting no client-side para ter um delay nas requisições antes delas serem enviadas ao servidor porém no server-side, o rate limiter é aplicado de forma centrazalida, tornando-o mais seguro, flexível e confiável para a proteção de seus recursos, que é o foco deste artigo

## Por que usar Rate Limiting?
Rate Limiting nos ajuda a protegar o sistema de *overloads*, abusos e exaustão de recursos:
- **Prevenção de abusos**: protege a API de requisições excessivas e maliciosas;
- **Melhora de performance**: mantém a responsividade;
- **Gerencia custos**: ajuda a prever e controlar despesas baseadas no uso;
- **Garante o uso justo**: todos os usuários têm acesso equitativo aos recursos.
- **Aprimoramento da segurança**: mitiga riscos como ataques de negação de serviço (DoS) restringindo a taxa de solicitações.

![[Pasted image 20260918122515.png]]

## Algoritmos de Rate Limiting
Diveros algoritmos são largamente usados para limitar a quantidade de requisições em um sistema, e também podemos aplicar nossa própria estratégias de rate limiting , porém isto acaba não sendo muito comum pois já temos estratégias que foram altamente testadas e estressadas para não termos problemas

Portanto, faremos um overview dos algoritmos mais famosos:

### Concurrency Limit
Esta é a forma mais simples de aplicar um rate limiting, aqui somente limitamos quantas requisições podemos processar ao mesmo tempo.

Exemplo: *Permita somente 2 requisições sendo executdas*
![[Pasted image 20260918123057.png]]

Este algoritmo é simples, previne o acúmulo de requisições, a exaustão de thread e funciona bem mesmo quando o número de requisições é imprevisível

Porém, esta estratégia tem diversos *problemas*:
- Requisições de longa duração reduzem a taxa de transferência geral (throughput);
- Podem causar injustiças (clientes lentos bloqueiam os mais rápidos);
- Não é adequado como única proteção contra DDoS ou picos de tráfego.

Como aplicar em .NET:
```csharp
builder.Services.AddRateLimiter(options =>  
{  
options.GlobalLimiter = PartitionedRateLimiter.Create<HttpContext, string>(httpContext =>  
{  
string partitionKey = "global";  
  
// 👇 choose rate limiting algorithm  
return RateLimitPartition.GetConcurrencyLimiter(  
partitionKey,  
partition => new ConcurrencyLimiterOptions  
{  
PermitLimit = 2, // allow 2 concurrent requests  
});  
});  
});
```
### Janela Fixa
Um limite de janela fixa aplica limites de requisições durante um período de tempo determinado. Quando a janela expira, o limite é redefinido e a contagem recomeça do início.

Exemplo: *Permita 5 requisições a cada 1 minuto*
![[Pasted image 20260918134851.png]]

Embora esta seja uma abordagem popular, já que fornece um limite de solicitações previsível por intervalo de tempo e previne requisições excessivas durante um período determinado, você pode reparar rapidamente alguns problemas aqui:
- Não funciona bem com tráfego uniformemente espaçado 
	- Permite picos de no início da janela (todas as requisições pode vir uma vez só);
	- Não lida com picos de solicitações no final da janela.
- Não previne a sobrecarga do sistema durante uma sequência de requisições.

Exemplo em .NET:
```csharp
builder.Services.AddRateLimiter(options =>  
{  
options.GlobalLimiter = PartitionedRateLimiter.Create<HttpContext, string>(httpContext =>  
{  
string partitionKey = "global";  
  
// 👇 choose rate limiting algorithm  
return RateLimitPartition.GetFixedWindowLimiter(  
partitionKey,  
partition => new FixedWindowRateLimiterOptions  
{  
PermitLimit = 5, // 5 requests  
Window = TimeSpan.FromMinutes(1), // per 1 minute  
});  
});  
});
```

Use a Janela Fixa quando deseja impor um limite de requisições previsível durante um período de tempo definido:
- APIs públicas com quotas claras (per-day, per-month);
- Impor uso justo por cliente ou por **API key**;
- Quando simplicidade e monitoramento fácil são mais importantes do que distribuição de solicitações perfeitamente tranquila

### Janela Deslizante
Este algoritmo é uma alternativa ao anterior, ele divide a janela em segmentos para limites mais detalhados.
- Requisições dentro da janela são limitados a um número determinado na propriedade "PermitLimit";
- A janela é dividida em segmentos determinados pela propriedade "SegmentsPerWindow" (exemplo, 3 segmentos);
- A janela avança um segmento de cada vez, com base no intervalo entre segmentos (duração da janela / segmentos por janela).
- Quando um segmento expira, ele libera o limite utilizado, restaurando a capacidade da janela.

Examplo: _“5 requisições para cada 1 minuto , que são reavaliadas a cada 20 segundos”._
![[Pasted image 20260918152254.png]]

Isso proporciona uma distribuição de requisições mais suave, com um controle de taxa mais preciso e um tratamento mais justo para solicitações de clientes com espaçamento uniforme.

Exemplo em .NET:
```csharp
builder.Services.AddRateLimiter(options =>  
{  
options.GlobalLimiter = PartitionedRateLimiter.Create<HttpContext, string>(httpContext =>  
{  
string partitionKey = "";  
  
// 👇 choose rate limiting algorithm  
return RateLimitPartition.GetSlidingWindowLimiter(  
partitionKey,  
partition => new SlidingWindowRateLimiterOptions  
{  
PermitLimit = 5, // 5 requests per window  
Window = TimeSpan.FromMinutes(1), // window time equals 1 minute  
SegmentsPerWindow = 3, // divides window into 3 smaller segments, 20 seconds each  
});  
});  
});
```

Este algoritmo funciona muito bem quando queremos impor um limite de requisições mais suave ao longo do tempo, evitando um "burst' que ocorre em Fixed Windows limites

### Token Bucket
Neste algoritmo, cada requisição consome um token, e estes token são recarregados a uma taxa fixa. Se caso ficarmos  sem tokens disponíveis, todas as requisições são rejeitadas.
- Um bucket tem uma capacidade máxima de tokens disponíveis (exemplo, 10 tokens);
- Novos  tokens são adicionados ao bucket a uma taxa fixa (exemplos, 2 tokens a cada 5 segundos);
- Cada requisição consome um token;
- Se ainda há tokens disponíveis, a requisição é permitida e o token é removido do bucket, caso contrario, a requisição é rejeitada

Exemplo: *"10 requisições no máximo, com recarga a cada 5 segundos"*
![[Pasted image 20260918153316.png]]

A abordagem de **token bucket** é popular porque:
- permite picos pequenos  sem exceder a  taxa de longo prazo.
- suaviza o tráfego mais efetivamente;
- flexível para ambas rate limiting quanto para o burst handling

Exemplo em .NET:
```csharp
builder.Services.AddRateLimiter(options =>  
{  
options.GlobalLimiter = PartitionedRateLimiter.Create<HttpContext, string>(httpContext =>  
{  
var partitionKey = "global";  
  
// 👇 choose rate limiting algorithm  
return RateLimitPartition.GetTokenBucketLimiter(  
partitionKey,  
partition => new TokenBucketRateLimiterOptions  
{  
TokenLimit = 10,  
TokensPerPeriod = 2,  
ReplenishmentPeriod = TimeSpan.FromSeconds(5),  
});  
});  
}
```

Útil quando queremos controlar a taxa de requisições a longo-termo enquanto ainda permitimos pequenos bursts no tráfego:
- APIs públicas onde picos ocasionais de curto prazo são aceitáveis, mas a taxa geral deve ser limitada.
- Endpoints onde queremos tanto justiça quanto flexibilidade para os clientes;
- Situações onde o throughput médio importa mais do que requisições instantaneas;

### Leaky Bucket
Nessa estratégia, o tráfego é suavizado para um fluxo constante. As requisições são adicionadas a uma fila (o "bucket") e processadas a uma taxa constante. Se o bucket estiver cheio, novas solicitações são rejeitadas.

Exemplo: *"Processe 1 requisição a cada segundo, em uma fila máxima de 100"*
![[Pasted image 20260918155539.png]]

O leaky bucket é sobre suavidade e não somente limite

Embora previna completamente os picos de tráfego, pode aumentar a latência devido ao enfileiramento.

Esta estratégia não tem diretamente suporte em .NET, porém conseguimos chegar o mais perto com o setup aproximado:
```csharp
builder.Services.AddRateLimiter(options =>  
{  
options.GlobalLimiter = PartitionedRateLimiter.Create<HttpContext, string>(httpContext =>  
{  
var partitionKey = "global";  
  
// 👇 choose rate limiting algorithm  
return RateLimitPartition.GetTokenBucketLimiter(  
partitionKey,  
partition => new TokenBucketRateLimiterOptions  
{  
TokenLimit = 1, // 👈  
TokensPerPeriod = 1,  
ReplenishmentPeriod = TimeSpan.FromSeconds(1),  
QueueLimit = 100, // 👈 bucket capacity  
});  
});  
}
```

## Request Queuing
Se você leu este artigo com cuidado, você talvez notou que o algoritmo Leaky Bucket usa um parametro chamado "QueueLimit". Na verdade, esse parâmetro é suportado por todos os demais algortimos
```csharp
builder.Services.AddRateLimiter(options =>  
{  
options.GlobalLimiter = PartitionedRateLimiter.Create<HttpContext, string>(httpContext =>  
{  
string partitionKey = "global";  
  
return RateLimitPartition.GetConcurrencyLimiter(  
partitionKey,  
partition => new ConcurrencyLimiterOptions  
{  
. . .  
QueueLimit = 5, // 👈  
QueueProcessingOrder = QueueProcessingOrder.OldestFirst, // 👈  
. . .  
});  
});  
});
```

Se temos uma fila habilitada, as requisições que excederem o limite serão enfileiradas ao invés de serem rejeitadas, esperando por uma permição disponível. Isso não somente permite limitar a taxa de requisições (request rate),  como também ter um fluxo mais estável.
![[Pasted image 20260918160718.png]]

Talvez possa te lembrar do padrão [Queue-Based Load Leveling](https://medium.com/@iamprovidence/queue-based-load-leveling-pattern-8aa7c31d0770) onde o trabalho recebido é armazenado em buffer e processado em um taxa controlada para proteger recursos subsequentes (*downstream resources*).

## Partições
Outra coisa que talvez você possa ter notado é que há o parametro "*partitionKey*" que não foi explicado ainda

```csharp
builder.Services.AddRateLimiter(options =>  
{  
options.GlobalLimiter = PartitionedRateLimiter.Create<HttpContext, string>(httpContext =>  
{  
string partitionKey = "global"; // 👈  
  
return RateLimitPartition.GetConcurrencyLimiter(  
partitionKey,  
partition => new ConcurrencyLimiterOptions());  
});  
});
```

Agora, o mesmo algoritmo de rate limiting é aplicado globalmente para todos os usuários, no entanto, você possa querer somente aplicar esta configuração para um grupo de usuários.
E i sso pode ser feito desta forma:
- Um usuário não pode exaurir o rate limit para todo mundo;
- Diferentes limites podem ser aplicados para cada usuário, cliente ou recurso;
- Isso permite uma proteção mais forte contra um abuso por alvo

As partições dividem o tráfeo vindo assim cada grupo podem ser trackeado e limitado independentemente
![[Pasted image 20260918161532.png]]

É comum usar o ***endereço IP, API key, user identity, tenant Id*** ou ***caminho do endpoint*** como ***partition key***

```csharp
builder.Services.AddRateLimiter(options =>  
{  
options.GlobalLimiter = PartitionedRateLimiter.Create<HttpContext, IPAddress>(httpContext =>  
{  
// 👇  
IPAddress partitionKey = context.Connection.RemoteIpAddress;  
// string apiKey = httpContext.Request.Headers["API-Key"].ToString() ?? "no-key";  
// string identity = httpContext.User.Identity?.Name ?? "anonymous";  
// string tenantId = httpContext.RequestServices.GetRequiredService<ICurrentTenantContext>().GetTenantId();  
// string path = httpContext.Request.Path.ToString().Containts("/api") ? "public-api": "private-api";  
  
return RateLimitPartition.GetConcurrencyLimiter(  
partitionKey,  
partition => new ConcurrencyLimiterOptions());  
});  
});
```
