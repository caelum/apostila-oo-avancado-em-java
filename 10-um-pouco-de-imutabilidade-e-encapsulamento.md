# Um pouco de imutabilidade e encapsulamento

## Violar o ISP ou o DIP?

No capítulo anterior, criamos as interfaces `Ebook` e `Capitulo` no pacote `cotuba.plugin`, que continham apenas _getters_ dos atributos, para que os plugins não tivessem acesso aos _setters_.

A solução atende ao ISP, já que não fornece aos clientes métodos desnecessários.

Mas será que a solução atende ao DIP?

As classes `Ebook` e `Capitulo` do pacote `cotuba.domain` implementam as interfaces de mesmo nome do pacote `cotuba.plugin`.

Classes relacionadas ao negócio implementam abstrações não relacionadas ao negócio.

Há uma violação do DIP.

Ao atendermos ao ISP, acabamos quebrando o DIP.

## Classes sem setters

A motivação para criamos as interfaces `Ebook` e `Capitulo` do pacote `cotuba.plugin` foi evitar que os plugins tivessem acessos aos setters.

Mas e se, nas próprias classes do pacote `cotuba.domain`, não tivéssemos esses setters?

Os plugins não poderiam chamar métodos que não existem.

Mas é possível criar uma classe sem setters?

Sim! Estaríamos caminhando na direção da imutabilidade.

## Em direção à imutabilidade

Vamos considerar uma classe `ContaCorrente`, que possui os atributos `saldo`, do tipo `BigDecimal` e `movimentacoes`, uma `List<Movimentacao>`. Os getters e setters são definidos para ambos os atributos. Há ainda os métodos `deposita`, `saca` e `adicionaMovimentacao`:

``` java
public class ContaCorrente {

  private BigDecimal saldo;
  private List<Movimentacao> movimentacoes;

  public BigDecimal getSaldo() {
    return saldo;
  }

  public List<Movimentacao> getMovimentacoes() {
    return movimentacoes;
  }

  public void setSaldo(BigDecimal saldo) {
    this.saldo = saldo;
  }

  public void setMovimentacoes(List<Movimentacao> movimentacoes) {
    this.movimentacoes = movimentacoes;
  }

  public void deposita(BigDecimal valor) {
    saldo = saldo.add(valor);
  }

  public void saca(BigDecimal valor) {
    saldo = saldo.subtract(valor);
  }

  public void adicionaMovimentacao(Movimentacao movimentacao) {
    movimentacoes.add(movimentacao);
  }

}
```

Essa _não_ é uma classe imutável.

Uma classe imutável é aquela cujos objetos, depois de instanciados, não mudam os valores de seus atributos.

Certamente, setters não devem ser definidos.

``` java
public class ContaCorrente {

  // código omitido...

  p̶u̶b̶l̶i̶c̶ ̶v̶o̶i̶d̶ ̶s̶e̶t̶S̶a̶l̶d̶o̶(̶B̶i̶g̶D̶e̶c̶i̶m̶a̶l̶ ̶s̶a̶l̶d̶o̶)̶ ̶{̶
    r̶e̶t̶u̶r̶n̶ ̶t̶h̶i̶s̶.̶s̶a̶l̶d̶o̶ ̶=̶ ̶s̶a̶l̶d̶o̶;̶
  }̶

  p̶u̶b̶l̶i̶c̶ ̶v̶o̶i̶d̶ ̶s̶e̶t̶M̶o̶v̶i̶m̶e̶n̶t̶a̶c̶o̶e̶s̶(̶L̶i̶s̶t̶<̶M̶o̶v̶i̶m̶e̶n̶t̶a̶c̶a̶o̶>̶ ̶m̶o̶v̶i̶m̶e̶n̶t̶a̶c̶o̶e̶s̶)̶ ̶{̶
    r̶e̶t̶u̶r̶n̶ ̶t̶h̶i̶s̶.̶m̶o̶v̶i̶m̶e̶n̶t̶a̶c̶o̶e̶s̶ ̶=̶ ̶m̶o̶v̶i̶m̶e̶n̶t̶a̶c̶o̶e̶s̶;̶
  }̶

   // código omitido...

}
```

Ainda assim, uma classe que não tem setters não necessariamente será imutável. Para que seja, é preciso que _nenhum outro_ método mude o estado de um objeto dessa classe, ou seja, que os valores dos seus atributos não sejam modificados.

Para garantir que não haverá mudança de estado, podemos definir os atributos como `final`. Qualquer tentativa de mudança ocasionará um erro de compilação:

``` java
public class ContaCorrente {

  private final BigDecimal saldo; // modificado
  private final List<Movimentacao> movimentacoes; // modificado

  // código omitido...

  public void deposita(BigDecimal valor) {
    saldo = saldo.add(valor); // error: cannot assign a value to final variable saldo
  }

  public void saca(BigDecimal valor) {
    saldo = saldo.subtract(valor); // error: cannot assign a value to final variable saldo
  }

  // código omitido...

}
```

Com atributos `final`, não podemos ter setters nem métodos que mudam seus valores. Mas, então, como fazer atribuições?

Por meio de construtores! Os atributos tem seus valores iniciais informados na criação do objeto. Uma vez definidos, esses valores não podem ser modificados:

```java
public class ContaCorrente {

  private final BigDecimal saldo;
  private final List<Movimentacao> movimentacoes;

  public ContaCorrente(BigDecimal saldo, List<Movimentacao> movimentacoes) {
    this.saldo = saldo;
    this.movimentacoes = movimentacoes;
  }

  // código omitido...

}
```

Se for necessário, podemos definir mais construtores, em que informamos apenas parte dos atributos.

Mas e os métodos que fazem mutações nos atributos? Passam a retornar novos objetos:

```java
public class ContaCorrente {

  // código omitido...

  public ContaCorrente deposita(BigDecimal valor) {
    BigDecimal novoSaldo = this.saldo.add(valor);
    return new ContaCorrente(novoSaldo, this.movimentacoes);
  }

  public ContaCorrente saca(BigDecimal valor) {
    BigDecimal novoSaldo = this.saldo.subtract(valor);
    return new ContaCorrente(novoSaldo, this.movimentacoes);
  }

  // código omitido...

}
```

Ainda não temos uma classe imutável.

Seria possível criar uma classe filha que se comportasse como se houvesse mudança de estado.

Porém, é possível usar `final` na classe, sinalizando que não poderá haver subclasses:

```java
public final class ContaCorrente {

  // código omitido...

}
```

Há ainda uma questão importante para chegarmos à imutabilidade de uma classe.

Se houver composição com objetos mutáveis, seria possível que outras classes mudassem seus estados. Bastaria que obtivessem referências a esses objetos via getters ou de alguma outra maneira.

É o que acontece com o método `getMovimentacoes` de `ContaCorrente`. É compartilhada com outros objetos uma `List<Movimentacao>`, que é mutável. Outros objetos podem adicionar, remover ou trocar valores dessa lista, mudando o estado da `ContaCorrente` indiretamente.

Para resolver isso, é preciso fazer uma cópia defensiva dos objetos mutáveis usados por uma classe que pretende ser imutável.

No caso de uma `List`, há o método `unmodifiableList` da classe utilitária `Collections`. Esse método retorna uma cópia _só para leitura_ da lista original:

```java
public final class ContaCorrente {

  private final BigDecimal saldo;
  private final List<Movimentacao> movimentacoes;

  public ContaCorrente(BigDecimal saldo, List<Movimentacao> movimentacoes) {
    this.saldo = saldo;
    this.movimentacoes = Collections.unmodifiableList(movimentacoes);
  }

    public BigDecimal getSaldo() {
    return saldo;
  }

  public List<Movimentacao> getMovimentacoes() {
    return movimentacoes;
  }

  // código omitido...

}
```

Se um objeto obtiver a lista de movimentações por meio do método `getMovimentacoes`, não será possível adicionar, remover ou trocar valores dessa lista. Qualquer chamada aos métodos que mudariam uma `unmodifiableList` faz com que seja lançada uma `UnsupportedOperationException`.

_Cá entre nós, manter a API mutável de List e lançar exceções é uma violação da LSP._

E a composição com `BigDecimal`? Sem problemas, pois essa classe é imutável!

### Classes imutáveis na plataforma Java

Boa parte das classes da biblioteca padrão do Java são imutáveis. Exemplos: `String`, classes wrappers como `Integer` e `Double`, `BigDecimal` e `BigInteger`, classes da API `java.time` como `LocalDate` e `LocalTime`, entre outras.

Algumas classes do Java são mutáveis como `Calendar` e `Date` e as da API de Collections, `ArrayList`, `HashSet` e `TreeMap`.

## Imutabilidade

Um dos motes de Joshua Bloch no livro [Effective Java](https://www.amazon.com/Effective-Java-Programming-Language-Guide/dp/0201310058) (BLOCH, 2001) é:

_Item 13: Favoreça a imutabilidade._

O autor, ainda no mesmo livro, lista cinco "regras" para tornar uma classe imutável:

- Não forneça nenhum método que modifique o estado do objeto
- Assegure que a classe não possa ser estendida
- Defina todos os atributos como `final`
- Faça com que todos os atributos sejam privados
- Assegure acesso exclusivo a quaisquer composição com objetos mutáveis

Algumas vantagens de objetos imutáveis citadas por Bloch no mesmo livro:

- São simples: só há um estado possível
- São _thread-safe_: como não há mudança de estado, não há problemas no acesso concorrente aos mesmos objetos.
- Podem ser compartilhados e reutilizados livremente

A grande desvantagem dos objetos imutáveis é no uso de memória: para cada valor distinto, é preciso um novo objeto.

> _Classes devem ser imutáveis ao menos que haja uma boa razão para torná-las mutáveis._
>
> _[...]_
>
> _Se uma classe não puder ser imutável, limite sua mutabilidade o máximo possível._
>
> Joshua Bloch, no livro [Effective Java](https://www.amazon.com/Effective-Java-Programming-Language-Guide/dp/0201310058) (BLOCH, 2001)

### Mais pragmatismo, menos fundamentalismo

Depois de ler a opinião de Joshua Bloch quanto à imutabilidade, dá aquela vontade de apagar tudo o que já implementamos, removendo setters, colocando atributos e classes como `final`, não é mesmo?

Tenhamos calma!

Na plataforma Java, é muito difícil criar código 100% imutável. As especificações e as bibliotecas que as implementam, em geral, requerem JavaBeans. É assim com JPA/Hibernate, com o JSF, com o JAX-B, com o JavaFX, entre outros. E um JavaBean precisa de getters, setters e de um construtor sem parâmetros.

Poderíamos deixar nossos objetos de alto nível como imutáveis, criando JavaBeans específicos para cada tecnologia da plataforma Java. Quando necessário, faríamos a conversão de/para os objetos imutáveis. Porém, quanto mais intermediários no código, mais complexidade.

Uma abordagem mais pragmática e menos idealista é trabalhar com objetos mutáveis, mesmo que os objetos de negócio acabem, aos poucos, sendo tomados por essa mutabilidade.

Escolher entre idealismo e pragmatismo é uma decisão que deve ser tomada de acordo com o contexto do projeto, da equipe e da empresa.

## Exercício: voltando atrás na segregação de interfaces

### Objetivo

Remova as interfaces `Ebook` e `Capitulo` do pacote `cotuba.plugin`.

Corrija os erros de compilação dos projetos `cotuba-cli` e `estatisticas-ebook`.

### Passo a passo

1. Apague `Ebook` e `Capitulo` do pacote `cotuba.plugin`:

  ```
  c̶o̶t̶u̶b̶a̶.̶p̶l̶u̶g̶i̶n̶.̶E̶b̶o̶o̶k̶
  c̶o̶t̶u̶b̶a̶.̶p̶l̶u̶g̶i̶n̶.̶C̶a̶p̶i̶t̶u̶l̶o̶
  ```
  
  No projeto `cotuba-cli`, devem ocorrer erros de compilação nas classes `Ebook` e `Capitulo`, ambas do pacote `cotuba.domain`, assim como na classe `Cotuba` e na interface `AoFinalizarGeracao`.

  Já no projeto `estatisticas-ebook`, deve ocorrer um erro de compilação na classe `CalculadoraEstatisticas`.

2. Deixe de implementar as interfaces removidas no passo anterior nas classes `Ebook` e `Capitulo`  do pacote `cotuba.domain`:

  ####### cotuba.domain.Ebook

  ```java
  public class Ebook i̶m̶p̶l̶e̶m̶e̶n̶t̶s̶ ̶c̶o̶t̶u̶b̶a̶.̶p̶l̶u̶g̶i̶n̶.̶E̶b̶o̶o̶k̶ {
  
    // código omitido...
  
  }
  ```

  ####### cotuba.domain.Capitulo

  ```java
  public class Capitulo i̶m̶p̶l̶e̶m̶e̶n̶t̶s̶ ̶c̶o̶t̶u̶b̶a̶.̶p̶l̶u̶g̶i̶n̶.̶C̶a̶p̶i̶t̶u̶l̶o̶ {
  
    // código omitido...
  
  }
  ```

3. Ajuste os imports da interface `AoFinalizarGeracao` do pacote `cotuba.plugin`:

  ####### cotuba.plugin.AoFinalizarGeracao

  ```java
  import cotuba.domain.Ebook;
  ```

  Nesse momento, todas as classes do projeto `cotuba-cli` devem compilar com sucesso.

4. Resolva os erros de compilação da classe `CalculadoraEstatisticas` do projeto `estatisticas-ebook` fazendo os imports corretos:

  ####### br.com.cognitio.estatisticas.CalculadoraEstatisticas

  ```java
  i̶m̶p̶o̶r̶t̶ ̶c̶o̶t̶u̶b̶a̶.̶p̶l̶u̶g̶i̶n̶.̶C̶a̶p̶i̶t̶u̶l̶o̶;̶
  i̶m̶p̶o̶r̶t̶ ̶c̶o̶t̶u̶b̶a̶.̶p̶l̶u̶g̶i̶n̶.̶E̶b̶o̶o̶k̶;̶
  import cotuba.domain.Capitulo;
  import cotuba.domain.Ebook;
  ```

  Não devem ocorrer outros erros de compilação.

## Exercício: um domain model imutável

### Objetivo

Faça com que as classes `Ebook` e `Capitulo` do pacote `cotuba.domain` sejam imutáveis.

### Passo a passo

1. Remova os setters de `Ebook` e `Capitulo`:

  ####### cotuba.domain.Ebook

  ```java
   public class Ebook {

    // código omitido...

    p̶u̶b̶l̶i̶c̶ ̶v̶o̶i̶d̶ ̶s̶e̶t̶F̶o̶r̶m̶a̶t̶o̶(̶F̶o̶r̶m̶a̶t̶o̶E̶b̶o̶o̶k̶ ̶f̶o̶r̶m̶a̶t̶o̶)̶ ̶{̶
      t̶h̶i̶s̶.̶f̶o̶r̶m̶a̶t̶o̶ ̶=̶ ̶f̶o̶r̶m̶a̶t̶o̶;̶
    }̶

    p̶u̶b̶l̶i̶c̶ ̶v̶o̶i̶d̶ ̶s̶e̶t̶A̶r̶q̶u̶i̶v̶o̶D̶e̶S̶a̶i̶d̶a̶(̶P̶a̶t̶h̶ ̶a̶r̶q̶u̶i̶v̶o̶D̶e̶S̶a̶i̶d̶a̶)̶ ̶{̶
      t̶h̶i̶s̶.̶a̶r̶q̶u̶i̶v̶o̶D̶e̶S̶a̶i̶d̶a̶ ̶=̶ ̶a̶r̶q̶u̶i̶v̶o̶D̶e̶S̶a̶i̶d̶a̶;̶
    }̶

    p̶u̶b̶l̶i̶c̶ ̶v̶o̶i̶d̶ ̶s̶e̶t̶C̶a̶p̶i̶t̶u̶l̶o̶s̶(̶L̶i̶s̶t̶<̶C̶a̶p̶i̶t̶u̶l̶o̶>̶ ̶c̶a̶p̶i̶t̶u̶l̶o̶s̶)̶ ̶{̶
      t̶h̶i̶s̶.̶c̶a̶p̶i̶t̶u̶l̶o̶s̶ ̶=̶ ̶c̶a̶p̶i̶t̶u̶l̶o̶s̶;̶
    }̶

   }
  ```

  ####### cotuba.domain.Capitulo

  ```java
  public class Capitulo {

    // código omitido...

    p̶u̶b̶l̶i̶c̶ ̶v̶o̶i̶d̶ ̶s̶e̶t̶T̶i̶t̶u̶l̶o̶(̶S̶t̶r̶i̶n̶g̶ ̶t̶i̶t̶u̶l̶o̶)̶ ̶{̶
      t̶h̶i̶s̶.̶t̶i̶t̶u̶l̶o̶ ̶=̶ ̶t̶i̶t̶u̶l̶o̶;̶
    }̶

    p̶u̶b̶l̶i̶c̶ ̶v̶o̶i̶d̶ ̶s̶e̶t̶C̶o̶n̶t̶e̶u̶d̶o̶H̶T̶M̶L̶(̶S̶t̶r̶i̶n̶g̶ ̶c̶o̶n̶t̶e̶u̶d̶o̶H̶T̶M̶L̶)̶ ̶{̶
      t̶h̶i̶s̶.̶c̶o̶n̶t̶e̶u̶d̶o̶H̶T̶M̶L̶ ̶=̶ ̶c̶o̶n̶t̶e̶u̶d̶o̶H̶T̶M̶L̶;̶
    }̶

  }
  ```

  Ao remover os setters, ocorrerão erros de compilação nas classes `Cotuba`, `RenderizadorMDParaHTMLComCommonMark` e `AplicadorTema`.

  Corrigiremos esses erros mais adiante.

2. Faça com que as classes `Ebook` e `Capitulo` não possam ter subclasses, definindo o modificador `final`:

  ####### cotuba.domain.Ebook

  ```java
  public final class Ebook { // modificado

    // código omitido...

  }
  ```

  ####### cotuba.domain.Capitulo

  ```java
  public final class Capitulo { // modificado

    // código omitido...

  }
  ```

3. Faça com que os atributos das classes `Ebook` e `Capitulo` sejam `final`:

  ####### cotuba.domain.Ebook

  ```java
  public final class Ebook {

    private final FormatoEbook formato; // modificado

    private final Path arquivoDeSaida; // modificado

    private final List<Capitulo> capitulos; // modificado

    // código omitido...

  }
  ```

  ####### cotuba.domain.Capitulo

  ```java
  public final class Capitulo {

    private final String titulo; // modificado

    private final String conteudoHTML; // modificado

    // código omitido...

  }
  ```

  Como o valor de nenhum desses atributos foi definido, todos terão erros de compilação.

4. Crie construtores para as classes `Ebook` e `Capitulo`, definindo os valores recebidos pelos parâmetros nos atributos.

  Dica: você pode usar o atalho _CTRL+3_, digitando _gcuf_ e, na tela que será aberta, selecionar a opção _Select All_.

  Faça com que a `List<Capitulo>` de `Ebook` seja imutável usando o método estático `unmodifiableList` da classe utilitária `Collections`.

  ####### cotuba.domain.Ebook

  ```java
  public final class Ebook {

    private final FormatoEbook formato;

    private final Path arquivoDeSaida;

    private final List<Capitulo> capitulos;

    // inserido
    public Ebook(FormatoEbook formato, Path arquivoDeSaida, List<Capitulo> capitulos) {
      this.formato = formato;
      this.arquivoDeSaida = arquivoDeSaida;
      this.capitulos = Collections.unmodifiableList(capitulos);
    }

    // código omitido...

  }
  ```

  Adicione o import:

  ####### cotuba.domain.Ebook

  ```java
  import java.util.Collections;
  ```

  ####### cotuba.domain.Capitulo

  ```java
  public final class Capitulo {

    private final String titulo;

    private final String conteudoHTML;

    // inserido
    public Capitulo(String titulo, String conteudoHTML) {
      this.titulo = titulo;
      this.conteudoHTML = conteudoHTML;
    }

    // código omitido...

  }
  ```

  Os erros de compilação nas classes `Cotuba`, `RenderizadorMDParaHTMLComCommonMark` e `AplicadorTema` continuam.

## Usando objetos imutáveis

Como resolver os erros de compilação causados pela remoção dos setters e imutabilidade nas classes `Ebook` e `Capitulo`?

### Corrigindo a classe Cotuba

Os erros na classe `Cotuba` acontecem porque é criado um `Ebook` com o construtor padrão, sem parâmetros, e, em seguida, são chamados os setters.

####### cotuba.application.Cotuba

```java
Ebook ebook = new Ebook(); // error: The constructor Ebook() is undefined
ebook.setFormato(formato); // error:  The method setFormato(FormatoEbook) is undefined for the type Ebook
ebook.setArquivoDeSaida(arquivoDeSaida); // error: The method setArquivoDeSaida(Path) is undefined for the type Ebook
ebook.setCapitulos(capitulos); // error: The method setCapitulos(List<Capitulo>) is undefined for the type Ebook
```

Nada disso existe mais!

A correção, nesse caso, é simples: basta usarmos o novo construtor da classe `Ebook`.

####### cotuba.application.Cotuba

```java
Ebook ebook = new Ebook(formato, arquivoDeSaida, capitulos);
```

### Corrigindo o aplicador de temas

Já na classe `AplicadorTema`, o caso é mais complicado.

Essa classe obtém o HTML de um capítulo e o modifica, inserindo o CSS de cada um dos plugins de tema.

Mas o setter do HTML não existem mais em `Capitulo`.

Então, ocorre um erro de compilação.

####### cotuba.tema.AplicadorTema

```java
public class AplicadorTema {

  public void aplica(Capitulo capitulo) {

    // código omitido...

    capitulo.setConteudoHTML(document.html()); // error: The method setConteudoHTML(String) is undefined for the type

  }

}
```

Como resolver?

Podemos alterar a assinatura do método `aplica` para retornar uma `String` como o novo HTML do capítulo.

Assim, não seria necessário usar o setter do conteúdo HTML.

####### cotuba.tema.AplicadorTema

```java
public class AplicadorTema {

  public String aplica(Capitulo capitulo) {

    // código omitido...

    return document.html();

  }

}
```

### Corrigindo o renderizador

O problema na classe `RenderizadorMDParaHTMLComCommonMark` é o mais complexo.

Há erros de compilação nas chamadas aos setters e ao construtor padrão de `Capitulo`:

####### cotuba.md.RenderizadorMDParaHTMLComCommonMark

```java
arquivosMD
  .filter(matcher::matches)
  .sorted()
  .forEach(arquivoMD -> {

    Capitulo capitulo = new Capitulo(); // error: The constructor Capitulo() is undefined

    // código omitido...

    String tituloDoCapitulo = ((Text) heading.getFirstChild()).getLiteral();

    capitulo.setTitulo(tituloDoCapitulo); // error: The method setTitulo(String) is undefined for the type Capitulo

    // código omitido...

    String html = renderer.render(document);

    capitulo.setConteudoHTML(html); // error: The method setConteudoHTML(String) is undefined for the type Capitulo

  });
```

A questão é que não basta agruparmos tudo em uma chamada ao construtor definido na classe `Capitulo`, que recebe o título e o conteúdo HTML.

No momento da criação do objeto, não temos todas as informações necessárias.

Precisamos processar os arquivos MD para descobrir o título de cada capítulo. Para ter o conteúdo, temos que renderizar cada MD para HTML.

Há várias soluções para esse caso específico, em que queremos construir um objeto aos poucos.

A maneira mais simples seria armazenar o título e, depois, o conteúdo HTML em variáveis para, só então, usar o construtor de `Capitulo`.

Mas podemos usar uma solução mais sofisticada.

## Design Pattern: Builder

A criação de alguns objetos é complexa.

Podem ser necessárias muitas informações e/ou muitos passos no seu processo de instanciação.

Uma nota fiscal, por exemplo, contém no mínimo:

- data de emissão
- nome completo do cliente
- endereço do cliente
- lista de produtos

Um construtor de uma classe `NotaFiscal` receberia muitos parâmetros.

```java
public final class NotaFiscal {

  // atributos omitidos...

  public NotaFiscal(LocalDate data, String nomeCliente, String enderecoCliente, List<Produto> produtos) {
    // parâmetros do construtor setados nos atributos...
  }

}
```

Qual a chance de trocarmos o nome do cliente pelo endereço? Grande, não?

E, provavelmente, teríamos mais atributos a serem definidos: endereço de cobrança, número de parcelas, método de pagamento, etc.

O construtor tende a crescer. E a dificuldade de manutenção do código aumentará.

Porém, podemos criar uma classe responsável por representar as etapas da construção do objeto.

É o que chamam no livro do [Design Patterns](https://www.amazon.com/Design-Patterns-Object-Oriented-Addison-Wesley-Professional-ebook/dp/B000SEIBB8) (GAMMA et al., 1994) de **Builder**.

Um possível uso de um Builder para a nota fiscal seria:

```java
NotaFiscal nota = 
  new NotaFiscalBuilder()
    .naData(2018, 9, 5)
    .paraOCliente("João da Silva")
    .doEndereco("Rua Vergueiro, 3185, 8º andar")
    .comOProduto("Livro Git e GitHub", 39.9)
    .constroi();
```

Como defini-lo?

Uma implementação simples seria:

```java
public class NotaFiscalBuilder {

  private LocalDate data;
  private String nomeCliente;
  private String endereco;
  private List<Produto> produtos = new ArrayList<>();

  public NotaFiscalBuilder naData(int ano, int mes, int dia) {
    this.data = LocalDate.of(ano, mes, dia);
    return this;
  }

  public NotaFiscalBuilder paraOCliente(String nome) {
    this.nomeCliente = nome;
    return this;
  }

  public NotaFiscalBuilder doEndereco(String endereco) {
    this.endereco = endereco;
    return this;
  }

  public NotaFiscalBuilder comOProduto(String nome, double preco) {
    this.produtos.add(new Produto(nome, preco));
    return this;
  }

  public NotaFiscal constroi() {
    return new NotaFiscal(data, nomeCliente, endereco, produtos);
  }

}
```

Perceba que os métodos intermediários do Builder retornam `this`, ou seja, o próprio Builder. Isso permite que as chamadas sejam encadeadas, dando maior fluência e legibilidade.

Há implementações mais elaboradas, que usam diferentes _Builders_ para cada etapa. Assim, a construção do objeto é guiada para ser usada em uma determinada ordem, evitando estados inválidos. Além disso, o _auto-complete_ das IDEs é potencializado.

> _Use o Builder pattern quando:_
>
> - _o algoritmo para criar um objeto complexo deve ser independente das partes que compõem o objeto e como elas são montadas._
>
> - _o processo de construção deve permitir diferentes representações para o objeto que é construído._
>
> GoF (Gamma & Helm & Johnson & Vlissides) no livro [Design Patterns](https://www.amazon.com/Design-Patterns-Object-Oriented-Addison-Wesley-Professional-ebook/dp/B000SEIBB8) (GAMMA et al., 1994)

## Exercício: um builder para os capítulos

### Objetivo

Resolva os erros de compilação do projeto `cotuba-cli`.

Na classe `Cotuba`, use o construtor apropriado de `Ebook`.

Faça `AplicadorTema` retornar uma `String` com o HTML que contém os temas.

Crie um Builder para `Capitulo` e o utilize na classe `RenderizadorMDParaHTMLComCommonMark`.

### Passo a passo

1. Na classe `Cotuba`, troque a chamada ao construtor padrão e aos setters pelo construtor de `Ebook`, passando os parâmetros:

  ####### cotuba.application.Cotuba

  ```java
  E̶b̶o̶o̶k̶ ̶e̶b̶o̶o̶k̶ ̶=̶ ̶n̶e̶w̶ ̶E̶b̶o̶o̶k̶(̶)̶;̶
  e̶b̶o̶o̶k̶.̶s̶e̶t̶F̶o̶r̶m̶a̶t̶o̶(̶f̶o̶r̶m̶a̶t̶o̶)̶;̶
  e̶b̶o̶o̶k̶.̶s̶e̶t̶A̶r̶q̶u̶i̶v̶o̶D̶e̶S̶a̶i̶d̶a̶(̶a̶r̶q̶u̶i̶v̶o̶D̶e̶S̶a̶i̶d̶a̶)̶;̶
  e̶b̶o̶o̶k̶.̶s̶e̶t̶C̶a̶p̶i̶t̶u̶l̶o̶s̶(̶c̶a̶p̶i̶t̶u̶l̶o̶s̶)̶;̶

  Ebook ebook = new Ebook(formato, arquivoDeSaida, capitulos); // inserido
  ```

2. Retorne o HTML com os temas no método `aplica` da classe `AplicadorTema`:

  ####### cotuba.tema.AplicadorTema

  ```java
  public class AplicadorTema {

    p̶u̶b̶l̶i̶c̶ ̶v̶o̶i̶d̶ ̶a̶p̶l̶i̶c̶a̶(̶C̶a̶p̶i̶t̶u̶l̶o̶ ̶c̶a̶p̶i̶t̶u̶l̶o̶)̶ ̶{̶
    public String aplica(Capitulo capitulo) { // modificado

      // código omitido...

      c̶a̶p̶i̶t̶u̶l̶o̶.̶s̶e̶t̶C̶o̶n̶t̶e̶u̶d̶o̶H̶T̶M̶L̶(̶d̶o̶c̶u̶m̶e̶n̶t̶.̶h̶t̶m̶l̶(̶)̶)̶;̶
      return document.html(); // inserido
    }

  }
  ```

3. Crie uma classe `CapituloBuilder` em um novo pacote `cotuba.domain.builder`:

  ####### cotuba.domain.builder.CapituloBuilder

  ```java
  package cotuba.domain.builder;

  import cotuba.domain.Capitulo;

  public class CapituloBuilder {

    private String titulo;
    private String conteudoHTML;

    public CapituloBuilder comTitulo(String titulo) {
      this.titulo = titulo;
      return this;
    }

    public CapituloBuilder comConteudoHTML(String conteudoHTML) {
      this.conteudoHTML = conteudoHTML;
      return this;
    }

    public Capitulo constroi() {
      return new Capitulo(titulo, conteudoHTML);
    }

  }
  ```

4. Na classe `RenderizadorMDParaHTMLComCommonMark`, troque a instanciação de `Capitulo` por `CapituloBuilder`:

  ####### cotuba.md.RenderizadorMDParaHTMLComCommonMark

  ```java
  arquivosMD
    .filter(matcher::matches)
    .sorted()
    .forEach(arquivoMD -> {

      C̶a̶p̶i̶t̶u̶l̶o̶ ̶c̶a̶p̶i̶t̶u̶l̶o̶ ̶=̶ ̶n̶e̶w̶ ̶C̶a̶p̶i̶t̶u̶l̶o̶(̶)̶;̶
      CapituloBuilder capituloBuilder = new CapituloBuilder(); // inserido

      // restante do código...
  ```

  Adicione o import necessário:

  ####### cotuba.md.RenderizadorMDParaHTMLComCommonMark

  ```java
  import cotuba.domain.builder.CapituloBuilder;
  ```

  Ocorrerão alguns erros de compilação nos usos da variável `capitulo`, que acabou de ser removida. Vamos corrigi-los!

5. Ainda em `RenderizadorMDParaHTMLComCommonMark`, use o builder para definir o título do capítulo, ao invés do setter:

  ####### cotuba.md.RenderizadorMDParaHTMLComCommonMark

  ```java
  c̶a̶p̶i̶t̶u̶l̶o̶.̶s̶e̶t̶T̶i̶t̶u̶l̶o̶(̶t̶i̶t̶u̶l̶o̶D̶o̶C̶a̶p̶i̶t̶u̶l̶o̶)̶;̶
  capituloBuilder.comTitulo(tituloDoCapitulo);
  ```

6. Em `RenderizadorMDParaHTMLComCommonMark`, remova o uso do setter para definir o HTML do capítulo.

  Como não temos mais um `Capitulo`, não podemos passá-lo como parâmetro para o método `aplica` do `AplicadorTema`.

  O que passar então? A `String` da variável `html`, que contém o HTML renderizado a partir do arquivo MD. No próximo passo, modificaremos `AplicadorTema`.

  Armazene o HTML com o CSS dos temas em uma `String` chamada `htmlComTemas`. Passe-a para o builder.

  Finalmente, instancie o `Capitulo` usando o método `constroi` do builder.

  ####### cotuba.md.RenderizadorMDParaHTMLComCommonMark

  ```java
  String html = renderer.render(document);

  c̶a̶p̶i̶t̶u̶l̶o̶.̶s̶e̶t̶C̶o̶n̶t̶e̶u̶d̶o̶H̶T̶M̶L̶(̶h̶t̶m̶l̶)̶;̶

  AplicadorTema tema = new AplicadorTema();

  t̶e̶m̶a̶.̶a̶p̶l̶i̶c̶a̶(̶c̶a̶p̶i̶t̶u̶l̶o̶)̶;̶
  String htmlComTemas = tema.aplica(html); // modificado

  capituloBuilder.comConteudoHTML(htmlComTemas); // inserido

  Capitulo capitulo = capituloBuilder.constroi();  // inserido

  capitulos.add(capitulo);
  ```

7. Modifique o método `aplica` de `AplicadorTema` para que receba uma `String` só com o HTML e não o `Capitulo`:

  ####### cotuba.tema.AplicadorTema

  ```java
  public class AplicadorTema {

    p̶u̶b̶l̶i̶c̶ ̶S̶t̶r̶i̶n̶g̶ ̶a̶p̶l̶i̶c̶a̶(̶C̶a̶p̶i̶t̶u̶l̶o̶ ̶c̶a̶p̶i̶t̶u̶l̶o̶)̶ ̶{̶
    public String aplica(String html) { // modificado

      S̶t̶r̶i̶n̶g̶ ̶h̶t̶m̶l̶ ̶=̶ ̶c̶a̶p̶i̶t̶u̶l̶o̶.̶g̶e̶t̶C̶o̶n̶t̶e̶u̶d̶o̶H̶T̶M̶L̶(̶)̶;̶

      // código omitido...

      return document.html();
    }

  }
  ```

  Remova o import desnecessário:

  ####### cotuba.tema.AplicadorTema

  ```java
  i̶m̶p̶o̶r̶t̶ ̶c̶o̶t̶u̶b̶a̶.̶d̶o̶m̶a̶i̶n̶.̶C̶a̶p̶i̶t̶u̶l̶o̶;̶
  ```

8. Todos os erros de compilação devem ter sido corrigidos.

  Faça o build dos projetos cujos códigos foram modificados: `cotuba-cli` e `estatisticas-ebook`.
  
  Teste a geração de PDFs e EPUBs. Deve funcionar!

## Código invejoso x Código tímido

Digamos que temos uma classe `Tributacao`, conforme a seguir:

```java
public class Tributacao {

  public BigDecimal calcula (NotaFiscal nota) {

    //...

    if (nota.getEndereco().getEstado() == Estado.SP &&
        nota.getEmpresa().getSegmento().getSetor() == Setor.SERVICOS &&
        nota.getEmpresa().getRegime() == RegimeFiscal.LUCRO_PRESUMIDO) {

      BigDecimal uniao = new BigDecimal("3.65");
      BigDecimal imcsEstadual = new BigDecimal("0.0");
      BigDecimal issMunicipal = new BigDecimal("5.0");

      BigDecimal tributacao = uniao.add(imcsEstadual).add(issMunicipal);

      BigDecimal total = BigDecimal.ZERO;
      List<Item> itens = nota.getItens();
      for (Item item : itens) {
        total = total.add(item.getValor());
      }

      BigDecimal tributo = total.multiply(tributacao).divide(new BigDecimal("100"));

      return tributo;

    }

    //...

  }

}
```

O método `calcula` de `Tributacao` retorna o valor dos tributos para uma dada uma nota fiscal. Para isso, baseia-se:

- no estado
- no setor e regime fiscal da empresa
- no valor total dos itens da nota
- em valores tributários pré-determinados para os âmbitos federal, estadual e municipal.

Qual é o problema com o código anterior?

Um dos maus cheiros de código descritos por Kent Beck e Martin Fowler no livro [Refactoring](https://www.amazon.com/Refactoring-Improving-Design-Existing-Code/dp/0201485672/) (FOWLER et al., 1999) é a **Inveja de funcionalidades**:

_A essência dos objetos é que eles são uma técnica para empacotar dados com os processamentos desses dados._

_Um indício clássico de problema é um método que parece mais interessado em uma classe diferente daquela na qual ele se encontra._

_O foco mais comum da inveja são os dados._

A classe `Tributacao` é invejosa. Veja dados de outras classes que são manipulados:

- `nota.getEndereco().getEstado()`
- `nota.getEmpresa().getSegmento().getSetor()`
- `nota.getEmpresa().getRegime()`
- `nota.getItens()`
- `item.getValor()`

O mais problemático, em termos de design de código, é que cálculos e procedimentos sobre esses dados estão sendo feitos fora dos objetos que os contém.

Para ilustrar o problema de classes invejosas, no artigo [The Art of Enbugging](https://media.pragprog.com/articles/jan_03_enbug.pdf) (HUNT; THOMAS, 2003), Andy Hunt e Dave Thomas citam uma analogia criada por David Bock:

_O entregador de jornais e a carteira_

_Suponha que o entregador de jornais vá até a sua porta, demandando o pagamento da semana. Você vira, o entregador puxa a carteira do bolso traseiro da sua calça, tira duas notas e devolve a carteira._

Um entregador de jornais puxar sua carteira e tirar o que quiser dela não parece uma boa ideia, não é mesmo? O correto seria você mesmo manipular a sua própria carteira, entregando somente as notas que você precisa.

A classe `Tributacao` é como o entregador de jornais: está "abrindo a carteira" das classes `Nota` e `Item` e fazendo processamentos que deveriam ser feitos pelas próprias classes.

No mesmo artigo, Andy Hunt e Dave Thomas sugerem que código bom é **tímido**:

_O objetivo fundamental [...] é escrever código tímido: código que não revela muito de si pra ninguém e não conversa com os outros mais do que o necessário._

_Código tímido evita contato com os outros, não é como aquele vizinho fofoqueiro que está envolvido nas idas e vindas de todo mundo._

_Código tímido nunca mostraria suas coisas “privadas” para os “amigos” [...]._

_Assim como no mundo real, boas cercas fazem bons vizinhos – contanto que você não olhe pela cerca._

## Encapsulamento

As analogias de código invejoso, do entregador de jornais enxerido e de código tímido trazem uma maneira bem humorada e ilustrativa de falar de uma ideia muito importante no bom design de código: o **encapsulamento**.

Encapsular é esconder, o máximo possível, as informações de um classe. Assim, evitamos que detalhes de implementação "vazem" para outras classes do sistema.

### Encapsulamento x Java Beans

Comumente, estudamos encapsulamento apenas como o uso do modificador _private_ no atributos de uma classe. Estaríamos restringindo a manipulação dos atributos à própria classe. Assim, não precisaríamos olhar toda a base de código para saber quais trechos manipulam esses atributos.

Atributos privados ajudam, mas não garantem o encapsulamento.

Ao definirmos getters e setters, podemos estar compartilhando detalhes da classe indevidamente, tornando fácil cálculos e processamentos sobre os dados fora da própria classe. Os getters e setters podem levar à inveja no nosso design!

O triste é constatar que várias bibliotecas da plataforma Java requerem Java Beans, que são classes com atributos privados, getters e setters e o construtor sem parâmetros. As bibliotecas usam esse formato comum para manipular, via Reflection API, os Java Beans. Mas o design acaba direcionado a um mau caminho.

Mesmo evitando setters, ainda não há garantia de encapsulamento: perceba que a classe `Tributacao` usa apenas getters. Porém, obtém detalhes das outras classes e realiza lógica de negócio com os dados obtidos.

### Encapsulamento x Herança

Conforme descrito no livro [Design Patterns](https://www.amazon.com/Design-Patterns-Object-Oriented-Addison-Wesley-Professional-ebook/dp/B000SEIBB8) (GAMMA et al., 1994), há um conflito entre herança e encapsulamento:

_Como a herança expõe uma subclasse a detalhes da implementação de sua superclasse, costuma-se dizer que "a herança quebra o encapsulamento"._

_A implementação de uma subclasse fica tão ligada à implementação de sua superclasse que qualquer alteração na implementação da superclasse forçará a subclasse a mudar._

## A Lei de Deméter e o lema "Diga, não pergunte"

Se atributos privados e a ausência de setters não garantem o encapsulamento, o que fazer?

Andy Hunt e Dave Thomas, em seu livro [Pragmatic Programmer](https://www.amazon.com.br/Pragmatic-Programmer-Journeyman-Master/dp/020161622X) (HUNT; THOMAS, 1999) citam o trabalho de Ian Holland na Northeastern University que, por volta de 1987, cunhou a **Lei de Deméter**.

Essa "lei" afirma que todo método de um objeto deve chamar apenas métodos pertencentes a:

- si mesmo
- quaisquer parâmetros que foram passados para o método
- quaisquer objetos criados
- qualquer composição

É uma maneira mais detalhada de declarar que um bom objeto apenas interage com seus "vizinhos" imediatos.

_Curiosidade: Deméter ou Demetra é a deusa grega da agricultura. Era o nome do projeto da Northeastern University que deu origem ao termo._

A classe `Tributacao` infringe a lei de Deméter porque chama métodos de objetos que não são seus colaboradores imediatos.

Um exemplo é o trecho `nota.getEmpresa().getSegmento().getSetor()`. Obtemos o setor do segmento da empresa da nota fiscal. É muita inveja e intromissão na vida alheia!

O fato de invocarmos percorrermos os itens da nota fiscal para calcular o valor total é outra violação dessa lei. Um item é um detalhe da nota. A classe `Tributacao` não deveria lidar com itens e nem sequer saber que eles existem!

No artigo [The Art of Enbugging](https://media.pragprog.com/articles/jan_03_enbug.pdf) (HUNT; THOMAS, 2003), Andy Hunt e Dave Thomas indicam que um lema de OO deveria ser **Tell, don't Ask**. Em português, algo como "Diga, não pergunte":

_Envie comandos para objetos dizendo o que você quer fazer._
_Explicitamente, não queremos consultar um objeto sobre seu estado, tomar uma decisão e, então, dizer ao objeto o que fazer._

> _Perceber se um código está bem encapsulado ou não, não é tão difícil._
>
> _[...] se pergunte:_
>
> _O que esse método faz? Provavelmente sua resposta será: eu sei o que o método faz pelo nome dele [...]_
>
> _Como ele faz isso? Sua resposta provavelmente é: se eu olhar só para esse código, não dá para responder._
>
> Maurício Aniche, no livro [OO e SOLID para Ninjas](https://www.casadocodigo.com.br/products/livro-oo-solid) (ANICHE, 2015)

## Maximizando o uso do this

Um outro modo, talvez um pouco mais suave, da gente enxergar o nível de encapsulamento das nossas classes é olhando o quanto estamos usando o **this**. Exclua os getters e setters, como está o uso do **this** no seu projeto? Por exemplo, no código relativo a classe `NotaFiscal`, em vez de usarmos todos os getters para executar uma lógica, poderíamos encapsular aquele trecho em um método da própria classe. O código atual está como segue abaixo:

```java
    if (nota.getEndereco().getEstado() == Estado.SP &&
        nota.getEmpresa().getSegmento().getSetor() == Setor.SERVICOS &&
        nota.getEmpresa().getRegime() == RegimeFiscal.LUCRO_PRESUMIDO)
```

Poderia ser algo como:

```java
    if (nota.algumNomeDeMetodoAqui(Estado.SP,Setor.Servicos,RegimeFiscal.LUCRO_PRESUMIDO))
```

Perceba que é um jeito super simples de você buscar juntar comportamento e estado, o que é a base da Orientação a Objetos.

## Quebra de encapsulamento no plugin de estatísticas

Repare no código da classe `CalculadoraEstatisticas` do projeto `estatisticas-ebook`:

```java
public class CalculadoraEstatisticas implements AoFinalizarGeracao {

@Override
public void aposGeracao(Ebook ebook) {

  ContagemPalavras contagemPalavras = new ContagemPalavras();

  // código que adiciona palavras omitido...

  for (Map.Entry<String, Integer> contagem : contagemPalavras.entrySet()) {
    String palavra = contagem.getKey();
    Integer ocorrencias = contagem.getValue();
    System.out.println(palavra + ": " + ocorrencias);
  }

}
```

São realizados os seguintes passos:

- um objeto da classe `ContagemPalavras` é instanciado
- o objeto é preenchido com as palavras extraídas do ebook
- através do método `entrySet`, é obtido um `Set` com os pares chave-valor
- o `Set` é percorrido em um _for-each_

O método `entrySet` retorna o `Set` de `Map.Entry` do mapa usado internamente por `ContagemPalavras`:

```java
public class ContagemPalavras {

  private Map<String, Integer> map = new TreeMap<>();

  // código omitido...

  public Set<Map.Entry<String, Integer>> entrySet() {
    return map.entrySet();
  }

}
```

O encapsulamento de `ContagemPalavras` é quebrado no método `entrySet`. Um detalhe de implementação, o `Map.Entry` de `Map`, vaza para as classes que usam esse método.

Para resolver esse problema seria interessante que pudéssemos percorrer com um for-each o próprio objeto da classe `ContagemPalavras`.

## Design Pattern: Iterator

O acesso às estruturas de dados usadas internamente, seja `List`, `Set` ou qualquer outra, deve ficar encapsulado ao objeto que as define. Não deve haver vazamentos.

Mas como acessar o conteúdo dessas estruturas de dados sem expô-las?

O livro [Design Patterns](https://www.amazon.com/Design-Patterns-Object-Oriented-Addison-Wesley-Professional-ebook/dp/B000SEIBB8) (GAMMA et al., 1994) cataloga o pattern **Iterator**, uma maneira de acessar sequencialmente um objeto agregado, como um lista, sem expôr sua representação interna.

Na API de Collections do Java, há a interface `java.util.Iterator` que define os métodos:

- `hasNext`, que retorna um `boolean` indicando se há mais elementos
- `next`, que retorna o próximo elemento da iteração

Várias das Collections do Java, como `List` e `Set`, possuem métodos que retornam um `Iterator`.

Podemos retornar o `Iterator` obtido a partir do `entrySet` do mapa que é usado internamente pela classe `ContagemPalavras`:

```java
public class ContagemPalavras {

  //...

  public Iterator<Map.Entry<String, Integer>> iterator() {
    Set<Map.Entry<String, Integer>> entrySet = this.map.entrySet();
    Iterator<Map.Entry<String, Integer>> iterator = entrySet.iterator();
    return iterator;
  }

}
```

Porém, o próprio `Map.Entry` está relacionado a um detalhe interno de `ContagemPalavras`: o uso de um `Map`.

Seria interessante definir uma classe de negócio que contenha uma palavra com a quantidade associada.

Podemos defini-la como uma classe aninhada dentro de `ContagemPalavras`:

```java
public class ContagemPalavras {

  public static final class Contagem {

    private final String palavra;
    private final int quantidade;

    public Contagem(String palavra, int quantidade) {
      this.palavra = palavra;
      this.quantidade = quantidade;
    }

    public String getPalavra() {
      return palavra;
    }

    public int getQuantidade() {
      return quantidade;
    }
  }

  // código omitido...

}
```

Então, trocaríamos o `Iterator<Map.Entry<String, Integer>>` por um `Iterator<Contagem>`.

Para isso, precisaríamos fornecer uma implementação da interface `Iterator` que pode ser uma classe anônima:

```java
public class ContagemPalavras {

  // código omitido...

  public Iterator<ContagemPalavras.Contagem> iterator() {

    Iterator<Map.Entry<String, Integer>> iterator = this.map.entrySet().iterator();

    return new Iterator<ContagemPalavras.Contagem>() {

      @Override
      public boolean hasNext() {
        return iterator.hasNext();
      }

      @Override
      public ContagemPalavras.Contagem next() {
        Map.Entry<String, Integer> entry = iterator.next();
        String palavra = entry.getKey();
        int quantidade = entry.getValue();
        return new ContagemPalavras.Contagem(palavra, quantidade);
      }
    };
  }

}
```

O `Iterator<Map.Entry<String, Integer>>` é usado internamente na implementação dos métodos `hasNext` e `next`.

No método `next`, instanciamos um objeto da classe `Contagem` a partir do par chave-valor obtido.

## Iteração externa x iteração interna

Na classe `CalculadoraEstatisticas`, que usa `ContagemPalavras`, percorreríamos o `Iterator` usando os métodos `hasNext` e `next`:

```java
Iterator<ContagemPalavras.Contagem> iterator = contagemPalavras.iterator();

while (iterator.hasNext()) {

  ContagemPalavras.Contagem contagem = iterator.next();

  String palavra = contagem.getPalavra();
  Integer ocorrencias = contagem.getQuantidade();
  System.out.println(palavra + ": " + ocorrencias);

}
```

Quando o cliente, ou seja, a classe que usa o Iterator controla a obtenção dos valores temos uma **iteração externa**.

A alternativa é a **iteração interna**, em que o próprio Iterator controla os valores que serão fornecidos para o cliente.

A plataforma Java, desde a versão 5, possui a interface `java.lang.Iterable`. Classes que implementam essa interface podem ser usadas diretamente em um for-each. Apenas um método é definido: o `iterator`, que retorna um `Iterator`.

Precisamos apenas definir a classe `ContagemPalavras` como implementação de `Iterable<ContagemPalavras.Contagem>`:

```java
public class ContagemPalavras implements Iterable<ContagemPalavras.Contagem> {
  // código omitido...
}
```

Como já havíamos definido o método `iterator`, a interface já está implementada.

No cliente de `ContagemPalavras`, a classe `CalculadoraEstatisticas`, podemos simplificar a iteração, deixando apenas o for-each:

```java
for (ContagemPalavras.Contagem contagem : contagemPalavras) {
  String palavra = contagem.getPalavra();
  Integer ocorrencias = contagem.getQuantidade();
  System.out.println(palavra + ": " + ocorrencias);
}
```

Dessa maneira, não há nenhuma menção a detalhes internos da classe `ContagemPalavras`. Apenas adicionamos palavras e percorremos os resultados, obtendo cada contagem. Não dá pra saber como isso é feito, apenas o que é feito. Detalhes encapsulados!

## Exercício: iteração interna para favorecer o encapsulamento

### Objetivo

Faça com que a classe `ContagemPalavras` implemente um `Iterable`.

Na classe `CalculadoraEstatisticas`, percorra a `ContagemPalavras` com um for-each.

### Passo a passo

1. Define uma classe imutável `Contagem`, dentro de `ContagemPalavras`, que contém os atributos `palavra`, do tipo `String`, e `quantidade`, do tipo `int`. Crie os getters e um construtor que recebe todos os atributos:

  ####### br.com.cognitio.estatisticas.ContagemPalavras

  ```java
  public class ContagemPalavras {

    // inserido
    public static final class Contagem {

      private final String palavra;
      private final int quantidade;

      public Contagem(String palavra, int quantidade) {
        this.palavra = palavra;
        this.quantidade = quantidade;
      }

      public String getPalavra() {
        return palavra;
      }

      public int getQuantidade() {
        return quantidade;
      }
    }

    // código omitido...

  }
  ```

2. Implemente, na classe `ContagemPalavras`, a interface `Iterable<ContagemPalavras.Contagem>`.

  Dica: digite CTRL+1 em cima do nome da classe e escolha a opção _Add unimplemented methods..._

  ####### br.com.cognitio.estatisticas.ContagemPalavras

  ```java
  // outros imports...
  import java.util.Iterator;

  p̶u̶b̶l̶i̶c̶ ̶c̶l̶a̶s̶s̶ ̶C̶o̶n̶t̶a̶g̶e̶m̶P̶a̶l̶a̶v̶r̶a̶s̶ ̶{̶
  public class ContagemPalavras implements Iterable<Contagem> { // modificado

    // código omitido...

    // inserido
    @Override
    public Iterator<ContagemPalavras.Contagem> iterator() {
      return null;
    }

  }
  ```

3. Complete o método `iterator` de `ContagemPalavras`, fornecendo uma classe anônima que implementa `Iterator<ContagemPalavras.Contagem>`:

  ####### br.com.cognitio.estatisticas.ContagemPalavras

  ```java
  Iterator<Map.Entry<String, Integer>> iterator = this.map.entrySet().iterator();

  return new Iterator<ContagemPalavras.Contagem>() {

    @Override
    public boolean hasNext() {
      return iterator.hasNext();
    }

    @Override
    public ContagemPalavras.Contagem next() {
      Map.Entry<String, Integer> entry = iterator.next();
      String palavra = entry.getKey();
      int quantidade = entry.getValue();
      return new ContagemPalavras.Contagem(palavra, quantidade);
    }
  };
  ```

4. No método `aposGeracao` da classe `CalculadoraEstatisticas`, modifique a iteração, removendo referências a detalhes internos da classe `ContagemPalavras`:

  ####### br.com.cognitio.estatisticas.CalculadoraEstatisticas

  ```java
  f̶o̶r̶ ̶(̶E̶n̶t̶r̶y̶<̶S̶t̶r̶i̶n̶g̶,̶ ̶I̶n̶t̶e̶g̶e̶r̶>̶ ̶c̶o̶n̶t̶a̶g̶e̶m̶ ̶:̶ ̶c̶o̶n̶t̶a̶g̶e̶m̶P̶a̶l̶a̶v̶r̶a̶s̶.̶e̶n̶t̶r̶y̶S̶e̶t̶(̶)̶)̶ ̶{̶
  for (ContagemPalavras.Contagem contagem : contagemPalavras) { // modificado

    S̶t̶r̶i̶n̶g̶ ̶p̶a̶l̶a̶v̶r̶a̶ ̶=̶ ̶c̶o̶n̶t̶a̶g̶e̶m̶.̶g̶e̶t̶K̶e̶y̶(̶)̶;̶
    String palavra = contagem.getPalavra(); // modificado

    I̶n̶t̶e̶g̶e̶r̶ ̶o̶c̶o̶r̶r̶e̶n̶c̶i̶a̶s̶ ̶=̶ ̶c̶o̶n̶t̶a̶g̶e̶m̶.̶g̶e̶t̶V̶a̶l̶u̶e̶(̶)̶;̶
    Integer ocorrencias = contagem.getQuantidade(); // modificado

    System.out.println(palavra + ": " + ocorrencias);

  }
  ```

  Limpe o import desnecessário:

  ####### br.com.cognitio.estatisticas.CalculadoraEstatisticas

  ```java
  i̶m̶p̶o̶r̶t̶ ̶j̶a̶v̶a̶.̶u̶t̶i̶l̶.̶M̶a̶p̶;̶
  ```

5. Faça o build do projeto `estatisticas-ebook` e teste a geração de PDFs e EPUBs. Veja se a contagem de palavras continua a funcionar.

## MVC ferido

Há um detalhe do projeto `estatisticas-ebook` que fere o MVC.

Mais especificamente, no método `aposGeracao` de `CalculadoraEstatisticas`:

```java
System.out.println(palavra + ": " + ocorrencias);
```

As impressões na saída padrão com `System.out.println()` são a UI do Cotuba. Nos termos do MVC, é parte da _View_.

No padrão MVC, o _Controller_ deveria ser o responsável por manipular a View.

Mas a View está sendo manipulada diretamente por uma mera implementação de um dos plugins.

Isso não é MVC!

## Um objeto para impressão

Qual classe deveria imprimir as mensagens? O nosso Controller: a `Main` do projeto `cotuba-cli`.

A classe `CalculadoraEstatisticas` deveria, de alguma maneira, enviar uma mensagem ao Cotuba.

Então, a mensagem seria repassada para a classe `Main`, que seria responsável por imprimi-la.

Mas como fazer essa implementação?

Poderíamos criar uma classe `ImprimeNoConsole`, que recebe uma mensagem e a imprime na saída padrão.

Em qual pacote colocar essa classe? No mesmo de `Main`!

```java
package cotuba.cli;

public class ImprimeNoConsole {

  public void imprime(String mensagem) {
    System.out.println(mensagem);
  }

}
```

A classe `Main`, ao invocar o método `executa` de `Cotuba`, passa uma instância de `ImprimeNoConsole`:

```java
public class Main {

  public static void main(String[] args) {

    // código omitido...

    Cotuba cotuba = new Cotuba();

    cotuba.executa(opcoesCLI, new ImprimeNoConsole()); // modificado

    // código omitido...

  }

}
```

Teríamos que alterar a assinatura do método `executa` do `Cotuba` para receber o `ImprimeNoConsole`, ajustando os imports:

```java
import cotuba.cli.ImprimeNoConsole; // inserido

public class Cotuba {

  public void executa(ParametrosCotuba parametros, ImprimeNoConsole impressao) { // modificado

    // código omitido...

  }

}
```

Opa! Peraí... Ao usar `ImprimeNoConsole` em `Cotuba`, desrespeitamos o DIP: a classe `Cotuba`, de alto nível, passaria a depender de uma classe de baixo nível.

Como resolver? Uma abstração de alto nível!

## Uma abstração de uma ação com o Command Pattern

Podemos ter uma abstração associada ao plugin `AoFinalizarGeracao` para uma ação menos específica que uma impressão.

Para isso podemos definir a interface `AcaoPosGeracao` no pacote `cotuba.plugin`, contendo um método `executa` que recebe uma mensagem:

```java
package cotuba.plugin;

public interface AcaoPosGeracao {
  void executa(String mensagem);
}
```

Essa abstração mais geral, que indica uma ação que deve ser feita sem definir exatamente o que é feito é catalogada no livro [Design Patterns](https://www.amazon.com/Design-Patterns-Object-Oriented-Addison-Wesley-Professional-ebook/dp/B000SEIBB8) (GAMMA et al., 1994) como o _Command Pattern_, conforme mencionado em capítulos anteriores.

A classe `ImprimeNoConsole` passaria a implementar `AcaoPosGeracao`. Para isso, teríamos que trocar o nome do método `imprime` por `executa`:

```java
import cotuba.plugin.AcaoPosGeracao;

public class ImprimeNoConsole implements AcaoPosGeracao {

  p̶u̶b̶l̶i̶c̶ ̶v̶o̶i̶d̶ ̶i̶m̶p̶r̶i̶m̶e̶(̶S̶t̶r̶i̶n̶g̶ ̶m̶e̶n̶s̶a̶g̶e̶m̶)̶ ̶{̶
  public void executa(String mensagem) {
    System.out.println(mensagem);
  }

}
```

A classe `Cotuba` passaria a receber uma implementação de `AcaoPosGeracao`, repassando-a ao método `gerou` de `AoFinalizarGeracao`.

O objeto da classe `ImprimeNoConsole` continuaria sendo instanciado por `Main`, já que essa classe implementa `AcaoPosGeracao`.

```java
i̶m̶p̶o̶r̶t̶ ̶c̶o̶t̶u̶b̶a̶.̶c̶l̶i̶.̶I̶m̶p̶r̶i̶m̶e̶N̶o̶C̶o̶n̶s̶o̶l̶e̶;̶

public class Cotuba {

  public void executa(ParametrosCotuba parametros, AcaoPosGeracao acaoPosGeracao) { // modificado

    // código omitido...

    AoFinalizarGeracao.gerou(ebook, acaoPosGeracao); // modificado

  }

}
```

Perceba que a dependência seria invertida! A classe de baixo nível `ImprimeNoConsole` não seria usada diretamente por `Cotuba`, de alto nível, que usaria a abstração de alto nível `AcaoPosGeracao`.

DIP respeitado!

Na interface `AoFinalizarGeracao`, receberíamos uma `AcaoPosGeracao` e a repassaríamos para os plugins:

```java
public interface AoFinalizarGeracao {

  void aposGeracao(Ebook ebook, AcaoPosGeracao acaoPosGeracao);  // modificado

  static void gerou(Ebook ebook, AcaoPosGeracao acaoPosGeracao) {  // modificado

    ServiceLoader<AoFinalizarGeracao> loader = ServiceLoader.load(AoFinalizarGeracao.class);
    for (AoFinalizarGeracao plugin : loader) {

      plugin.aposGeracao(ebook, acaoPosGeracao); // modificado

    }
  }

}
```

No projeto `estatisticas-ebook`, a classe `CalculadoraEstatisticas` passaria uma `AcaoPosGeracao`.

Ao invés de invocar `System.out.println()`, passaríamos a usar o método `executa` da `AcaoPosGeracao`.

```java
public class CalculadoraEstatisticas implements AoFinalizarGeracao {

  @Override
  public void aposGeracao(Ebook ebook, AcaoPosGeracao acaoPosGeracao) { // modificado

    // código omitido...

    for (ContagemPalavras.Contagem contagem : contagemPalavras) {
      String palavra = contagem.getPalavra();
      Integer ocorrencias = contagem.getQuantidade();

      S̶y̶s̶t̶e̶m̶.̶o̶u̶t̶.̶p̶r̶i̶n̶t̶l̶n̶(̶p̶a̶l̶a̶v̶r̶a̶ ̶+̶ ̶"̶:̶ ̶"̶ ̶+̶ ̶o̶c̶o̶r̶r̶e̶n̶c̶i̶a̶s̶)̶;̶
      acaoPosGeracao.executa(palavra + ": " + ocorrencias); // modificado

    }

  }

}
```

## Uma abordagem mais funcional

Poderíamos omitir a classe `ImprimeNoConsole`.

Para isso, na classe `Main`, poderíamos definir uma classe anônima como implementação de `AcaoPosGeracao`:

```java
cotuba.executa(opcoesCLI,  new AcaoPosGeracao () {

  @Override
  public void executa(String mensagem) {
    System.out.println(mensagem);
  }

});
```

Com o Java 8, surgiu o conceito de _Functional Interface_, ou interface funcional, uma interface que define apenas um método em seu contrato.

Como possui apenas o método `executa`, a interface `AcaoPosGeracao` é uma interface funcional.

Podemos usar expressões _lambda_ para definir implementações de interfaces funcionais. A sintaxe é bem mais enxuta que a das classes anônimas:

```java
cotuba.executa(opcoesCLI, (String mensagem) -> {
  System.out.println(mensagem);
});
```

As expressões lambdas possuem inferência de tipos, o que nos permite omitir a declaração do tipo do parâmetro `mensagem`:

```java
cotuba.executa(opcoesCLI, mensagem -> {
  System.out.println(mensagem);
});
```

As chaves (`{` e `}`) na definição da implementação podem ser omitidas quando há apenas uma linha:

```java
cotuba.executa(opcoesCLI, mensagem -> System.out.println(mensagem));
```

Quando a expressão lambda apenas repassa o parâmetro para um método, podemos usar um _method reference_:

```java
cotuba.executa(opcoesCLI, System.out::println);
```

## Usando uma interface funcional pré-definida

Podemos ir além. A própria biblioteca padrão do Java, em sua versão 8, passou a ter interfaces funcionais  pré-definidas no pacote `java.util.function`.

Entre essas, há a interface `Consumer<T>`, uma abstração de um objeto que recebe um valor e o consome. É definido o método `accept`, que recebe um valor do tipo `T` e cujo retorno é `void`.

A interface `AcaoPosGeracao` poderia ser removida.

Em `Main`, continuaríamos passando o method reference `System.out::println`.

Já em `Cotuba`, o método `executa` passaria a receber um `Consumer<String>`. A invocação do método `gerou` de `AoFinalizarGeracao` continuaria intacta:

```java
// outros imports...
i̶m̶p̶o̶r̶t̶ ̶c̶o̶t̶u̶b̶a̶.̶p̶l̶u̶g̶i̶n̶.̶A̶c̶a̶o̶P̶o̶s̶G̶e̶r̶a̶c̶a̶o̶;̶
import java.util.function.Consumer;

public class Cotuba {
  public void executa(ParametrosCotuba parametros, Consumer<String> acaoPosGeracao) { // modificado
    // código omitido...

    AoFinalizarGeracao.gerou(ebook, acaoPosGeracao);
  }
}
```

Tanto o método `aposGeracao` quanto o método estático `gerou` de `AoFinalizarGeracao` passariam a receber um `Consumer<String>`:

```java
// outros imports...
i̶m̶p̶o̶r̶t̶ ̶c̶o̶t̶u̶b̶a̶.̶p̶l̶u̶g̶i̶n̶.̶A̶c̶a̶o̶P̶o̶s̶G̶e̶r̶a̶c̶a̶o̶;̶
import java.util.function.Consumer;

public interface AoFinalizarGeracao {

  void aposGeracao(Ebook ebook, Consumer<String> acaoPosGeracao); // modificado

  static void gerou(Ebook ebook, Consumer<String> acaoPosGeracao) { // modificado

    ServiceLoader<AoFinalizarGeracao> loader = ServiceLoader.load(AoFinalizarGeracao.class);
    for (AoFinalizarGeracao plugin : loader) {
      plugin.aposGeracao(ebook, acaoPosGeracao);
    }
  }

}
```

Na classe `CalculadoraEstatisticas`, do projeto `estatisticas-ebook`, usaríamos o método `accept` do `Consumer<String>` para passarmos a mensagem do plugin ao Cotuba:

```java
// outros imports...
i̶m̶p̶o̶r̶t̶ ̶c̶o̶t̶u̶b̶a̶.̶p̶l̶u̶g̶i̶n̶.̶A̶c̶a̶o̶P̶o̶s̶G̶e̶r̶a̶c̶a̶o̶;̶
import java.util.function.Consumer;

public class CalculadoraEstatisticas implements AoFinalizarGeracao {

  @Override
  public void aposGeracao(Ebook ebook, Consumer<String> acaoPosGeracao) { // modificado

    // código omitido...

    for (ContagemPalavras.Contagem contagem : contagemPalavras) {
      String palavra = contagem.getPalavra();
      Integer ocorrencias = contagem.getQuantidade();

      S̶y̶s̶t̶e̶m̶.̶o̶u̶t̶.̶p̶r̶i̶n̶t̶l̶n̶(̶p̶a̶l̶a̶v̶r̶a̶ ̶+̶ ̶"̶:̶ ̶"̶ ̶+̶ ̶o̶c̶o̶r̶r̶e̶n̶c̶i̶a̶s̶)̶;̶
      acaoPosGeracao.accept(palavra + ": " + ocorrencias); // modificado

    }

  }

}
```

## Exercício opcional: respeitando o MVC com uma abordagem funcional

1. Na classe `Main`, passe o method reference `System.out::println` ao método `executa` de `Cotuba`:

  ####### cotuba.cli.Main

  ```java
  Cotuba cotuba = new Cotuba();
  cotuba.executa(opcoesCLI, System.out::println);
  ```

2. Receba o `Consumer<String>` no método `executa` da classe `Cotuba`, repassando-o para o método estático `gerou` de `AoFinalizarGeracao`:

  ####### cotuba.application.Cotuba

  ```java
  public class Cotuba {

    public void executa(ParametrosCotuba parametros, Consumer<String> acaoPosGeracao) { // modificado

      // código omitido...

      AoFinalizarGeracao.gerou(ebook, acaoPosGeracao); // modificado

    }
  
  }
  ```

  Não esqueça de fazer o import correto:

  ####### cotuba.application.Cotuba

  ```java
  import java.util.function.Consumer;
  ```

3. Altere o método estático `gerou` de `AoFinalizarGeracao` para receber o `Consumer<String>`, repassando-o para o método `aposGeracao` de cada plugin:

  ####### cotuba.plugin.AoFinalizarGeracao

  ```java
  public interface AoFinalizarGeracao {

    void aposGeracao(Ebook ebook, Consumer<String> acaoPosGeracao); // modificado

    static void gerou(Ebook ebook, Consumer<String> acaoPosGeracao) { // modificado

      ServiceLoader<AoFinalizarGeracao> loader = ServiceLoader.load(AoFinalizarGeracao.class);
      for (AoFinalizarGeracao plugin : loader) {
        plugin.aposGeracao(ebook, acaoPosGeracao); // modificado
      }
    }

  }
  ```

  Faça o import necessário:

  ####### cotuba.plugin.AoFinalizarGeracao

  ```java
  import java.util.function.Consumer;
  ```

4. No projeto `estatisticas-ebook`, adeque o método `aposGeracao` de `CalculadoraEstatisticas` para receber o `Consumer<String>`.

  Troque a chamada à saída padrão pela invocação método `accept` do `Consumer`:

  ####### br.com.cognitio.estatisticas.CalculadoraEstatisticas

  ```java
  public class CalculadoraEstatisticas implements AoFinalizarGeracao {

    @Override
    public void aposGeracao(Ebook ebook, Consumer<String> acaoPosGeracao) { // modificado

      // código omitido...

      for (ContagemPalavras.Contagem contagem : contagemPalavras) {
        String palavra = contagem.getPalavra();
        Integer ocorrencias = contagem.getQuantidade();

        S̶y̶s̶t̶e̶m̶.̶o̶u̶t̶.̶p̶r̶i̶n̶t̶l̶n̶(̶p̶a̶l̶a̶v̶r̶a̶ ̶+̶ ̶"̶:̶ ̶"̶ ̶+̶ ̶o̶c̶o̶r̶r̶e̶n̶c̶i̶a̶s̶)̶;̶
        acaoPosGeracao.accept(palavra + ": " + ocorrencias); // modificado

      }

    }

  }
  ```

  Ajuste os imports, adicionando:

  ####### cotuba.plugin.AoFinalizarGeracao

  ```java
  import java.util.function.Consumer;
  ```

5. Faça o build dos projetos `cotuba-cli` e `estatisticas-ebook` e teste a geração de PDFs e EPUBs. Veja se a contagem de palavras continua a funcionar.
