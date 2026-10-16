Circuit Breaker é um design pattern que é usado para parar o processo de request-response quando um serviço não está funcionando.

![[Pasted image 20260918173427.png]]

Como no exemplo acima, imagine que um consumidor envia uma solicitação para receber dados de vários serviços. Porém, um dos serviços está indisponível devido a problemas técnicos. Nesse caso, enfrentaremos principalmente dois problemas:
- Primeiro, como o consumidor não tem conhecimento de que um determinado serviço esta indisponível (falhou), ele continua a enviar requisições normalmente para este serviço;
- Segundo, os recursos de rede se esgostarão, resultando em baixo desempenho e má experiência do usuário.

Tendo o uso do padrão Circuit Breaker conseguimos mitigar estes problemas, assim o consumir usará esse padrão para chamar serviço remoto por meio de um proxy e esse proxy se comportará com uma barreira.
Quando um número de falhas atinge um determinado limite, o "disjuntor" desarma por um período de tempo definido

Durante esse tempo de inatividade, todas as requisições ao servidor offline irão falhar. Ao fim desse período, o "disjuntor" permitirá que um número limitado de testes seja aprovado e se essas requisições forem bem sucedidas, o "disjuntor" retornará ao à operação normal. Se ocorrer uma falha, o período de inatividade começará novamente.

## Estados do Circuit Breaker
O padrão Circuit Breaker possui 3 estados:
- **Open**;
- **Closed**;
- **Half Open**;

O Circuit Breaker funciona normalmente quando está no estado **Closed**, mas quando as falhas excedem o limite predefinido, o "disjuntor" desarma, e assim ele vai para o estado **Open**.
Quando o Circuit Breaker está no estado de Open, as solicitações recebidas retornarão um erro sem qualquer tentativa de executar a operação real.

Após um determinado período, o Circuit Breaker entra no estado **Half Open**. Nesse estado, o Circuit Breaker permite a passagem de um número limitado de requisições de teste e, se as requisições forem bem-sucedidas, Circuit Breaker reinicia e retorna ao estado Closed, permitindo que o tráfego flua normalmente, porém se alguma solicitação falhar, o Circuit Breaker retorna ao estado Aberto até que ocorra outro tempo limite.

> [!example]
> Exemplo: O serviço A deve responder em até 200 ms.
> 
> 0 ms - 100 ms: intervalo de atraso esperado.  
>100 ms - 200 ms: arriscado.
>
>Se o tempo de resposta for superior a 200 ms, o serviço será interrompido.
>
>Ao criar um painel de monitoramento, você pode acompanhar as solicitações e seus tempos de resposta. Com base nisso, você pode definir um nível de limite.
  

O padrão Circuit Breaker funciona da seguinte forma: se o número de solicitações (ex: 75% das solicitações) atingir o limite superior (150–200 ms), isso significa que o serviço está falhando lentamente.

![[Pasted image 20260918180156.png]]

![[Pasted image 20260918180200.png]]

Se o número de ocorrências exceder 200 ms (o limite máximo definido para o serviço), o proxy identificará que este serviço não está mais respondendo.

O que acontece em seguida é que a solicitação de acesso ao serviço A será rejeitada. Isso interrompe a conexão entre seu proxy e o serviço A.

Agora seu proxy não irá para o serviço A. Isso significa que ele não ficará esperando.