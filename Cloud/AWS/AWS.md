# Artigos:

[[EC2]]
[[Bucket S3]]
[[IAM]]
# Sobre a AWS

## **Como o usuário consegue acessar os serviços da AWS (AWS Console ou AWS Services)?**

Basicamente, o usuário precisa criar uma conta, e existem dois tipos de contas da AWS, sendo eles a Personal Account ou a Business Account. Se alguém usa a Personal Account, provavelmente deve estar estudando para tirar uma certificação ou testando algum serviço, se caso essa pessoa tenha uma conta Business ou trabalha para uma empresa que já tem uma conta AWS, o pessoal responsável irão criar um usuário, e a senha dessa conta deverá ser atualizada no primeiro acesso à conta, é claro que esse usuário e senha serão atrelados a algumas permissões, como quais serviços essa pessoa pode usar, se ela pode acessar a console ou não, quais menus estarão disponíveis, se pode acessar a parte de billing, isso vai depender muito da posição e departamento que essa pessoa trabalha. Tendo uma conta Business a AWS pode gerar nota fiscal para a empresa em que trabalhamos.
Essa conta pessoal tanto Business quanto Personal da AWS, ela pode ser gratuita inicialmente, existe um sistema que a AWS criou chamado Free Tier, que dá acesso gratuito a alguns serviços da AWS para que possamos testar e rodar alguns projetos pequenos e verificarmos se o serviços da AWS são o que precisamos.

## **Sobre o Free Tier**

Desde o começo a AWS disponibilizou alguns serviços gratuitamente para que possamos testar, é claro que é disponibilizado os serviços mais básicos, por exemplo, o EC2 onde podemos criar máquinas virtuais, onde podemos configurar memória, processamento, rede, porém quanto de cada qual? É claro que a AWS quer que testemos esse serviços, então ele disponibiliza uma memória baixa, um processamento baixo e uma rede com trafego bem baixo para que possamos testar, assim podemos reparar que o Free Tier é para ambientes de teste e de desenvolvimento.

## Quais serviços estão dentro do Free Tier?

No link “[http://aws.amazon.com/free](http://aws.amazon.com/free)” podemos ver todos serviços disponíveis para o Free Tier e qual a limitação temos em cada serviço.

# Cloud Computing

Antes de termos os serviços em nuvem ou cloud computing, separávamos um espaço dentro da empresa para a famosa “sala de servidor”, que basicamente era onde os servidores dedicados da empresa ficavam, por exemplo, parte de arquivos, autenticação do usuários, onde se fazia a liberação dos IPs das máquinas, segurança, onde tinhamos a intranet muitas das vezes. Porém com isso, se trazia algumas desvantagens como energia alta, segurança e troca de peças, sejam eles que queimaram ou upgrade, e além de tudo isso, o servidor ficava na responsabilidade, algumas das vezes, de uma pessoa só.

Agora depois do invento da Cloud Computing, essas “server rooms” deixaram de existir, e quase tudo foi migrado para os serviços em nuvem, que basicamente trabalha com a conexão de varios DataCenters.

![[Cloud/AWS/Attachments/image.png]]

imagem: Data Center da AWS

Agora que ja sabemos sobre Cloud Computing, iremos ver agora as vantagens de migrar para cloud. Vamos imaginar uma empresa que ainda não migrou para cloud (claro que existe kkk), o que essa empresa esta perdendo:

1. **Custo reduzido** → A computação em nuvem vai ajudar a empresa a reduzir os custos de capital gastos em hardware, software, instalação, implementação e manutenção dos Data Centers. Tudo vai estar dentro da e gerenciado também pela provedora Cloud, nesse caso a AWS;
2. **Escala Global** → Aqui teremos a habilidade de implementar os serviços, de forma horizontal, para atender a demanda e pagar menos. Se caso tivermos clientes nos EUA, na UK e aqui no BR, conseguimos implementar facilmente o nossos serviços mais próximos dos clientes, sem dar aquela sensação de “lag”, ou seja, alta latência.
3. **Perfomance** → Ganhamos performance pelo uso de uma rede mundial do Data Centers, e isso oferece algumas vantagens sobre o Data Center em ponto único, como dito anteriormente.
4. **Velocidade e Agilidade** → Conseguimos acessar os serviços com um click de distância e também conseguimos iniciar serviços em minutos, o que de forma tradicional, iríamos gastar horas, abrindo ticket, conversando com um “Sys Admin”.
5. **Aumento de Produtividade, Segurança e Flexibilidade** → Podemos aumentar cada tópico desse pois, como visto no ponto anterior, invés de passarmos por todo o processo antiquado de compra, vista com o vendedor de servidor, etc, conseguimos deixar com que o time de desenvolvimento resolva.

# Tipos de Clouds

## IAAS

Esse tipo siginifica “Infrastructure As A Service¨ ou “Infraestrutura Como Serviço”, que basicamente é alugar serviços de TI para o seu cliente, e esses serviços podem ser servidores, VMs, Storage.

## PAAS

Esse aqui é “Platform As A Service”, que é alugarmos um SO, um sistema de banco de dados, hospedagem de sites. Nesse caso, estamos falando de coisas físicas mesmo e não somente os serviços lógicos.

## SAAS

“Software As A Service” é o nível mais alto em computação em nuvem, pois os usuários adquirem acesso à uma aplicação, como por exemplo uma aplicação de email como o Gmail, ou o Dropbox, Driver, 

Agora que vimos os 3 segmentos de serviços em nuvem, precisamos ver onde podemos instalar um desses segmentos, pois onde iremos instalá-los também influencia. Podemos ter:

- **Nuvem Pública** → Elas são públicas onde todo mundo pode entrar e criar uma conta e ter seus recursos dentro daquela conta, e não que todos irão ver seus arquivos e serviços, por exemplo AWS, Azure e GCP;
- **Nuvem Privada** → Bancos e Governos preferem mais a nuvem privada, mesmo podendo usar a AWS, eles podem usar um servidor dentro de um Cloud Provider dedicados à instituição, porém irão pagar MUITO mais. Esse tipo pode estar dentro da AWS, ou fisicamente, depende do contexto;
- **Nuvem Híbrida** → Aqui é o misto das duas outros nuvens. Imaginemos que temos 3 servidores em nossa topologia, e os 2 primeiros não armazenam nenhum dado sensitível, consequemente podemos migrar eles para um hardware compartilhado, porém no terceiro mantemos em um servidor exclusivo, e se estivermos operando em uma Cloud Provider, esses 3 servidores conseguem conversar tranquilamente;