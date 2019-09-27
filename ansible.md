
# Ansible

O Ansible é uma ferramenta utilizada para provisionamento. Não trabalha com agentes, a execução pode vir de qualquer máquina que tenha acesso via SSH as demais. 
Utiliza-se de arquivos descritivos escritos em YAML chamados de playbooks.

## Conexão

O Ansible se conecta a outras máquinas Linux através do protocolo SSH. Para tanto, é necessário que as máquinas consigam se comunicar e já se conheçam previamente.

## Inventário

O inventário do Ansible é um arquivo em formato **.ini** localizado por padrão em /etc/ansible/hosts. Nele são definidos as máquinas que podemos acessar, separadas em grupos ou não, e com configurações específicas, caso necessário:

```ini
[desenvolvimento]
maquina01.dragonflycreative.com.br
maquina02.dragonflycreative.com.br
209.208.27.243

[producao]
maquina03.dragonflycreative.com.br
209.208.27.244
```

## Configuração

Alguns valores em comuns como usuário remoto e chave privada podem ser modificados no arquivo **/etc/ansible/ansible.cfg**.

## Modulos

Cada comando que o Ansible executa, utiliza-se de um módulo, os mais utilizados são:

-   **package** - Gerencia pacotes abstraindo o comando da distribuição
-   **service** - Gerencia os serviços - systemd ou systemctl
-   **copy** - Copia arquivos de uma máquina para outra
-   **unarchive** - Copia arquivos compactados de uma máquina para outra, descompactando
-   **get_url** - Baixa arquivos de endereços remotos

## Ad-Hocs

Os comandos ad-hocs são comandos simples que executamos para testes ou instação de pacotes de forma muito simplista:

```bash
ansible all -m 'ping'
ansible all -m 'package' -a 'name=vim state=present'
```

O Ansible recolhe as características das máquinas - fatos - através de um módulo chamado **setup**:

```bash
ansible all -m 'setup'
```

Esses fatos estão disponíveis dentro das playbooks como varíaveis.

## Playbooks

As playbooks são um conjunto de instruções, geralmente mais complexas que os comandos ad-hocs, e podem fazer muitas coisas como comparações, loops e blocos.

Um exemplo de uma playbook que configura um servidor web com uma página html copiada da máquina que rodou a playbook:

**deploy-site.yml**

```yml
- name: Instalar o Lighttpd
  hosts: all
  become: yes
  tasks:
  - package:
      name: lighttpd
      state: present
  - service:
      name: lighttpd
      state: started
      enabled: true
  - copy:
      src: /home/dragonfly/index.html
      dest: /var/www/html/index.html
```

Uma playbook que padroniza o `/etc/hosts` das máquinas:

**etc-hosts.yml**

```yml
- name: Garantindo /etc/hosts
  become: yes
  hosts: all
  tasks:
  - lineinfile:
      path: /etc/hosts
      line: '{{item}}'
    loop:
    - '192.168.33.10 devops.example.com'
    - '192.168.33.20 docker.example.com'
    - '192.168.33.30 automation.example.com'
```

Uma playbook que instala o Gitlab Omnibus:

**gitlab-install.yml**

```yml
# Playbook de instalação do GitLab em Ansible
- name: Instalação GitLab
  hosts: devops
  become: yes
  tasks:
  - package:
      name: ['curl','openssh-server','ca-certificates']
      state: present
  - get_url:
      url: https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh
      dest: /tmp/
      mode: 0755
  - shell: /tmp/script.deb.sh
  - package:
      name: gitlab-ce
      state: present
```

Um exemplo com blocos e condições da instalação do **Puppet** em máquinas de sistemas operacionais diferentes:

**puppet-install.yml**

```yml
- name: Provisionando o Puppet
  hosts: all
  become: yes
  tasks:
  - name: CentOS
    block:
    - yum:
        name: https://yum.puppet.com/puppet6-release-el-7.noarch.rpm
        state: present
    - package:
        name: puppetserver
        state: present
    when: ansible_distribution|lower == 'centos'
  - name: Debian
    block:
    - apt:
        deb: https://apt.puppetlabs.com/puppet6-release-xenial.deb
    - apt:
        name: puppet
        state: absent
        update_cache: yes
    - apt:
        name: puppet-agent
        state: present
        update_cache: yes
    when: ansible_distribution|lower == 'debian'
```
