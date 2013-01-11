---
layout: post
title: "Migrando da FAST ESP para o elasticsearch"
date: 2013-01-11 00:16
comments: true
categories: 
---
Durante o ano de 2012 me deparei com um grande desafio profissional: migrar a plataforma de busca da globo.com 
para uma nova solução. Nós tinhamos a busca toda baseada na 
<a href="http://www.microsoft.com/enterprisesearch/en/us/fast-customer.aspx">FAST</a>, uma solução
de busca que foi incorporada pela Microsoft e como consequência deixou de ter suporte a servidores
UNIX, o que já não mais nos atendia.

A nova solução de busca deveria atender algumas características, como por exemplo, ser de preferência
open source, escalável, possuir uma API bem definida e principalmente não limitar a criatividade dos
desenvolvedores com schemas engessados. Além disso, era desejável que a solução fosse distribuída para
que pudesse ser implantada na cloud da globo.com que está saindo do forno.

Dentre as várias soluções que foram levantadas o <a href="http://elasticsearch.org">Elasticsearch</a>
foi a que mais atendia aos requisitos. Ele é uma máquina de busca construída sobre o 
<a href="http://lucene.apache.org/">Apache Lucene</a> escalável e distribuída, RESTful,
com um setup bastante intuitivo e livre de schema. Além disso provê busca em real-time e 
<a href="http://en.wikipedia.org/wiki/Multitenancy">Multitenancy</a> de maneira simples.

<!-- more -->
