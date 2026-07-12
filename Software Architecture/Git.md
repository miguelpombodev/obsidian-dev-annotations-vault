**git checkout {nome da branch}** —> Para trocar para a branch especificada

**git branch -d  <{nome da branch}>** —> Para deletar uma branch local

**git push origin —delete <{nome da branch}>** —> Para delete uma branch remota

**git merge <{branch}> —>Estando dentro da master, esse comando da merge com uma branch específica** 

# Modelo Mental Base

O Git não trabalha com “**arquivos versionados**”.

Ele trabalha com commits encadeados, formando um **grafo direcionado acíclico (DAG)**.

Cada commit aponta para (um ou mais) commits anteriores.

Quando falamos de merge, rebase e squash, estamos falando de como combinar históricos de commits.

# Diferença entre Merge, Rebase e Squash

## 1️⃣ Merge

O merge une duas branches criando um novo commit de merge.

Suponhamos que temos:

```bash
main:    A---B---C
               \
feature:        D---E
```

Se fizermos: 

```bash
git checkout main
git merge feature
```

Temos o resultado:

```bash
A---B---C-------M
         \     /
          D---E
```

Vamos para a explicação:

- **“M”** é um merge commit;
- Ele tem dois pais;
- O histórico é preservado exatamente como aconteceu;

### Características

- Não reescreve o histórico;
- Seguro para branches compartilhadas;
- Mantém a topologia real do trabalho;
- Pode gerar histórico mais “poluído”

### Quando usar?

- Branches públicas;
- Times grandes;
- Quando queremos preservar o contexto real do desenvolvimento

## 2️⃣ Rebase

O rebase move (replica) commits de uma branch para outro ponto da história

Vamos manter o mesmo cenário que o anterior;

```bash
main:    A---B---C
               \
feature:        D---E
```

Se fizermos:

```bash
git checkout main
git merge feature
```

Temos o seguinte resultado:

```bash
A---B---C---D'---E'
```

### Características

- Reescreve o histórico;
- Produz histórico linear;
- Mais limpo visualmente;
- Pode causar problemas se usado em branch pública;

<aside>
💡

A regra de ouro do rebase é - Nunca faça rebase em branch já compartilhada com outras pessoas. Porque assim mudamos o SHA dos commits.

</aside>

### Quando usar?

- Antes de abrir PR;
- Para manter histórico linear;
- Em branch local

## 3️⃣ Squash

O squash transforma vários commits em apenas um.

Existem duas formas principais:

### Squash via merge

```bash
git merge --squash feature
```

Dessa forma:

- Junta todas as mudanças;
- Mas cria um único commit;
- Não preserva histórico interno da branch;

Resultado:

```bash
A---B---C---S
```

Onde “S” contém tudo de “D” e “E”.

### Interactive rebase (mais comum)

```bash
git rebase -i HEAD~3
```

Podemos marcar os commits como **squash** ou **fixup**.

Exemplo:

```bash
pick D
squash E
squash F
```

Resultado:

```bash
A---B---C---X
```

Onde “X” contém D+E+F

### Características

- Histórico mais limpo;
- Remove granularidade;
- Ideal para commits intermediários como:
    - “fix typo”;
    - “ajuste”;
    - “corrigindo teste”

# Além desse temos outros tipos

## Fast-forward merge

Se não houver divergência:

```bash
A---B---C
           \
            D---E
```

Se a “main” não teve commits novos;

```bash
git merge feature
```

Resultado:

```bash
A---B---C---D---E
```

Sem merge commit, isso é chamado de **fast-forward**.

## Cherry-pick

O cherry-pick aplica um commit específico em outra branch;

```bash
git cherry-pick <commit>
```

Ele cria um novo commit com o mesmo conteúdo