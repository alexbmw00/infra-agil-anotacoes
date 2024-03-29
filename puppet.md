
# Puppet

O puppet é um software de gerenciamento de configuração, pode ser utilizado em modo standalone ou agente e servidor.
As instruções executadas pelo puppet podem ser passadas através de linha de comando - como instruções mais simples - ou grandes scripts executando dezenas ou até mesmo centenas de instruções, esses scripts recebem o nome de **manifests**.

## Standalone

No modo standalone utilizamos o puppet somente para aplicar configurações na máquina local, essas configurações podem ser simples linhas de comando adicionando usuários, pacotes e configurando serviços como manifestos que foram adquiridos de alguma forma.
A vantagem deste modo é que não precisamos configurar um servidor, apenas o pacote **puppet-agent** é necessário.

## Agente e Servidor

No mode agente/servidor, o puppet procura por um servidor que através de uma troca de certificados de confiança passa a fornecer manifestos para que ele execute localmente.
O servidor, por sua vez, possuí diretórios específicos que guardam os manifestos que podem ser entregues e uma autoridade certificadora que assina os certificados dos agentes.
Para esse tipo de instalação o pacote **puppetserver** precisará ser instalado na máquina master.

# Instalação

### Debian 9

Para habilitar o repositório mais recente em distribuições baseadas em Debian:

```bash
sudo wget https://apt.puppetlabs.com/puppet6-release-xenial.deb
sudo dpkg -i puppet6-release-xenial.deb
sudo apt-get update
```

### CentOS 7

Para habilitar o repositório mais recente em distribuições baseadas em RedHat:

```bash
sudo rpm -Uvh https://yum.puppet.com/puppet6/puppet6-release-el-7.noarch.rpm
```

## Puppet Agent

### Debian

Para habilitar o repositório mais recente em distribuições baseadas em Debian:

```bash
apt-get install -y puppet-agent
```

### CentOS

Para habilitar o repositório mais recente em distribuições baseadas em RedHat:

```bash
yum install -y puppet-agent
```

## Puppet Server

### Debian

Para habilitar o repositório mais recente em distribuições baseadas em Debian:

```bash
apt-get install -y puppetserver
puppsetserver ca setup
systemctl start puppetserver
systemctl enable puppetserver
```

### CentOS

Para habilitar o repositório mais recente em distribuições baseadas em RedHat:

```bash
yum install -y puppetserver
puppsetserver ca setup
systemctl start puppetserver
systemctl enable puppetserver
```

# puppet

O commando utilizado tanto pelo agente como para os comandos locais é o `puppet`.
As ações são tomadas a partir de "resources". Existem muitos resouces, por exemplo, para usuários, pacotes, serviços, arquivos e etc.

## CLI

Podemos por exemplo, listar todos os serviços:

	puppet resource service

Através disso, podemos escolher um serviço específico e pará-lo, sem se preocupar com a sintaxe do sistema em questão:

	puppet resource service cron ensure=stopped

Podemos cadastrar um usuário com apenas um simples comando do puppet - sem considerar a senha:

```bash
PASSWD=$(echo '4linux' | openssl passwd -1 -stdin)
puppet resource user ken shell='/bin/bash' home='/home/ken' password="$PASSWD"
```

Podemos gerar exemplos de manifestos com os comandos para editarmos mais tarde:

```bash
puppet resource user ken shell='/bin/bash' home='/home/ken' > meu_manifesto.pp
puppet package vim >> meu_manifesto.pp
```

## Manifests

Os manifestos são um conjunto de instruções utilizados pelo puppet, são fáceis de ser versionados e transferidos:

`meu_manifesto.pp`

```puppet
user { 'ken':
  ensure   => 'present',
  home     => '/home/ken',
  password => '$1$56DSrUyJ$b1NsvSzju36aKFKX.Kzz4.',
  shell    => '/bin/bash',
}

$pacotes = ['htop','curl','telnet','wget']
package { $pacotes:
  ensure   => 'present',
}
```

**Obs:** veja que podemos modificar o "nome" do recurso para referenciar uma variável em forma de array, desta forma instalamos todos os pacotes de uma só vez.

Para executar esse manifesto, basta executar:

	puppet apply meu_manifesto.pp

# Problemas

Resolução dos problemas mais comuns em sala de aula:

## TCP connection to puppet:8140

> Error: Could not request certificate: Failed to open TCP connection to puppet:8140

Modificar o arquivo **/etc/puppetlabs/puppet/puppet.conf** adicionando a seção **main** indicando o Puppet Server:

`/etc/puppetlabs/puppet/puppet.conf`
```ini
[agent]
server = devops.example.com
```

## Memória Insuficiente

Geralmente o serviço falha ao iniciar. O comportamento pode ser observado com o comando:

```bash
journalctl -u puppet -f
```

Para gerenciar a quantidade de memória que o Puppet Server utiliza, por exemplo, de 2GB para 512MB, modificar o arquivo em **/etc/default/puppetserver**:

```ini
vim /etc/default/puppetserver
### Modificar de ###
JAVA_ARGS="-Xms2g -Xmx2g -Djruby.logger.class=com.puppetlabs.jruby_utils.jruby.Slf4jLogger"
### Para ###
JAVA_ARGS="-Xms512m -Xmx512m -Djruby.logger.class=com.puppetlabs.jruby_utils.jruby.Slf4jLogger"
```

## Problemas com certificado TLS

No nosso caso, em que o puppet está configurado em um ambiente de testes, caso o Puppet Server tenha sido configurado de forma errada, ou apresentou problemas de certificado por motivos diversos, tudo pode ser reiniciado de forma simples:

```bash
systemctl stop puppetserver
rm -rf /etc/puppetlabs/puppet/ssl
puppetserver ca setup
systemctl start puppetserver
```
