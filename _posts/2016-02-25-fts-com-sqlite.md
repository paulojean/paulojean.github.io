---
layout: post
title: Full Text Search com SQLite
description: "Utilizando full text search com SQLite"
modified: 2015-08-23
tags: [fts, sqlite, mobile]
image:
    feature:
    credit:
    creditlink:
feature: 
credit: 
creditlink: 
---

Esse post tem alguns objetivos.

O primeiro, que acaba sendo específico para Windows Phone, é informar é possível utilizar [Full Text Search](https://www.sqlite.org/fts3.htm) (FTS) no SQLite (ao menos utilizando o [SQLite.Net](https://github.com/oysteinkrog/SQLite.Net-PCL) acaba sendo bem simples) e que ele já vem habilitado por padrão. Esse "informe" pode parecer idiota (e talvez até seja mesmo), mas não há muita informação por aí sobre isso, então não custa nada frisar ;)

Já no caso do Android e iOS encontra-se mais facilmentent informaões a respeito de quais bibliotecas já trazem o FTS habilitado por padrão.

O segundo é falar do FTS em si e mostrar como usá-lo. Para isso é bom começarmos vendo o que ele é:

[Full Text Search](https://en.wikipedia.org/wiki/Full_text_search) é a técnica de busca de dados por meio de palavras ou expressões em uma tabela.


Para explicar a ideia, vamos considerar o exemplo de um cardápio. Nesse cardápio nós tempos os pratos (que será a representação de nossa tabela) e os pratos possuem:

{% highlight bash %}
- título
- descrição
- ingredientes
{% endhighlight %}

##Criando uma tabela

Então vamos criar uma tabela onde os pratos ficarão guardados. Criar uma tabela onde possamos utilizar FTS é bem simples, basta executar o comando

{% highlight sql %}
CREATE VIRTUAL TABLE pratos USING fts4(titulo, descricao, ingredientes)
{% endhighlight %}

Onde `pratos` é o nome da tabela e `titulo`, `descricao` e `ingredientes` são nosso atributos.

A ideia é que, quando formos procurar por pratos de forma genérica, ou seja, não temos um `id` (e/ou não queremos procurar por um prato específico etc etc), temos apenas uma palavra, por exemplo, *pimenta* e queremos obter todos os pratos que possuem essa palavra, não importando se ela está em um ou mais atributos, nós consigamos fazer isso de forma simples e otimizadada. Aqui é importante ressaltar que não faria sentido realizar uma busca textual em um campo `boolean` então **ao criar uma tabela com FTS todas as colunas serão interpretadas como texto**.

##Inserindo dado

Antes de saber como realizamos a pesquisa, é importante sabermos como inserimos dados:

{% highlight sql %}
INSERT INTO pratos VALUES('Macarrão apimentado', 'um bom macarrão', 'macarrão pimenta salsinha')
INSERT INTO pratos VALUES('Omelete', 'omelete especial', 'ovo sal queijo')
INSERT INTO pratos VALUES('Burrito com pimenta', 'burrito apimentado', 'carne moída cebola pimenta')
{% endhighlight %}

##Buscando dados

E para procurarmos por pratos que possuem a palavra **pimenta**

{% highlight sql %}
SELECT * FROM pratos WHERE pratos MATCH 'pimenta'
{% endhighlight %}

É possível fazer queries mais interessantes, como aponta na própria documentação do [SQLite](https://www.sqlite.org/fts3.html#section_3).

Bem, a busca feita anteriormente trará os dados dos pratos **Macarrão apimentado** e **Burrito com pimenta**.

##Ranquando os dados

É importante notar que, nesse exemplo não teremos uma ordenação inteligente. Isso é, os itens serão trazidos na ordem em que forem encontrados. Mas há casos em que é interessante mudar a ordem dos itens baseado na busca, por exemplo: se o termo buscado estiver no *título* e nos *ingredientes* de um prato e só nos *ingredientes* de outro, seria interessante que o primeiro estivesse à frente do segundo.

Felizmente, o SQLite provê funções que possibilitam isso. Uma delas é a [matchinfo](https://www.sqlite.org/fts3.html#matchinfo), uma função que retorna um `blob` contendo informações a respeito do resultado da busca.

Utiliazar essa função é bem fácil, basta chamá-la da seguinte maneira

{% highlight sql %}
SELECT *, matchinfo(pratos) FROM pratos WHERE pratos MATCH 'pimenta'
{% endhighlight %}

Assim obteremos, além de todos os atributos da tabela, um quarto atributo no formato de `byte array`, com ele em mãos, basta aplicar a regra que ranqueamente que se queira e pronto! [Aqui](https://www.sqlite.org/fts3.html#appendix_a) você vê dicas de como trabalhar em cima disso.

Tá, acho que já temos uma quantidade considerável de informações, mas em que situação seria interessante usar isso, ao invés de deixar o servidor fazer a busca pra a gente?

Um caso é justamente quando o dispositivo não tiver acesso ao servidor (por exemplo, no caso de um app com suporte a acesso offline) mas tivermos os dados em cache e queiramos oferecer uma busca um pouco mais completa ao usuário, mesmo offline.