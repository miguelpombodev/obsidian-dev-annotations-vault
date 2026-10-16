#### Comandos de conexão e expansão
Aqui são comandos que tem a capacidade de redirecionar a entrada e saída dos comandos para outros comandos ou arquivos.

Comandos:
| (pipe) -> O pipe conecta a saída de um comando com a entrada de outro, por exemplo:
`history | grep <comando>`

& -> O "&" pode ser tanto usado para iniciarmos processos em background:
```bash
	# O comando LS executa em background porém vai mostrar um output padrão e uma mensagem de erro também padrão
	ls &
	
	# Conseguimos ver o status do comando com o comando:
	jobs -l
```

como também podemos fazer uma sequencia de comandos:
```bash
# Aqui baixamos as versões mais recentes da nossas libs que estão no sistema e depois instalamos de fato estas versões
sudo apt update && sudo apt upgrade -y
```

; -> Ele também faz uma sequencialidade de comandos, porém temos a resalva de que se caso o primeiro comando der erro com "&&" ele irá cancelar todo o processo, enquanto com o ";" mesmo tendo erro no  primeiro comando seguimos para os demais:
```
sudo apt update ; sudo apt upgrade -y
```

( ) ->

< > ->
