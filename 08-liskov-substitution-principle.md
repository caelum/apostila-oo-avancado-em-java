# Liskov Substitution Principle: herança do jeito certo

## Continuando a implementação das estatísticas

A empresa Cognitio ainda não terminou a implementação do plugin de estatísticas do ebook.

O projeto `estatisticas-ebook` está apenas listando as palavras encontradas.

Precisamos fazer a contagem dessas palavras.

Também seria interessante remover pontuação, acentuação, nao diferencias maiúsculas de minúsculas.

### Removendo pontuação

Ainda são impressas pontuações como `.`, `?`, `:`, `(`, `)`, entre outros.

O método `replaceAll` de `String` recebe uma expressão regular. Nas expressões regulares do Java, é possível usar classes de caracteres POSIX, algo baseado na linguagem  Perl. A classe de caracteres `\p{Punct}` é equivalente a qualquer um dos caracteres a seguir:

```
!"#$%&'()*+,-./:;<=>?@[\]^_`{|}~
```

Com isso em mente, podemos trocar pontuações por espaços com o seguinte código:

```java
String textoDoCapituloSemPontuacao = textoDoCapitulo.replaceAll("\\p{Punct}", " ");
```

### Removendo acentuação

Outra coisa que seria interessante é limpar acentuação como `ê`, `ú`, `ç`, `ã`, etc.

Para isso, podemos usar a classe `Normalizer`, do pacote `java.text`, disponível a partir do Java 6. O método `normalize` dessa classe retorna uma `String` cujos caracteres são decompostos de acordo com o padrão Unicode.

A classe de caracteres POSIX `\p{ASCII}` contém todos os caracteres da tabela ASCII: letras sem acentuação e pontuação básica.

A partir da `String` decomposta, podemos trocar por uma `String` vazia tudo o que _não_ estiver na classe `\p{ASCII]` com a expressão regular `[^\p{ASCII}]`:

```java
String decomposta = Normalizer.normalize(textoDoCapituloSemPontuacao, Normalizer.Form.NFD).
String textoDoCapituloSemAcentos = decomposta.replaceAll("[^\\p{ASCII}]", "");
```

### Tudo maiúsculo

Para a contagem, palavras com letras maiúsculas não são diferentes de minúsculas.

Podemos ignorar essa diferença de alguma forma, passando todas as letras para maiúsculas:

```java
String emMaiusculas = palavra.toUpperCase();
contagemPalavras.put(emMaiusculas, 1);
```

### Uma contagem preliminar

Para contar as palavras, uma ideia é usar um `Map`, associando uma palavra ao seu número de ocorrências:

```java
Map<String, Integer> contagemPalavras = new HashMap<>();
```

Mas e a contagem em si? Dá pra tentar fazer algo como:

```java
contagemPalavras.put(palavra, 1);
```

### Percorrendo as entradas do Map

É possível percorrer os valores de dentro de um `Map` usando o _entry set_, que disponibiliza cada par chave-valor (_key-value_):

```java
for (Map.Entry<String, Integer> contagem : contagemPalavras.entrySet()) {

  String palavra = contagem.getKey();

  Integer ocorrencias = contagem.getValue();

  System.out.println(palavra + ": " + ocorrencias);
}
```

Perceba que os pares key-value são representado por um `Map.Entry`, que é uma interface  aninhada definida dentro da própria interface `Map`.

Será que funciona?

## Exercício: tentando contar as palavras

### Objetivo

Faça a limpeza de pontuação e acentuação na classe `CalculadoraEstatisticas`.

Faça com que as palavras sejam impressas em letras maiúsculas.

### Passo a passo

1. No projeto `estatisticas-ebook`, modifique o método `aposGeracao` da classe `CalculadoraEstatisticas`, para que a pontuação e acentuação do texto do capítulo sejam removidas. Faça com que cada palavra tenha apenas letras maiúsculas:

  ####### br.com.cognitio.estatisticas.CalculadoraEstatisticas

  ```java
  String textoDoCapitulo = doc.body().text();

  String textoDoCapituloSemPontuacao = 
    textoDoCapitulo.replaceAll("\\p{Punct}", " "); // inserido

  String textoDoCapituloSemAcentos =
    Normalizer.normalize(textoDoCapituloSemPontuacao, Normalizer.Form.NFD).
    replaceAll("[^\\p{ASCII}]", ""); // inserido

  ̶S̶t̶r̶i̶n̶g̶[̶]̶ ̶p̶a̶l̶a̶v̶r̶a̶s̶ ̶=̶ ̶t̶e̶x̶t̶o̶D̶o̶C̶a̶p̶i̶t̶u̶l̶o̶.̶s̶p̶l̶i̶t̶(̶"̶\̶\̶s̶+̶"̶)̶;̶
  String[] palavras = textoDoCapituloSemAcentos.split("\\s+"); // modificado

  for (String palavra : palavras) {

    String emMaiusculas = palavra.toUpperCase(); // inserido

    S̶y̶s̶t̶e̶m̶.̶o̶u̶t̶.̶p̶r̶i̶n̶t̶l̶n̶(̶p̶a̶l̶a̶v̶r̶a̶)̶;̶
    System.out.println(emMaiusculas); // modificado

  }
  ```

  Adicione o import correto:

  ####### br.com.cognitio.estatisticas.CalculadoraEstatisticas

  ```java
  import java.text.Normalizer;
  ```

2. No topo do método `aposGeracao` de `CalculadoraEstatisticas`, crie um `HashMap`. Dentro do _for_ das palavras dos capítulos, insira cada palavra encontrada como chave do `Map`, associando ao valor `1`.  Remova a impressão no console.

  ####### br.com.cognitio.estatisticas.CalculadoraEstatisticas

  ```java
  Map<String, Integer> contagemPalavras = new HashMap<>(); // inserido

  for (Capitulo capitulo : ebook.getCapitulos()) {

    // código omitido...

    String[] palavras = textoDoCapituloSemAcentos.split("\\s+");

    for (String palavra : palavras) {

      String emMaiusculas = palavra.toUpperCase();

      S̶y̶s̶t̶e̶m̶.̶o̶u̶t̶.̶p̶r̶i̶n̶t̶l̶n̶(̶e̶m̶M̶a̶i̶u̶s̶c̶u̶l̶a̶s̶)̶;̶
      contagemPalavras.put(emMaiusculas, 1); // modificado
    }

  }
  ```

  Ajuste os imports:

  ####### br.com.cognitio.estatisticas.CalculadoraEstatisticas

  ```java
  import java.util.Map;
  import java.util.HashMap;
  ```

3. Ao final do método `aposGeracao`, percorra o entry set do `Map`, imprimindo cada palavra encontrada e o respectivo número de ocorrências:

 ####### br.com.cognitio.estatisticas.CalculadoraEstatisticas

  ```java
  for (Map.Entry<String, Integer> contagem : contagemPalavras.entrySet()) {

    String palavra = contagem.getKey();

    Integer ocorrencias = contagem.getValue();

    System.out.println(palavra + ": " + ocorrencias);
  }  
  ```

4. Faça o build do projeto `estatisticas-ebook`. Copie-o para o `Desktop`. Execute o Cotuba novamente.

  Veja que as palavras são impressas sem acentuação nem pontuação, e em letras maiúsculas.

## Contando palavras

Todas as palavras do ebook estão sendo impressas em letras maiúsculas, sem pontuação nem acentuação. Porém, a contagem ainda não está sendo feita. É como se cada palavra fosse única:

```
SEGREGATION: 1
MARTIN: 1
DEPENDENCIAS: 1
PROBLEMA: 1
PRINCIPLES: 1
```

O problema é que usamos um `Map`, mais especificamente um `HashMap`, que não faz contagem nem acumula valores. Cada chave só pode ter um valor associado.

Mas podemos criar um `HashMap` que conta a quantidade de valores repetidos.

Para isso, podemos usar herança.

## Exercício: implementando a contagem de palavras

### Objetivo

No projeto `ebook-estatisticas`, crie uma classe `ContagemPalavras` que herda de `HashMap`.

Verifique se uma palavra já foi adicionada, mantendo a contagem de quantas vezes houve repetição.

Use a nova classe em `CalculadoraEstatisticas`.

### Passo a passo

1. No pacote `br.com.cognitio.estatisticas` do projeto `ebook-estatisticas`, adicione uma classe `ContagemPalavras`. Faça com a classe estenda de `HashMap`, colocando os tipos genéricos apropriados. Defina um método `adicionaPalavra`:

  ####### br.com.cognitio.estatisticas.ContagemPalavras

  ```java
  package br.com.cognitio.estatisticas;
  
  import java.util.HashMap;

  public class ContagemPalavras extends HashMap<String, Integer> {

    private static final long serialVersionUID = 1L;

    public void adicionaPalavra(String palavra) {

    }

  }
  ```

2. Implemente o método `adicionaPalavra`, incrementando a contagem se a palavra estiver repetida:

  ####### br.com.cognitio.estatisticas.ContagemPalavras

  ```java
  Integer contagem = get(palavra);

  if (contagem != null) {
    contagem++;
  } else {
    contagem = 1;
  }

  put(palavra, contagem);
  ```

3. Em `CalculadoraEstatisticas`, use a nova classe:

  ####### br.com.cognitio.estatisticas.CalculadoraEstatisticas

  ```java
  M̶a̶p̶<̶S̶t̶r̶i̶n̶g̶,̶ ̶I̶n̶t̶e̶g̶e̶r̶>̶ ̶c̶o̶n̶t̶a̶g̶e̶m̶P̶a̶l̶a̶v̶r̶a̶s̶ ̶=̶ ̶n̶e̶w̶ ̶T̶r̶e̶e̶M̶a̶p̶<̶>̶(̶)̶;̶
  ContagemPalavras contagemPalavras = new ContagemPalavras(); // modificado

  for (Capitulo capitulo : ebook.getCapitulos()) {

    // código omitido...

    String[] palavras = textoDoCapituloSemAcentos.split("\\s+");

    for (String palavra : palavras) {

      String emMaiusculas = palavra.toUpperCase();

      c̶o̶n̶t̶a̶g̶e̶m̶P̶a̶l̶a̶v̶r̶a̶s̶.̶p̶u̶t̶(̶e̶m̶M̶a̶i̶u̶s̶c̶u̶l̶a̶s̶,̶ ̶1̶)̶;̶
      contagemPalavras.adicionaPalavra(emMaiusculas); // modificado
    }

  }
  ```

  Remova os imports desnecessários:

  ####### br.com.cognitio.estatisticas.CalculadoraEstatisticas

  ```java
  i̶m̶p̶o̶r̶t̶ ̶j̶a̶v̶a̶.̶u̶t̶i̶l̶.̶M̶a̶p̶;̶
  i̶m̶p̶o̶r̶t̶ ̶j̶a̶v̶a̶.̶u̶t̶i̶l̶.̶T̶r̶e̶e̶M̶a̶p̶;̶
  ```

4. Faça o build do projeto `estatisticas-ebook`. Copie-o para o `Desktop`. Execute o Cotuba novamente.

  Veja que as palavras são impressas e a contagem está correta!

## Palavras em ordem alfabética

A contagem está sendo feita. As palavras estão sem pontuação nem acentuação. E todas as letras estão maiúsculas. Ótimo!

Mas como um `HashMap` não tem garantias de qual chave vem primeiro, as palavras são exibidas em uma ordem estranha:

```
SEGREGATION: 1
MARTIN: 11
DEPENDENCIAS: 6
PROBLEMA: 1
PRINCIPLES: 5
```

Como ordenar um `Map`?

Há uma implementação dessa interface que mantém as chaves ordenadas: é a classe `TreeMap`.

Essa classe usa como estrutura de dados uma implementação de uma árvore de busca binária balanceada chamada árvore rubro-negra. Busca, inserção e remoção são feitas em _O(log n)_.

A cada inserção, as chaves são mantidas em _ordem natural_: ordem crescente para números, ordem alfabética para textos. Para objetos, é necessário implementar a interface `Comparable`, que define o método `compareTo`.

Usando um `TreeMap`, as palavras ficariam em ordem alfabética ao serem impressas:

```
DEPENDENCIAS: 6
MARTIN: 11
PRINCIPLES: 5
PROBLEMA: 1
SEGREGATION: 1
```

Vale notar que a API de Collections do próprio Java  mostra o poder de um bom design. A interface `Map`, por exemplo, é uma abstração de uma associação chave-valor que é implementada por diferentes classes: `HashMap`, `LinkedHashMap` e `TreeMap`. Há garantias de um comportamento comum, mas cada implementação tem comportamentos próprios que trazem bastante flexibilidade para quem as usa.

## Exercício: ordenando as palavras

### Objetivo

Faça com que as palavras sejam impressas em ordem alfabética. Use um `TreeMap`.

### Passo a passo

1. Na classe `ContagemPalavras`, herde de `TreeMap` ao invés de `HashMap:`

  ####### br.com.cognitio.estatisticas.ContagemPalavras

  ```java
  p̶u̶b̶l̶i̶c̶ ̶c̶l̶a̶s̶s̶ ̶C̶o̶n̶t̶a̶g̶e̶m̶P̶a̶l̶a̶v̶r̶a̶s̶ ̶e̶x̶t̶e̶n̶d̶s̶ ̶H̶a̶s̶h̶M̶a̶p̶<̶S̶t̶r̶i̶n̶g̶,̶ ̶I̶n̶t̶e̶g̶e̶r̶>̶ ̶{̶
  public class ContagemPalavras extends TreeMap<String, Integer> { // modificado

    // código omitido...

  }
  ```

  Ajuste os imports:

  ####### br.com.cognitio.estatisticas.ContagemPalavras

  ```java
  i̶m̶p̶o̶r̶t̶ ̶j̶a̶v̶a̶.̶u̶t̶i̶l̶.̶H̶a̶s̶h̶M̶a̶p̶;̶
  import java.util.TreeMap;
  ```

2. Faça o build do projeto `estatisticas-ebook`. Copie-o para o `Desktop`. Execute o Cotuba novamente.

  Veja que as palavras são impressas em ordem alfabética!

## Herdando inutilidades

O cálculo de estatísticas funcionou. Mas o design do código não está satisfatório.

Há um ponto bem ruim: será que queremos mesmo herdar de `TreeMap` na classe `ContagemPalavras`?

Podemos ler o `extends` como _é um_ ou _é um tipo especial de_.

Não faz sentido falar que a classe `ContagemPalavras` é um tipo especial de `TreeMap`.

A questão principal aqui queremos apenas usar `ContagemPalavras` para adicionar palavras, acumulando a sua contagem.

Não queremos fazer tudo o que é possível com `TreeMap`.

Por exemplo, não precisamos dos métodos:

- `clear`, que limpa o conteúdo
- `values`, que obtém uma coleção com todos os valores
- `replace`, que troca o valor associado a uma chave retornado o valor antigo (ou `null`)

Não queremos que as classes que usam `ContagemPalavras`, no caso, apenas `CalculadoraEstatisticas`, possam chamar esses métodos sem uso.

Na verdade, não precisamos da maioria dos métodos de `TreeMap`. Precisamos apenas de alguns:

- os métodos `get` e `put` de `TreeMap` são usados internamente na classe `ContagemPalavras`.
- o método `entrySet` é usado em um _for-each_ na classe `CalculadoraEstatisticas`

Poderíamos sobrescrever todos os métodos desnecessários e, se forem chamados, lançar uma exceção:

```java
public class ContagemPalavras extends TreeMap<String, Integer> {

  // código omitido...

  @Override
  public void clear() {
    throw new UnsupportedOperationException();
  }

  @Override
  public Collection<Integer> values() {
    throw new UnsupportedOperationException();
  }

  @Override
  public Integer replace(String key, Integer value) {
    throw new UnsupportedOperationException();
  }

  // o mesmo para todos os outros métodos que não queremos usar...

}
```

Parece uma boa solução?

Queremos usar de fato uma parcela muito pequena da classe que herdamos.

## OO não é só herança

Logo depois de uma introdução a OO, a ideia de herança é a que fica.

Ao falar em OO, o que vêm à mente, em geral, é: especialização, subclasses e sobrescrita de métodos.

A tentação de herdar de uma superclasse para reaproveitar código é muito forte.

Mas um especialista em OO raramente usa esses recursos.

Então, quando usar herança é justificado?

> _Se um gato possui raça e patas, e um cachorro possui raça, patas e tipoDoPelo, logo `Cachorro extends Gato`?_
> _Pode parecer engraçado, mas é [...] herança por preguiça, por comodismo, por que vai dar uma ajudinha._
> _A relação “é um” não se encaixa aqui, e vai nos gerar problemas._
>
> Paulo Silveira, no post [Como não aprender orientação a objetos: Herança](http://blog.caelum.com.br/como-nao-aprender-orientacao-a-objetos-heranca/) (SILVEIRA, 2006)

## O Princípio da Substituição de Liskov

Uncle Bob resgata o trabalho de Barbara Liskov no artigo [Data Abstraction and Hierarchy](https://pdfs.semanticscholar.org/36be/babeb72287ad9490e1ebab84e7225ad6a9e5.pdf) (LISKOV, 1988), que afirma:

_Se para cada objeto o1 do tipo S há um objeto o2 do tipo T que para todos os programas P definidos em termos de T, o comportamento de P não é modificado quando o1 é substituído por o2, então S é um subtipo de T._

Uma afirmação muito complexa, não? Uncle Bob simplifica:

> **Liskov Substitution Principle (LSP)**
>
> _Subtipos devem ser substituíveis por seus tipos base._

É o princípio da _substituibilidade_.

Quando usamos herança do jeito certo, deve ser possível usar qualquer método das classes filhas ao recebermos uma variável cujo tipo é o da classe mãe.

Uma classe filha herda todos os comportamentos (métodos) da classe mãe.

Porém, não podemos usar só parte desses comportamentos.

Alguns comportamentos podem ser sobrescritos e modificados, mas não podem ser desligados nem removidos.

## Favorecendo composição à herança

Voltando ao design da classe `ContagemPalavras`, para atender ao LSP, não devemos herdar de `TreeMap` porque não queremos dar suporte à boa parte dos métodos.

A classe `ContagemPalavras` não é substituível por `TreeMap` pois faz menos coisas que sua superclasse.

Qual uma implementação melhor?

Devemos simplesmente usar um `TreeMap` e não ser um `TreeMap`:

```java
public class ContagemPalavras e̶x̶t̶e̶n̶d̶s̶ ̶T̶r̶e̶e̶M̶a̶p̶<̶S̶t̶r̶i̶n̶g̶,̶ ̶I̶n̶t̶e̶g̶e̶r̶>̶ {

  private Map<String, Integer> map = new TreeMap<>();

  // ...
}
```

Ter uma referência de um objeto como atributo privado, conforme o código anterior, é algo que chamamos de **composição**.

Fazer uma composição com uma dependência é preferível a criar um tipo especial dessa dependência.

Alguns livros clássicos de design OO contém lemas alinhados com essa ideia:

- [Design Patterns](https://www.amazon.com/Design-Patterns-Object-Oriented-Addison-Wesley-Professional-ebook/dp/B000SEIBB8) (GAMMA et al., 1994): _"Favoreça composição à herança"_
- [Effective Java](https://www.amazon.com/Effective-Java-Programming-Language-Guide/dp/0201310058) (BLOCH, 2001): _"Item 14: Prefira composição à herança"_

## Exercício: composição na contagem de palavras

### Objetivo

Troque a herança de `ContagemPalavras` em relação a `TreeMap` por composição.

### Passo a passo

1. Remova a herança de `TreeMap` da classe `ContagemPalavras`:

  ####### br.com.cognitio.estatisticas.ContagemPalavras

  ```java
  public class ContagemPalavras e̶x̶t̶e̶n̶d̶s̶ ̶T̶r̶e̶e̶M̶a̶p̶<̶S̶t̶r̶i̶n̶g̶,̶ ̶I̶n̶t̶e̶g̶e̶r̶>̶ {

    p̶r̶i̶v̶a̶t̶e̶ ̶s̶t̶a̶t̶i̶c̶ ̶f̶i̶n̶a̶l̶ ̶l̶o̶n̶g̶ ̶s̶e̶r̶i̶a̶l̶V̶e̶r̶s̶i̶o̶n̶U̶I̶D̶ ̶=̶ ̶1̶L̶;̶

    // código omitido...
  }
  ```

2. Adicione um atributo do tipo `Map`, com os tipos genéricos apropriados. Inicialize o atributo com uma instância de `TreeMap` e use-o no método `adicionaPalavra`:

  ####### br.com.cognitio.estatisticas.ContagemPalavras

  ```java
  public class ContagemPalavras {

    private Map<String, Integer> map = new TreeMap<>();

    public void adicionaPalavra(String palavra) {

      I̶n̶t̶e̶g̶e̶r̶ ̶c̶o̶n̶t̶a̶g̶e̶m̶ ̶=̶ ̶g̶e̶t̶(̶p̶a̶l̶a̶v̶r̶a̶)̶;̶
      Integer contagem = map.get(palavra); // modificado

      if (contagem != null) {
        contagem++;
      } else {
        contagem = 1;
      }

      p̶u̶t̶(̶p̶a̶l̶a̶v̶r̶a̶,̶ ̶c̶o̶n̶t̶a̶g̶e̶m̶)̶;̶
      map.put(palavra, contagem); // modificado

    }

  }
  ```

  Certifique-se que os imports estão corretos:

  ####### br.com.cognitio.estatisticas.ContagemPalavras

  ```java
  import java.util.Map;
  import java.util.TreeMap;
  ```

3. Deve acontecer um erro de compilação na classe `CalculadoraEstatisticas`, que espera que `ContagemPalavras` possua um método `entrySet`. Crie-o, retornando um `Set` com os pares chave-valor do `Map`:

  ####### br.com.cognitio.estatisticas.ContagemPalavras

  ```java
  public class ContagemPalavras {

    // código omitido

    // inserido
    public Set<Map.Entry<String, Integer>> entrySet() {
      return map.entrySet();
    }

  }
  ```

  Faça os imports adequados:

  ####### br.com.cognitio.estatisticas.ContagemPalavras

  ```java
  import java.util.Set;
  ```

4. Faça o build do projeto `estatisticas-ebook`. Copie-o para o `Desktop`. Execute o Cotuba novamente.

  Deve continuar funcionando!

## A forte intimidade entre uma (classe) filha e sua mãe

No livro [Refactoring](https://www.amazon.com/Refactoring-Improving-Design-Existing-Code/dp/0201485672/) (FOWLER et al., 1999), Kent Beck e Martin Fowler listam alguns problemas de design comuns, que chamam de maus cheiros de código.

Entre esse maus cheiros, há a _intimidade inadequada_:

_Às vezes as classes se tornam íntimas demais e gastam tempo demais sondando as partes privadas das outras._
_Podemos não ser pudicos quando o assunto são pessoas, mas achamos que nossas classes devem seguir regras puritanas rígidas._

_A herança pode muitas vezes levar à intimidade excessiva._
_Subclasses sempre saberão mais sobre seus pais do que esses gostariam que elas soubessem._

O uso de herança faz com que a classe filha tenha um forte acoplamento (ou intimidade) com sua classe mãe.

Esse forte acoplamento faz com que especialistas em OO como Joshua Bloch, no livro [Effective Java](https://www.amazon.com/Effective-Java-Programming-Language-Guide/dp/0201310058) (BLOCH, 2001), indique que a maioria das classes de uma aplicação deveria proibir herança:

_"Item 15: Faça um design e documente pensando em herança ou proíba-a"._

No Java, com o uso de `final` antes do nome da classe, é possível fazer com que nenhuma outra classe consiga herdá-la.

Joshua Bloch sugere, no mesmo livro, que evitemos herança para abstrações:

_"Item 16: Prefira interfaces a classes abstratas"._

Classes abstratas podem definir atributos, métodos concretos e métodos abstratos, que contém apenas assinaturas. Interfaces são mais leves: apenas definem assinaturas de métodos que devem ser implementados. Por isso, há menos acoplamento.

## Quando usar herança?

Joshua Bloch diz no livro [Effective Java](https://www.amazon.com/Effective-Java-Programming-Language-Guide/dp/0201310058) (BLOCH, 2001):

_A herança só é apropriada em circunstâncias em que a subclasse é realmente um subtipo da superclasse._
_Em outras palavras, uma classe B deve estender uma classe A somente se um relacionamento "é-um" existir entre as duas classes._
_Se você ficar tentado a fazer uma classe B estender uma classe A, faça a pergunta: cada B é realmente um A?_
_Se não puder responder sim a essa pergunta com convicção, B não deve estender A._
_Quando a resposta é não, geralmente é o caso em que B deve conter uma instância privada de A e expor uma API menor e mais simples: A não é uma parte essencial de B, apenas um detalhe de sua implementação._

No caso do Cotuba, poderíamos ter um tipo especial de ebook que, além de fazer tudo o que um ebook faz e conter tudo o que um ebook contém, ainda tem um método que adiciona propagandas em algumas páginas. Definiríamos uma classe `EbookComPropagandas` que herda de `Ebook`, por exemplo.



## Para saber mais: o uso incorreto de herança na plataforma Java

Paulo Silveira, no post [Como não aprender orientação a objetos: Herança](http://blog.caelum.com.br/como-nao-aprender-orientacao-a-objetos-heranca/) (SILVEIRA, 2006), cita alguns casos da própria plataforma Java em que há mau uso de herança.

### Properties

Um exemplo é a classe `java.util.Properties`:

```java
public class Properties extends Hashtable<Object,Object> {
  //...
}
```

A classe `Properties`, definida no Java 1.0, herda de `Hashtable`.

Isso traz efeitos indesejados, fazendo com que a chamada de alguns métodos não seja recomendada.

A classe filha deseja fazer menos que sua mãe. Ou seja, há uma quebra do LSP.

Isso é explicado no próprio [Javadoc](https://docs.oracle.com/javase/10/docs/api/java/util/Properties.html):

_Como `Properties` herda de `Hashtable`, os métodos `put` e `putAll` podem ser aplicados a um objeto `Properties`._
_Seu uso é fortemente desencorajado, pois permitem que sejam inseridas entradas cujas chaves ou valores não são `String`._

### HttpServlet

Para definir uma Servlet em um projeto Java para Web, devemos herdar de `javax.servlet.http.HttpServlet` e sobrescrever os métodos apropriados como `doGet` para requisições HTTP do tipo GET, `doPost` para POST ou `service` para qualquer tipo de requisição.

Para obter configurações na inicialização de uma Servlet, podemos sobrescrever o método `init` que recebe um `ServletConfig`.

Mas há inconsistências terríveis: no caso do `doGet`, `doPost` ou `service`, não podemos chamar o método da classe mãe, ocorre uma exceção.

Já no caso do `init`, devemos obrigatoriamente chamar o método `init` da classe mãe, passando o `ServletConfig` como parâmetro.

```java
public class OiServlet extends HttpServlet {

  @Override
  public void init(ServletConfig config) throws ServletException {
    super.init(config); //se NÃO chamar, dá erro...
  }

  @Override
  protected void service(HttpServletRequest req, HttpServletResponse res) throws IOException, ServletException {
    super.service(req, res); //se chamar, dá erro...
  }

}
```

## Qual a necessidade de um método se não há nada pra ser feito?

O Cotuba agora tem dois pontos de extensão:

- um plugin de tema, usado pela empresa Paradizo
- um plugin que é chamado ao finalizar a geração do ebook, usado pela empresa Cognitio para calcular estatísticas do ebook

Tudo compilou e a execução teve sucesso.

Mas há algo incômodo nos service providers, que implementam a SPI `Plugin`.

Observe que nada é feito no método `aposGeracao` da classe `TemaParadizo`:

```java
public class TemaParadizo implements Plugin {

  @Override
  public String cssDoTema() {
    return FileUtils.getResourceContents("/tema.css");
  }

  @Override
  public void aposGeracao(Ebook ebook) {
    // NADA AQUI!
  }

}
```

O caso da classe `CalculadoraEstatisticas` é pior ainda: retornamos `null` no método `cssDoTema`, já que não queremos definir um tema.

```java
public class CalculadoraEstatisticas implements Plugin {

  @Override
  public String cssDoTema() {
    return null; // OLHA SÓ!
  }

  @Override
  public void aposGeracao(Ebook ebook) {

    ContagemPalavras contagemPalavras = new ContagemPalavras();

    for (Capitulo capitulo : ebook.getCapitulos()) {
      // código omitido...
    }

  }

}
```

O método estático `listaDeTemas` de `Plugin` insere esse valor nulo na lista de temas retornados.

O tema nulo, ou `<style>null</style>`, é adicionado ao `<head>` do HTML pela classe `AplicadorTema`.

Entretanto, não há efeito visual.

Será que retornar nulo é a melhor alternativa para o método `cssDoTema` de `CalculadoraEstatisticas`?

Ou será que deveríamos lançar uma exceção, como a seguir:

```java
public class CalculadoraEstatisticas implements Plugin {

  @Override
  public String cssDoTema() {
    throw new UnsupportedOperationException("Não há suporte a temas.");
  }

  // código omitido...
}
```

Se escolhermos lançar a exceção, a geração de ebooks deixará de funcionar.

Poderíamos colocar uma checagem se o retorno do método `cssDoTema` é nulo (ou um _try-catch_) no método estático `listaDeTemas` de `Plugin`, evitando inserir valores nulos na lista de temas.

Isso minimizaria os efeitos do problema mas não resolveria a causa raiz.

E qual é essa causa raiz?

Dependendo de quem estiver implementando a interface `Plugin`, só definirá uma coisa ou outra.

No fim das contas, há uma **quebra do LSP**: as implementações não são substituíveis pela interface, já que fornecem só parte do comportamento definido pela abstração.

Como resolver? Isso é assunto para o próximo capítulo!
