---
layout: post
title: "A estrutura de dados do elasticsearch e sua API de indexação"
date: 2013-02-06 23:45
comments: true
categories:
---

Estrutura de dados
------------------

A estrutura de dados do elasticsearch é baseada em documentos e campos. O documento
é modelado como um objeto JSON:

{% codeblock lang:javascript %}
{
    "id": "paulaner",
    "nome": "paulaner",
    "website": "http://www.paulaner.com/",
    "localizacao": {
      "pais": "Alemanha",
      "cidade": "Munique"
    }
}
{% endcodeblock %}

Existem campos de metadados que fazem parte do núcleo do elasticsearch:
    _id         => Identificador único de um documento
    _type       => Tipo do documento
    _source     => Armazena o documento original que foi indexado
    _all        => Campo que indexa todos os valores de todos os campos do documento
    _timestamp  => Timestamp em que o documento foi indexado
    _ttl        => Define um tempo de expiração do documento
    _size       => Indexa o tamanho do campo _source

Os tipos de dados que os campos dos documentos podem assumir são:

**Tipos principais**: string, number, boolean, datetime, binary (base64)<br>
**Tipos complexos**: Array e Object<br>
**Outros tipos**: geo_point, ip, attachment, multi-field

<!-- more -->

API de criação de índices
-------------------------

A API de criação de índice é bem simples, utiliza o verbo PUT e o nome do índice.
Opcionalmente é possível passar algumas configurações, como o número de shards e
replicas e o mapping específico do índice:

{% codeblock lang:bash %}
curl -XPUT 'localhost:9200/cervejas' -d '{
    "settings": {
      ...
    },
    "mappings": {
      ...
    }
}'
{% endcodeblock %}

API de indexação de documentos
------------------------------

A API de indexação recebe um índice, tipo e opcionalmente um identificador:

{% codeblock lang:bash %}
curl -XPUT 'localhost:9200/cervejas/trigo/Weihenstephaner' -d '{
    "tipo": "German Hefeweizen",
    "teorAlcoolico": 5.4,
    "origem": "Alemanha"
}'

{
    "ok": true,
    "_index": "cervejas",
    "_type": "trigo",
    "_id": "Weihenstephaner",
    "_version": 1
}
{% endcodeblock %}

O elasticsearch responde com dois status code:<br>
201 (CREATED) se o documento é novo e foi criado corretamente<br>
200 (OK) se o documento foi atualizado

A parte do _id na URL é opcional. Alternativamente é possível extrair essa informação
de um campo do documento ou pode ser gerado automaticamente. Nesse caso deve-se usar
o verbo <strong>POST</strong>.

{% codeblock lang:bash %}
curl -XPOST 'localhost:9200/cervejas/trigo' -d '{
    "tipo": "German Hefeweizen",
    "teorAlcoolico": 5.4,
    "origem": "Alemanha"
}'

{
    "ok": true,
    "_index": "cervejas",
    "_type": "trigo",
    "_id": "2txgvYM5Tiy_su-ZB-LbqQ",
    "_version": 1
}
{% endcodeblock %}

A operação de índice também é utilizada para reindexação. Baseada no id de um documento se um documento
o mesmo id já existir o mesmo será substituído pelo novo. A reindexação na verdade é um operação de
delete seguida de uma nova indexação, isso porque o Lucene não aceita operações de reindexação. Caso
seja necessário criar um documento somente se ele não exisir basta utilizar o operador _create:

{% codeblock lang:bash %}
curl -XPUT 'localhost:9200/cervejas/trigo/Weihenstephaner/_create' -d '{
    "tipo": "German Hefeweizen",
    "teorAlcoolico": 5.4,
    "origem": "Alemanha"
}'

{
    "error": "DocumentAlreadyExistsException[[cervejas][0] [trigo][Weihenstephaner]: document already exists]",
    "status": 409
}
{% endcodeblock %}

O índice será criado automaticamente se necessário, utilizando as configurações e mappings default.
A criação automática pode ser desativada utilizando configuração a nível de nó:

{% codeblock lang:bash %}
action.auto_create_index: false
{% endcodeblock %}

Por fim vamos falar de como funciona a execução distribuída da API de indexação. Vejam a ilustração
a seguir:

{% img center /images/index_execution.png %}

(I) Uma requisição de indexação é endereçada a um nó específico.<br>
(II) O elasticsearch calcula o hash (por default utiliza o valor do _id) e escolhe
o shard em que o documento deverá ser armazenado (shard 1 do exemplo). Acha onde o
shard primário está armazenado e redireciona o request para ele.<br>
(III) Uma vez que a request foi executada com sucesso, replica o documento em todas as
réplicas desse shard.
