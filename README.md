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





