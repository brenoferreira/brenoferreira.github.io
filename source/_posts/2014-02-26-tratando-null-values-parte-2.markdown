---
layout: post
title: "Tratando Null Values - Parte 2"
date: 2014-02-26 16:24
comments: true
categories: [Functional Programming, Option, Monads, C#]
---

> Veja a parte 1 [aqui](http://brenoferreira.github.io/blog/2013/12/23/tratando-null-values-like-a-boss-com-o-tipo-option/)

Está rolando uma thread no grupo .NET Architects sobre o Null Object Pattern. O assunto começou com uma dúvida sobre como criar mapeamentos do NHibernate com classes que possuem referencias a um Null Object.

Foram dadas algumas respostas, mas a thread continuou com o assunto e dúvidas surgiram sobre: Null Object é Anti-Pattern? Como usar? Non-Nullable References é melhor?

Vou dar algumas opiniões minhas aqui sobre esse assunto.

##É Anti-Pattern?

Não acho que seja. Mas pode vir a ser. Patterns são coisas que não devem ser abusadas, e é uma linha bem fina sobre quando é necessário e quando não é.

###Algumas reflexões

Em muitas estruturas de dados, é comum o uso de um "Null Object". Por exemplo:

####Listas

Existe um objeto que representa uma lista vazia. Geralmente é chamado de `Nil` ou `Empty`.

####Árvores

Existe os nós das árvores. Cada um com seu tipo: `Node` geralmente é o tipo base, e daí existe os tipos `Leaf`, que representa o último nó de um branch da árvore, e é comum existir o que representa um nó vazio, também chamado de `Empty` ou `Nil`. Nós `Leaf` geralmente usam esse tipo para representar seus sub-nós (que não existem).

Em ambos esses casos, seria melhor ou pior usar um valor `null` para representar uma lista vazia, ou os filhos de um nó da árvore que não existem? Minha opinião é: seria muito pior. Tornaria o design da API mais feio, e também deixaria o uso da API mais propenso a erros.

##Non-Nullable References

A ideia é muito boa, mas na prática, não é tão simples assim de se implementar. Na thread postaram links para quatro posts falando sobre o assunto:

[Parte 1](http://blogs.msdn.com/b/cyrusn/archive/2005/04/25/411617.aspx) 
[Parte 2](http://blogs.msdn.com/b/cyrusn/archive/2005/04/25/411630.aspx) 
[Parte 3](http://blogs.msdn.com/b/cyrusn/archive/2005/04/26/412040.aspx) 
[Parte 4](http://blogs.msdn.com/b/cyrusn/archive/2005/04/27/412444.aspx)

O Eric Lippert (ex-membro do time do C#) também [postou em seu blog](http://blog.coverity.com/2013/11/20/c-non-nullable-reference-types/) sobre o assunto: 

É uma pena, pois acho que nunca veremos isso no C#.

Uma linguagem que tem suporte a esse recurso é [Kotlin](http://confluence.jetbrains.com/display/Kotlin/Null-safety). Kotlin é uma linguagem criada pela Jetbrains, que roda sobre a JVM. Essa é uma das poucas linguagens que eu já vi que se preocupou com isso desde o início.

##Option type

Já [postei aqui](http://brenoferreira.github.io/blog/2013/12/23/tratando-null-values-like-a-boss-com-o-tipo-option/) sobre o assunto. E, como a thread é do .NET Architects, eu e o Abner (trabalha comigo na Lambda3) criamos um [projeto no Github](https://github.com/abnerdasdores/CSharp-OptionType) com uma implementação do tipo Option em C#.

Alguns exemplos:

``` csharp
var pessoa = new Pessoa
{
    Nome = "Robb Stark"
};

var optionPessoa = Option.Create(pessoa);

var logradouro = optionPessoa
                        .Map(p => p.Endereco)
                        .Map(e => e.Cidade)
                        .GetOrElse("Não informado");

logradouro.Should().Be("Não informado");
```

``` csharp
var option = Option.From(1);

option.Should().Be(Option.From(1));
```

``` csharp
var option = Option.From<string>(null);

option.Should().Be(Option.None<string>());
```

Inclusive com suporte (inicial) a Linq.

``` csharp
var pessoa = new Pessoa
{
    Nome = "Robb Stark"
};

var res = (from p in Option.From(pessoa)
          from e in p.Endereco.ToOption()
          select e.Cidade).GetOrElse("Sem endereco");

res.Should().Be("Sem endereco");
```

``` csharp
var pessoa = new Pessoa
{
    Nome = "Robb Stark",
    Endereco = new Endereco
    {
        Cidade = "Winterfell"
    }
};

var res = from p in Option.From(pessoa)
          from e in p.Endereco.ToOption()
          select e.Cidade;

res.Should().Be(Option.From("Winterfell"));

```

O que acham? "Linq-To-Option" ficou legal?