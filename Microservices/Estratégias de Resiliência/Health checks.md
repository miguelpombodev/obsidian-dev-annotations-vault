Health checks são serviços que verificam a saúde do serviço, seja o liveness ou readiness, porém não verificamos somente a saúde exclusivamente do serivço mas também dos outros serviços que este depende, como bancos de dados, cache, workers, etc.

Sistemas que não estão saudáveis possuem chance de se recuperar caso o tráfego pare de ser redirecionado para eles, este é um mecanismo que chamamos de "**self-healing**".

Para termos um healthcheck de qualidade, ou seja, confiável, precisamos ter em mente que existem 2 tipos de health checks, um passivo e um ativo.

O HC passivo funciona de forma que só conseguimos ter o status de saúde ao acessar o endpoint de, por exemplo,  ***/health***. 
Já o HC ativo, é quando o próprio sistema fica a executar o seu health check e checando o status ou utilizaremos de outros softwares, como Nginx ou Kubernetes, para fazer este trabalho.

> [!warning] Apesar deste assunto ser muito importante, muitos desenvolvedores e engenheiros acabam deixando de lado. Porém quando estamos lidando com sistemas distribuídos, resiliência e alta disponibilidade, os Health cecks são mais do que essenciais de existir em arquitetura deste tipo

*Conceitos de HC do Kubernetes*:
![[Pasted image 20260918115515.png]]
