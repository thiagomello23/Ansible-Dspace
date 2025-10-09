
# Script Ansible para DSpace

O objetivo desse script é instalar, configurar e rodar primariamente as versões DSpace a partir da 7.x quando o DSpace separou em Front e Back. Por padrão
o projeto é feito para instalar o DSpace 9.x.

### Requisitos

- Ter ansible instalado e uma chave SSH para ele se conectar com os servidores do qual vai rodar o script.
- As maquinas que o DSpace vai ser instalado precisam necessáriamente serem derivadas do Debian por conta do uso do gerenciador de pacotes APT (Ex: Ubuntu).

## Configuração

Essa aba é separada em 3 tipos de configuração, uma configuração necessária para rodar os scripts ansible, outra opcional de configuração do DSpace
e por fim uma específica dos artefatos do DSpace, como por exemplo se o seu Frontend for customizado e quiser trocar a versão de instalação.

### Configurações necessárias para rodar o script

Configurações básicas para ter o script rodando, como IP dos hosts que o ansible vai acessar assim como qual o usuário dos servidores, o dominio e etc.

Este script foi pensado para esperar o mesmo usuário de servidor e o mesmo domínio nos servidores, caso por exemplo em sua instalação do DSpace seja um dominio
diferente para cada servidor basta configurar e rodar o script duas vezes separadamente, mais sobre isso na sessão "Como Funciona".

Configs:
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

Para configurar o usuário do servidor, o dominio e o caminho da chave SSH edite "group_vars/dspace_prod.yaml" igual no exemplo abaixo:
```
ansible_user: ubuntu
# chave-ansible é o nome padrão da chave ssh esperada pelo sistema
ansible_ssh_private_key_file: "{{ home }}/chave-ansible.pem"
# Usei sslip.io como exemplo de dominio
DSPACE_SERVER_DOMAIN: "{{ ansible_host | replace('.', '-') }}.sslip.io"
```

### Configuração do próprio DSpace

Configs:
- Templates de configuração do DSpace Backend
- Templates de configuração do DSpace Frontend
- Templates de configuração dos Certificados

As configurações de template de configuração são opcionais para o funcionamento do sistema mas caso queira configurar para adicionar coisas extras
como um SMTP para o dspace basta acessar o diretório de "/templates", os arquivos funcionam da seguinte forma:
- Backend --> templates/local.cfg.j2, arquivo padrão de configuração do DSpace back
- Frontend --> templates/config.prod.yaml.j2, arquivo padrão de configuração do DSpace front
- Certificados --> templates/dspace-nginx.j2, arquivo de configuração dos certificados SSL para o NGINX

Como pode notar todos os arquivos de template usam a extensão "j2" oque significa que os templates usam o Jinja2 para serem gerados pois eles aceitam variáveis do ansible
deixando tudo mais dinâmico como no exemplo abaixo no "dspace-nginx.j2":
```
server {
    listen 80;
    server_name {{ DSPACE_SERVER_DOMAIN }};

    # Security Headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    ...
```

E para adicionar novas variáveis basta ou colocar hard-coded na própria configuração ou adicionar em "group_vars/dspace_prod.yaml"

### Configurações de versão dos artefatos

A maioria das dependencias do DSpace estão em formato de artefato, ou seja elas não são baixadas diretamente da internet, são enviadas para o servidor
usando o diretório de "/artifacts".

Por padrão o projeto já vem com os artefatos necessários para rodar o DSpace 9.x como Node 22, Maven 3.9.11 e etc. É possível a qualquer momento trocar quais artefatos
estão sendo utilizados, recomendo manter os mesmos nomes dos artefatos já utilizados apenas substituindo os arquivo, e os arquivos devem estar em mesmo formato, se o arquivo estiver em
"tar.gz" deve ser mantido da mesma forma.

Artefatos:
- DSpace Front
- DSpace Back
- Apache Maven
- Icu4j e Lucene (libs necessárias para rodar o Solr)
- Node
- Solr

## Como Funciona

Esse script é dividido em três partes, backend, frontend e certificado. As versões mais recentes do DSpace a partir da 7.x começaram
a fazer essa separação entre Backend e Frontend então resolvi manter o padrão e se quiser pode instalar apenas um dos dois separadamente
assim como pode instalar apenas o DSpace e não gerar o certificado pelo script (caso queira por exemplo usar outro proxy reverso).

Comando para rodar os scripts:
- ansible-playbook -i inventory.yaml playbooks/deploy-dspace-backend-master.yaml -vvv (Backend)
- ansible-playbook -i inventory.yaml playbooks/deploy-dspace-front-master.yaml -vvv (Frontend)
- ansible-playbook -i inventory.yaml playbooks/deploy-nginx-certificate-master.yaml -vvv (Certificates)

Por padrão são instalados dois daemons básicos para o controle do backend, um para o próprio DSpace e o outro para o Solr, uma dependência do DSpace, ambos
podem ser sobrescritos caso queria configurações a mais no daemon.

Daemons:
- /daemons/dspace.service
- /daemons/solr.service

Para o gerenciamento do Frontend do DSpace é utilizado o PM2.

Os objetos do script como daemons, artefatos, templates e etc são todos arquivos enviados para o servidor.

