---
layout: post
title: SQLite3 - Importando e exportando dados pela linha de comando
description: "Demonstração de manipulação de banco da dados pelo terminal"
modified: 2015-08-11
tags: [sqlite3, terminal]
image:
  feature: SQLite370.svg
  credit: D. Richard Hipp
  creditlink: https://commons.wikimedia.org/wiki/File:SQLite370.svg#/media/File:SQLite370.svg
---

Usando SQLite3 pelo terminal podemos acabar inicializando-o sem especificar o banco que queremos, escrevendo apenas

    {% highlight bash %}
user@host:~$ sqlite3
    {% endhighlight %}

O que implica que o trabalho que executarmos a partir daqui não será salvo automaticamente. Também pode ocorrer de inicializarmos um banco já existente, criarmos tabelas/inserirmos dados e, só depois, percebermos que não era naquele banco que queríamos inserir os dados.

Considerando o primeiro caso apresentado, a solução é simples e direta, podemos usar o comando `.backup`. Para isso, basta escrever:

    {% highlight sqlite3 %}
sqlite> .backup meuBanco.db
    {% endhighlight %}

Sendo que `meuBanco.db` foi o nome que escolhi como novo banco de dados (caso já exista um arquivo com o nome escolhido, seus dados serão sobrescrito).

Em seguida basta sair da solução atual e entrar no banco criado

    {% highlight bash %}
sqlite> .exit
user@host:~$ .sqlite3 meuBanco.db
sqlite> 
    {% endhighlight %}

No segundo caso, consideremos os seguintes bancos:

    {% highlight sqlite3 %}
user@host:~$ .sqlite3 meuBanco.db
sqlite> .schema
CREATE TABLE alunos(matricula INTEGER NOT NULL, nome VARCHAR(255) NOT NULL))
CREATE TABLE professores(registro VARCHAR(255) NOT NULL, nome VARCHAR(255) NOT NULL))
    {% endhighlight %}

e o seguinte banco (sendo que a última tabela de `teste.db` foi criada e preenchida, porém, ela deveria estar em `meuBanco.db`)

    {% highlight sqlite3 %}
user@host:~$ .sqlite3 teste.db
sqlite> .schema
CREATE TABLE irrelevante1(nome VARCHAR(255), sobrenome VARCHAR(255)))
CREATE TABLE irrelevante2(idade INTEGER, acertos INTEGER NOT NULL))
CREATE TABLE estacionamento(vaga INTEGER NOT NULL, cliente VARCHAR(255) NOT NULL, identificador INTEGER))
    {% endhighlight %}

Para transferirmos a última tabela para `meuBanco.db` podemos seguir os paços:

    {% highlight sqlite3 %}
user@host:~$ .sqlite3 teste.db
sqlite> .output temp.sql
sqlite> .dump estacionamento
sqlite> .output stdout
    {% endhighlight %}

Sendo que os comando:

**.output temp.sql** faz com que o resultado de coisas que normalmente seriam exibidas no terminal, como comando de listagem de tabela, seja enviado para um arquivo (no caso, o nome escolhido para o arquivo foi `temp.sql`), caso já exista um arquivo com o nome escolhido ele será sobreescrito.

**.dump estacionamento** despeja o conteúdo da tabela escolhida em formato SQL (caso não seja especificada uma tabela, será despejado todo o conteúdo do banco). Como optamos por jogar as saídas em um arquivo específico (com o comando anterior), os dados serão despejados naquele arquivo.

**.output stdout** faz com que a saída volte para a tela, ou seja, a partir daqui os comandos efetuados serão efetuados serão aplicados ao banco atual, e não mais ao arquivo escolhido anteriormente.

Agora podemos importar a tabela para `meuBanco.db`. Uma forma da fazer isso (caso não exista uma tabela com o nome `estacionamento` no banco)

    {% highlight bash %}
user@host:~$ .sqlite3 teste.db < temp.sql
    {% endhighlight %}

se a tabela já existir devemos exluí-la antes da importação dos novos dados, caso isso não seja feito haverá dados duplicados na tabela resultante (já que, na importação, ocorre a união entre as tabelas).

Alguma dúvida, correção ou adendo sobre o que foi postado? Escreva um comentário ou mande um email :D 
