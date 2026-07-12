
O EC2 é um dos serviços mais utilizados dentro da AWS pois é onde as empresas sempre começam sua migração da estrutura física para a estrutura cloud. EC2 significa Elastic Compute Coud, e é o service de IAAS (Infrastructure as a Service) para VMs.

Antigamente tínhamos um servidor físico, que tinha uma quantidade grande de RAM, de CPU, de disco e rede, e com isso instalávamos um SO, que poderia ser Linux ou Windows, e após isso instalávamos nossas aplicações que nos seriam úteis. Porém existia um problema de sobra de recurso, por exemplo, se tivéssemos instalados 128GB de RAM no nosso servidor e só usássemos, por alguma razão 10GB, teríamos uma grande sobre de 118GB de memória e isso significa dinheiro jogado fora.

Por conta desse problema que o EC2 foi criado, pois a AWS tem um cluster de servidores que trabalha com virtualização (Hypervisor), e dessa forma nos deixa trabalhar com SOs no topo, dessa forma podemos ter várias instâncias de varios SOs com várias aplicações, sendo assim a quantidade dos recursos são controlados pela própria instancia.

# Tipos de EC2

A AWS têm vários tipos de instâncias otimizadas para cada caso de uso. Vamos pegar para exemplo uma máquina “m7g”:

![[EC2_image.png]]

Ela é uma máquina que tem uma CPU virtual (vCPU), com 4 GB de memória, o armazenamento dela é EBS que significa Elastic Block Store, que é o disco da máquina, com largura de banda de rede de até 12,5Gbps, e largura de banda de acesso aos discos (EBS) de até 10Gbps.

E é assim que podemos ver as características de nossas instâncias.

# Modelos de Preços

Agora iremos falar sobre quanto que custa rodar um EC2, mas antes de tudo devemos falar sobre **modelos de preços** ou **modelos de aquisição**:

- Sob Demand (On Demand) → Esse modelo é um dos mais caro que temos, é aquele que começamos a criar a máquina, escolhemos o tipo e o modelo, e ela ja está funcionando;
- Plano para economizarmos (Saving Plans) →  Esses planos são os mais baratos, temos ele para o EC2, Sagemaker e Fargate, porém para obtê-los precisamos ter um contrato de 1 a 3 anos com a AWS;
- Spot → Aqui pagamos um valor baixo que pode chegar até 90% a menos do que o “On Demand”, porém ele pega as instâncias que a AWS não esta utilizando mas depois pode utilizar, então podemos criar um servidor usando esse modelo e depois a AWS pode nos tomar para outro cliente, usamos esse modelo para trabalhos (workloads) que são tolerantes a falha ou estamos testando, aplicativos que precisam ser testados por alguns minutos ou horas;
- Hosts Dedicados → Esse modelo é o mais caro que temos, pois estaremos pagando por um host em si, quem usa são empresas financeiras, governos ou empresas que tem dados que nao podem estar em rede pública.
- Capacidade por demanda → É um modelo que usamos para máquina com recursos especificos por um tempo específico, por exemplo “preciso de uma máquina com X de RAM, Y de CPU, W de disco por Z horas”, esse modelo é um pouco mais barato que o “On Demand”.