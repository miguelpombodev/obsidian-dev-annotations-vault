O SQL, Structured Query Language ou Linguagem de Pequisa Estruturada, é uma linguagem de pesquisa de forma declarativa e padronizada, para bancos de dados relacionais. Ela foi criada na década de 90 pela IBM e até hoje muito da sua estrutura e de suas características principais se mantêm, como por exemplo:

- **DML** → Linguagem de Manipulação de Dados: INSERT, UPDATE e DELETE;
- **DDL** → Linguagem de Definição de Dados: CREATE e DROP;
- **DCL** → Linguagem de Controle de Dados:
    - GRANT: Autoriza o usuário a executar comandos definidos
    - REVOKE: Remove ou restringe a capacidade de um usuário
- **DQL** → Linguagem de Consulta de Dados: SELECT;
- **DTL** → Linguagem de Transação de Dados: BEGIN WORK (ou BEGIN TRANSACTION), COMMIT, ROLLBACK

Todos essas linguagens acima, irei falar melhor delas no decorrer do artigo.

## Principais Sistemas de Gerenciamento de Banco de Dados

**SGBD** é um conjunto de software utilizado para o gerenciamento de uma base de dados, responsáveis por controlar, acessar, organizar e proteger as informações de uma aplicação, tendo como principal objetivo gerenciar as bases de dados utilizadas por aplicações clientes e remover esta responsabilidade das mesmas.

Abaixo, está uma listagem dos principais SGBDs**,** que são usado no mercado nos dias de hoje:

- Oracle - [https://www.oracle.com/database/](https://www.oracle.com/database/) → **PAGO;**
- PostgreSQL - [https://www.postgresql.org/](https://www.postgresql.org/);
- Microsoft SQL Server - [https://www.microsoft.com/sql-server](https://www.microsoft.com/sql-server) → **PAGO**;
- MySQL - [https://www.mysql.com/](https://www.mysql.com/);
- MariaDB (fork do MySQL)- [https://mariadb.org/](https://mariadb.org/);
- IBM DB2 - [https://www.ibm.com/db2](https://www.ibm.com/db2) → **PAGO**;

# DQL

DQL ou Data Query Language (Linguagem de Consulta de Dados) é o conjunto de comandos (statements) que usados para executar consultas dos dados inseridos nas tabelas de um banco de dados. O comando mais utilizado para executar essas consultas é o comando **SELECT**.

## SELECT

O comando SELECT é o responsável para mostrar o dados de uma tabela, sejam eles específicados ou geral, variando pela necessidade da consulta

```sql
--Consulta geral, ou seja, iremos mostrar todos as colunas dessa tabela
SELECT * FROM Users

--Consulta especificada, ou seja, iremos mostrar somente as colunas que necessitamos
SELECT Name, Email, Date_Birth FROM Users
```

### WHERE

A cláusula **WHERE** é usada para filtrar dados dentro de uma consulta.

```sql
-- Essa consulta está filtrando somente as linhas onde o nome do usuário seja 'Jonh Doe'
SELECT * FROM User WHERE Name = 'Jonh Doe'
```

A cláusula **WHERE** pode concatenar, ou seja, adicionar outros campos a serem filtrados, sendo possível pelo uso dos **operadores lógicos**, os quais falaremos sobre eles logo abaixo.

```sql
-- Na consulta abaixo, adicionamos a coluna "Gender" para aumentar nossa filtragem das linhas da tabela
SELECT * FROM User WHERE Name = 'Jonh Doe' AND Gender = 'Male'
```

### Operadores Lógicos