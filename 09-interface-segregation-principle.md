# Interface Segregation Principle: clientes separados, interfaces separadas

## Uma interface, dois usos

A interface `Plugin` do Cotuba é implementada tanto pelo plugin de temas da Paradizo como pelo plugin de estatísticas da Cognitio.

No plugin de temas, não fazemos nada no método `aposGeracao`.

No plugin de estatísticas, que usa o _hook_ chamado ao final da geração do ebook, ou retornamos um valor nulo ou lançamos uma exceção no método `cssDoTema`.

Uma implementação não deveria fornecer apenas parte de uma abstração. É uma quebra da LSP.

Mas a questão principal é que temos dois usos diferentes, temas e finalização da geração, para a mesma interface `Plugin`.

Cada interface deveria ter um uso específico.

## O Princípio da Segregação de Interfaces

Uncle Bob trata do problema de diferentes usos para uma mesma interface definindo o seguinte princípio:

> **Interface Segregation Principle (ISP)**
>
> _Clientes não devem ser obrigados a depender de métodos que eles não usam._

Retornar um valor nulo ou lançar uma exceção não deveriam ser opções.

No artigo [Design Principles and Design Patterns](http://www.cvc.uab.es/shared/teach/a21291/temes/object_oriented_design/materials_adicionals/principles_and_patterns.pdf) (MARTIN, 2000), diz:

_Muitas interfaces específicas para cada cliente são melhores que uma interface de propósito geral._

De certa forma, o ISP trata de coesão para interfaces.

No livro [OO e SOLID para Ninjas](https://www.casadocodigo.com.br/products/livro-oo-solid) (ANICHE, 2015), Maurício Aniche declara:

_Interfaces coesas são aquelas cujos comportamentos são simples e bem definidos._
_[...]_
_Classes que dependem de interfaces leves sofrem menos com mudanças em outros pontos do sistema._

No fim das contas, não devemos ter abstrações genéricas.

Devemos pensar sempre no código que as usa.

Se o uso de uma abstração é diferente, devemos ter mais de uma interface (ou classe abstrata).

> _Clientes separados, interfaces separadas._
>
> Uncle Bob, no livro [Agile Principles, Patterns, and Practices in C#](https://www.amazon.com.br/Agile-Principles-Patterns-Practices-C/dp/0131857258) (MARTIN, 2006)

## Exercício: uma interface separada para temas

### Objetivo

No projeto `cotuba-cli`, crie uma interface `Tema` no pacote `cotuba.plugin`. Essa interface deve conter o método `cssDoTema` e o método estático `listaDeTemas`. Use a nova interface na classe `AplicadorTema`.

No projeto `tema-paradizo`, faça com que a classe `TemaParadizo` implemente a nova interface. Renomeie o arquivo de configuração do provider em `META-INF/services`.

### Passo a passo

1. No pacote `cotuba.plugin` do projeto `cotuba-cli`, adicione a interface `Tema`. Copie o método `cssDoTema` da interface `Plugin` para a nova interface:

  ####### cotuba.plugin.Tema

  ```java
  package cotuba.plugin;

  public interface Tema {

    String cssDoTema();

  }
  ```

2. Copie o método estático `listaDeTemas` da interface `Plugin` para a nova interface `Tema`. Mude o tipo passado ao `ServiceLoader` para `Tema`:

  ####### cotuba.plugin.Tema

  ```java
  public interface Tema {

    String cssDoTema();

    static List<String> listaDeTemas() {
      List<String> temas = new ArrayList<>();

      S̶e̶r̶v̶i̶c̶e̶L̶o̶a̶d̶e̶r̶<̶P̶l̶u̶g̶i̶n̶>̶ ̶l̶o̶a̶d̶e̶r̶ ̶=̶ ̶S̶e̶r̶v̶i̶c̶e̶L̶o̶a̶d̶e̶r̶.̶l̶o̶a̶d̶(̶P̶l̶u̶g̶i̶n̶.̶c̶l̶a̶s̶s̶)̶;̶
      ServiceLoader<Tema> loader = ServiceLoader.load(Tema.class); // modificado

      f̶o̶r̶ ̶(̶P̶l̶u̶g̶i̶n̶ ̶p̶l̶u̶g̶i̶n̶ ̶:̶ ̶l̶o̶a̶d̶e̶r̶)̶ ̶{̶
      for (Tema plugin : loader) { // modificado
        String css = plugin.cssDoTema();
        temas.add(css);
      }

      return temas;
    }

  }
  ```

  Não esqueça de fazer os imports:

  ####### cotuba.plugin.Tema

  ```java
  import java.util.ArrayList;
  import java.util.List;
  import java.util.ServiceLoader;
  ```

3. Na classe `AplicadorTema`, passe a usar o método estático `listaDeTemas` da interface `Tema`:

  ####### cotuba.tema.AplicadorTema

  ```java
  public class AplicadorTema {

    public void aplica(Capitulo capitulo) {
      // código omitido...

      L̶i̶s̶t̶<̶S̶t̶r̶i̶n̶g̶>̶ ̶l̶i̶s̶t̶a̶D̶e̶T̶e̶m̶a̶s̶ ̶=̶ ̶P̶l̶u̶g̶i̶n̶.̶l̶i̶s̶t̶a̶D̶e̶T̶e̶m̶a̶s̶(̶)̶;̶
      List<String> listaDeTemas = Tema.listaDeTemas(); // modificado

      // código omitido...
    }

  }
  ```

  Ajuste os imports:

  ####### cotuba.tema.AplicadorTema

  ```java
  i̶m̶p̶o̶r̶t̶ ̶c̶o̶t̶u̶b̶a̶.̶p̶l̶u̶g̶i̶n̶.̶P̶l̶u̶g̶i̶n̶;̶
  import cotuba.plugin.Tema;
  ```

4. No projeto `tema-paradizo`, faça com que a classe `TemaParadizo` implemente a interface `Tema`. Remova o método `aposGeracao`.

  ####### br.com.paradizo.tema.TemaParadizo

  ```java
  p̶u̶b̶l̶i̶c̶ ̶c̶l̶a̶s̶s̶ ̶T̶e̶m̶a̶P̶a̶r̶a̶d̶i̶z̶o̶ ̶i̶m̶p̶l̶e̶m̶e̶n̶t̶s̶ ̶P̶l̶u̶g̶i̶n̶ ̶{̶
  public class TemaParadizo implements Tema { // modificado

    @Override
    public String cssDoTema() {
      return FileUtils.getResourceContents("/tema.css");
    }

    @̶O̶v̶e̶r̶r̶i̶d̶e̶
    p̶u̶b̶l̶i̶c̶ ̶v̶o̶i̶d̶ ̶a̶p̶o̶s̶G̶e̶r̶a̶c̶a̶o̶(̶E̶b̶o̶o̶k̶ ̶e̶b̶o̶o̶k̶)̶ ̶{̶
    }̶

  }
  ```

  Arrume os imports:

  ```java
  i̶m̶p̶o̶r̶t̶ ̶c̶o̶t̶u̶b̶a̶.̶d̶o̶m̶a̶i̶n̶.̶E̶b̶o̶o̶k̶;̶
  i̶m̶p̶o̶r̶t̶ ̶c̶o̶t̶u̶b̶a̶.̶p̶l̶u̶g̶i̶n̶.̶P̶l̶u̶g̶i̶n̶;̶
  import cotuba.plugin.Tema;
  ```

5. Ainda no projeto `tema-paradizo`, renomeie o arquivo de configuração do provider:

  - de `META-INF/services/cotuba.plugin.Plugin`
  - para `META-INF/services/cotuba.plugin.Tema`
  
  O conteúdo do arquivo deve ser mantido.

## Exercício: uma interface separada para a finalização da geração do ebook

### Objetivo

No projeto `cotuba-cli`, defina uma nova interface no pacote `cotuba.plugin` chamada `AoFinalizarGeracao`. O método `aposGeracao` e o método estático `gerou` devem ser definidos na nova interface. Mude a classe `Cotuba` para usar a nova interface.

No projeto `estatisticas-ebook`, a classe `CalculadoraEstatisticas` deve implementar a nova interface. O arquivo de configuração do provider em `META-INF/services` deve ser renomeado.

### Passo a passo

1. Crie a interface `AoFinalizarGeracao` no pacote `cotuba.plugin` do projeto `cotuba-cli`. O método `aposGeracao` da interface `Plugin` deve ser copiado para a nova interface:

  ####### cotuba.plugin.AoFinalizarGeracao

  ```java
  package cotuba.plugin;

  import cotuba.domain.Ebook;

  public interface AoFinalizarGeracao {

    void aposGeracao(Ebook ebook);
  }
  ```

2. O método estático `gerou` deve ser copiado da interface `Plugin` para a nova interface `AoFinalizarGeracao`. Ao `ServiceLoader`, deve ser passado o tipo `AoFinalizarGeracao`:

  ####### cotuba.plugin.AoFinalizarGeracao

  ```java
  public interface AoFinalizarGeracao {

    void aposGeracao(Ebook ebook);

    static void gerou(Ebook ebook) {

      S̶e̶r̶v̶i̶c̶e̶L̶o̶a̶d̶e̶r̶<̶P̶l̶u̶g̶i̶n̶>̶ ̶l̶o̶a̶d̶e̶r̶ ̶=̶ ̶S̶e̶r̶v̶i̶c̶e̶L̶o̶a̶d̶e̶r̶.̶l̶o̶a̶d̶(̶P̶l̶u̶g̶i̶n̶.̶c̶l̶a̶s̶s̶)̶;̶
      ServiceLoader<AoFinalizarGeracao> loader = ServiceLoader.load(AoFinalizarGeracao.class); // modificado

      f̶o̶r̶ ̶(̶P̶l̶u̶g̶i̶n̶ ̶p̶l̶u̶g̶i̶n̶ ̶:̶ ̶l̶o̶a̶d̶e̶r̶)̶ ̶{̶
      for (AoFinalizarGeracao plugin : loader) {  // modificado
        plugin.aposGeracao(ebook);
      }

    }
  }
  ```

  Verifique os imports:

  ####### cotuba.plugin.AoFinalizarGeracao

  ```java
  import java.util.ServiceLoader;
  import cotuba.domain.Ebook;
  ```

3. Use o método estático `gerou` da interface `AoFinalizarGeracao` na classe `Cotuba`:

  ####### cotuba.application.Cotuba

  ```java
  public class Cotuba {

    public void executa(ParametrosCotuba parametros) {

      // código omitido...

      P̶l̶u̶g̶i̶n̶.̶g̶e̶r̶o̶u̶(̶e̶b̶o̶o̶k̶)̶;̶
      AoFinalizarGeracao.gerou(ebook); // modificado

    }

  }
  ```

  Ajuste os imports:

  ####### cotuba.application.Cotuba

  ```java
  i̶m̶p̶o̶r̶t̶ ̶c̶o̶t̶u̶b̶a̶.̶p̶l̶u̶g̶i̶n̶.̶P̶l̶u̶g̶i̶n̶;̶
  import cotuba.plugin.AoFinalizarGeracao;
  ```

4. No projeto `estatisticas-ebook`, faça com que a classe `CalculadoraEstatisticas` passe a implementar `AoFinalizarGeracao`, removendo o método `cssDoTema`:

  ####### br.com.cognitio.estatisticas.CalculadoraEstatisticas

  ```java
  p̶u̶b̶l̶i̶c̶ ̶c̶l̶a̶s̶s̶ ̶C̶a̶l̶c̶u̶l̶a̶d̶o̶r̶a̶E̶s̶t̶a̶t̶i̶s̶t̶i̶c̶a̶s̶ ̶i̶m̶p̶l̶e̶m̶e̶n̶t̶s̶ ̶P̶l̶u̶g̶i̶n̶ ̶{̶
  public class CalculadoraEstatisticas implements AoFinalizarGeracao { // modificado

    @̶O̶v̶e̶r̶r̶i̶d̶e̶
    p̶u̶b̶l̶i̶c̶ ̶S̶t̶r̶i̶n̶g̶ ̶c̶s̶s̶D̶o̶T̶e̶m̶a̶(̶)̶ ̶{̶
      r̶e̶t̶u̶r̶n̶ ̶n̶u̶l̶l̶;̶
    }̶

    @Override
    public void aposGeracao(Ebook ebook) {
      for (Capitulo capitulo : ebook.getCapitulos()) {
        // código omitido...
      }
    }

  }
  ```

  Certifique-se que os imports estão corretos:

  ```java
  i̶m̶p̶o̶r̶t̶ ̶c̶o̶t̶u̶b̶a̶.̶p̶l̶u̶g̶i̶n̶.̶P̶l̶u̶g̶i̶n̶;̶
  import cotuba.plugin.AoFinalizarGeracao;
  ```

5. Mude o nome do arquivo de configuração do provider, no projeto `estatisticas-ebook`:

  - de `META-INF/services/cotuba.plugin.Plugin`
  - para `META-INF/services/cotuba.plugin.AoFinalizarGeracao`
  
  O conteúdo do arquivo deve ser mantido.

6. Volte ao projeto `cotuba-cli`. Agora, a interface `Plugin`, do pacote `cotuba.plugin`, não está sendo usada mais. Remova-a!

  ```
  c̶o̶t̶u̶b̶a̶.̶p̶l̶u̶g̶i̶n̶.̶P̶l̶u̶g̶i̶n̶
  ```

  Pronto! Ao invés de uma interface em comum, temos interfaces para cada uso específico.

  ![Interfaces segregadas no Cotuba {w=74}](assets/imagens/cap09-interface-segregation-principle/plugin-de-tema-e-ao-finalizar-geracao.png)

7. Teste a geração do PDF e/ou EPUB, fazendo o build dos projetos `cotuba-cli`, `tema-paradizo` e `estatisticas-ebook`.

  O ebook deve ser gerado com os temas e as palavras devem ser exibidas.

## Separando classes, não apenas interfaces

Considere que temos a seguinte classe:

```java
public class NotaFiscal {

  private Cliente cliente;

  private Endereco entrega;
  private Endereco cobranca;

  private List<Item> itens = new ArrayList<>();
  
  private List<Desconto> descontos = new ArrayList<>();
  private FormaDePagamento pagamento;
  private BigDecimal valorTotal;

  // getters...

  // restante do código...
}
```

> Esse exemplo é baseado no livro [OO e SOLID para Ninjas](https://www.casadocodigo.com.br/products/livro-oo-solid) (ANICHE, 2015).

A classe `NotaFiscal` do código anterior possui vários atributos relacionados à emissão de notas fiscais.

Vamos considerar que a complexidade da classe é justificada pela dificuldade do problema que está sendo resolvido.

Uma `NotaFiscal` é passada como parâmetro para o método `calcula` da classe a seguir:

```java
public class CalculadoraDeImposto {

  private static final BigDecimal VALOR_LIMITE = new BigDecimal(1000);
  private static final BigDecimal TAXA_MAIOR = new BigDecimal("0.02");
  private static final BigDecimal TAXA_MENOR = new BigDecimal("0.01");

  public BigDecimal calcula(NotaFiscal nota) { // nota fiscal usada aqui
    BigDecimal total = BigDecimal.ZERO;

    for (Item item : nota.getItens()) { // e também aqui

      if (item.getValor().compareTo(VALOR_LIMITE) > 0) {
        total = total.add(item.getValor().multiply(TAXA_MAIOR));

      } else {
        total = total.add(item.getValor().multiply(TAXA_MENOR));
      }

    }

    return total;
  }
}
```

Perceba que a classe `CalculadoraDeImposto` usa apenas o método `getItens` da `NotaFiscal`.

Para o cálculo de impostos, não precisamos de outras informações da nota fiscal.

Os atributos completos da nota fiscal seriam necessários para uma classe que gera uma representação em PDF, para um DANFe (Documento Auxiliar da NF-e), ou em XML.

Já para uma classe envolvida com relacionamento com o cliente (CRM) seria interessante apenas os dados do cliente, o valor total e os endereços de entrega e cobrança.

Ou seja, dependendo de quem usa a `NotaFiscal`, precisaríamos de acesso a diferentes conjuntos de atributos.

Deixar o acesso a todos os atributos pode ser ruim em termos de design de código.

Poderíamos trabalhar com dados de cliente ou endereços dentro de uma classe responsável por cálculos financeiros, como `CalculadoraDeImposto`.

Ou misturar lógica de CRM com emissão de DANFe.

O que nos impediria, além do bom senso?

### Uma interface específica para quem usa

No caso da `CalculadoraDeImposto`, poderíamos separar apenas aquilo que desejamos da classe `NotaFiscal` por meio de uma interface bem enxuta:

```java
public interface Tributavel {
  List<Item> itensASeremTributados();
}
```

A interface `Tributavel` deveria ser implementada por `NotaFiscal` :

```java
public class NotaFiscal implements Tributavel {

  // código omitido...

  public List<Item> itensASeremTributados() {
    return itens;
  }

}
```

Na classe `CalculadoraDeImposto`, passaríamos a usar a interface `Tributavel`:

```java
public class CalculadoraDeImposto {

  // código omitido...

  public BigDecimal calcula(Tributavel tributavel) { // modificado
    BigDecimal total = BigDecimal.ZERO;

    for (Item item : tributavel.itensASeremTributados()) { // modificado
      // código omitido...
    }

  }

}
```

Não teríamos mais acesso a todas as informações da nota fiscal. Apenas às necessárias.

> _Se você tiver uma classe que tenha vários clientes, em vez de carregar a classe com todos os métodos de que os clientes precisam, crie interfaces específicas para cada cliente e implemente-as na classe._
>
> [Design Principles and Design Patterns](http://www.cvc.uab.es/shared/teach/a21291/temes/object_oriented_design/materials_adicionals/principles_and_patterns.pdf) (MARTIN, 2000)

## Interfaces específicas para o cliente no Cotuba

No método `aposGeracao` do plugin `AoFinalizarGeracao`, passamos o `Ebook` que acabou de ser gerado.

Há um problema: o `Ebook` possui _setters_, que poderiam ser usados pelos service providers para alterar informações do ebook.

Um service provider bem intencionado, como a `CalculadoraEstatisticas` da empresa Cognitio, não faria nenhuma alteração.

Mas uma implementação maliciosa do plugin `AoFinalizarGeracao` poderia remover ou trocar os capítulos, mudar o formato do ebook e/ou o arquivo de saída.

```java
public class ServiceProviderMalicioso implements AoFinalizarGeracao {

  @Override
  public void aposGeracao(Ebook ebook) {

    ebook.setArquivoDeSaida(null);
    ebook.setCapitulos(null);
    ebook.setFormato(FormatoEbook.EPUB);

  }
}
```

Uma solução é criar uma interface `Ebook` específica para os plugins, que seria somente para leitura. Ou seja, teria apenas os _getters_ do formato do ebook, do arquivo de saída e dos capítulos.

Mas o problema ainda persiste: não podemos expor os setters de `Capitulo`. Um service provider malicioso pode explorar um _setter_ para modificar o conteúdo HTML do capítulo:

```java
public class ServiceProviderMalicioso implements AoFinalizarGeracao {

  @Override
  public void aposGeracao(Ebook ebook) {

    for (Capitulo capitulo : ebook.getCapitulos()) {

      capitulo.setTitulo(null);
      capitulo.setConteudoHTML(null);

    }

  }

}
```

A ideia é criar uma interface para o capítulo específica para os plugins, também só pra leitura.

_Em um capítulo posterior veremos uma solução diferente_.



### Atitude limitante vs. permissiva

No post [Software Development Attitude](https://www.martinfowler.com/bliki/SoftwareDevelopmentAttitude.html) (FOWLER, 2004), Martin Fowler argumenta que há duas atitudes quando discutimos sobre linguagens, metodologias, ferramentas e design de código:

  - _Directing attitude_: uma atitude limitante, controladora, direcionadora, restritiva. É uma mentabilidade preocupada em prevenir desenvolvedores ruins de causar danos. Leva a designs robustos, mas que limitam as possibilidades.
  - _Enabling attitude_: uma atitude permissiva, habilitante, tolerante, capacitadora. É uma mentalidade que dá liberdade aos desenvolvedores, sem prevenir que façam coisas ruins. Leva a designs flexíveis, mas que podem ser usados de maneira errada.

## Exercício: protegendo o domain model com interfaces

### Objetivo

Crie interfaces só com getters para `Ebook` e `Capitulo`.

Use as novas interfaces no plugin `AoFinalizarGeracao` e no seu service provider, `CalculadoraEstatisticas`.

### Passo a passo

1. Crie uma interface `Capitulo` no pacote `cotuba.plugin`, baseada na classe de mesmo nome do pacote `cotuba.domain`, mas contendo apenas os getters:

  ####### cotuba.plugin.Capitulo

  ```java
  package cotuba.plugin;

  public interface Capitulo {

    String getTitulo();

    String getConteudoHTML();

  }
  ```

2. A classe `Capitulo` do pacote `cotuba.domain` deve implementar a nova interface:

  ####### cotuba.domain.Capitulo

  ```java
  public class Capitulo implements cotuba.plugin.Capitulo { // modificado

    // código omitido...

  }
  ```

  Os métodos definidos na interface já estão implementados pela classe `Capitulo`.

  Como os nomes são os mesmos, é preciso usar o _fully qualified name_ da interface.

3. Baseando-se em `Ebook` do pacote `cotuba.domain`, crie uma interface de mesmo nome apenas com os getters, no pacote `cotuba.plugin`:

  ####### cotuba.plugin.Ebook

  ```java
  package cotuba.plugin;

  import java.nio.file.Path;
  import java.util.List;

  import cotuba.plugin.Capitulo;
  import cotuba.domain.FormatoEbook;

  public interface Ebook {

    FormatoEbook getFormato();

    Path getArquivoDeSaida();

    List<? extends Capitulo> getCapitulos();

  }
  ```

  O import de `Capitulo` deve apontar para a interface do pacote `cotuba.plugin`.

  Para podermos retorna uma lista com qualquer subtipo da interface `Capitulo`, incluindo a classe do pacote `cotuba.domain`, devemos usar o _bounded wildcard_ `List<? extends Capitulo>`.

4. A nova interface deve ser implementada pela classe `Ebook` do pacote `cotuba.domain`:

  ####### cotuba.domain.Ebook

  ```java
  public class Ebook implements cotuba.plugin.Ebook { // modificado

    // código omitido...

  }
  ```

  Assim como anteriormente, os métodos já estão implementados e devemos usar o _fully qualified name_ da interface.

5. Como as interfaces `AoFinalizarGeracao` e `Ebook` estão no mesmo pacote `cotuba.plugin`, podemos remover o import:

  ```java
  i̶m̶p̶o̶r̶t̶ ̶c̶o̶t̶u̶b̶a̶.̶d̶o̶m̶a̶i̶n̶.̶E̶b̶o̶o̶k̶;̶
  ```

6. No projeto `estatisticas-ebook`, ajuste os imports para que a classe `CalculadoraEstatisticas` use as novas interfaces:

  ####### br.com.cognitio.estatisticas.CalculadoraEstatisticas

  ```java
  i̶m̶p̶o̶r̶t̶ ̶c̶o̶t̶u̶b̶a̶.̶d̶o̶m̶a̶i̶n̶.̶C̶a̶p̶i̶t̶u̶l̶o̶;̶
  i̶m̶p̶o̶r̶t̶ ̶c̶o̶t̶u̶b̶a̶.̶d̶o̶m̶a̶i̶n̶.̶E̶b̶o̶o̶k̶;̶
  import cotuba.plugin.Capitulo;
  import cotuba.plugin.Ebook;
  ```

  Pronto!
  
  Agora não há chance de um plugin modificar o ebook!

7. Teste a geração do PDF e/ou EPUB, fazendo o build dos projetos `cotuba-cli` e `estatisticas-ebook`.

  O ebook deve ser gerado com os temas e as palavras devem ser exibidas.

![Interfaces de domínio segregadas {w=71}](assets/imagens/cap09-interface-segregation-principle/interfaces-de-dominio-segregadas.png)

## Desafio (opcional): generalizando o plugin de tema

### Objetivo

O plugin `Tema`, chamado após a renderização de cada capítulo de MD para HTML, é específico para a aplicação de CSS.

Generalize esse plugin para algo como `AoRenderizarHTML`.

Reorganize o código para que o CSS seja aplicado no novo plugin.


