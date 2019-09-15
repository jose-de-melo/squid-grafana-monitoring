# Monitorando o uso de servidor proxy Squid utilizando Graylog, Elasticsearch e Grafana no CentOS 7

## Pré-requisitos

- **Elasticsearch** - Armazenamento dos logs e ferramenta de busca.
- **Graylog Server** - Análise e coleta dos logs. Insere os logs no Elasticsearch.
- **MongoDB** - Armazena as configurações e metadados do Graylog.
- **Grafana** - Visualização dos dados coletados pelo Graylog e inseridos no Elasticsearch.



## Desativando o SELinux
```shell
# systemctl stop firewalld.service
# systemctl disable firewalld.service
# sed -i s/SELINUX=enforcing/SELINUX=disabled/g /etc/selinux/config
# setenforce 0
# reboot
```


## Instalando o Elasticsearch 6.x

O Graylog ainda não funciona com o Elasticsearch 7.x. O Elasticsearch é construído usando Java e requer pelo menos o Java 8 para ser executado. Portanto, antes de instalar o Elasticsearch, é necessário instalar o Java 8.

```shell
# yum install java-1.8.0-openjdk.x86_64
```
Você pode verificar a versão do Java do sistema utilizando o comando **java -version**.

```shell
# java -version
openjdk version "1.8.0_222"
OpenJDK Runtime Environment (build 1.8.0_222-b10)
OpenJDK 64-Bit Server VM (build 25.222-b10, mixed mode)
```

Vamos instalar a versão 6.8.3 do Elasticsearch.
```shell
# wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.8.3.rpm

# rpm --install elasticsearch-6.8.3.rpm
```

## Configurando o Elasticsearch

Em sua configuração básica, o Graylog requer que o nome do cluster do **Elasticsearch** seja definido como graylog. Portanto, vamos editar o arquivo de configuração do **Elasticsearch** localizado em **/etc/elasticsearch/elasticsearch.yml**.

```shell
# vim /etc/elasticsearch/elasticsearch.yml
```

```diff
...
# -------------------- Cluster -------------------
#
# Use a descriptive name for your cluster:
#
- cluster.name: my-application
+ cluster.name: graylog
...
```

Feito isso, inicie o **Elasticsearch** e permita que ele seja executado na inicialização do sistema.

```shell
# systemctl daemon-reload
# systemctl start elasticsearch
# systemctl enable elasticsearch
```
Para verificar se está tudo correto com a instalação e configuração do **Elasticsearch**, execute o seguinte comando:

```shell
# curl -X GET http://localhost:9200
{
  "name" : "YCN0CIt",
  "cluster_name" : "graylog",
  "cluster_uuid" : "LsPVhXKvRaWmFQlxKSPZ0Q",
  "version" : {
    "number" : "6.8.3",
    "build_flavor" : "default",
    "build_type" : "rpm",
    "build_hash" : "0c48c0e",
    "build_date" : "2019-08-29T19:05:24.312154Z",
    "build_snapshot" : false,
    "lucene_version" : "7.7.0",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
  },
  "tagline" : "You Know, for Search"
}
```

## Instalando o MongoDB 4.0

Primeiro, crie o repositório YUM do MongoDB da seguinte forma:

```shell
# vim /etc/yum.repos.d/mongodb-org-4.0.repo
```

```
[mongodb-org-4.0]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/7/mongodb-org/4.0/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-4.0.asc
```

Em seguida, instale o MongoDB através do gerenciador de pacotes **YUM**.

```shell
# yum install mongodb-org
```

Após a instalação, inicie o serviço MongoDB e o habilite a executar na inicialização do sistema com os comandos a seguir:

```shell
# systemctl start mongod
# systemctl enable mongod
```
Para verificar se o MongoDB está rodando, execute o comando:

```shell
# systemctl status mongod
● mongod.service - MongoDB Database Server
   Loaded: loaded (/usr/lib/systemd/system/mongod.service; enabled; vendor preset: disabled)
   Active: active (running) since Sat 2019-09-14 17:20:31 -03; 2h 50min ago
     Docs: https://docs.mongodb.org/manual
 Main PID: 841 (mongod)
   CGroup: /system.slice/mongod.service
           └─841 /usr/bin/mongod -f /etc/mongod.conf
```

Para verificar a versão do MongoDB, utilize o seguinte comando:
```shell
# mongod --version
db version v4.0.12
git version: 5776e3cbf9e7afe86e6b29e22520ffb6766e95d4
OpenSSL version: OpenSSL 1.0.1e-fips 11 Feb 2013
allocator: tcmalloc
modules: none
build environment:
    distmod: rhel70
    distarch: x86_64
    target_arch: x86_64
```

## Instalando o Graylog 3.0

Execute o comando abaixo para instalar o repositório RPM do Graylog 3.0.

```shell
# rpm -Uvh https://packages.graylog2.org/repo/packages/graylog-3.0-repository_latest.rpm
```

Em seguida, instale o Graylog Server 3.0.

```shell
# yum install graylog-server
```
### Configurando o Graylog

Arquivo de configuração do Graylog: **/etc/graylog/server/server.conf**

Primeiramente, vamos gerar uma senha de segurança para proteger o armazenamento das senhas dos usuários. Para isso, vamos usar o gerador de senhas aleatórias, **pwgen**. 

Para instalar o **pwgen**, precisaremos instalar o pacote **epel-release** previamente com o seguinte comando:

```shell
# yum install epel-release
```

Em seguida, vamos instalar o **pwgen** com o seguinte comando:

```shell
# yum install pwgen
```

Para gerar a senha de segurança, execute o seguinte comando: 
```shell
# pwgen -N 1 -s 96
FTeIZznMtCRXmP066trf4sd8tNMCMrOIkEVW0npqwiuMHjFK8otA8IIXggE0bVonVChzVsVLkpWidmsJQdH1wRLQNodRmIhY
```

Em seguida, vamos atribuir a senha gerada acima ao parâmetro ***password_secret*** no arquivo de configuração do Graylog.

```shell
# vim /etc/graylog/server/server.conf
```

```diff
...
- password_secret =
+ password_secret = FTeIZznMtCRXmP066trf4sd8tNMCMrOIkEVW0npqwiuMHjFK8otA8IIXggE0bVonVChzVsVLkpWidmsJQdH1wRLQNodRmIhY
...
```

Para gerar o hash da senha do usuário **admin**, utilize o seguinte comando:

```shell
# echo -n "SuaSenhaAqui" | sha256sum | cut -d" " -f1
a115236c51dd5498e7683f79d9de387c842f70c258f69719855184ed54ecea56
```

Depois, vamos atribuir o hash gerado acima ao parâmetro ***root_password_sha2*** do arquivo de configuração do Graylog.

```shell
# vim /etc/graylog/server/server.conf
```

```diff
...
- root_password_sha2 =
+ root_password_sha2 = a115236c51dd5498e7683f79d9de387c842f70c258f69719855184ed54ecea56
...
```

Para acessar publicamente o Graylog, defina o endereço IP correto para os parâmetros ***http_bind_address*** e ***http_publish_uri***:

```shell
# vim /etc/graylog/server/server.conf
```

```diff
...
- http_bind_address = 127.0.0.1:9000
+ http_bind_address = 10.3.1.26:9000

...

- http_publish_uri = http://127.0.0.1:9000/
+ http_publish_uri = http://10.3.1.26:9000/
...
```

Altere também o valor do parâmetro ***elasticsearch_shards*** para 1, pois iremos utilizar um Elasticsearch de nó único.
```shell
# vim /etc/graylog/server/server.conf
```

```diff
...
- elasticsearch_shards = 4
+ elasticsearch_shards = 1
...
```

### Executando o Graylog
Execute os comandos abaixo para iniciar e permitir que o servidor Graylog seja executado na reinicialização do sistema.

```shell
# systemctl start graylog-server
# systemctl enable graylog-server
```

### Acessando a interface web do Graylog

Agora que o Graylog está rodando, você pode acessá-lo através de um browser usando o endereço: **http://ip_do_servidor:9000**.

<img src="img/graylog-login.png" alt="Tela de login Graylog" style="margin-left: 20%;margin-top:10px;margin-bottom:10px;">

Entre com o nome de usuário **admin** e a senha cujo hash foi gerado acima. Ao fazer o login, teremos acesso ao dashboard do Graylog.

<img src="img/graylog-dashboard.png" alt="Dashboard - Graylog" style="margin-left: 20%;margin-top:10px;margin-bottom:10px;">








