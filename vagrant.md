# Vagrant

O vagrant é um provisionador de máquinas virtuais. É utilizado para facilitar tarefas repetitivas e acelerar o provisionamento.
Executa comandos diretos no *hypervisor* existente, criando as máquinas, seus discos e suas interfaces de rede.
Chama as imagens das máquinas virtuais de **boxes**.

### Vagrantfile

É o arquivo que o Vagrant utiliza para criar e provisionar as máquinas. Pode ser criado a mão, ou com o auxílio do comando **vagrant init <nomedaimagem>**.
Ao executar o Vagrantfile através do comando **vagrant up**, o Vagrant verifica se as boxes existem na máquina local, e caso não existam, baixa através do site [app.vagrantup.com](https://app.vagrantup.com/).

##### Configurações

Os arquivos de configuração das máquinas como discos e chave privada ficam no diretório oculto **.vagrant** junto ao Vagrantfile.
As imagens originais, as quais o Vagrant utiliza para gerar as novas máquinas, ficam dentro da home do usuário em questão no diretório **~/.vagrant.d**.

### Exemplos

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
