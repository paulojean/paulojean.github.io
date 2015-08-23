---
layout: post
title: Teste de unidade em Haskell 
description: "Exemplos de teste de unidade em haskell"
modified: 2015-08-23
tags: [haskell, testes, hspec]
image:
    feature:
    credit:
    creditlink:
feature: 
credit: 
creditlink: 
---
       
Acredito que uma das coisas mais importantes ao se aprender uma nova linguagem é a fazer testes nela. Isso porque, com testes, podemos validar rapidamente e, mais importante, de forma automática o que acabamos de escrever.

Em Haskell, pelo que vi, a melhor opção é o [HSpec](http://hspec.github.io/). Uma alternativa seria o [HUnit](http://hackage.haskell.org/package/HUnit), mas basta uma olhada rápida nos exemplos pra perceber que o HSpec é muito mais legível e prático.

##Instalando
Você pode obter o HSpec pelo `cabal` com o comando

    {% highlight bat %}
cabal update && cabal install hspec
    {% endhighlight %}
##Escrevendo os testes
A escrita dos testes são bem simples, um exemplo de arquivo de teste

    {% highlight haskell %}
---arquivo MultipleOf3And5Spec.hs
module MultipleOf3And5Spec where

import Test.Hspec
import MultipleOf3And5
import Prelude hiding (sum)

main :: IO ()
main = hspec $ do
    describe "sum of natural numbers, multiples of 3 and 5" $ do
        it "return the sum below 10" $
            sum 10 `shouldBe` 23
        it "return the sum below 1000" $
            sum 1000 `shouldBe` 233168

    {% endhighlight %}

considerando que estamos testando as funções do arquivo

    {% highlight haskell %}
---arquivo MultipleOf3And5.hs
module MultipleOf3And5 where
        
sum :: Int -> Int
sum n = Prelude.sum $ filter isMultipleOf3Or5 numbers
        where   isMultipleOf3Or5 = (\x -> x `mod` 3 == 0 || x `mod` 5 == 0)
                numbers = [1..n-1]
    {% endhighlight %}

Ou seja, no nosso arquivo de teste precisamos importar o `Teste.Hspec` e demais módulos necessários.

Os `describe` servem para que agrupemos os testes que pertencem a mesmas categorias (isso facilitará a leitura ao executarmos). Cada `it`, idealmente, terá apenas uma asserção.

Nesse exemplo foi usado apenas a função `shouldBe` para validarmos as expectativas, mas o HSpec tem outras, como `shouldSatisfy` e `shouldThrow`.

#Executando os testes
Para executarmos os testes basta rodarmos o comando

    {% highlight bat %}
runhaskell MultipleOf3And5Spec.hs
    {% endhighlight %}
com isso obteremos uma saída como


    {% highlight bat %}
sum of natural numbers, multiples of 3 and 5
  return the sum below 10
  return the sum below 1000

Finished in 0.0026 seconds
2 examples, 0 failures
    {% endhighlight %}

#Observações sobre os códigos
Ao importar um módulo o recomendado é trazer apenas o que será usado, como

    {% highlight haskell %}
        import ModuloCustomizado (funcao1, funcao2)
    {% endhighlight %}

Assim teremos acesso apenas ao que pedimos e evitamos possíveis ocorrências ambíguas (ao importar tudo de todos os módulos há uma grande chance de haver dois módulos que definiram alguma função com mesmo nome).

um *workaraound* para evitar referência ambígua é usar o `hiding`, como em
    {% highlight haskell %}
        import Prelude hiding (sum)
    {% endhighlight %}

Mas se importarmos apenas o aquilo que precisamos, esse tipo de coisa não será necessária. No caso do `Prelude` não temos escolha, pois ele sempre será carregado em seus testes.
