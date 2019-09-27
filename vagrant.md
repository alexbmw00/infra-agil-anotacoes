
# Vagrant

O vagrant é um provisionador de máquinas virtuais. É utilizado para facilitar tarefas repetitivas e acelerar o provisionamento.
Executa comandos diretos no *hypervisor* existente, criando as máquinas, seus discos e suas interfaces de rede.
Chama as imagens das máquinas virtuais de **boxes**.

## Vagrantfile

É o arquivo que o Vagrant utiliza para criar e provisionar as máquinas. Pode ser criado a mão, ou com o auxílio do comando **vagrant init <nomedaimagem>**.
Ao executar o Vagrantfile através do comando **vagrant up**, o Vagrant verifica se as boxes existem na máquina local, e caso não existam, baixa através do site [app.vagrantup.com](https://app.vagrantup.com/).

## Configurações

Os arquivos de configuração das máquinas como discos e chave privada ficam no diretório oculto **.vagrant** junto ao Vagrantfile.
As imagens originais, as quais o Vagrant utiliza para gerar as novas máquinas, ficam dentro da home do usuário em questão no diretório **~/.vagrant.d**.

# Exemplos

Cria o **Vagrantfile** utilizando uma imagem do Debian Stretch, e a acessa através de SSH:

```
vagrant init debian/stretch64
vagrant up
vagrat ssh
```

Lista as **boxes** presentes na máquina local, e adiciona uma imagem de CentOS 7:

```
vagrant box list
vagrant box add centos/7
```

Loga na máquina virtual específica quando há mais de uma máquina definida no mesmo arquivo:

```
vagrant ssh maquina1
```

Desliga as máquinas de forma abrupta, e as liga logo em seguida:

```
vagrant halt -f
vagrant up
```

Reinicia todas as máquinas e faz com que as definições de provisionamento sejam reexecutadas:

```
vagrant reload --provision
```

Desliga e destroi todos os vestígios das máquinas:

```
vagrant destroy -f
```

Um exemplo de Vagrantfile com três máquinas e algum provisionamento:

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.box_check_update = false

  config.vm.define "devops" do |devops| 
    devops.vm.box = "debian/buster64"
    devops.vm.hostname = "devops.example.com"
    devops.vm.network "private_network", ip: "192.168.33.10"
    devops.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
      vb.cpus = "2"
    end
    devops.vm.provision "shell", inline: <<-SHELL
      apt-get install -y dirmngr gnupg2
      echo 'deb http://ppa.launchpad.net/ansible/ansible/ubuntu trusty main' > /etc/apt/sources.list.d/ansible.list
      apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 93C4A3FD7BB9C367
      apt-get update
      apt-get install -y ansible
    SHELL
  end

  config.vm.define "docker" do |docker| 
    docker.vm.box = "debian/buster64"
    docker.vm.hostname = "docker.example.com"
    docker.vm.network "private_network", ip: "192.168.33.20"
    docker.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
      vb.cpus = "2"
    end
    docker.vm.provision "shell", inline: <<-SHELL
      apt-get update
      apt-get -y install apt-transport-https ca-certificates curl gnupg2 software-properties-common
      curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add -
      add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable"
      apt-get update
      apt-get install -y docker-ce docker-ce-cli containerd.io
      usermod -G docker -a vagrant
      systemctl enable docker
      systemctl start docker
    SHELL
  end

  config.vm.define "automation" do |automation| 
    automation.vm.box = "centos/7"
    automation.vm.hostname = "automation.example.com"
    automation.vm.network "private_network", ip: "192.168.33.30"
    automation.vm.provider "virtualbox" do |vb|
      vb.memory = "2048"
      vb.cpus = "2"
    end
    automation.vm.provision "shell", inline: <<-SHELL
      yum install -y java-1.8.0 wget
      rpm -Uvh https://repo.rundeck.org/latest.rpm
      wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
      rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
      yum install -y rundeck jenkins
      sed -i 's/localhost/192.168.33.30/g' /etc/rundeck/framework.properties
      sed -i 's/localhost/192.168.33.30/g' /etc/rundeck/rundeck-config.properties
      systemctl start rundeckd jenkins
      systemctl enable rundeckd jenkins
    SHELL
  end
end
```
