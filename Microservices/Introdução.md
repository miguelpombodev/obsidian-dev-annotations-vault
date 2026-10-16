## O que são microserviços? 🤔
De forma resumida, microserviços são aplicações como qualquer outra aplicação, em aspecto de código, linguagem de programação, não temos diferenças, PORÉM, em aspecto de objetivos, os microserviços tem seus objetivos bem definidos e é aqui que diferencia bastante as aplicações em microserviços das demais, por exemplo se compararmos com os monolitos.

> [!info] **UM GRANDE PONTO DE ATENÇÃO ⚠️**
> Ter objetivo claro e definido **NÃO** significa que a aplicação precisa ser pequena, com 10 método, etc.

### Microserviços fazem partes de um ecossistema
Quando dizemos que "**microserviços fazem parte de ecossistemas**" significa que um microsserviço não estará isolado, trabalhando de forma única, ele fará parte de um contexto ou domínio com outros microsserviços.

### Microserviços são autônomos
Esta é uma das partes mais desafiadoras de trabalhar com microsserviços, pois sempre que implementamos, por exemplo, uma maneira de comunicação do microsserviço trabalhado com qualquer outra aplicação, seja outro microsserviço ou não,  e com isso acabamos gerando uma série de dependencias e estas dependencias fazem, de maneira geral, com que o nosso microsserviço não sobreviva sozinho. Por exemplo, se nosso microsserviço depende da resposta de uma aplicação B e esta aplicação B cai, o nosso microsserviço também irá cair. 

Com isso, devemos implementar tratativas às estas dependencias (como fallback, resiliencia, circuit break, etc) para que a premissa de independencia do microsserviço seja garantida, mesmo que ele trabalhe de forma mais limitada porém ainda consegue exercer sua função.

### Microsserviços se comunicam o tempo todo
Mas como assim? Pois é, mesmo que tenhamos falado sobre as dependencias dentro de microsserviço, microsserviços estão sempre a se falar, porém, as vezes, não de forma direta, por rede por exemplo (via HTTP), eles podem se comunicar usando mensagerias, brokers, event hubs, etc.

### Microsserviço VS Monolitos
Aqui veremos as algumas diferenças entre microsserviços e monolitos, porém não estamos dizendo se um é melhor que o outro:
- Microsserviços tem objetivos bem definidos enquanto monolitos tem toda a aplicação, ou seja, varios contextos dentro de um mesmo sistema;
- Microsserviços são agnosticos à linguagens de programação, enquanto monolitos são construídos sobre uma linguagem somente;
- Em caso de uma caída de serviço, em uma arquitetura de microsserviços, somente uma aplicação cairá enquanto o restante fica intacto, já em monolitos a aplicação cai por um todo;
- Com microsserviços conseguimos ter uma equipe por aplicação, enquanto em um monolito teremos todas as equipes trabalhando no mesmo sistema, e com isso podemos também ter mais domínio de cada equipe;
- Projetos monolitos ajudam muito a começar POCs, projetos simples, etc;

---
## Características de Microsserviços

_Fonte_: https://martinfowler.com/articles/microservices.html

### Componentization via service
Componente é uma unidade do sistema (leia-se **software**) que é independentemente substituível e atualizável.
Por exemplo, microsserviços utilizam bibliotecas, porém a principal forma de componentizar o próprio software é dividindo-o em serviços. Definimos bibliotecas como componentes que são vinculados a um programa e chamados por meio de chamadas de função em memória, enquanto **serviços** são componentes fora do processo que se comunicam com um mecanismo como uma requisição de serviço web.

### Organizado ao redor das aŕeas de negócios
Seguindo a Lei de Conway:

>[!quote]  Organizações que projetam sistemas (no sentido amplo do termo) são obrigadas a produzir projetos que são cópias das estruturas de comunicação dessas organizações

Com isso, ao trabalharmos com microsserviços, ficamos preocupados em desenvolvermos área de negócios da empresa invés das divisões de função, por exemplo um UX que precisa trabalhar com todos os designs dos produtos da empresa ou um dev backend que precisa cuidar de X sistemas
Aqui voltamos ao passo de ter domínio sobre o que desenvolvemos somente.

**Com divisão de função:**
![[Pasted image 20260916195157.png]]

Com divisão por áreas de negócio:
![[Pasted image 20260916195523.png]]

### Produtos e não projetos
Este ponto é bastante direto, é tratarmos os sistemas não como somente projetos e sim como produtos. Pois quando tratamos somente como projetos e temos em mente a prática de gestão de projetos, sabemos que todo projeto tem seu ciclo de vida e após esse ciclo de vida, o time que desenvolveu o "projeto" é desolvido e poucas pessoas ficam para manter o "projeto". Enquanto quando tratamos como produto, seguimos a premissa "quem desenvolve, é quem cuida", pois esta mesma pessoa tem domínio sobre regras, problemas, falhas comuns, idéias novas, etc.

### Smart endpoints e dumb pipes
Quando estamos construindo comunicação entre diferentes serviços, devemos manter o canal de comunicação, como mensagerias ou bus ouo até mesmo protocolos como HTTP, o mais simples possível, de modo que não tenhamos regra alguma dentro deste processos, enquanto os endpoints que lidarão de como a mensagem precisa chegar até o outro lado.


