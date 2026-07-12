# Azure

# Introdução

A Azure Cloud é o provider de computação na nuvem da Microsoft, e mesmo sendo da Microsoft não estamos limitados a somente usar produtos da mesma.

## Particularidades da Azure 🔎

Primeiro, todos os serviços e recursos devem estar vinculados à um Resource Group, e esse Resource Group está vinculado a uma região.

Regiões da Azure (2026):

![[Cloud/Azure/Attachments/image.png]]

Para conhecer os produtos da Azure, podemos ir no link abaixo:

[Produtos de nuvem | Microsoft Azure](https://azure.microsoft.com/pt-br/products)

## Portal Azure

O caminho mais fácil para criar e gerenciar recursos da Azure é pelo Portal da Azure:

[https://portal.azure.com](https://portal.azure.com)

Interface do Portal:

![[Cloud/Azure/Attachments/image 1.png]]

Podemos usar esse portal para criar uma rede virtual, configurar as VMs, configurar as contas de armazenamento, definir políticas de acesso… Bom, basicamente conseguimos fazer tudo através dele.

**DNS (Domain Name Service) →** É serviço onde temos um dominio que será atribuído ao IP do nosso webservice

Resource Groups (Grupo de Recursos)→ Vão criar como se fosse uma rede no Azure, onde teremos nossa API que depende do BD, Blob Storage, etc. Temos que garantir que a app e o banco estão no mesmo Resource Group.

Dentro do RG (Resource Group), temos a App Service, Virtual Machines e a Functios Apps ou Azure Functions, os dois recursos são para hostear apps, sendo eles respectivamente, PaaS (Platform as a Service), IaaS (Infrastructure as a Service) e FaaS (Function as a Service).