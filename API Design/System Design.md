# System Design

# ☝🏽Ponto único de falha

De forma resumida, é o serviço, dentro da arquitetura da nossa aplicação, que se ele tiver algum problema, quebrar, ter um erro, ou seja, ficar inacessível, nossa aplicação não estará disponível.

Quando estamos desenhando um System Design, devemos ficar bem alertas em possíveis pontos assim.

# Cache Server (Cache-Aside Pattern)

Quando usamos servidores de cache como o **Redis**, também podemos escalá-lo, sendo isso de forma vertical ou de forma horizontal. Porém, se estivermos falando de aplicações de alta disponibilidade, talvez escalar vertical será a melhor, tendo em vista os princípios de [**failover**](http://www.cloudflare.com/pt-br/learning/performance/what-is-server-failover/) e **redundância**.

E também temos alguns serviços que já nos ajudam a provisionar isso, como o ElasticCache da AWS, serviços nos quais conseguimos criar um cluster de vários servidores de cache, onde basicamente teremos somente o endereço de um servidor e o load balancer interno desse cluster vai cuidar do restante.

# 🆙 Auto-Scale

Basicamente, é quando configuramos um auto provisionamento de instâncias baseado na quantidade de requisições que estão sendo enviadas para o servidor, no qual o load balancer verifica essa quantidade. Chamamos isso de elasticidade das aplicações, e com isso só pagamos aquilo que usamos, caso não usarmos as instâncias ociosas são deletadas.

Na AWS, temos um cara chamado de Auto Scaling Group, onde podemos “combar” o ELB, com o AutoScale e com as EC2.

![[API Design/Attachments/image.png]]

# 🗺️Multi-AZ