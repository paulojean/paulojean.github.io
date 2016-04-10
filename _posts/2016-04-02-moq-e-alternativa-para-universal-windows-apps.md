---
layout: post
title: Moq e uma alternativa em Universal Windows Apps
description: "Utilizando mocks em .Net e Universal Windows Apps"
modified: 2015-04-09
tags: [uwp, testes, mock, moq, lightmock]
image:
    feature:
    credit:
    creditlink:
feature: 
credit: 
creditlink: 
---

Hoje em dia boa parte dos desenvolvedores já reconhece as vantagens que ter testes automatizados trás para a manutenção de uma aplicação (e para sua saúde mental, já que refatorar/acrescentar funcionalidades sem ter uma forma automática de atestar que nada foi "quebrado" é uma boa fonte de dor de cabeça).

E, em se tratando de testes de unidade, é fundamental que tenhamos uma forma de simular o comportamento de alguns objetos. Isso porque:

- Quando estamos testando uma classe, não importa como as suas dependências são implementadas, só estamos interessados em o que elas retornam.
Por exemplo: queremos testar o método `Cadastrar(Artefato artefato)` da classe `Estante`. Digamos que esse `Cadastrar` faça algumas coisas e, em seguida, chame um banco de dados para salvar o conteúdo. Dado o que queremos testar, não é interessante que precisemos instanciar um banco de dados e que ele seja chamado pela nossa classe (afinal estamos testando um método de nossa classe e não a persistência no banco), então é aí que entra o `simular o comportamento de alguns objetos`. Ao invéz de inicializarmos nossa classe com um banco real, podemos inicializar com outra classe que simula o comportamento do banco (recebendo os mesmos parâmetros que o banco receberia e retornando os mesmos tipos).

## Moq
Bem, agora vejamos como podemos fazer isso utilizando o [Moq](https://github.com/moq/moq4/)

    {% highlight csharp %}
var mock = new Mock<BancoDeDados>();
mock.Setup(meuBanco => meuBanco.Salvar(It.Is<Artefato>(artefato => artefato != null)))
    .Returns(true);
var estante = new Estante(mock.Object);
estante.Salvar(null);
    {% endhighlight %}

Vamos por partes:

- `var mock = new Mock<BancoDeDados>():` inicializamos o mock informando a classe/interface que iremos *mockar* (mas, saiba que, dado como o Moq é implementado, em classes só consigueremos sobrescrever métodos `virtual`).
- `mock.Setup(meuBanco => meuBanco.Salvar(...)`: informamos qual método iremos simular
- `It.Is<Artefato>(artefato => artefato != null)`: isso é o parâmetro que aceitaremos, escrevendo dessa forma podemos filtrar quais aceitaremos, da forma como está estamos informando que aceitaremos qualquer coisa que seja `Artefato` mas que seja diferente de nule (e, por isso **esse o teste falhará, já que estamos passando nulo na última linha**)
- `.Returns(true)`: como sugere, informamos o retorno. Ou seja, dado um parâmetro que bata com o que definimos no passo anterior, `meuBanco.Salvar()` sempre retornará `true`.
- `new Estante(mock.Object)`: inicializamos a classe que queremos testar.
- `mock.Object`: Pegamos o objeto *mockado* (que virá com todas as configurações que passamos em `mock.Setup`)
- `estante.Salvar(null)`: executamos o método que estamos testando (como dito anteriormente, esse teste falhará, já que estamos passando um parâmetro que o `mock` não saberá tratar)

Uma coisa de utilizar `mock`s é que não precisamos implementar todo um objeto, sendo que ele nem é o que importa. Outra coisa é que definiremos comportamentos apenas para os métodos do `mock` que serão utilizados, o que trás outra vantagem: se tentarmos invocar um método que não foi definido no `mock.Setup()` o teste falhará, sendo assim garantimos que, durante a execução do método que estamos testando, nenhum outro método da classe de `mocck` é chamada.

Bem, tudo isso é muito legal e facilita bastante coisa, mas (infelizmente) não podemos utilizar o [Moq em Universal Windows Apps](https://github.com/moq/moq4/issues/195), isso devido ao proxy que ele utilizar para sobrescrever os métodos com nossas definições. Sendo assim, precisamos de uma alternativa.

## Universal Windows Apps
Em UWA não há uma opção que chegue a ser tão boa quanto o `Moq`, mas possui algumas que até que quebram um galho.

A primeira é o [mMckJockey](http://tylerhoersch.com/mockJockey). Ele trás algumas facilidades, como a forma como faz o setup do `mock`, mas ele não tem muitas funcionalidades importantes, como acesso ao número de vezes que um método foi chamado.
Outra, que é a que considero mais completa, é o [LightMock](https://github.com/seesharper/LightMock), e é deles que falaremos de forma mais detalhada.

## LightMock
Pra falarmos dele precisamos falar do seu problema (até aonde usei, é o único): você precisa criar uma classe de mock pra a interface que vai *mockar*. Isso é consideravelmente irritante, mas dados as limitações do UWA, do que o LightMock fornece e por não ter uma alternativa tão completa, acaba sendo aceitável.

Vamos ao que precisamos:

Nossa interface

    {% highlight csharp %}
public interface MeuBanco{
    bool Salvar(Artefato artefato);
}
    {% endhighlight %}

Uma classe de `mock` para a interface

    {% highlight csharp %}
public class MeuBancoMock : MeuBanco{
    private readonly IInvocationContext<IFoo> context;

    MockMeuBanco(IInvocationContext<IFoo> context){
        this.context = context;
    }

    public bool Salvar(Artefato artefato) => context.Invoke(b => b.Salvar(artefato));
}
    {% endhighlight %}

E o nosso teste

    {% highlight csharp %}
var mockContext = new MockContext<MeuBanco>();
var meuBando = new MeuBancoMock(mockContext);
mockContext.Arrange(b => b.Salvar(It.Is<Artefato>(artefato => artefato != null)))
                .Returns(true);
var estante = new Estante(meuBanco);
estante.Salvar(new Artefato());
mockContext.Assert(b => b.Salvar(It.Is<Artefato>(artefato => artefato != null)), 
                Invoked.Once);
    {% endhighlight %}

No teste em si, a diferença em relação ao Moq é que precisamos de um passo intermediário (instanciar a classe de mock), fora isso ele nos dá basicamente o mesmo que o Moq (dadas as limitações da plataforma). Por esse motivo, o considero a melhor opção como biblioteca de `mock` para Universal Windows Apps.