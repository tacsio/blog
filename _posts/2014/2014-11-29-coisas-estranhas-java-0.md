---
layout: article
title: Coisas ‘estranhas’ do Java[0]
date: 2014-11-29 21:00:00-0300
coverPhoto: https://tacsio.github.io/contents/images/tanoshi-alpha.png
---

Inicialmente quero deixar claro que não tenho preferência por linguagem de programação, tenho preferência por paradigma, mas não por linguagem (Ruby, Python e JavaScript.. ~~as modinhas~~ são, de fato, legais de trabalhar, mas eu não diria que “visto a camisa” da linguagem como vejo muita gente fazendo)…

Nesse quesito de ‘vestir a camisa’ lembrei de uma linguagem que tenho um ódio peculiar… [tcl][tcl]. ~~Acho que o capiroto me fez chegar perto desse negócio~~

Mas deixando de blá blá blá, e focando na linguagem do título dessa postagem, Java, decidi apresentar algumas coisas ‘estranhas’ que me ocorreram ao utilizar uma construção, aparentemente comum, de qualquer linguagem de programação.

Depois de pesquisar um pouco, entendi o motivo da ‘bizarrice’ e um trecho de código muito ‘louco’ onde em um cenário semelhante, uma velha conhecida de quem programa em Java pode surgir.

## Conditional expressions of DOOM

```Java 
Object value;
 
if (true)
  value = new Integer(1);
else
  value = new Double(2.0);
 
System.out.println(value);
```

Analisando o código acima, é bem claro que teremos como retorno o valor **1**. 

Como a condição é sempre verdadeira, o valor avaliado da expressão new *Integer(1)*; será atribuído a variável *value*.

Agora, caso eu seja um programador que acha mais elegante esse tipo de construção…

```Java
Object value = true ? new Integer(1) : new Double(2.0);
```

O resultado será o mesmo?

‘Estranhamente’… o resultado da execução do  segundo trecho tem como retorno **1.0**… supostamente deveriam ser equivalentes.

Pesquisando rapidamente no Google, surge a explicação na [especificação da linguagem][1]: construções utilizando o operador condicional realizam um mecanismo chamado *numeric promotion*, que é basicamente o ato de ‘promover’ um tipo (sem que haja perda de informação) a outro. Esse tipo de promoção pode ser realizada de 3 formas:

1. ***identity conversion***: conversão de um tipo A para o tipo A [identidade]

2. ***widening primitive conversion***: a conversão de tipos mais conhecida.. **int -> long, double ou float** [além de outros tipos como byte, short, char. Lembrado que não pode haver perda de informação… *e.g.* **double -> int** .. esse cenário não se caracteriza como uma promoção de numérica]

3. ***unboxing conversion***: basicamente realizar o auto-unboxing, que é converter expressões ou referências de um determinado tipo para um tipo primitivo correspondente… *e.g.* **Double -> double, Integer -> int**
Então, construções utilizando o operador condicional (ou operador ternário) realizam o unboxing automático, por isso, mesmo que o new Double(2.0);  nunca venha a ser avaliado (dead code +_+), a promoção e o unboxing serão realizados.

Ainda neste cenário, eis que surge o seguinte código:

```Java
Integer i = null;
Double d = new Double(2.0);
Object o;
 
if(true)
  o = i;	
else
  o =d;
System.out.println(o); // Print null \o/

o = true ? i : d; // NullPointerException!  =(
```

Por conta da *numeric promotion* que ocorre nas construções utilizando operador ternário, temos um famigerado **NullPointerException** quando é utilizado esse tipo de construção. Eu tinha me deparado com a promoção automática um dia desses por acaso, pesquisando vi um trecho semelhante ao código acima (adicionado pelo usuário do reddit paul_miner).

Bem…. é isto, espero que tenham gostado!

![tanoshi][tanoshi]

[tanoshi]: https://tacsio.github.io/contents/images/tanoshi-alpha.png
[tcl]: https://en.wikipedia.org/wiki/Tcl
[1]: https://docs.oracle.com/javase/specs/jls/se7/html/jls-5.html