---
layout: post
title:  "Docker + Apache + PHP no Mac OSX Mavericks"
date:   2015-08-16 22:00:00 -0300
categories: Programming osx docker
---

Olá.

### O que é Docker?
Docker é um aplicativo que facilita a execução de aplicações de forma isolada no Linux. Funciona de forma semelhante a uma máquina virtual, porém, as máquinas virtuais utilizam uma virtualização de hardware, enquanto os containers realizam uma virtualização a nível de sistema operacional. Apesar de não ser uma tecnologia nova, o docker facilita a utilização de containers, utilizando por baixo dos panos o LXC (Linux Containers) e um sistema de arquivos incremental chamado btrfs e os control groups (cgroups) do Linux. Antes de mais nada é necessário compreendermos o grande ganho desta nova forma de virtualização. 

O principal objetivo é a capacidade em empacotar tudo que a sua aplicação precisa em um grande pagote e executá-lo em um servidor com a garantia que não existirão conflitos com outras aplicações (seja por causa de versões, portas em uso, arquivos compartilhados etc). Até aí nenhuma novidade, já era possível fazer isso com máquinas virtuais, basta criar uma máquina virtual para cada aplicação e pronto. O problema com esta abordagem é o peso adicional que o sistema operacional, que precisa ser instalado individualmente para cada maquina virtual, causa. 

A virtualização por containers permite que exista apenas um sistema operacional sendo executado e os containers são "plugados" e executados. Cada container executa um processo e possui seu próprio sistema de arquivos, interface de rede virtual totalmente independente de outros containers. 

Sem a sobrecarga de um sistema operacional virtualizado adicional para cada aplicação, é possível executar uma quantidade muito maior de containers em uma máquina. Outra diferença importante é sentida no tempo de inicialização. Utilizando containers a inicialização/finalização de uma aplicação acontece em segundos, contra minutos em uma virtualização utilizando máquinas virtuais.

#### Mas então não preciso mais de máquinas virtuais?
Depende do ambiente, nada impede a utilização de containers Docker serem executados dentro de uma máquina virtual. Um exemplo é a utilização de máquinas virtuais na Amazon EC2, onde uma máquina virtual pode ser utilizada para receber e executar diversos containers Docker. 

#### Como fica o gerencialmento dos recursos (CPU, memória, I/O)?

O Docker utiliza o cgroups do Linux para gerenciar os recursos dos processos Docker. Desta forma é possível limitar os recursos para cara um dos containers executados na máquina, de forma semelhante a utilização de máquinas virtuais.

#### Porque eu preciso instalar o VirtualBox no Mac OSX ou Windows?

O Docker utiliza diversos recursos do kernel do Linux para sua execução, estes recursos não estão disponíveis no OSX nem no Windows. Para que seja possível executar containers Docker nestes sistemas operacionais é necessário executar uma máquina virtual com Linux e o Docker instalados. 

#### Mas se cada container contém a base do sistema de arquivos de uma distribuição Linux não vai ocupar muito espaço?

Para resolver este problema o Docker utiliza um sistema de arquivos incremental. Você faz download de uma imagem, por exemplo do Ubuntu, instala as dependências e aplicações que você deseja executar e comita estas mudanças na imagem. Funciona de forma semelhante a um brach no git. Se uma máquina estiver executando 10 containers, cada um utilizando a mesma imagem do Ubuntu  a única diferença entre os containers são as aplicações adicionadas na imagem. Portanto o espaço ocupado corresponde a apenas uma única imagem do Ubuntu + as aplicações de cada container e não 10 imagens do Ubuntu+aplicações. 

Se as alterações de um container não forem comitadas elas serão perdidas se o container parar e for executado novamente. 


### Instalação

Vamos para a parte que interessa, a instalação! No Mac OSX é disponibilizado um aplicativo chamado boot2docker, este aplicativo efetua o download de uma distribuição Linux extremamente pequena, com o Docker instalado e executa esta máquina virtual no Virtualbox. Como o OSX não pode executar o Docker nativamente, é disponibilizado um aplicativo de linha de comando chamado Docker que funciona como um client e envia os comandos para o Docker real executado dentro da máquina virtual do boot2docker.

Atualize o homebrew:

	brew update

Instale o boot2docker:

	brew install boot2docker

Instale o cliente do docker usando o homebrew:

	brew install docker

Inicie o boot2docker:

	boot2docker init
    
Será feito o download da imagem necessária, você deve ver um retorno como este:

    Virtual machine 'boot2docker-vm' is created and registered.
    UUID: 8f0e81f6-4772-4d7f-a4de-ebe690f05e0e
    Settings file: '/Volumes/Dados/VMS/boot2docker-vm/boot2docker-vm.vbox'
    [2014-05-21 17:22:27] Apply interim patch to VM boot2docker-vm (https://www.virtualbox.org/ticket/12748)
    [2014-05-21 17:22:27] Setting VM settings
    [2014-05-21 17:22:27] Setting VM networking
    [2014-05-21 17:22:27] Setting VM disks
    [2014-05-21 17:22:28] Done.
    [2014-05-21 17:22:28] You can now type boot2docker up and wait for the VM to start.

Para iniciar a execução da máquina virtual digite o comando:

	boot2docker up

O terminal deve mostrar as informações abaixo:

    [2014-05-21 17:25:22] Starting boot2docker-vm...
    [2014-05-21 17:25:54] Started.

    To connect the docker client to the Docker daemon, please set:
    export DOCKER_HOST=tcp://localhost:4243

Conforme solicitado no terminal, é necessário exportar a variável de ambiente DOCKER_HOST para indicar para o aplicativo cliente Docker qual é a máquina que ele deve se conectar para enviar os comandos.
	
    export DOCKER_HOST=tcp://localhost:4243

Neste momento é possível fazer um pequeno teste para ver se está tudo bem, no terminal digite:

	docker verion

Deve ser mostrado um resultado semelhante a este:

	Client version: 0.11.1
    Client API version: 1.11
    Go version (client): go1.2.1
    Git commit (client): fb99f99
    Server version: 0.11.1
    Server API version: 1.11
    Git commit (server): fb99f99
    Go version (server): go1.2.1
    Last stable version: 0.11.1


### Configurando as portas de rede

Antes de continuar com a instalação do restante das coisas é necessário configurar a rede da máquina virtual iniciada pelo boo2docker para que as portas de rede sejam redirecionadas para a máquina real (osx). Para isso pare a máquina virtual com o comando:

	boot2docker down

Execute o comando abaixo (ele demora alguns minutos). Ele fará com que todas as portas iniciando em 10000 até 10999 sejam redirecionadas:

	for i in {10000..10999}; do
		VBoxManage modifyvm "boot2docker-vm" --natpf1 "tcp-port$i,tcp,,$i,,$i";
		VBoxManage modifyvm "boot2docker-vm" --natpf1 "udp-port$i,udp,,$i,,$i";
	done

Inicie novamente a máquina virtual:
	
    boot2docker up
    

### Criando um container Docker

A próxima etapa consiste na instalação de uma imagem base para o Docker. Existem diversas imagens, neste tutorial utilizaremos a ubuntu. Para isso digite:
	
    docker pull ubuntu
    
O download tem aproximadamente 400Mb e pode demorar alguns minutos dependendo da sua conexão de internet. 

Feito o download é possível realizarmos um novo teste para verificar se tudo está ok até o momento. Execute o comando abaixo:

	docker run ubuntu /bin/echo "Hello"
    
Deve aparecer no terminal o texto:
	
    Hello
    
Vamos olhar com mais atenção o que aconteceu. O comando docker run é reponsável por executar uma aplicação dentro de um container. Este comando tem os seguintes parâmetros:

	docker run <imagem> <aplicação> <parâmetros_da_aplicação>

No nosso caso a imagem chama-se ubuntu, a aplicação é a echo e o parâmetro da aplicação "Hello". Um container docker é executado apenas enquanto existe algum processo dentro dele em execução. Se o processo para o container também para.

### Instalando as dependências necessárias

Até este momento efetuamos o download de uma nova imagem, mas ela ainda não possui nenhuma dependência nem o software que desejamos executar (apache e php). Para executar a instalação dentro de uma imagem docker podemos continuar utilizando do comando docker run

	docker run ubuntu apt-get -y install apache2 php5 
    
Este comando executa o apt-get install (opção -y elimina a necessidade de confirmar a instalação) para instalar os pacotes apache2 e php5.

O Docker utiliza um sistema de arquivos incremental, para persistir nossa instalação é necessário comitar as alterações. Execute o comando abaixo para verificar qual é a *id* da alteração realizada:

	docker ps -l

Deve aparecer a seguinte informação:

	CONTAINER ID        IMAGE               COMMAND                CREATED            
	14f40656f466        ubuntu:14.04        apt-get -y install a   About a minute ago   

Execute o comando abaixo para comitar as alterações:

	docker commit 14f40656f466 ubuntu

Onde o *id* após o commit é o *id* que apareceu no **docker ps -l**.

Como aplicação simbólica, vamos criar uma única página php que imprime as configurações do servidor. Para criar a página precisamos acessar o sistema de arquivos da imagem docker. Para isso execute o comando:

	docker run -i -t ubuntu /bin/bash
    
A partir de agora você está dentro da imagem do docker. Para criar nossa página navegue até o diretório /var/www/html

	cd /var/www/html
    
Utiliando o vi crie um arquivo com o nome teste.php e adicione o conteúdo abaixo:

	<?php phpinfo(); ?>

Salve o arquivo e saia do terminal utilizando o comando **exit** e volte para o terminal do osx.

Persista a alteração novamente executando os comandos **docker ps -l**  e **docker commit id ubuntu**. 

Agora que temos nossas dependências (apache e php) e nossa aplicação (teste.php) configuradas vamos executá-las. No terminal execute o comando:
	
    docker run -d -p 10080:80 ubuntu /usr/sbin/apache2ctl -D FOREGROUND
    
Este comando é semelhate o comando que utilizamos antes mas contém algumas informações adicionais. O opção **-p** permite especificar as portas que desejamos redirecionar. Depois de tantos redirecionamentos de portas fica um pouco complicado de lembrar, então vamos recaptular os detalhes:

1. A máquina real (osx) redireciona a porta 10080 para a porta 10080 do boo2docker (virtualbox) 
2. O boot2docker (virtualbox) redireciona a porta 10080 de entrada para a porta 80 do container docker (ubuntu+apache+php).

A opção **-d** executa o docker em modo *detached* onde você pode fechar a janela do terminal que ele continuará sendo executado.

O caminho **/usr/sbin/apache2ctl** é o caminho do script de controle do apache dentro da imagem **ubuntu**

A opção **-D FOREGROUND** força o apache a executar em primeiro plano pois o docker reconhece um processo como *rodando* apenas se ele estiver em primeiro plano.

Após este procedimento você deve ser capaz de abrir o seu browser no Mac OSX e entrar no endereço *http://localhost:10080/teste.php* 
### Copiando uma imagem docker para outra máquina

Agora que temos nossa aplicação configurada e rodando é necessário exportarmos a imagem para que ela seja executada em um servidor. Aqui vamos exportar a imagem manualmente, sem utilizar os [repositórios públicos](http://docs.docker.io/use/workingwithrepository/).

Para exportar a imagem para um arquivo image.tar verifique o *id* do container com o comando **docker ps -l** e execute o comando:

	sudo docker export $CONTAINER_ID > image.tar

Copie o arquivo image.tar para o servidor de destino e importe-o com o comando abaixo:

	cat image.tar | sudo docker import - nome_nova_imagem

Após importar a imagem você poderá iniciar a execução do container novamente no novo ambiente com o comando:

	docker run -d -p 80:80 ubuntu /usr/sbin/apache2ctl -D FOREGROUND
    
Abraço!