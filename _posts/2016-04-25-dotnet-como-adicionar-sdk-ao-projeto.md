---
layout: post
title: Como adicionar dependências de SDK a um projeto .Net
description: ".Net: Adicionando dependências de SDK ao projeto"
modified: 2016-04-25
tags: [.net, visual-studio]
image:
    feature:
    credit:
    creditlink:
feature: 
credit: 
creditlink: 
---

Nesse post vamos ver como adicionar dependências de SDKs ao projeto, por exemplo o SQLite em um projeto UWA, pois quando precisamos utilizá-lo não o pegamos por meio do Nuget e sim instalamos a extensão ao Visual Studio e então adicionamos uma referência ao projeto.

Mas, antes de ver como, precisamos ter um porque de fazer isso. E eu encontrei dois motivos:

  - Facilitar o desenvolvimento quando há mais de uma pessoa alterando o código do projeto. Isso porque, como se trata de uma SDK, se alguém atualizá-la (em seu Visual Studio) todas as outras pessoas terão que atualizar também. Além disso, no setup de um novo ambiente é preciso ficar lembrando quais extensões estão instaladas, se a extensão estiver no projeto bastará instalar os pacotes do Nuget e pronto.
  - Possibilitar CI. Em UWA é comum precisar de dependências que são SDKs, mas se você tentar fazer o build utilizando o [MSBuild](https://github.com/Microsoft/msbuild) (que é como o [VSTS](https://www.visualstudio.com/en-us/products/visual-studio-team-services-vs.aspx) e o [AppVeyor](https://www.appveyor.com) realizam o build) você verá que o build falhará, no caso de estar usando o [SQLite](https://visualstudiogallery.msdn.microsoft.com/4913e7d5-96c9-4dde-a1a1-69820d615936) será exibida uma mensagem de erro contento o seguinte `Could not find SDK "SQLite.WP81, Version=X.XX.X"`. Isso porque as dependências do projeto que foram obtidas por meio de alguma SDK não estarão acessíveis ao MSBuild.

Agora vamos ver como fazer isso:

## Obtendo os arquivos necessárois
O primeiro passo é encontrar os arquivos da SDK, isso pode ser feito abrindo a parte de referências do projeto, clicando com o botão direito na referência que quer e selecionando `Propriedades`:

<img src="{{ site.url }}/images/selecionando-propriedades-visual-studio.png" alt="Selecionando propriedades de uma referência no Visual Studio">

Em seguida basta copiar o `Path` do arquivo

<img src="{{ site.url }}/images/selecionando-path-de-referencia-visual-studio.png" alt="Visualizando Path de referência no Visual Studio">

Nesse caso, o `Path` do SQLite.WP1 está apontando para `C:\Program Files (x86)\Microsoft SDKs\WindowsPhoneApp\v8.1\ExtensionSDKs\SQLite.WP81\3.12.1\`. Devemos voltar algumas pastas, ficando em `C:\Program Files (x86)\Microsoft SDKs\WindowsPhoneApp\v8.1\ExtensionSDKs\`. Aqui vamos copiar a pasta `SQLite.WP81`.

Bem, achamos os arquivos que queríamos, agora vamos ver aonde precisamos colocá-los.

## Salvando no projeto

Vamos considerar que o nosso projeto está com a seguinte estrutura de arquivos:
{% highlight cmd %}
SampleApp/
SampleApp.sln
{% endhighlight %}

E que queremos guardar as SDKs dentro de uma pasta chamada `libs`, ficando com a seguinte estrutura:

{% highlight cmd %}
SampleApp/
libs/
SampleApp.sln
{% endhighlight %}

Nós não podemos simplesmente jogar a pasta que copiamos dentro da pasta `libs`, pois o Visual Studio não sabeira encontrá-la durante o `build`, precisamos seguir o padrão de pastas que há dentro de `C:\Program Files (x86)\Microsoft SDKs\`. Pra ficar mais claro, vamos considerar que o projeto contempla Windows e WindowsPhone (ambos na versão 8.1), precisaremos criar a seguinte estrutura de pasta dentro de `libs/`

{% highlight cmd %}
SampleApp/
libs/
  Windows/v8.1/ExtensionSDKs/
  WindowsPhoneApp/v8.1/ExtensionSDKs/
SampleApp.sln
{% endhighlight %}

Agora jogaremos a pasta que copiamos anteriormente para dentro dos subdiretórios `ExtensionSDKs/`.

**OBS:** lembre-se de fazer o processo para obter o `Path` das referências para cada projeto, e não jogar para dentro da pasta `Windows/v8.1/ExtensionSDKs/` uma pasta copiada de uma referência do WindowsPhone e vice-versa.

Feito isso, já temos tudo preparado, agora precisamos indicar no nosso projeto que quando adicionarmos uma referência dessa SDK queremos que ele pegue a que está dentro da pasta `libs/` e não da padrão. Fazermos isso editando o `.csproj` de cada projeto, precisamos adicionar o seguinte código:

{% highlight cmd %}
<Project ...>
    <PropertyGroup>
        <SDKReferenceDirectoryRoot>
         $(SolutionDir)libs;$(SDKReferenceDirectoryRoot)
        </SDKReferenceDirectoryRoot>
    </PropertyGroup>
</Project>
{% endhighlight %}

Preste atenção no trecho `$(SolutionDir)libs;` é aí que indicamos onde deixamos as SDKs. Se você optou por outro nome de pasta, deve modificar o trecho a cima.

Após isso, talvez seja necessário reiniciar o Visual Studio. Em seguida, ao visualizar o `Path` da referência que moveu para o projeto, ela deve estar da seguinte forma:

{% highlight cmd %}
caminho/para/seu/projeto/libs/WindowsPhoneApp/v8.1/ExtensionSDKs/SQLite.WP81/3.12.1
{% endhighlight %}

Caso ainda não tenha atualizado, você pode ir em `References -> Add Reference`, na parte de extensões estarão listadas as duas referências para a SDK, então basta marcar a que estiver com o `Path` do projeto e clicar em `Ok`.