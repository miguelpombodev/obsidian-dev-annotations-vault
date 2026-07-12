São soluções generalistas para problemas recorrentes durante o desenvolvimento de um software, ou seja, são modelos de soluções abstratas de alto nível para resolver problemas comuns. Onde cada padrão vai descrever um problema que ocorre com frequência e vai descrever o cerne da solução de forma a poder **reusar** a solução muitas vezes em situações diferentes.

> 🚨 Os Design Patterns não são um framework, não são uma linguagem, e não é um código pronto…
  (Também não são a panaceia universal, nem ‘a bala de prata’)


> ⚠️ Muitos devs, após lerem um livro de padrões, caem na armadilha de tentar aplicar padrões para tudo o que estão fazendo, conseguindo exatamente, o oposto do que os padrões são propostos, que é fazer as coisas de maneira simples.



## Quando tudo começou?

Os padrões de projeto começaram a ser reconhecidos mais formalmente pelo inicios do anos 90, resultado de estudo de vários especialista, dentre eles, o mais conhecido foi **Erich Gamma**, e como fruto saiu a publicação do famosos livro “_Design Patterns: Elements of Reusable Object-Oriented Software_”.

![image.png](Design%20Patterns/image.png)

Este livro, também conhecido como **Gang Of Four (*GOF*)**, teve um impacto profundo sobre aqueles que buscam enteder sobre usar os padrões de projeto. Neste livro, são descritos 23 padrões de projeto de ocorrência mais comum, com comentários de quando e como podemos aplicá-los.

Os autores dividiram os padrões em três tipos:
- **Criacionais** (Creational);
- **Estruturais** (Structural);
- **Comportamentais** (Behavioral)

Vamos aos seguintes benefícios em usar **Design Patterns**:
- Fornecem soluções que já foram testadas e aprovadas, sendo assim, não precisamos fazer testes desde o início;
- Tornam o sistema mais fácil de entender e manter;
- Facilitam o desenvolvimento de módulos coesos;
- A comunicação entre os participantes do projeto fica mais eficiente

## Bom, então como e quando usar?

Bem primeiramente, precisamos ter bom senso em analisar o problema que precisamos resolver, depois faça sua parte bem feita, como desenvolvedor, usando as boas práticas de desenvolvimento, procurando sempre mantendo um código limpo. Depois implemente a solução e ver se ela funciona de como adequado, resolvendo o problema. Após, verifique se ela pode ser otimizada e aí então podemos identificar e utilizar o padrão de projeto que se ajusta ao caso. Sendo assim, veremos que o resultado final deve melhorar as deficiências verificadas no projeto

Resumindo os passos são:

- Analise;
- Implemente;
- Teste;
- Verifique;
- Ao otimizar, pode-se procurar e utilizar o padrão de projeto necessário
- Finalmente, ver se o resultado foi melhorado (ou não…)

## Fundamentos

A razão fundamental para o uso do padrões de projeto é manter os módulos separados e com responsabilidades definidas e únicas, e evitar que eles tenham que conhecer as suas implementações. Usar os padrões de projeto nos ajuda a evitar a reinvenção da roda e permite que descrevamos a abordagem de programação que foi usada, de forma resumida e de forma que outros devs possam entender. Sendo assim, existe uma série de estratégias que nós como desenvolvedores usamos para conseguir essa separação de responsabilidade dos módulos de um projeto. Sendo elas as seguinte:

- Programar para uma interface e não para uma implementação;
- Favorecer a composição sobre herança;
- Usar os princípios **S.O.L.I.D**;
- **Evitar** o forte acomplamento entre as classes;
- **Manter** uma alta coesão entre as classe

---

# Padrões Criacionais

## Definição:

Esses padrões abstraem e/ou adiam o processo de criação dos objetos, e fornecem mecanismos de criação de objetos que aumentam a flexibilidade e a reutilização do código existente. Os padrões criacionais fornecem uma interface para criar objetos em uma classe-pai, mas permitem que as subclasses ou classes-filhas alterem o tipo de objetos que serão criados.

## Tipos de Padrões Criacionais:

- Abstract Factory;
- Builder;
- Factory Method;
- Prototype;
- Singleton;

## Conceito de Factory

Um **Factory** é um objeto usado para criar outros objetos, e na sua implementação geralmente temos uma classe com um método usado para criar e retornar diversos tipos de objetos. Dessa forma, usando o padrão **Factory** criamos objetos sem expor a lógica de criação ao cliente. Essa padrão tem diferentes implementações, tais como:

- **Factory Method;**
- **Abstract Factory**

Porém o primeiro padrão não está catalogado dentre os 5 padrões criacionais **GOF**, e ele é conhecido como **Simple Factory**.

Um Simple Factory é um objeto que é usado para criar instâncias de um objeto para o cliente, permitindo usar interfaces para criar objetos sem expor a lógica da criação do objeto ao cliente.

Como no exemplo abaixo:

![[DP_image.png]]

A seguir vemos o mesmo exemplo, porém usando código C#, primeiro veremos sem usar o Simple Factory, e logo após veremos usando o padrão:

![[DP_Pizzaria]]

## Factory Method

### Objetivo

A intenção e objetivo do **Factory Method** é definir uma interface para criar objetos mas permitir que as subclasses decidam qual classe instanciar, ou seja, permitir que o cliente não precise saber quais tipos de objetos serão criados. Para isso, se usa a herança e depende e depende-se de uma subclase para tratar com a instanciação do objeto desejado e também se cria uma instância de várias classes derivadas dependendo do cenário.

Sendo assim o padrão **Factory Method** tem o mesmo objetivo que o padrão Simple Factory e os demais padrões criacioinais que é:

>☝🏽**Ser usado para criar objetos sem expor a lógica de criação do objeto ao cliente**


O seguinte diagrama UML explica se forma mais ilustrada:

![[DP_image 2.png]]

### Benefícios do padrão

- Nesse padrão, as subclasse Factory possuem controle tottal sobre o processo de criação dos objetos, podendo retornar o tipo desejado ou uma subclasse;
- Remoção do forte acoplamento entre as classes;
- Encapsulamento do código que varia (instanciação da classe) em único local;
- Facilita a manutenção do código

A seguir, temos um exemplo de código usando o padrão:

[FactoryMethod.zip](Design%20Patterns/FactoryMethod.zip)

![[DP_image 3.png]]

Acima segue o diagrama UML do projeto mostrado

## Abstract Factory

### Objetivo

Prover uma interface para criação de famílias de objetos relacionados sem especificar sua classe concreta, ou seja, objetos diferentes porém da mesma família.

Fornece uma forma de encapsular um grupo de fábricas individuais que possuem um tema em comum sem especificar suas classes concretas. Esse padrão é considerado uma fábrica de fábricas de objetos. E ele usa a composição de objetos para desacoplar o cliente das implementações específicas.

Segue o diagrama UML para ilustração:

![[DP_image 4.png]]

### Quando usar?

- O sistema deve ser configurado para funcionar com famílias de produtos;
- Você deseja criar um conjunto de objetos relacionados ou dependentes que devem ser usados juntos;
- O sistema precisar ser independente de como seus produtos são criados, compostos ou representados;
- Você quer fornecer uma biblioteca de classes e quer revelar somente suas interfaces, não suas implementações.
- As classes concretas devem ser desacopladas dos clientes;

### Vantagens e Desvantagens

- Isola as classes concretas dos clientes
- Facilita a troca de família de produtos
- Promove a consistência de produtos (não mistura objetos de família diferentes)
- Dificulta a criação de novos produtos ligeiramente diferentes (pois temos que modificar a fabrica abstrata e todas as fabricas concretas)
- Se tivermos muitas famílias de produtos teremos excesso de classes das fábricas concretas