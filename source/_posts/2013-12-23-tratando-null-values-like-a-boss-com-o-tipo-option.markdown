---
layout: post
title: "Tratando null values like a boss com o tipo Option"
date: 2013-12-23 20:06
comments: true
categories: [Functional Programming, Option, Monads, Scala]
---

Um dos pesadelos de todo desenvolvedor, é uma `NullPointerException`. Tratar objetos que podem conter um valor, ou não, é chato, e muito propenso à erros. Quem nunca esqueceu de fazer um check contra valores nulos?

``` scala
if(value == null) { throw new InvalidArgumentException() }
```

ou ainda

``` scala
var pessoa = pessoas.find(x => x.Nome == "Breno")
pessoa.AumentarSalario(100) // NullReferenceException
```

Desde que comecei a me aventurar pelo mundo de linguagens funcionais, percebi que lá, eu não me preocupava com isso. E isso se devia ao fato de que eu usava um tipo de dados bem interessante: o tipo `Option`.

``` scala
trait Option[+T] {
  def value:T
  def hasValue:Boolean
}
```

Como voce pode ver, o tipo Option simplesmente encapsula um valor possivelmente nulo. E existem os tipos concretos, chamados `Some` e `None`. Abaixo a declaração de ambos.

``` scala
case class Some[T](val value:T) extends Option[T] {
  def hasValue = true
}

case object None extends Option[Nothing] {
  def value = throw new Exception("No value")
  def hasValue = false
}
```

E também temos um objeto Option com o método apply que cria um valor do tipo Option com base no valor passado como parâmetro:

``` scala
object Option {
  def apply[A](x: A): Option[A] = if (x == null) None else Some(x)
}
```

Assim, podemos criar valores do tipo `Option` da seguinte maneira:

``` scala
val ten = Some(10)
println(ten.value) //10
```

``` scala
val none = None
println(ten.value) //throws Exception
```

Como voces podem ver, o encapsulamento do valor é feito, mas ainda sim, podemos cair em uma Exception no caso de tentarmos acessar o `value` do Option. Quer dizer então que o Option não serve para nada? Não é bem assim.

O poder verdadeiro do Option está em suas higher-order functions `map` e `flatMap`. Na definição da trait `Option`, temos a declaração dos métodos:

``` scala
def flatMap[B](f: T => Option[B]):Option[B] =
  if(hasValue) f(value) else None

def map[B](f: T => B):Option[B] =
  flatMap(_ => Option(f(value)))
```

Qual a útilidade desses métodos? Simples:

* Ter acesso ao valor armazenado no `Option`, se houver um
* Fazer combinações de valores do tipo `Option`

Como assim?

``` scala
val ten = Some(10)
val timesTwo = ten.flatMap(x => Some(x*2))

timesTwo.map(x => println(x)) //20
```

Como vimos, com os métodos `flatMap` e `map` podemos acessar o valor contido dentro do `Option`, também é possível transformar o valor em um outro valor do tipo `Option`. No exemplo acima, criamos um `Some(10)` e depois o transformamos, multiplicando o valor por 2. Assim, obtemos um `Some(20)`. Logo em seguida, usamos o método `map` para chamarmos o método `println` que imprime o valor na tela. E o legal é que, caso em alguma chamada a `map` ou `flatMap`, apareça um valor `null`, o resultado vai ser um `None`. E qualquer chamada a uma dessas funções sobre um valor `None`, nada irá acontecer, e assim, nenhuma `NullReferenceException` será lançada. Legal não é?

Também é possível obter o valor contido dentro do `Option` através de Pattern Matching:

``` scala
val none = None
none.map(_ => 10) match {
  case Some(x) => println(x)
  case None => println("Vazio")
}
//Vazio
```

No exemplo acima, como o valor é um `None`, o map não faz nada, simplesmente retorna um `None`, e quando foi feito um Pattern Matching sobre o valor resultante, ele caiu no segundo caso, que imprimiu "Vazio" na tela.

Um exemplo mais "mundo real" seria:

``` scala
val joao = pessoas.find(p => p.Nome == "João")//Retorna um Option[Pessoa]
val salarioDoJoao = joao.map(p => p.Salario) //Retorna um Option[Double]
println(salarioDoJoao.getOrElse(0.0))
```

No exemplo acima, temos uma lista de pessoas `pessoas`, e chamamos o método `find` para tentarmos achar dentro dessa lista, uma pessoa chamada "João". Como o João pode não existir dentro da lista, ele retorna um `Option[Pessoa]`. Daí, pegamos o salário do possível valor do `joao` com a funcão `map`. No final, imprimimos o valor do salário. Mas, como o resultado final pode ser um `None`, chamamos mais uma função interessante chamada `getOrElse`. O que essa função faz é simples.

``` scala
def getOrElse[B >: T](default:B) =
  this match {
    case Some(x) => x
    case None => default
  }
```

Como podemos ver, ela faz um Pattern Matching sobre o próprio valor, e caso ele seja um `Some[T]`, retornamos o próprio valor, caso seja um `None`, retornamos o valor default passado como parâmetro.

Então, no exemplo do João acima, caso o João exista na lista, o seu salário será impresso na tela, caso não, será impresso o valor 0.0.

O que você achou?

Abraços

Breno