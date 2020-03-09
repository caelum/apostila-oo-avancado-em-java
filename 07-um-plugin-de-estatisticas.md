# Um plugin de estatísticas

## Um plugin menos específico

O plugin que criamos serve somente para retornar temas que são aplicados no HTML de cada capítulo.

São comuns plugins mais alinhados ao ciclo de vida da aplicação, também chamados de _hooks_ (ganchos, em inglês).

No caso do Cotuba, seria interessante ter um _plugin_ para ser chamado logo depois do ebook ser gerado.

## Exercício: mais um plugin no Cotuba

### Objetivo

Adicione um método `aposGeracao` ao plugin do Cotuba, que recebe o ebook que acabou de ser gerado.

Acrescente também um método estático `gerou` que, usando a Service Loader API, invoca todos os plugins.

Na classe `Cotuba`, invoque o novo método estático, logo depois da geração do ebook.

### Passo a passo

1. Defina um método chamado `aposGeracao`, que recebe um `Ebook` como parâmetro, na interface `Plugin`:

  ####### cotuba.plugin.Plugin

  ```java
  // outros imports...

  public interface Plugin {

    String cssDoTema();

    static List<String> listaDeTemas() {
      // código omitido...
    }

    void aposGeracao(Ebook ebook); // inserido

  }
  ```

  Defina o novo import:

  ####### cotuba.plugin.Plugin

  ```java
  import cotuba.domain.Ebook;
  ```

2. Inclua, em `Plugin`, um método estático `gerou`, que recebe um `Ebook`. Use um `ServiceLoader` para obter todos os service providers, invocando o método `aposGeracao`:

  ####### cotuba.plugin.Plugin

  ```java
  public interface Plugin {

    // código omitido...

    static void gerou(Ebook ebook) {

      ServiceLoader<Plugin> loader = ServiceLoader.load(Plugin.class);

      for (Plugin plugin : loader) {
        plugin.aposGeracao(ebook);
      }

    }

  }
  ```

3. Na classe `Cotuba`, invoque o método `gerou` de `Plugin`, logo após a geração do ebook:

  ####### cotuba.application.Cotuba

  ```java
  // outros imports...

  public class Cotuba {

    public void executa(ParametrosCotuba parametros) {

      // código omitido...

      GeradorEbook gerador = GeradorEbook.cria(formato);
      gerador.gera(ebook);

      Plugin.gerou(ebook); // inserido

    }

  }
  ```

  Insira o import a seguir:

  ####### cotuba.application.Cotuba

  ```java
  import cotuba.plugin.Plugin;
  ```

4. (opcional) Use recursos do Java 8 como Lambdas e Streams para trabalhar com o `ServiceLoader`.

5. (desafio) Otimize a chamada ao método estático `load`, da classe `ServiceLoader`, armazenando a instância de `ServiceLoader` retornada.







## Um problema no plugin de tema

No momento em que colocamos mais um método na interface `Plugin` do Cotuba, ocorreu um erro de compilação.

A classe `TemaParadizo` do projeto `tema-paradizo`, um service provider, precisa implementar todos os métodos declarados na SPI `Plugin`.

Por isso, precisamos declarar uma implementação para o novo método `aposGeracao` de `Plugin`.

O que fazer na classe de tema, ao final da geração do ebook?

A resposta é fácil: nada!

## Exercício: corrigindo o problema do tema

### Objetivo

Faça com que a classe `TemaParadizo` não apresente erros de compilação.

### Passo a passo

1. No projeto `tema-paradizo`, faça com que a classe `TemaParadizo` implemente o novo método `aposGeracao` de `Plugin`.

  Dica: digite _CTRL + 1_ em cima do nome da classe e escolha a opção _Add unimplemented methods_.

  O resultado deve ser o seguinte:

  ####### br.com.paradizo.tema.TemaParadizo

  ```java
  public class TemaParadizo implements Plugin {

    @Override
    public String cssDoTema() {
      return FileUtils.getResourceContents("/tema.css");
    }

    @Override
    public void aposGeracao(Ebook ebook) { // inserido

    }

  }
  ```

  Acrescente o import:

  ####### br.com.paradizo.tema.TemaParadizo

  ```java
  import cotuba.domain.Ebook;
  ```

  Se testarmos a geração de ebooks, nada deve ser alterado.

## Um plugin para calcular estatísticas do ebook

Quais as palavras mais comuns no ebook?

A empresa Cognitio quer saber!

Para implementar a contagem de palavras, usarão o novo plugin, que é chamado após a geração do livro.

Para descobrir quais são as palavras do ebook, os desenvolvedores da Cognitio precisam:

  - percorrer a lista de capítulos
  - obter o HTML de cada capítulo
  - remover as tags HTML, deixando só o texto

Por enquanto, apenas listarão as palavras. A contagem em si ficará para depois.

## Exercício: iniciando a implementação do plugin de estatísticas

### Objetivo

Crie um outro projeto Maven chamado `estatisticas-ebook`.

Adicione como dependência o projeto `cotuba-cli`.

Insira também uma dependência à biblioteca Jsoup, que será usada para remover as tags HTML.

Crie uma classe `CalculadoraEstatisticas`, que servirá como service provider para a SPI `Plugin`.

No método `aposGeracao`, imprima no Console todas as palavras de todos os capítulos.

### Passo a passo

1. No Eclipse, vá em  _File > New > Maven Project_.

  Marque a opção _Create a simple project (skip archetype selection)_.

  Desmarque a opção _Use default Workspace location_.

  Em _Location_, coloque `/home/<usuario-do-curso>/estatisticas-ebook`.

  Não esqueça de trocar `<usuario-do-curso>` pelo nome de usuário do curso.

  Clique em _Next_.

  Na próxima tela, preencha:

  - _Group Id_: `br.com.cognitio`
  - _Artifact Id_: `estatisticas-ebook`

  Deixe o campo _Version_ como `0.0.1-SNAPSHOT` e _Packaging_ como `jar`.

  Os demais campos podem ficar vazios.

  Clique em _Finish_.

2. No `pom.xml` do projeto `estatisticas-ebook`, corrija a codificação de caracteres e a versão do Java.

  Como dependências, declare o projeto Cotuba e a biblioteca Jsoup.

  ####### estatisticas-ebook/pom.xml

  ```xml
  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
  </properties>

  <dependencies>

    <dependency>
      <groupId>cotuba</groupId>
      <artifactId>cotuba-cli</artifactId>
      <version>0.0.1-SNAPSHOT</version>
    </dependency>

    <dependency>
      <groupId>org.jsoup</groupId>
      <artifactId>jsoup</artifactId>
      <version>1.11.2</version>
    </dependency>

  </dependencies>
  ```

  Clique com o botão direito no projeto `estatisticas-ebook` e vá em _Maven > Update project..._.
  
  Selecione o projeto `estatisticas-ebook` e clique em _OK_.

  Assim, as mudanças no `pom.xml` terão efeito.

3. Crie, no pacote `br.com.cognitio.estatisticas`, a classe `CalculadoraEstatisticas`.
  Implemente a interface `Plugin` de Cotuba, definindo os métodos necessários.

  ####### br.com.cognitio.estatisticas.CalculadoraEstatisticas

  ```java
  package br.com.cognitio.estatisticas;

  import cotuba.domain.Ebook;
  import cotuba.plugin.Plugin;

  public class CalculadoraEstatisticas implements Plugin {

    @Override
    public String cssDoTema() {
      return null;
    }

    @Override
    public void aposGeracao(Ebook ebook) {

    }

  }
  ```

4. Implemente o método `aposGeracao`, percorrendo a lista de `Capitulo` do `Ebook`, obtendo o HTML de cada capítulo e extraindo as palavras, com a ajuda da biblioteca Jsoup.

  Dica: o método `split` de `String` quebra um texto de acordo com uma expressão regular. A expressão regular `\s+` pega vários espaços adjacentes.

  ####### br.com.cognitio.estatisticas.CalculadoraEstatisticas

  ```java
  for (Capitulo capitulo : ebook.getCapitulos()) {

    String html = capitulo.getConteudoHTML();

    Document doc = Jsoup.parse(html);

    String textoDoCapitulo = doc.body().text();

    String[] palavras = textoDoCapitulo.split("\\s+");

    for (String palavra : palavras) {
      System.out.println(palavra);
    }

  }
  ```

  Realize os imports necessários:

  ####### br.com.cognitio.estatisticas.CalculadoraEstatisticas

  ```java
  import org.jsoup.Jsoup;
  import org.jsoup.nodes.Document;

  import cotuba.domain.Capitulo;
  ```

5. Crie os subdiretórios `META-INF` e `services` no diretório `src/main/resources`.

  Dentro, adicione um arquivo `cotuba.plugin.Plugin` e defina o nome do service provider:

  ####### src/main/resources/META-INF/services/cotuba.plugin.Plugin

  ```
  br.com.cognitio.estatisticas.CalculadoraEstatisticas
  ```



## Exercício: testando o plugin de estatísticas

### Objetivo

Gere os JARs dos projetos `cotuba-cli`, `tema-paradizo` e `estatisticas-ebook`.

Teste a geração de um ebook e veja se as palavras são impressas no console.

### Passo a passo

1. Abra um Terminal e entre na pasta do projeto `cotuba-cli`:

  ```sh
  cd ~/cotuba
  ```

2. Faça o build do `cotuba-cli` usando o Maven:

  ```sh
  mvn install
  ```

3. Descompacte o `.zip` gerado para o seu Desktop com o comando:

  ```sh
  unzip -o target/cotuba-*-distribution.zip -d ~/Desktop
  ```

4. Vá até a pasta do projeto `tema-paradizo`:

  ```sh
  cd ~/tema-paradizo
  ```

5. Faça o build do `tema-paradizo` usando o Maven:

  ```sh
  mvn install
  ```

6. Copie o JAR do plugin `tema-paradizo` para a pasta `libs` do Cotuba, que está no Desktop:

  ```sh
  cp target/tema-paradizo-*.jar ~/Desktop/libs/
  ```

7. Vá até a pasta do projeto `estatisticas-ebook`:

  ```sh
  cd ~/estatisticas-ebook
  ```

8. Faça o build do `estatisticas-ebook` usando o Maven:

  ```sh
  mvn install
  ```

9. Copie o JAR do plugin `estatisticas-ebook` para a pasta `libs` do Cotuba, que está no Desktop:

  ```sh
  cp target/estatisticas-ebook-*.jar ~/Desktop/libs/
  ```

10. Vá até o Desktop:

  ```sh
  cd ~/Desktop
  ```

11. Gere o PDF:

  ```sh
  ./cotuba.sh -d ~/cotuba/exemplo -f pdf
  ```

  Verifique se cada palavra do ebook foi impressa.
