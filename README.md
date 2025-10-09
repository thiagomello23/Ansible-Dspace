
# Script Ansible para DSpace

O objetivo desse script é instalar, configurar e rodar o DSpace 9.x.

### Requisitos

- Ter ansible instalado e uma chave SSH para ele se conectar com os servidores do qual vai rodar o script.
- As maquinas que o DSpace vai ser instalado precisam necessáriamente serem derivadas do Debian por conta do uso do gerenciador de pacotes APT (Ex: Ubuntu).

### Configuração

Configurações necessárias para rodar o script:
- Definir IP's
- Adicionar chave SSH de acesso
- Qual a home do projeto
- Qual o usuário do servidor
- Qual o dominio que vai ser utilizado (caso resolva gerar o certificado pelo proprio script)

Para configurar a home e definir os IP's vá em "inventory.yaml" e modifique o arquivo igual no exemplo abaixo:
```
dspace_prod:
  hosts:
    dspace_host_01:
      ansible_host: 127.0.0.1
    dspace_host_02:
      ansible_host: 127.0.0.2
  vars:
      home: /home/meu-usuario/ansible-for-dspace
```

Para configurar o usuário do servidor, o dominio e o caminho da chave SSH edite "roup_vars/dspace_prod.yaml" igual no exemplo abaixo:
```
ansible_user: ubuntu
# chave-ansible é o nome padrão da chave ssh esperada pelo sistema
ansible_ssh_private_key_file: "{{ home }}/chave-ansible.pem"
# Usei sslip.io como exemplo de dominio
DSPACE_SERVER_DOMAIN: "{{ ansible_host | replace('.', '-') }}.sslip.io"
```

### Como funciona

Esse script é dividido em três partes, backend, frontend e certificado. As versões mais recentes do DSpace a partir da 7.x começaram
a fazer essa separação entre Backend e Frontend então resolvi manter o padrão e se quiser pode instalar apenas um dos dois separadamente
assim como pode instalar apenas o DSpace e não gerar o certificado pelo script (caso queira por exemplo trocar de proxy reverso).

Comando para rodar os scripts:
- ansible-playbook -i inventory.yaml playbooks/deploy-dspace-backend-master.yaml -vvv (Backend)
- ansible-playbook -i inventory.yaml playbooks/deploy-dspace-front-master.yaml -vvv (Frontend)
- ansible-playbook -i inventory.yaml playbooks/deploy-nginx-certificate-master.yaml -vvv (Certificates)

Por padrão são instalados dois daemons básicos para o controle do backend, um para o próprio DSpace e o outro para o Solr, uma dependência do DSpace.
- /daemons/dspace.service
- /daemons/solr.service

Para o gerenciamento do Frontend do DSpace é utilizado o PM2.
