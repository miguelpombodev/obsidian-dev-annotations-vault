# 👬 Database Replication

Essa estratégia também é conhecida como “Master/Slave”, basicamente é quando separamos nosso banco de dados em dois tipos, o de escrita e o de leitura, onde o banco principal será o de escrita e o de nome “Master”, enquanto para leitura faremos réplicas do banco somente para leitura, como podemos ver na imagem abaixo:

![[Database/Attachments/image.png]]

Na grande maioria, usamos esse desenho, no qual temos o somente o um banco de escrita pois costumamos ter mais volume de leitura do que de escrita em nossas bases.

## Configuração do Backend

Para fazer configuração do backend quando usamos essa estratégia é bem simples, quase todos os ORMs fazem esse mapeamento de forma fácil e rápida.

Aqui temos algumas demonstrações de como é no .NET com o EF Core:

[Enhance Your .NET Core Application with Master-Slave Database Replication](https://medium.com/@shoeb.hasan/mastering-master-slave-replication-enhance-your-net-6536809a3862)

[Using Separate Read/Write Models with EF Core and CQRS](https://www.youtube.com/watch?v=iKDITShiZy4)

# 🔀 Database Sharding