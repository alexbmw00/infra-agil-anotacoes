# Docker

Docker é uma ferramenta utilizada para criar aplicações que consigam funcionar isoladamente sem depender dos binários e bibliotecas do sistema operacional. Essas aplicações são transformadas no que chamamos de imagens. Através dessas imagens, podemos criar o que chamamos de contêineres.

Hoje, podemos dizer que o docker está separado em três ferramentas:

 - docker-cli - executa os comandos desejados chamando a API REST
 - docker-engine - uma API REST que recebe os comandos e os passa para o containerd
 - containerd - aquele que de fato cria os contêineres

### Instalação

#### Debian

```bash
apt-get update
apt-get -y install apt-transport-https ca-certificates curl gnupg2 software-properties-common
curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add -
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable"
apt-get update
apt-get install -y docker-ce docker-ce-cli containerd.io
usermod -G docker -a vagrant
```

Após a instalação, você pode executar o comando de teste para verificar se tudo está funcionando corretamente:

	docker run hello-world

Ao executar o comando **run** o Docker verifica se a imagem está disponível na máquina local, e caso não esteja ele faz o download no **registry** público padrão - um repositório de imagens - localizado em [https://hub.docker.com/](https://hub.docker.com/).

O comando **run** é a junção de dois outros comandos:

 - docker create
 - docker run
 
### Comandos de exemplo

```
docker pull alpine
docker run -d --name servidor -p 8080:80 nginx:alpine
docker run -ti debian
docker exec -ti ac10f32ed87e sh
docker top ac10f32ed87e
docker stats
```

Os contêineres quase sempre são efêmeros, e quando são removidos seus dados são descartados, a não ser que sejam persistidos em um volume. Este volume pode ser um diretório na própria máquina ou mesmo um serviço de NFS ou iSCSI. Também podemos "montar arquivos" diretamente dentro do container.
Para montar volumes dentro do contêiner utilizamos a flag **-v** ou **--mount**.

```
docker run -d --name apache -v /root/html:/usr/local/apache2/htdocs/ --publish 9090:80 httpd:end
docker volume create portainer_data # opcional
docker run -d -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer
```

Também é possível passar variáveis de ambiente para dentro do contêiner, para facilitar alteração de endereços, senhas, portas e etc ou mesmo utilizá-las para a inicialização de um contêiner:

```
docker run -d -e MYSQL_ROOT_PASSWORD=4linux -e MYSQL_USER=hector -e MYSQL_PASSWORD=123 -e MYSQL_DATABASE=docker --name mysql mysql:5.7
```

Podemos fazer alterações dentro de um contêiner e salvá-lo como uma image:

```
docker run -ti --name gambiarra alpine
apk add vim curl telnet
# CTRL + P + Q
docker commit gambiarra gambiarra
```

Podemos salvar o contêiner como uma imagem, transferí-la para outro computador e carregá-la, sem precisar de acesso a internet ou a um registry local:

```
docker save gambiarra -o gambiarra.tar
docker load -i gambiarra.tar
```

# Dockerfile

O Dockerfile é o arquivo utilizado para criar imagens do docker. São muito simples, leves e fáceis de versionar.
Clone o repositório https://github.com/hector-vido/sti-lighttpd.git e entre na pasta e crie o arquivo:

**Dockerfile**
```
FROM alpine
EXPOSE 80
ENV VERSION=0.1
RUN apk add lighttpd
WORKDIR /var/www/localhost/htdocs/ 
COPY . /var/www/localhost/htdocs/
# USER lighttpd
CMD lighttpd -f /etc/lighttpd/lighttpd.conf -D
```

Feito isso, utilize o comando para criar sua imagem através de um Dockerfile:

	docker build -t minha-imagem .

Os principais comandos são:

 - **FROM** - imagem base utilizada para a criação
 - **RUN** - roda comandos durante a criação do contêiner
 - **COPY** - copia arquivos da sua máquina para dentro do contêiner
 - **CMD** - comando utilizado quando o contêiner inicia
 
 # Redes
 
 Criar uma rede é a única forma de atribuir um valor de IP fixo a um contêiner.
 É possível adicionar uma nova rede em contêineres que já estão cordando, ou adicioná-la no momento em que o contêiner será criado.
 
```
docker network create --sub-net 11.11.11.0/24 minha-rede
docker run -dti --network minha-rede --ip 11.11.11.10 --name meu-container alpine
docker inspect meu-container
```

# Registry

Um registry é um repositório de imagens do Docker. Até então o único registry que utilizamos é o registry padrão público que consultamos suas imagens em [https://hub.docker.com](https://hub.docker.com).
As vezes as pessoas precisam de repositórios privados e utilizam o registry do próprio Docker ou soluções mais robuscas como **Nexus**.
Um exemplo bem simples é criar o registry com um container chamado **registry/v2**. Neste exemplo, vamos aproveitar e criar um registry com certificado TLS e autenticação básica.

### Diretórios

Crie uma pasta chamada **registry**, vamos trabalhar dentro dela. E nesta pasta crie outras três:

 - registry
   - auth
   - cache
   - certs

### Certificados

Utilize o comando **openssl** para criar um certificado auto-assinado:

```
openssl req -newkey rsa:4096 -nodes -sha256 -keyout certs/domain.key -x509 -days 365 -out certs/domain.crt
```

### HTPasswd

Vamos colocar nossa autenticação em um arquivo de texto com a senha criptografada. Esse método é muito comum para liberar acesso a arquivos através do servidor web Apache:

```
apt-get install -y apache2-utils
htpasswd -Bbc auth/htpasswd admin 123
```

### Contêiner

Execute um **docker run** passando os parâmetros necessários para inicializar o contêiner com autenticação e certificado tls:

```
docker run -d -p 5000:443 --restart=always --name registry \
-v `pwd`/certs:/certs \
-v `pwd`/cache:/var/lib/registry \
-v `pwd`/auth:/auth \
-e REGISTRY_AUTH='htpasswd' \
-e REGISTRY_AUTH_HTPASSWD_REALM='Registry Healm' \
-e REGISTRY_AUTH_HTPASSWD_PATH='/auth/htpasswd' \
-e REGISTRY_HTTP_ADDR='0.0.0.0:443' \
-e REGISTRY_HTTP_TLS_CERTIFICATE='/certs/domain.crt' \
-e REGISTRY_HTTP_TLS_KEY='/certs/domain.key' \
-e REGISTRY_PROXY_REMOTEURL='http://registry-1.docker.io' \
registry
```

### Insecure Registry

Por padrão o docker se recusa a comunicar-se com registries inseguros, mas é possível adicionar uma lista de locais permitidos. Nas versões mais novas este arquivo fica em **/etc/docker/daemon.json**:

```
{
  "insecure-registries" : ["localhost:5000"]
}
```

Para se logar o contêiner e poder subir imagens utilize:

```
docker login https://localhost:5000
docker tag alpine localhost:5000/alpine
docker push localhost:5000/alpine
```

Para consultar o registry, é preciso utilizar usuário e senha. Você pode fazê-lo pelo navegador ou utilizar o **curl**:

```
curl -k -u admin:123 https://localhost:5000/v2/_catalog
curl -k -u admin:123 https://127.0.0.1:5000/v2/alpine/tags/list
```
# Swarm

O **swarm** é o orquestrador padrão do Docker, já embutido no **docker-ce** muito simples de utilizar e muito poderoso.
Um orquestrador de contêineres é responsável por gerenciar o estado do cluster, recriando contêineres que por alguma razão pararam, fornecendo balanceamento de carga e compartilhamento de configurações.

Como analogia bem simplista, dentro do **swarm** podemos fazer a seguinte comparação:

 - docker container create -> docker service create
 - docker-compose create -> docker stack deploy

Para facilitar o provisionamento das máquinas do cluster na nossa máquina local vamos utilizar o **docker-machine**.

# Docker Machine

O **docker-machine** é um software que facilita o gerenciamento de máquinas docker.
Com ele é possível provisionar máquinas virtual muito leves para fazer testes localmente, ou podemos provisionar máquinas nas clouds públicas como Digital Ocean, AWS, Azure e Google Cloud.

Por ser um binário único e independente, basta baixá-lo e colocá-lo em um diretório padrão de executáveis:

```
curl -L https://github.com/docker/machine/releases/download/v0.16.0/docker-machine-$(uname -s)-$(uname -m) > /tmp/docker-machine &&
sudo install /tmp/docker-machine /usr/local/bin/docker-machine
```

O comando para criar uma máquina virtual local utilizando-se do Virtualbox é muito simples, criaremos 3 para fazer parte de nosso cluster do Swarm:

```
docker-machine create -d virtualbox default
docker-machine create -d virtualbox worker1
docker-machine create -d virtualbox worker2
```

Para iniciar o Swarm podemos entrar na máquina digitando apenas **docker-machine ssh <nome>** mas também podemos executar o comando diretamente:

```
docker-machine ssh default docker swarm init --advertise-addr 192.168.99.100

```

Este comando iniciará o cluster de Swarm e nos fornecerá um **token** para ser utilizado na adição de novos workers, por exemplo:

```
docker-machine ssh worker1 docker swarm join --token SWMTKN-1-5oap2z60b7awm9or2y54pq1wtj91ozytbeb4kz7zkhfefvfy8q-d01y9wthhtv9naed1vbcfbs0d 192.168.99.100:2377
docker-machine ssh worker2 docker swarm join --token SWMTKN-1-5oap2z60b7awm9or2y54pq1wtj91ozytbeb4kz7zkhfefvfy8q-d01y9wthhtv9naed1vbcfbs0d 192.168.99.100:2377
```

Para podermos utilizar o **docker-cli** diretamente na máquina **manager** precisamos importar as variáveis necessárias na nossa sessão:

```
eval $(docker-machine eval default)
docker run hello-world
```

A partir de agora, todos os comandos executados partirão do princípio de que o **docker-cli** está direcionado para a máquina **default**.

### Utilizando o swarm

É possível provisionar sua aplicação dentro do cluster, fornecendo o número de réplicas desejadas:

```
docker service create --name cgi --replicas 3 -p 8080:8080 hectorvido/sh-cgi
docker service ls
docker service ps cgi
```

### Utilizando um compose-file

Podemos provisionar um compose-file dentro do cluster, por exemplo:

**docker-compose.yml**
```
version: '3.7'

services:

  php:
    depends_on:
    - mysql
    image: hectorvido/php-app
    environment:
      DB_HOST: mysql
      DB_PORT: 3306
      DB_PASS: Abc123!
      DB_USER: tre
      DB_NAME: tre
    ports:
    - "8080:8080"

  mysql:
    image: mysql
    environment:
      MYSQL_ROOT_PASSWORD: Abc123!
      MYSQL_USER: tre
      MYSQL_PASSWORD: 4linux
      MYSQL_DATABASE: tre
    command: --default-authentication-plugin=mysql_native_password
```

Basta utilizar o comando:

```
docker stack deploy --compose-file docker-compose.yml app
```
