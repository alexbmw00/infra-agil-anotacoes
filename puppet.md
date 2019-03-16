# Puppet

O puppet é um software de gerenciamento e configuração.

# Repositórios

##### Debian
Para habilitar o repositório mais recente em distribuições baseadas em Debian:
```
sudo wget https://apt.puppetlabs.com/puppet6-release-xenial.deb
sudo dpkg -i puppet6-release-xenial.deb
sudo apt-get update
```
##### CentOS
Para habilitar o repositório mais recente em distribuições baseadas em RedHat:
```
sudo rpm -Uvh https://yum.puppet.com/puppet6/puppet6-release-el-7.noarch.rpm
```

##### Binários
Adicionar ao **$PATH** do usuário, o caminho dos binários do Puppet, e recarregar as configurações:
```
echo 'PATH=$PATH:/opt/puppetlabs/bin/' > ~/.bashrc
source ~/.bashrc
```

### Puppet Agent

##### Debian
Para habilitar o repositório mais recente em distribuições baseadas em Debian:
```
apt-get install -y puppet-agent
```
##### CentOS
Para habilitar o repositório mais recente em distribuições baseadas em RedHat:
```
yum install -y puppet-agent
```

### Puppet Server

##### Debian
Para habilitar o repositório mais recente em distribuições baseadas em Debian:
```
apt-get install -y puppetserver
puppsetserver ca setup
systemctl start puppetserver
systemctl enable puppetserver
```
##### CentOS
Para habilitar o repositório mais recente em distribuições baseadas em RedHat:
```
yum install -y puppetserver
puppsetserver ca setup
systemctl start puppetserver
systemctl enable puppetserver
```

## Problemas

Resolução dos problemas mais comuns em sala de aula:

#### Error: Could not request certificate: Failed to open TCP connection to puppet:8140
Modificar o arquivo **/etc/puppetlabs/puppet/puppet.conf** adicionando a seção **main** indicando o Puppet Server:
```
vim /etc/puppetlabs/puppet/puppet.conf
### Adicionar ###
[main]
server = devops.dexter.com.br
```

#### Memória Insuficiente
Para gerenciar a quantidade de memória que o Puppet Server utiliza, por exemplo, de 2GB para 512MB, modificar o arquivo em **/etc/default/puppetserver**:
```
vim /etc/default/puppetserver
### Modificar de ###
JAVA_ARGS="-Xms2g -Xmx2g -Djruby.logger.class=com.puppetlabs.jruby_utils.jruby.Slf4jLogger"
### Para ###
JAVA_ARGS="-Xms512m -Xmx512m -Djruby.logger.class=com.puppetlabs.jruby_utils.jruby.Slf4jLogger"
```

#### Problemas com certificado TLS
Caso o Puppet Server tenha sido configurado de forma errada, ou apresentou problemas de certificado por motivos diversos, tudo pode ser reiniciado de forma simples:
```
systemctl stop puppetserver
rm -rf /etc/puppetlabs/puppet/ssl
puppetserver ca setup
systemctl start puppetserver
```
