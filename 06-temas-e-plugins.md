# Temas e Plugins

## Aplicando CSS no ebook

Queremos estilizar cada capítulo, por meio de CSS. Esse estilo deverá ser inserido no HTML que foi renderizado a partir do Markdown.

Para isso, criaremos uma classe responsável por aplicar um tema. Essa classe recebe um capítulo e, usando o Jsoup, insere o estilo CSS no HTML.

### A biblioteca Jsoup

Jsoup é uma biblioteca feita em Java que provê uma API baseada no jQuery para manipular HTML.

Considere o HTML a seguir:

```html
<div class="curso">
  <h2 class="curso__titulo">Curso Design de código SOLID em Java</h2>
  <p class="curso__info"><span>20</span> horas/aula</p>
</div>
```

Podemos usar o Jsoup para extrair texto e até mudar o HTML:

```java
String html = //...

Document doc = Jsoup.parse(html);

Elements info = doc.select(".curso__info"); // <p>

String texto = info.text(); // "20 horas/aula"

doc.select(".curso").append("<a href=\"#turmas\">Turmas</a>"); // adiciona link depois do <p>

String novoHtml = doc.html(); // HTML atualizado
```

## Exercício: uma classe para aplicar temas

### Objetivo

Crie uma classe `AplicadorTema`, no pacote `cotuba.tema`, que recebe um `Capitulo` e insere uma borda tracejada abaixo do título do capítulo.

Use a biblioteca Jsoup.

### Passo a passo

1. No `pom.xml`, declare como dependência a biblioteca Jsoup:

  ####### cotuba-cli/pom.xml
  
  ```xml
  <dependency>
    <groupId>org.jsoup</groupId>
    <artifactId>jsoup</artifactId>
    <version>1.11.2</version>
  </dependency>
  ```

2. Crie a classe `AplicadorTema` em um novo pacote `cotuba.tema` e defina o método `aplica`, que recebe um `Capitulo` como parâmetro.

  ####### cotuba.tema.AplicadorTema

  ```java
  package cotuba.tema;

  import cotuba.domain.Capitulo;

  public class AplicadorTema {

    public void aplica(Capitulo capitulo) {

    }

  }
  ```

3. Implemente o método `aplica`, usando a biblioteca Jsoup e definido o CSS apropriado:

  ####### cotuba.tema.AplicadorTema

  ```java
  String html = capitulo.getConteudoHTML();

  Document document = Jsoup.parse(html);

  String css = "h1 { border-bottom: 1px dashed black; }";

  document.select("head").append("<style> " + css + " </style>");

  capitulo.setConteudoHTML(document.html());
  ```

  Certifique-se que os imports estão corretos:

  ####### cotuba.tema.AplicadorTema

  ```java
  import org.jsoup.Jsoup;
  import org.jsoup.nodes.Document;

  import cotuba.domain.Capitulo;
  ```

## Explorando o design: responsabilidades e dependências

Quem deve chamar o `AplicadorTema`?

Temos duas opções:

- a classe `RenderizadorMDParaHTML` chama o `AplicadorTema` logo após renderizar cada capítulo

![RenderizadorMDParaHTML chama AplicadorTema {w=34}](imagens/cap06-temas-e-plugins/renderizador-chama-aplicador-tema.png)

- a classe `Cotuba` chama o `AplicadorTema`, depois de receber a lista de capítulos de `RenderizadorMDParaHTML`

![Cotuba chama RenderizadorMDParaHTML e AplicadorTema {w=35}](imagens/cap06-temas-e-plugins/cotuba-chama-renderizador-e-aplicador-tema.png)

### Responsabilidades

Se pensarmos em termos de responsabilidades, qual decisão é a mais acertada?

Ao fazermos `RenderizadorMDParaHTML` invocar `AplicadorTema`, estaríamos adicionando mais uma responsabilidade a essa classe: disparar a aplicação do estilo, além de renderizar o Markdown para HTML. Por outro lado, aplicar um CSS está relacionado com a geração do HTML, que é a responsabilidade de `RenderizadorMDParaHTML`.

Já se fizermos `Cotuba` invocar `AplicadorTema`, seria mais uma classe para ser coordenada.

### Dependências

E considerando as dependências? O que seria mais correto?

Precisamos considerar se a classe `AplicadorTema` é de alto ou baixo nível. É algo um tanto subjetivo. Aplicar temas é ou não parte da regra de negócio do nosso gerador de ebooks?

Outra questão é que, ao colocarmos `AplicadorTema` como dependência de `RenderizadorMDParaHTML`, passaríamos a ter dependências com classes do Cotuba, do Java NIO, do CommonMark e, agora, com a nova classe.

Se colocarmos como dependência de `Cotuba`, teríamos dependências com `AplicadorTema`, além de com as classes de domínio `Ebook` e `Capitulo` e
as abstrações `ParametrosCotuba`, `RenderizadorMDParaHTML`, `GeradorPDF` e `GeradorEPUB`. Se considerarmos `AplicadorTema` de baixo nível teríamos que inverter a dependência, criando uma nova abstração.

### Escolhendo um design

Não há uma resposta certa em um design. Podemos caminhar para um lado ou para outro. Se detectarmos que o caminho escolhido não é o melhor, podemos refatorar, melhorando o design.

No nosso caso, vamos fazer com que `RenderizadorMDParaHTML` chame `AplicadorTema`, considerando que a responsabilidade dessa classe é relacionada a um detalhe da geração do HTML e é de baixo nível.

## Exercício: aplicando o tema

### Objetivo

Faça com que `RenderizadorMDParaHTML` chame `AplicadorTema` para cada capítulo, logo depois de renderizar o HTML.

### Passo a passo

1. Na classe `RenderizadorMDParaHTMLComCommonMark`, instancie de `AplicadorTema` e invoque o método `aplica` passando o `Capitulo`, logo depois de setar o HTML:

  ####### cotuba.md.RenderizadorMDParaHTMLComCommonMark

  ```java
  HtmlRenderer renderer = HtmlRenderer.builder().build();
  String html = renderer.render(document);

  capitulo.setConteudoHTML(html);
  
  AplicadorTema tema = new AplicadorTema(); // inserido
  tema.aplica(capitulo); // inserido
  
  capitulos.add(capitulo);
  ```

  Não esqueça de fazer o import:
  
  ####### cotuba.md.RenderizadorMDParaHTMLComCommonMark

  ```java
  import cotuba.tema.AplicadorTema;
  ```

2. Teste a geração do PDF e do EPUB. Veja a borda nos títulos dos capítulos!

## A necessidade de plugins

A empresa Paradizo, nossa cliente, quer definir seu próprio tema.

Para isso, os desenvolvedores da Paradizo querem inserir seu próprio CSS no HTML de cada capítulo.

Uma opção seria pedir que o time da Paradizo nos mandasse um arquivo `.css`. No código do Cotuba, aplicaríamos os estilos deles no HTML dos capítulos.

Mas **temos outros clientes**! Alguns desses clientes também querem seus temas customizados, com seu próprio CSS. Outros não querem nenhum tema.

É inviável incluir, no código do Cotuba, os estilos CSS de todos os clientes, além de uma lista de quais não têm nenhum estilo.

Uma _outra opção_ seria pedir que os clientes fornecessem JARs contendo classes que retornariam o CSS.

Seriam classes que estariam em outros JARs, fora do `cotuba-cli.jar`.

Mas não sabemos quais são esses outros JARs nem o nome das classes e métodos que precisamos chamar. Não sabemos nem se esses JARs existem. Não podemos depender deles!

![Cotuba chamando o Tema Paradizo NÃO é uma boa ideia {w=59}](imagens/cap06-temas-e-plugins/cotuba-chamando-plugin.png)

Precisamos de pontos de extensão, ou _plugins_, para o Cotuba. Se existirem, serão aplicados. Mas como implementá-los?

## Pontos de extensão por meio de interfaces

O Cotuba não pode depender das classes de terceiros, mas essas podem depender do Cotuba. Podemos _inverter as dependências_!

![Tema Paradizo implementa interfaces do Cotuba {w=67}](imagens/cap06-temas-e-plugins/plugin-chamando-cotuba.png)

Mais uma vez, é o código de alto nível, do Cotuba, fornecendo abstrações para um código de baixo nível, que fornece detalhes de implementação.

É o DIP!

Para isso, vamos fornecer essa abstração de um ponto de extensão por meio da interface `Plugin`.

## Exercício: um plugin para o Cotuba

### Objetivo

Defina uma interface `Plugin` no pacote `cotuba.plugin`. Defina um método chamado `cssDoTema`, que retorna uma `String`.

### Passo a passo

1. No pacote `cotuba.plugin`, crie a interface `Plugin`, conforme código a seguir:

  ####### cotuba.plugin.Plugin

  ```java
  package cotuba.plugin;

  public interface Plugin {

    String cssDoTema();

  }
  ```

## Exercício: uma implementação do plugin

### Objetivo

Crie um outro projeto Maven chamado `tema-paradizo`.

Defina um recurso com o CSS do tema: uma borda sólida abaixo dos títulos dos capítulos e das seções e uma borda sólida envolvendo as citações.

Implemente a interface `Plugin` do Cotuba, retornando o conteúdo do CSS.

### Passo a passo

1. No Eclipse, vá em  _File > New > Maven Project_.

  Marque a opção _Create a simple project (skip archetype selection)_.

  Desmarque a opção _Use default Workspace location_.

  Em _Location_, coloque `/home/<usuario-do-curso>/tema-paradizo`.

  Não esqueça de trocar `<usuario-do-curso>` pelo nome de usuário do curso.

  Clique em _Next_.

  Na próxima tela, preencha:

  - _Group Id_: `br.com.paradizo`
  - _Artifact Id_: `tema-paradizo`

  Deixe o campo _Version_ como `0.0.1-SNAPSHOT` e _Packaging_ como `jar`.

  Os demais campos podem ficar vazios.

  Clique em _Finish_.

2. No `pom.xml` do novo projeto, declare a codificação de caracteres e a versão do Java.

  Declare também o Cotuba como dependência.

  ####### tema-paradizo/pom.xml

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

  </dependencies>
  ```

  Para que as configurações tenham efeito, clique com o botão direito no projeto e vá em _Maven > Update project..._. Selecione o projeto `tema-paradizo` e clique em _OK_.

3. Crie um arquivo `tema.css` no diretório `src/main/resources` do projeto `tema-paradizo`, com o seguinte conteúdo:

  ####### tema-paradizo/src/main/resources/tema.css

  ```css
  h1 {
    border-bottom: 1px dashed black;
    font-size: 3em;
  }

  h2 {
    border-left: 1px solid black;
    padding-left: 5px;
    border-bottom: 1px solid black;
  }

  blockquote {
    border: 1px solid black;
    padding: 5px;
  }
  ```

4. Crie uma classe `TemaParadizo` no pacote `br.com.paradizo.tema` que implementa a interface `Plugin` do Cotuba:

  ####### br.com.paradizo.tema.TemaParadizo

  ```java
  package br.com.paradizo.tema;

  import cotuba.plugin.Plugin;

  public class TemaParadizo implements Plugin {

    @Override
    public String cssDoTema() {
      return null;
    }

  }
  ```

5. Para obter o `tema.css` a partir da classe `TemaParadizo`, vamos definir uma classe utilitária `FileUtils`, no mesmo pacote `br.com.paradizo.tema`.

  Essa classe ajuda a obter, de maneira simples, o conteúdo de recursos que estão (ou não) em JARs.

  Você pode encontrar o código abaixo na seguinte URL: http://bit.ly/fj38-file-utils

  ####### br.com.paradizo.tema.FileUtils

  ```java
  package br.com.paradizo.tema;

  public class FileUtils {

    public static String getResourceContents(String resource) {
      try {
        Path resourcePath = getResourceAsPath(resource);
        return getPathContents(resourcePath);
      } catch(URISyntaxException | IOException ex) {
        throw new RuntimeException(ex);
      }
    }

    private static Path getResourceAsPath(String resource) throws URISyntaxException, IOException {
      URI uri = FileUtils.class.getResource(resource).toURI();

      if (isResourceInJar(uri)) {
        return getResourceFromJar(uri);
      } else {
        return Paths.get(uri);
      }
    }

    private static boolean isResourceInJar(URI uri) {
      return uri.getScheme().equals("jar");
    }

    private static Path getResourceFromJar(URI fullURI) throws IOException {
      String[] uriParts = fullURI.toString().split("!");
      URI jarURI = URI.create(uriParts[0]);

      FileSystem fs;

      try {
        fs = FileSystems.newFileSystem(jarURI, Collections.<String, String>emptyMap());
      } catch (FileSystemAlreadyExistsException ex) {
        fs = FileSystems.getFileSystem(jarURI);
      }
      String resourceURI = uriParts[1];
      return fs.getPath(resourceURI);
    }

    private static String getPathContents(Path path) throws IOException {
      return new String(Files.readAllBytes(path));
    }

  }
  ```

  Não esqueça de fazer os imports adequados:

  ####### br.com.paradizo.tema.FileUtils

  ```java
  import java.io.IOException;
  import java.net.URI;
  import java.net.URISyntaxException;
  import java.nio.file.FileSystem;
  import java.nio.file.FileSystemAlreadyExistsException;
  import java.nio.file.FileSystems;
  import java.nio.file.Files;
  import java.nio.file.Path;
  import java.nio.file.Paths;
  import java.util.Collections;
  ```

6. Na classe `TemaParadizo`, use `FileUtils` para obter o conteúdo de `tema.css`:

  ####### br.com.paradizo.tema.TemaParadizo

  ```java
  public class TemaParadizo implements Plugin {

    @Override
    public String cssDoTema() {
      return FileUtils.getResourceContents("/tema.css");
    }

  }
  ```



## Ligando os pontos (de extensão) com a Service Loader API

Temos um ponto de extensão no Cotuba através da interface `Plugin`.

Temos uma implementação desse ponto de extensão através da classe `TemaParadizo`.

Mas como ligar uma coisa com a outra, sem fazer com que o Cotuba dependa do código da Paradizo?

A ideia é que as implementações dos pontos de extensão do Cotuba sejam aplicadas pela simples presença de seus JARs.

### A Service Loader API

Antigamente, na plataforma Java, para ligar um plugin de uma aplicação a uma implementação era necessário:

  - criar uma solução caseira usando a Reflection API
  - usar bibliotecas como [JPF](http://jpf.sourceforge.net/index.html) ou [PF4J](https://github.com/pf4j/pf4j)
  - usar uma especificação robusta, mas complexa, como [OSGi](https://www.osgi.org/)

Porém, a partir do Java SE 6, a própria JRE contém uma solução: a **Service Loader API**.

Na _Service Loader API_, um ponto de extensão é chamado de _service_.

Para provermos um service precisamos de:

- **Service Provider Interface (SPI)**: interfaces ou classes abstratas que definem a assinatura do ponto de extensão. No nosso caso, a interface `Plugin`.
- **Service Provider**: uma implementação da SPI. No nosso caso, a classe `TemaParadizo`.

Para ligar a SPI com seu _service provider_, o JAR do provider precisa definir o _provider configuration file_: um arquivo com o nome da SPI dentro da pasta `META-INF/services`. O conteúdo desse arquivo deve ser o _fully qualified name_ da classe de implementação.

No projeto que define a SPI, carregamos as implementações usando a classe `java.util.ServiceLoader`.

A classe `ServiceLoader` possui o método estático `load` que recebe uma SPI como parâmetro e, depois de vasculhar os diretórios `META-INF/services` dos JARs disponíveis no Classpath, retorna uma instância de `ServiceLoader` que contém todas as implementações.

O `ServiceLoader` é um `Iterable` e, por isso, pode ser percorrido com um _for-each_. Caso não haja nenhum service provider para a SPI, o `ServiceLoader` se comporta como uma lista vazia.

Perceba que uma mesma SPI pode ter vários service providers, o que traz bastante flexibilidade.

Com a Service Loader API, a simples presença de um `.jar` que a implemente a abstração do plugin (ou SPI) fará com que o comportamento da aplicação seja estendido, sem precisarmos modificar nenhuma linha de código.

É o OCP ao extremo!

## Exercício: ligando a SPI com o Service Provider

### Objetivo

No projeto `tema-paradizo`, ligue a SPI `Plugin` com o service provider `TemaParadizo`.

### Passo a passo

1. No diretório `src/main/resources`, crie o subdiretório `META-INF` e, dentro desse, o `services`.

2. Dentro do diretório `src/main/resources/META-INF/services`, crie um arquivo `cotuba.plugin.Plugin` (assim mesmo, com os pontos). Defina como conteúdo desse arquivo, o nome do service provider:

  ####### src/main/resources/META-INF/services/cotuba.plugin.Plugin

  ```
  br.com.paradizo.tema.TemaParadizo
  ```

## Exercício: carregando o service provider

### Objetivo

No projeto `cotuba-cli`, use a classe `ServiceLoader` para carregar o service provider.

Use o tema retornado pelo service provider na classe `AplicadorTema`.

### Passo a passo

1. Na interface `Plugin`, defina um método estático `listaDeTemas` que retorna uma `List<String>`:

  ####### cotuba.plugin.Plugin

  ```java
  public interface Plugin {

    String cssDoTema();

    static List<String> listaDeTemas() { // inserido
      List<String> temas = new ArrayList<>();
      return temas;
    }

  }
  ```

  ####### cotuba.plugin.Plugin

  Adicione os imports:

  ```java
  import java.util.ArrayList;
  import java.util.List;
  ```

2. Ainda no método `listaDeTemas` de `Plugin`, use a classe `ServiceLoader` para obter todos os temas dos service providers:

  ####### cotuba.plugin.Plugin

  ```java
  List<String> temas = new ArrayList<>();

  // inserido
  ServiceLoader<Plugin> loader = ServiceLoader.load(Plugin.class);
  for (Plugin plugin : loader) {
    String css = plugin.cssDoTema();
    temas.add(css);
  }
  ```

  Não esqueça do import:

  ####### cotuba.plugin.Plugin

  ```java
  import java.util.ServiceLoader;
  ```

3. No método `aplica` da classe `AplicadorTema`, chame o método `listaDeTemas` de `Plugin` e aplique os CSS retornados no HTML:

  ####### cotuba.tema.AplicadorTema

  ```java
  public class AplicadorTema {

    public void aplica(Capitulo capitulo) {
      // código omitido...

      S̶t̶r̶i̶n̶g̶ ̶c̶s̶s̶ ̶=̶ ̶"̶h̶1̶ ̶{̶ ̶b̶o̶r̶d̶e̶r̶-̶b̶o̶t̶t̶o̶m̶:̶ ̶1̶p̶x̶ ̶d̶a̶s̶h̶e̶d̶ ̶b̶l̶a̶c̶k̶;̶ ̶}̶"̶;̶

      // inserido
      List<String> listaDeTemas = Plugin.listaDeTemas();
      for (String css : listaDeTemas) {
        document.select("head").append("<style> " + css + " </style>");
      }

      // código omitido...
    }
  }
  ```

  Não deixe de fazer o import:

  ####### cotuba.tema.AplicadorTema

  ```java
  import cotuba.plugin.Plugin;
  ```

4. (opcional) No método `listaDeTemas` da classe `Plugin`, use recursos do Java 8 como Lambdas e Streams para trabalhar com o `ServiceLoader`.

5. (desafio) O método estático `load`, da classe `ServiceLoader`, vasculha o Classpath em busca de implementações da SPI, o que é um processo lento. Otimize o código armazenando a instância de `ServiceLoader` retornada.







## Exercício: testando o plugin de tema

### Objetivo

Gere os JARs dos projetos `cotuba-cli` e `tema-paradizo`.

Teste a geração de um ebook e veja se o tema foi aplicado.

### Passo a passo

1. Abra um Terminal e entre na pasta do projeto `cotuba-cli`:

  ```sh
  cd ~/cotuba
  ```

2. Faça o build usando o Maven:

  ```sh
  mvn install
  ```

3. Descompacte o `.zip` gerado para o seu Desktop com o comando:

  ```sh
  unzip -o target/cotuba-*-distribution.zip -d ~/Desktop
  ```

4. Vá até o Desktop:

  ```sh
  cd ~/Desktop
  ```

5. Faça o teste de geração do PDF:

  ```sh
  ./cotuba.sh -d ~/cotuba/exemplo -f pdf
  ```

  Como ainda não colocamos o JAR do plugin no Classpath, não deve haver nenhum estilo.

6. Vá até a pasta do projeto `tema-paradizo`:

  ```sh
  cd ~/tema-paradizo
  ```

7. Faça o build do `tema-paradizo` usando o Maven:

  ```sh
  mvn install
  ```

8. Copie o JAR do plugin `tema-paradizo` para a pasta `libs` do Cotuba, que está no Desktop:

  ```sh
  cp target/tema-paradizo-*.jar ~/Desktop/libs/
  ```

9. Volte ao Desktop:

  ```sh
  cd ~/Desktop
  ```

10. Gere o PDF novamente:

  ```sh
  ./cotuba.sh -d ~/cotuba/exemplo -f pdf
  ```

  Veja que o tema foi aplicado!

  Teste também a geração do EPUB.

## Para saber mais: Alguns usos da Service Loader API na plataforma Java

O mecanismo de Service Provider já existia internamente desde a [JDK 1.3](http://lampwww.epfl.ch/java/jdk1.3/docs/guide/jar/jar.html#Service%20Provider).

A partir do Java SE 6, a Service Loader API ficou pública e disponível para aplicações. A API passou a ser usada em diferentes especificações da plataforma Java.

### PersistenceProvider do JPA

É possível usar o JPA em uma aplicação Java SE, fora de um servidor de aplicação Java EE.

Para isso, precisamos de uma implementação da interface `EntityManager`. Mas como instanciar essa abstração?

```java
EntityManager manager = // ???
```

A especificação JPA determina que devemos usar a interface `EntityManagerFactory`.

Porém, o problema continua: como instanciar a Factory?

```java
EntityManagerFactory factory = // ???
EntityManager manager = factory.createEntityManager();
```

Pela especificação, para criar uma `EntityManagerFactory`, devemos usar um método estático da classe `Persistence`, passando o nome de uma _Persistence Unit_:

```java
EntityManagerFactory factory = Persistence.createEntityManagerFactory("financas");
EntityManager manager = factory.createEntityManager();
```

Todas essas classes e interfaces são do pacote `javax.persistence`.

Como ligá-las com uma implementação do JPA, como o Hibernate ou o Eclipse Link?

Por meio da SPI [`javax.persistence.spi.PersistenceProvider`](https://docs.oracle.com/javaee/5/api/javax/persistence/spi/PersistenceProvider.html).

O [Hibernate](https://github.com/hibernate/hibernate-orm/blob/master/hibernate-core/src/main/resources/META-INF/services/javax.persistence.spi.PersistenceProvider) tem dentro do `hibernate-core.jar` o arquivo:

####### META-INF/services/javax.persistence.spi.PersistenceProvider

```
org.hibernate.jpa.HibernatePersistenceProvider
```

Já o `eclipselink.jar` terá, no [mesmo arquivo](http://git.eclipse.org/c/eclipselink/eclipselink.runtime.git/tree/jpa/org.eclipse.persistence.jpa/resource/META-INF/services/javax.persistence.spi.PersistenceProvider):

####### META-INF/services/javax.persistence.spi.PersistenceProvider

```
org.eclipse.persistence.jpa.PersistenceProvider
```

É interessante notar que a especificação JPA 1.0, parte da EJB 3.0 e Java EE 5, foi lançada para uso com o J2SE 5.0. Portanto, a Service Loader API ainda não era pública. O mecanismo de disponibilização das implementações era responsabilidade do JPA Provider.

### Drivers JDBC

Antes do Java SE 6, era necessário carregar programaticamente a implementação da interface `java.sql.Driver` de um driver JDBC.

Para isso, antes de obter uma conexão, usávamos um código parecido com o seguinte:

```java
Class.forName("com.mysql.jdbc.Driver");
```

Esse código aparentemente inútil tinha, como efeito colateral, o carregamento das implementações de `Driver` que seriam usadas posteriormente pela classe `DriverManager`.

Do Java SE 6 em diante, a classe [`DriverManager`](https://docs.oracle.com/javase/6/docs/api/java/sql/DriverManager.html) usa a Service Loader API para carregar automaticamente todos os drivers na inicialização. A chamada anterior ao método `forName` de `Class` passou a ser desnecessária.

A interface `java.sql.Driver` passou a ser uma SPI.

No caso do MySQL, o arquivo  `mysql-connector-java.jar` passou a ter o arquivo:

####### META-INF/services/java.sql.Driver

```
com.mysql.jdbc.Driver
```

Um fato interessante é que o [Tomcat 7+](http://tomcat.apache.org/tomcat-7.0-doc/jndi-datasource-examples-howto.html#DriverManager,_the_service_provider_mechanism_and_memory_leaks) desliga o carregamento automático de drivers via Service Loader API para evitar vazamento de memória.

### Configuração programática da Servlet 3.0

A partir da especificação Servlet 3.0, parte do Java EE 6, é possível configurar Servlets e Filters sem ter que digitar várias linhas no `web.xml`.

Podemos usar usar as anotações `@WebServlet` e `@WebFilter` em cima de nossas classes.

Mas e para Servlets e Filters de frameworks e bibliotecas, cujo código não conseguimos modificar? Estamos fadados ao `web.xml`?

Há a SPI [`javax.servlet.ServletContainerInitializer`](https://docs.oracle.com/javaee/6/api/javax/servlet/ServletContainerInitializer.html
), que define o método `onStartup`.

Um Servlet Container compatível com a Servlet 3.0, como o Tomcat 7+ ou Jetty 8+, usa a Service Loader API para carregar e executar as implementações de `ServletContainerInitializer` na inicialização do servidor.

O [Spring](https://github.com/spring-projects/spring-framework/blob/master/spring-web/src/main/resources/META-INF/services/javax.servlet.ServletContainerInitializer), por exemplo, tem em seu `spring-web.jar` o arquivo:

####### META-INF/services/javax.servlet.ServletContainerInitializer 

```
org.springframework.web.SpringServletContainerInitializer
```

Esse mecanismo faz com que o Spring dispare chamadas a classes como `AbstractAnnotationConfigDispatcherServletInitializer` sem a necessidade de configuração XML.
