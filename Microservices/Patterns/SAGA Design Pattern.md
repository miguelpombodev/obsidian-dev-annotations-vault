Vamos pensar que temos um cenário onde estamos lidando com uma aplicação de compras de pacotes de viagens, como garantimos que, quando o cliente executar o pagamento e o mesmo for de fato abatido, a reserva do vôo, do hotel e do carro foi de fato feita?

![[Pasted image 20260819212228.png]]

Isso tudo sabendo que os serviços são totalmente apartados, com bancos de dados separados e nenhum deles sabe a existência um do outro?
É sobre cenários assim que aplicamos o padrão **SAGA**.

## Explicação
O SAGA pattern é uma maneira na qual usamos para orquestrar múltiplos serviços diferentes quando não conseguimos empacotar tudo em uma única transação ACID

> Lembrando que ACID significa **A**tomicity, **C**onsistency, **I**solation e **D**urability.