---
layout: post
title: "Como adicionar uma camada de segurança sobre o elasticsearch: autenticação e autorização"
date: 2013-03-06 08:37
comments: true
categories: 
---

Atualmente o elasticsearch não provê nenhuma camada de segurança para as chamadas à API REST.
Existem algumas issues abertas no github pedindo a implementação dessa camada, incluindo,
<a href="https://github.com/elasticsearch/elasticsearch/issues/1379" target="_blank">autenticação e autorização</a>
e <a href="https://github.com/elasticsearch/elasticsearch/issues/664" target="_blank" >suporte HTTPS</a>. Porém os
desenvolvedores do elasticsearch já sinalizaram que essas issues tem baixa prioridade pois podem ser contornadas
com proxies e plugins. Além disso o elasticsearch sempre deve ser utilizado em ambientes confiáveis, como a rede
interna de sua empresa, por exemplo. Por tanto a melhor forma de deployar o elasticsearch é com um webserver na 
frente com a responsabilidade de controlar o acesso e os tipos de queries que poderão ser feitas. Sabendo
disso, existem algumas alternativas que são mais usadas, vamos dar uma passada por cada uma delas.

<!-- more -->

Elasticsearch-jetty plugin
--------------------------
O <a href="https://github.com/sonian/elasticsearch-jetty" target="_blank">elasticsearch-jetty plugin</a> suporta 
conexões SSL, autenticação HTTP básica e loga todas as requests em formato de texto ou JSON.

Ele pode ser instado como qualquer outro plugin para elasticsearch:

{% codeblock lang:bash %}
$ bin/plugin -url \ 
  https://oss-es-plugins.s3.amazonaws.com/elasticsearch-jetty/elasticsearch-jetty-0.20.1.zip \
  -install elasticsearch-jetty-0.20.1
{% endcodeblock %}

O que ele faz é substituir o NettyHttpServerTransport pelo JettyHttpServerTransport. Para ativá-lo
é preciso desativar o netty http transport adicionando a configuração a seguir no arquivo de configuração
elasticsearch.yml:

{% codeblock lang:bash %}
http.type: com.sonian.elasticsearch.http.jetty.JettyHttpServerTransportModule
{% endcodeblock %}

Feito isso é necessário configurar os arquivos jetty*.xml para as funcionalidades desejadas. Na página
do <a href="https://github.com/sonian/elasticsearch-jetty#configuration-files" target="_blank">plugin no github</a> esses
passos estão muito bem explicados.

NGINX
-----
O NGINX é um poderoso webserver que pode ser usar como um proxy reverso por cima do elasticsearch
restringindo rotas e fazendo autenticação HTTP básica.

Costuma-se, por exemplo, restringir o acesso à Cluster API, permitir somente o verbo GET ou até mesmo
só permitir o acesso a um índice para um usuário devidamente autenticado.

A seguir o trecho do arquivo de configuração do NGINX que faz isso:

{% codeblock %}
server { 
    listen       8080;
    server_name  es.suaempresa.com;

    error_log   elasticsearch-errors.log;
    access_log  elasticsearch.log;

    location / {

        # Proíbe o acesso à Cluster API
        if ($request_filename ~ "_cluster") {
            return 403;
            break;
        }

        # Permite somente o método GET
        if ($request_method !~ "GET") {
            return 403;
            break;
        }

        # Repassa a requisição para o elasticsearch
        proxy_pass http://localhost:9200;
        proxy_redirect off;

        proxy_set_header  X-Real-IP  $remote_addr;
        proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header  Host $http_host;

        # Autentica o usuário
        auth_basic           "elasticsearch";
        auth_basic_user_file arquivo_de_senhas;

        # Roteia todas as requisições para os índices de 
        # propriedade do usuário autenticado
        rewrite  ^(.*)$  /$remote_user$1  break;
        rewrite_log on;

        return 403;
    }
}
{% endcodeblock %}

Outras medidas de segurança
---------------------------

Além de proteger a API de acessos indevidos através de um webserver devemos nos atentar
também para as configurações de segurança que o próprio elasticsearch provê. Com elas
podemos nos prevenir de catastrofes como apagar todos os índices de uma vez, criação
de índices desnecessários, desligar nós e evitar um splitbrain.

Para desativar a criação automática de índices, a deleção de todos os índices de uma vez
e o shutdown do nós basta acrescentar as configurações a seguir no arquivo elasticsearch.yml:

{% codeblock %}
action.auto_create_index: 0
action.disable_delete_all_indices: true
action.disable_shutdown: true
{% endcodeblock %}

O splitbrain é um evento catastrófico para o elasticsearch. Ele acontece quando
o cluster se divide em dois e ambos elegem um master, a partir daí os dois lados
do cluster irão divergir e não será possível restaurar o cluster sem matar uma
metade do cluster. Para evitar esse cenário podemos utilizar a seguinte configuração:

{% codeblock %}
discovery.zen.minimum_master_nodes: N/2 + 1
{% endcodeblock %}

onde N é o número de máquinas que compões o cluster.
