### O que é um usuário?
Uma conta de usuário fornece limites de segurança entre diferentes pessoas e programas que podem executar os comandos.
Então, os usuários Linux tem nomes para identificá-los para os usuários humanos e para facilitar o nosso trabalho.
De forma interna, o sistema distingue as contas de usuários pelo número de ID do usuário ou o UUID que atribuído aos usuários.
Na maioria dos casos, se a conta de usuário é de um humano, o sistema pede uma senha para provar que é ele mesmo que esta se autenticando no sistema.
Com isso, podemos afirmar que as contas de usuários são essenciais para a segurança do sistema Linux.

Cada processo (ou programa) que executamos é executado com um usuário específico, mesmo que este usuário não tenha permissões de login, o que é muito comum, exemplificando **quando baixamos um programa de banco de dados, o Linux atribuí para este processo de download um usuário para executar a tarefa**.

Também temos esta separação de contas com arquivos, ou seja, cada arquivo tem seu proprietário, para assim termos um controle de acesso ao arquivo em espécie.

Esses são os tipos de usuários que temos:
- **Super User ou Administrador**-> Este tipo administra todo o sistema, o que seria o **user root**;
- **Usuário comum** -> Este tipo é da maioria dos usuários em um sistema Linux;
- **Usuários de sistema** -> Este tipo é designado para fins do próprio sistema, este usuários não tem login
#### Comandos
Para vermos os arquivos que um usuário é proprietário:
```bash
# -l -> informa tudo sobre o arquivo
# -h -> formata para um formato entendível para humanos
ls -lh
```

Para vermos os processos que estão rodando e seus usuários:
```bash
ps -au
```
Exemplo de display do comando:
![[Pasted image 20261012195944.png]]

Para vermos todos os usuários de sistemas, ou seja, usuários especiais:
```bash
cat /etc/passwd
```

Como aparece:
![[Pasted image 20261012200123.png]]

### Grupos de usuários
Os grupos de usuários são coleções de usuários que compartilhar algum acesso, seja este acesso um arquivo ou um recurso do sistema.
Estes grupos podem ser usados para conceder acesso a um usuário específico ou a um conjunto de usuários e assim como os usuários, os grupos de também tem nomes para facilitar o nosso reconhecimento e ele também tem um ID único, o GID.
Quando criamos um usuários, automaticamente criamos um grupo também, que leva o mesmo nome do usuário.
Por padrão, os sistemas Linux, para armazenar os grupos, usam o arquivo:
```bash
cd /etc/group
```


### Criação de usuários
Para criar um usuário, usamos o comando:
```bash
# -c -> Seta um codenome para aquele usuário.
# -g -> Nome do grupo primário que o usuário vai pertencer
# -G -> Nome do grupo secundário que o usuário vai pertencer
# -s -> Tipo de shell que ele pode usar
useradd -c "Jonh Doe" -g users -G users -s /bin/bash <username>
```

Para criar um senha para o usuário:
```bash
passwd <username>
```

Display do comando:
![[Pasted image 20261012201749.png]]

Para entrarmos no usuário root:
```sh
sudo su
```

Para sabermos em qual usuário estamos:
```
whoami
```

Display do comando:
![[Pasted image 20261012203937.png]]

