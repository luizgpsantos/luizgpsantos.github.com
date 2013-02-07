---
layout: post
title: "Introdução ao elasticsearch"
date: 2013-02-04 14:27
comments: true
categories: 
---

Prover uma busca textual para aplicações tem se tornado cada vez mais fácil com as máquinas
de busca open source disponíveis na internet. O elasticsearch, por exemplo é uma máquina de
busca cujo modelo de dados é baseado em banco de dados schema-free (não é necessário definir
a estrutura dos dados a priori) e orientados a documentos. Esse modelo como já mostrado pelo
movimento #nosql é bastante efetivo para se criar aplicações. O modelo adotado pelo elasticsearch
é o <a href="http://json.org">JSON</a> e a máquina por trás é o poderoso 
<a href="http://lucene.apache.org/">Apache Lucene</a>.

Uma das principais características do elasticsearch que chama atenção é o fato dele ser distribuído
o que permite uma fácil escalabilidade 
<a href="http://en.wikipedia.org/wiki/Scalability#Scale_horizontally_vs._vertically">tanto vertical como horizontal</a> 
e alta disponibilidade. Diversas plataformas de cloud se aproveitam dessa natureza para oferecê-lo como serviço.

O elasticsearch é capaz de fazer consultas livres como: ache todos os fabricantes de cerveja, consultas
estruturadas como: ache todos os fabricantes de cerveja fundadas entre 2011 e 2012. É possível também
classificar resultados como: retorne o número de latinhas de cerveja fabricadas por cada fabricante. E
combinar isso tudo: retorne o número médio de latinhas de cerveja fabricadas por cada empresa no período
de 2011 a 2012, tudo em tempo quase real!

<!-- more -->

A distribuição do elasticsearch inclui os seguintes diretórios e arquivos:

![Arquivos e diretórios da distribuição do elastisearch](/images/tree_elasticsearch.png "Arquivos e diretórios da distribuição do elastisearch")

Dentro do diretório *bin* estão os scripts executáveis, dentro de config estão os arquivos de configuração
do cluster e de logging. Dentro de lib estão as bibliotecas e ainda existe o velho diretório de logs.

Para startar um nó basta executar o comando:

{% codeblock lang:bash %}
# executa em foreground
bin/elasticsearch -f

# executa em background
bin/elasticsearch

# loga o arquivo de pid
bin/elasticsearch -p /var/elasticsearch.pid
{% endcodeblock %}

Algumas variáveis de ambiente importantes:

{% codeblock lang:bash %}
ES_HOME       -> O diretório raiz da instalação
ES_HEAP_SIZE  -> O HEAP da JVM
ES_JAVA_OPTS  -> Parâmetros adicionais para a JVM
{% endcodeblock %}

Uma introdução rapidinha à API:

{% codeblock lang:bash %}
# Indexar um documento
curl -XPUT 'localhost:9200/cerveja/weissbeer/paulanerhefeweissbeer' -d '{
    "pais": "Alemanha",
    "ingredientes": "Água, maltes de trigo e cevada, levedura e lúpulo.",
    "temperatura": "5-7 ºC"
  }'

# Buscar documentos
curl -XGET 'localhost:9200/cerveja/_search?q=paulaner'

# Recuperar um documento específico
curl -XGET 'localhost:9200/cerveja/weissbeer/paulanerhefeweissbeer'

# Apagar um documento
curl -XDELETE 'localhost:9200/cerveja/weissbeer/paulanerhefeweissbeer'
{% endcodeblock %}

No próximo post irei explicar com mais detalhes como utilizar a API do elasticsearch para indexar,
recuperar e apagar documentos, com algumas dicas importantes para quem deseja fazer um uso mais avançado
da ferramenta. Até lá!
