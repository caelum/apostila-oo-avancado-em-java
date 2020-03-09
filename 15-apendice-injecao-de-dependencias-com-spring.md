# Apêndice: Injeção de Dependências com Spring

## Exercício: gerenciando o Cotuba CLI com Spring

1. Adicione o `spring-context` como dependência no `pom.xml` do supermódulo `cotuba`. O Spring Context deve ficar disponível para todos os submódulos.

  ####### cotuba/pom.xml

  ```xml
  <dependencies>

    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>5.0.9.RELEASE</version>
    </dependency>

  </dependencies>
  ```

2. No arquivo `module-info.java` do módulo `cotuba-cli`, adicione o Spring Context como dependência.

  ####### cotuba-cli/src/main/java/module-info.java

  ```java
  module cotuba.cli {
    requires commons.cli;
    requires cotuba.core;

    requires spring.context; // inserido
  }
  ```

3. Crie a classe `SpringConfig` no pacote `cotuba.cli` do módulo `cotuba-cli`. Essa classe deverá ser anotada com `@Configuration`, tornando-a o ponto de entrada para as configurações do Spring. Use também a anotação `@ComponentScan` para buscar as classes gerenciadas pelo Spring, tendo `cotuba` como pacote base.

  ####### cotuba.cli.SpringConfig

  ```java
  package cotuba.cli;

  import org.springframework.context.annotation.ComponentScan;
  import org.springframework.context.annotation.Configuration;

  @Configuration
  @ComponentScan("cotuba")
  public class SpringConfig {

  }
  ```

4. Observe que ocorrerá um erro de compilação cuja mensagem é semelhante à que vem a seguir:

  ```txt
  The type org.springframework.beans.factory.support.BeanNameGenerator cannot be resolved.
  It is indirectly referenced from required .class files
  ```

  Para corrigir esse erro de compilação, precisamos adicionar uma dependência ao módulo `spring.beans` no `module-info.java` do `cotuba-cli`. O Spring Beans é uma dependência do Spring Context, cujo JAR já foi obtido pelo Maven.

  ####### cotuba-cli/src/main/java/module-info.java

  ```java
  module cotuba.cli {
    requires commons.cli;
    requires cotuba.core;

    requires spring.context;
    requires spring.beans; // inserido
  }
  ```

  Feito isso, o código deverá ser compilado com sucesso.

5. Renomeie a classe `Main` do pacote `cotuba.cli` para `CotubaCLI`.

  Anote a nova classe com `@Component`.

  Renomeie o método `main` para `executa`, fazendo com que não seja mais `static`. O método ainda deve recebe o parâmetro `args` do tipo `String[]`.

  Ao invés de instanciar a classe `Cotuba`, a receba no construtor e armazene a instância em um atributo `final`.

  ####### cotuba.cli.CotubaCLI

  ```java
  @Component // inserido
  p̶u̶b̶l̶i̶c̶ ̶c̶l̶a̶s̶s̶ ̶M̶a̶i̶n̶ ̶{̶
  public class CotubaCLI {

    private final Cotuba cotuba; // inserido

    public CotubaCLI(Cotuba cotuba) { // inserido
      this.cotuba = cotuba;
    }

    p̶u̶b̶l̶i̶c̶ ̶s̶t̶a̶t̶i̶c̶ ̶v̶o̶i̶d̶ ̶m̶a̶i̶n̶(̶S̶t̶r̶i̶n̶g̶[̶]̶ ̶a̶r̶g̶s̶)̶ ̶{̶
    public void executa (String[] args) {

        // código omitido...

        C̶o̶t̶u̶b̶a̶ ̶c̶o̶t̶u̶b̶a̶ ̶=̶ ̶n̶e̶w̶ ̶C̶o̶t̶u̶b̶a̶(̶)̶;̶
        cotuba.executa(opcoesCLI, mdsDoDiretorio, System.out::println);

        System.out.println("Arquivo gerado com sucesso: " + arquivoDeSaida);

        // código omitido...
    }

  }
  ```

  Não deixe de adicionar o import correto:

  ####### cotuba.cli.CotubaCLI

  ```java
  import org.springframework.stereotype.Component;
  ```

6. Defina uma nova classe `Main` no pacote `cotuba.cli`, que define o método estático `main`.

  Use um `AnnotationConfigApplicationContext`, passando a classe de configurações do Spring, a `SpringConfig`.

  A partir do `ApplicationContext`, obtenha uma instância de `CotubaCLI`, que conterá todas as dependências já injetadas. Invoque o método `executa` desse instância, passando os argumentos da linha de comando.

  ```java
  package cotuba.cli;

  import org.springframework.context.ApplicationContext;
  import org.springframework.context.annotation.AnnotationConfigApplicationContext;

  public class Main {

    public static void main(String[] args) throws Exception{

      ApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);

      CotubaCLI cli = context.getBean(CotubaCLI.class);
      cli.executa(args);

    }

  }
  ```

7. Deve ocorrer um erro de compilação na classe `Main`, com a seguinte mensagem:

  ```txt
  The type org.springframework.core.AliasRegistry cannot be resolved.
  It is indirectly referenced from required .class files
  ```

  Essa classe é definida pelo Spring Core, já obtido pelo Maven, e é uma dependência do Spring Beans.
  
  Vamos declarar o módulo `spring.core` como dependência no `module-info.java` do módulo `cotuba.cli`:

  ####### cotuba-cli/src/main/java/module-info.java

  ```java
  module cotuba.cli {
    requires commons.cli;
    requires cotuba.core;

    requires spring.context;
    requires spring.beans;
    requires spring.core; // inserido
  }
  ```

  Feito isso, o código deve ser compilado com sucesso.

8. Se tentarmos executar a classe `Main`, obteremos uma exceção parecida com a seguinte:

  ```txt
  Exception in thread "main" java.lang.NoClassDefFoundError: java/sql/SQLException
    at spring.context@5.0.9.RELEASE/org.springframework.context.support.AbstractApplicationContext.resetCommonCaches(AbstractApplicationContext.java:914)
    ...

  Caused by: java.lang.ClassNotFoundException: java.sql.SQLException

    at java.base/jdk.internal.loader.BuiltinClassLoader.loadClass(BuiltinClassLoader.java:582)
    at java.base/jdk.internal.loader.ClassLoaders$AppClassLoader.loadClass(ClassLoaders.java:190)
    at java.base/java.lang.ClassLoader.loadClass(ClassLoader.java:499)
  ... 4 more
  ```

  O módulo Spring Beans depende do Java SQL, que não é adicionado por padrão no Modulepath.

  Para adicioná-lo, devemos declarar a dependência no arquivo `module-info.java`:

  ####### cotuba-cli/src/main/java/module-info.java

  ```java
  module cotuba.cli {
    requires commons.cli;
    requires cotuba.core;

    requires spring.context;
    requires spring.beans;
    requires spring.core;

    requires java.sql; // inserido
  }
  ```

## Exercício: abrindo o módulo Cotuba CLI para Reflection

1. Ao tentarmos executar novamente a classe `Main`, teremos uma outra exceção:

  ```txt
  Exception in thread "main" java.lang.IllegalStateException: Cannot load configuration class: cotuba.cli.SpringConfig
    ...
  Caused by: org.springframework.cglib.core.CodeGenerationException:
    ...

  Caused by: java.lang.IllegalAccessException:
      class org.springframework.cglib.proxy.Enhancer (in module spring.core)
      cannot access class cotuba.cli.SpringConfig$$EnhancerBySpringCGLIB$$410590db
      (in module cotuba.cli) because module cotuba.cli
      does not export cotuba.cli to module spring.core

    at java.base/jdk.internal.reflect.Reflection.newIllegalAccessException(Reflection.java:360)
    at java.base/java.lang.reflect.AccessibleObject.checkAccess(AccessibleObject.java:589)
    at java.base/java.lang.reflect.Field.checkAccess(Field.java:1075)
    at java.base/java.lang.reflect.Field.set(Field.java:778)
    at spring.core@5.0.9.RELEASE/org.springframework.cglib.proxy.Enhancer.wrapCachedClass(Enhancer.java:715)
    ... 20 more
  ```

  O Spring utiliza a API de Reflection para gerenciar objetos. Porém, o JPMS impede por padrão o acesso via reflexão.

  Para isso, precisamos abrir para reflexão o módulo todo com `open` ou cada pacote com `opens`.

  Vamos alterar o `module-info.java` do módulo `cotuba.cli` para que o pacote `cotuba.cli` seja aberto:

  ####### cotuba-cli/src/main/java/module-info.java

  ```java
  module cotuba.cli {
    requires commons.cli;
    requires cotuba.core;

    requires spring.context;
    requires spring.beans;
    requires spring.core;

    requires java.sql;

    opens cotuba.cli; // inserido
  }
  ```

2. Ao tentar executar `Main` mais uma vez, teríamos uma exceção diferente:

  ```txt
  Exception in thread "main" org.springframework.beans.factory.UnsatisfiedDependencyException:
    Error creating bean with name 'cotubaCLI' defined in file [/home/<...>/modulos/cotuba/cotuba-cli/target/classes/cotuba/cli/CotubaCLI.class]:
    Unsatisfied dependency expressed through constructor parameter 0;
    nested exception is ...

  ...

  Caused by: org.springframework.beans.factory.NoSuchBeanDefinitionException:
    No qualifying bean of type 'cotuba.application.Cotuba' available:
      expected at least 1 bean which qualifies as autowire candidate.
    Dependency annotations: {}

  ... 14 more
  ```

  A exceção indica que o Spring não conseguiu encontrar uma definição para a classe `Cotuba`.

  Precisamos configurá-la para que o Spring a gerencie.

## Exercício: gerenciando o Cotuba Core com Spring

1. Adicione, no arquivo `module-info.java` do módulo `cotuba.core`, dependências aos módulos do Spring:

  ####### cotuba-core/src/main/java/module-info.java

  ```java
  module cotuba.core {
    exports cotuba.application;
    exports cotuba.plugin;
    exports cotuba.domain;

    requires jsoup;
    requires org.commonmark;

    requires spring.context; // inserido
    requires spring.beans; // inserido
    requires spring.core; // inserido

    uses cotuba.plugin.Tema;
    uses cotuba.plugin.AoFinalizarGeracao;
    uses cotuba.plugin.GeradorEbook;

  }
  ```

2. Crie uma classe `TemaConfig`, no pacote `cotuba.plugin`, que será responsável por fornecer todas as implementações do plugin `Tema`.

  Anote a nova classe com `@Configuration`.
  
  Use o `ServiceListFactoryBean` do Spring para carregar os Service Providers da determinada SPI. Anote com `@Bean` os métodos que produzem instâncias que serão injetadas em outros objetos.

  ####### cotuba.plugin.TemaConfig

  ```java
  @Configuration
  public class TemaConfig {

    @Bean("temas")
    public ServiceListFactoryBean temasFactory() {
      ServiceListFactoryBean serviceListFactoryBean = new ServiceListFactoryBean();
      serviceListFactoryBean.setServiceType(Tema.class);
      return serviceListFactoryBean;
    }

    @Bean
    public List<Tema> listaDeTemas(@Qualifier("temas") ServiceListFactoryBean temasFactory) {
      try {
        return (List<Tema>) temasFactory.getObject();
      } catch (Exception e) {
        throw new RuntimeException(e);
      }
    }

  }  
  ```

  A lista de service providers da SPI `Tema` poderá ser injetada pelo Spring em outros objetos.

  Lembre-se de fazer os imports:

  ####### cotuba.plugin.TemaConfig

  ```java
  import java.util.List;

  import org.springframework.beans.factory.annotation.Qualifier;
  import org.springframework.beans.factory.serviceloader.ServiceListFactoryBean;
  import org.springframework.context.annotation.Bean;
  import org.springframework.context.annotation.Configuration;
  ```

3. Remova da interface `Tema` o método estático que retorna a lista de temas carregados a partir do Service Loader.

  Deixe apenas a definição do método `cssDoTema`.

  ####### cotuba.plugin.Tema

  ```java
  i̶m̶p̶o̶r̶t̶ ̶j̶a̶v̶a̶.̶u̶t̶i̶l̶.̶A̶r̶r̶a̶y̶L̶i̶s̶t̶;̶
  i̶m̶p̶o̶r̶t̶ ̶j̶a̶v̶a̶.̶u̶t̶i̶l̶.̶L̶i̶s̶t̶;̶
  i̶m̶p̶o̶r̶t̶ ̶j̶a̶v̶a̶.̶u̶t̶i̶l̶.̶S̶e̶r̶v̶i̶c̶e̶L̶o̶a̶d̶e̶r̶;̶

  public interface Tema {

    String cssDoTema();

    s̶t̶a̶t̶i̶c̶ ̶L̶i̶s̶t̶<̶S̶t̶r̶i̶n̶g̶>̶ ̶l̶i̶s̶t̶a̶D̶e̶T̶e̶m̶a̶s̶(̶)̶ ̶{̶
      L̶i̶s̶t̶<̶S̶t̶r̶i̶n̶g̶>̶ ̶t̶e̶m̶a̶s̶ ̶=̶ ̶n̶e̶w̶ ̶A̶r̶r̶a̶y̶L̶i̶s̶t̶<̶>̶(̶)̶;̶
      S̶e̶r̶v̶i̶c̶e̶L̶o̶a̶d̶e̶r̶<̶T̶e̶m̶a̶>̶ ̶l̶o̶a̶d̶e̶r̶ ̶=̶ ̶S̶e̶r̶v̶i̶c̶e̶L̶o̶a̶d̶e̶r̶.̶l̶o̶a̶d̶(̶T̶e̶m̶a̶.̶c̶l̶a̶s̶s̶)̶;̶
      f̶o̶r̶ ̶(̶T̶e̶m̶a̶ ̶p̶l̶u̶g̶i̶n̶ ̶:̶ ̶l̶o̶a̶d̶e̶r̶)̶ ̶{̶
        S̶t̶r̶i̶n̶g̶ ̶c̶s̶s̶ ̶=̶ ̶p̶l̶u̶g̶i̶n̶.̶c̶s̶s̶D̶o̶T̶e̶m̶a̶(̶)̶;̶
        t̶e̶m̶a̶s̶.̶a̶d̶d̶(̶c̶s̶s̶)̶;̶
      }̶
      r̶e̶t̶u̶r̶n̶ ̶t̶e̶m̶a̶s̶;̶
    }̶

  }
  ```

4. Anote a classe `AplicadorTema` com `@Component`, para que ela seja gerenciada pelo Spring.

  Defina um atributo que armazena uma lista de `Tema` e a receba no construtor.

  No método `aplica`, percorra a lista de temas, obtendo cada CSS.

  ####### cotuba.tema.AplicadorTema

  ```java
  @Component // inserido
  public class AplicadorTema {

    private final List<Tema> temas; // inserido

    public AplicadorTema(List<Tema> temas) { // inserido
      this.temas = temas;
    }

    public String aplica(String html) {
      Document document = Jsoup.parse(html);

      L̶i̶s̶t̶<̶S̶t̶r̶i̶n̶g̶>̶ ̶l̶i̶s̶t̶a̶D̶e̶T̶e̶m̶a̶s̶ ̶=̶ ̶T̶e̶m̶a̶.̶l̶i̶s̶t̶a̶D̶e̶T̶e̶m̶a̶s̶(̶)̶;̶
      f̶o̶r̶ ̶(̶S̶t̶r̶i̶n̶g̶ ̶c̶s̶s̶ ̶:̶ ̶l̶i̶s̶t̶a̶D̶e̶T̶e̶m̶a̶s̶)̶ ̶{̶
      for (Tema tema : temas) { // inserido
        String css = tema.cssDoTema(); // inserido
        document.select("head").append("<style> " + css + " </style>");
      }

      return document.html();
    }

  }
  ```

  Faça o import correto:

  ####### cotuba.tema.AplicadorTema

  ```java
  import org.springframework.stereotype.Component;
  ```

5. Faça com que o Spring gerencie a classe `RenderizadorMDParaHTMLComCommonMark`, anotando-a com `@Component`.

  Defina um construtor que recebe um `AplicadorTema` e armazene a instância recebida em um atributo.

  Remova a instanciação de `AplicadorTema` do método `renderiza`.

  ```java
  @Component // inserido
  public class RenderizadorMDParaHTMLComCommonMark implements RenderizadorMDParaHTML {

    private final AplicadorTema tema; // inserido

    public RenderizadorMDParaHTMLComCommonMark(AplicadorTema tema) { // inserido
      this.tema = tema;
    }

    @Override
    public List<Capitulo> renderiza(RepositorioDeMDs repositorioDeMDs) {

      List<Capitulo> capitulos = new ArrayList<>();

      // código omitido...

      A̶p̶l̶i̶c̶a̶d̶o̶r̶T̶e̶m̶a̶ ̶t̶e̶m̶a̶ ̶=̶ ̶n̶e̶w̶ ̶A̶p̶l̶i̶c̶a̶d̶o̶r̶T̶e̶m̶a̶(̶)̶;̶
      String htmlComTemas = tema.aplica(html);
      capituloBuilder.comConteudoHTML(htmlComTemas);

      // código omitido...

      return capitulos;

    }

  }
  ```

6. Remova, da inteface `RenderizadorMDParaHTML`, o método estático `cria`.

  ####### cotuba.application.RenderizadorMDParaHTML

  ```java
  public interface RenderizadorMDParaHTML {

    List<Capitulo> renderiza(RepositorioDeMDs repositorioDeMDs);

    p̶u̶b̶l̶i̶c̶ ̶s̶t̶a̶t̶i̶c̶ ̶R̶e̶n̶d̶e̶r̶i̶z̶a̶d̶o̶r̶M̶D̶P̶a̶r̶a̶H̶T̶M̶L̶ ̶c̶r̶i̶a̶(̶)̶ ̶{̶
      r̶e̶t̶u̶r̶n̶ ̶n̶e̶w̶ ̶R̶e̶n̶d̶e̶r̶i̶z̶a̶d̶o̶r̶M̶D̶P̶a̶r̶a̶H̶T̶M̶L̶C̶o̶m̶C̶o̶m̶m̶o̶n̶M̶a̶r̶k̶(̶)̶;̶
    }̶

  }
  ```

  Limpe o import desnecessário:

  ####### cotuba.application.RenderizadorMDParaHTML

  ```java
  i̶m̶p̶o̶r̶t̶ ̶c̶o̶t̶u̶b̶a̶.̶m̶d̶.̶R̶e̶n̶d̶e̶r̶i̶z̶a̶d̶o̶r̶M̶D̶P̶a̶r̶a̶H̶T̶M̶L̶C̶o̶m̶C̶o̶m̶m̶o̶n̶M̶a̶r̶k̶;̶
  ```

7. Crie uma classe `AoFinalizarGeracaoConfig`, no pacote `cotuba.plugin`, que será responsável por fornecer todas as implementações do plugin `AoFinalizarGeracao`.

  Anote a nova classe com `@Configuration`.
  
  Use o `ServiceListFactoryBean` do Spring para carregar os Service Providers da SPI. Anote com `@Bean` os métodos que produzem instâncias que serão injetadas em outros objetos.

  ####### cotuba.plugin.TemaConfig

  ```java
  @Configuration
  public class AoFinalizarGeracaoConfig {

    @Bean("finalizacoes")
    public ServiceListFactoryBean finalizacoesFactory() {
      ServiceListFactoryBean serviceListFactoryBean = new ServiceListFactoryBean();
      serviceListFactoryBean.setServiceType(AoFinalizarGeracao.class);
      return serviceListFactoryBean;
    }

    @Bean
    public List<AoFinalizarGeracao> listaDeFinalizacoes(@Qualifier("finalizacoes") ServiceListFactoryBean finalizacoesFactory) {
      try {
        return (List<AoFinalizarGeracao>) finalizacoesFactory.getObject();
      } catch (Exception e) {
        throw new RuntimeException(e);
      }
    }

  }
  ```

  A lista de service providers da SPI `AoFinalizarGeracao` poderá ser injetada pelo Spring em outros objetos.

  Faça os imports corretos:

  ####### cotuba.plugin.AoFinalizarGeracao

  ```java
  import java.util.List;

  import org.springframework.beans.factory.annotation.Qualifier;
  import org.springframework.beans.factory.serviceloader.ServiceListFactoryBean;
  import org.springframework.context.annotation.Bean;
  import org.springframework.context.annotation.Configuration;
  ```

8. Remova da interface `AoFinalizarGeracao` o método estático `gerou`. Mantenha somente o método `aposGeracao`.

  ####### cotuba.plugin.AoFinalizarGeracao

  ```java
  public interface AoFinalizarGeracao {

    void aposGeracao(Ebook ebook, Consumer<String> acaoPosGeracao);

    s̶t̶a̶t̶i̶c̶ ̶v̶o̶i̶d̶ ̶g̶e̶r̶o̶u̶(̶E̶b̶o̶o̶k̶ ̶e̶b̶o̶o̶k̶,̶ ̶C̶o̶n̶s̶u̶m̶e̶r̶<̶S̶t̶r̶i̶n̶g̶>̶ ̶a̶c̶a̶o̶P̶o̶s̶G̶e̶r̶a̶c̶a̶o̶)̶ ̶{̶
      S̶e̶r̶v̶i̶c̶e̶L̶o̶a̶d̶e̶r̶<̶A̶o̶F̶i̶n̶a̶l̶i̶z̶a̶r̶G̶e̶r̶a̶c̶a̶o̶>̶ ̶l̶o̶a̶d̶e̶r̶ ̶=̶ ̶S̶e̶r̶v̶i̶c̶e̶L̶o̶a̶d̶e̶r̶.̶l̶o̶a̶d̶(̶A̶o̶F̶i̶n̶a̶l̶i̶z̶a̶r̶G̶e̶r̶a̶c̶a̶o̶.̶c̶l̶a̶s̶s̶)̶;̶
      f̶o̶r̶ ̶(̶A̶o̶F̶i̶n̶a̶l̶i̶z̶a̶r̶G̶e̶r̶a̶c̶a̶o̶ ̶p̶l̶u̶g̶i̶n̶ ̶:̶ ̶l̶o̶a̶d̶e̶r̶)̶ ̶{̶
        p̶l̶u̶g̶i̶n̶.̶a̶p̶o̶s̶G̶e̶r̶a̶c̶a̶o̶(̶e̶b̶o̶o̶k̶,̶ ̶a̶c̶a̶o̶P̶o̶s̶G̶e̶r̶a̶c̶a̶o̶)̶;̶
      }̶
    }̶

  }
  ```

  Remova o import desnecessário:

  ####### cotuba.plugin.AoFinalizarGeracao

  ```java
  i̶m̶p̶o̶r̶t̶ ̶j̶a̶v̶a̶.̶u̶t̶i̶l̶.̶S̶e̶r̶v̶i̶c̶e̶L̶o̶a̶d̶e̶r̶;̶
  ```

9. Defina uma classe `GeradorEbookConfig`, anotada com `@Configuration`.

  Por meio da classe `ServiceListFactoryBean`, obtenha as instâncias dos Service Providers que implementam a SPI `GeradorEbook`.

  A partir de cada instância, monte um `Map` que associa o `FormatoEbook` à implementação de `GeradorEbook`.

  ####### cotuba.plugin.GeradorEbookConfig

  ```java
  @Configuration
  public class GeradorEbookConfig {

    @Bean("geradores")
    public ServiceListFactoryBean geradoresFactory() {
      ServiceListFactoryBean serviceListFactoryBean = new ServiceListFactoryBean();
      serviceListFactoryBean.setServiceType(GeradorEbook.class);
      return serviceListFactoryBean;
    }

    @Bean
    public Map<FormatoEbook, GeradorEbook> formatosParaGeradores(
        @Qualifier("geradores") ServiceListFactoryBean geradoresFactory) {
      try {
        List<GeradorEbook> geradores = (List<GeradorEbook>) geradoresFactory.getObject();
        return geradores.stream().collect(Collectors.toMap(GeradorEbook::formato, Function.identity()));
      } catch (Exception e) {
        throw new RuntimeException(e);
      }

    }

  }
  ```

  Não deixe de fazer os imports:

  ####### cotuba.plugin.GeradorEbookConfig

  ```java
  import java.util.List;
  import java.util.Map;
  import java.util.function.Function;
  import java.util.stream.Collectors;

  import org.springframework.beans.factory.annotation.Qualifier;
  import org.springframework.beans.factory.serviceloader.ServiceListFactoryBean;
  import org.springframework.context.annotation.Bean;
  import org.springframework.context.annotation.Configuration;

  import cotuba.domain.FormatoEbook;
  ```

10. Remova o método estático `cria` da interface `GeradorEbook`. Deixe apenas os métodos `gera` e `formato`.

  ####### cotuba.plugin.GeradorEbook

  ```java
  public interface GeradorEbook {

    void gera(Ebook ebook);

    FormatoEbook formato();

    p̶u̶b̶l̶i̶c̶ ̶s̶t̶a̶t̶i̶c̶ ̶G̶e̶r̶a̶d̶o̶r̶E̶b̶o̶o̶k̶ ̶c̶r̶i̶a̶(̶F̶o̶r̶m̶a̶t̶o̶E̶b̶o̶o̶k̶ ̶f̶o̶r̶m̶a̶t̶o̶)̶ ̶{̶
      f̶o̶r̶ ̶(̶G̶e̶r̶a̶d̶o̶r̶E̶b̶o̶o̶k̶ ̶g̶e̶r̶a̶d̶o̶r̶ ̶:̶ ̶S̶e̶r̶v̶i̶c̶e̶L̶o̶a̶d̶e̶r̶.̶l̶o̶a̶d̶(̶G̶e̶r̶a̶d̶o̶r̶E̶b̶o̶o̶k̶.̶c̶l̶a̶s̶s̶)̶)̶ ̶{̶
        i̶f̶ ̶(̶g̶e̶r̶a̶d̶o̶r̶.̶f̶o̶r̶m̶a̶t̶o̶(̶)̶.̶e̶q̶u̶a̶l̶s̶(̶f̶o̶r̶m̶a̶t̶o̶)̶)̶ ̶{̶
          r̶e̶t̶u̶r̶n̶ ̶g̶e̶r̶a̶d̶o̶r̶;̶
        }̶
      }̶
      t̶h̶r̶o̶w̶ ̶n̶e̶w̶ ̶R̶u̶n̶t̶i̶m̶e̶E̶x̶c̶e̶p̶t̶i̶o̶n̶(̶"̶F̶o̶r̶m̶a̶t̶o̶ ̶d̶o̶ ̶e̶b̶o̶o̶k̶ ̶i̶n̶v̶á̶l̶i̶d̶o̶:̶ ̶"̶ ̶+̶ ̶f̶o̶r̶m̶a̶t̶o̶)̶;̶
    }̶

  }  
  ```

  Limpe o import:

   ####### cotuba.plugin.GeradorEbook

  ```java
  i̶m̶p̶o̶r̶t̶ ̶j̶a̶v̶a̶.̶u̶t̶i̶l̶.̶S̶e̶r̶v̶i̶c̶e̶L̶o̶a̶d̶e̶r̶;̶
  ```

11. Anote a classe `Cotuba` com `@Component`, fazendo com que ela seja gerenciada pelo Spring.

  Receba, pelo construtor, o `RenderizadorMDParaHTML`, o `Map` de formatos para geradores e `List` de ações pós geração do ebook. Armazene todas essas instâncias em atributos e as use no método `executa`.

  ####### cotuba.application.Cotuba

  ```java
  @Component // inserido
  public class Cotuba {

    private final RenderizadorMDParaHTML renderizador;  // inserido
    private final Map<FormatoEbook, GeradorEbook> geradores;  // inserido
    private final List<AoFinalizarGeracao> finalizacoes;  // inserido

     // inserido
    public Cotuba(RenderizadorMDParaHTML renderizador,
                   Map<FormatoEbook, GeradorEbook> geradores,
                        List<AoFinalizarGeracao> finalizacoes) {
      this.renderizador = renderizador;
      this.geradores = geradores;
      this.finalizacoes = finalizacoes;
    }

    public void executa(ParametrosCotuba parametros, RepositorioDeMDs repositorioDeMDs,
        Consumer<String> acaoPosGeracao) {

      FormatoEbook formato = parametros.getFormato();
      Path arquivoDeSaida = parametros.getArquivoDeSaida();

      R̶e̶n̶d̶e̶r̶i̶z̶a̶d̶o̶r̶M̶D̶P̶a̶r̶a̶H̶T̶M̶L̶ ̶r̶e̶n̶d̶e̶r̶i̶z̶a̶d̶o̶r̶ ̶=̶ ̶R̶e̶n̶d̶e̶r̶i̶z̶a̶d̶o̶r̶M̶D̶P̶a̶r̶a̶H̶T̶M̶L̶.̶c̶r̶i̶a̶(̶)̶;̶
      List<Capitulo> capitulos = renderizador.renderiza(repositorioDeMDs); // inserido

      Ebook ebook = new Ebook(formato, arquivoDeSaida, capitulos);

      G̶e̶r̶a̶d̶o̶r̶E̶b̶o̶o̶k̶ ̶g̶e̶r̶a̶d̶o̶r̶ ̶=̶ ̶G̶e̶r̶a̶d̶o̶r̶E̶b̶o̶o̶k̶.̶c̶r̶i̶a̶(̶f̶o̶r̶m̶a̶t̶o̶)̶;̶
      GeradorEbook gerador = geradores.get(formato);  // inserido
      if (gerador == null) {  // inserido
        throw new RuntimeException("Não há gerador de livro para o formato " + formato);
      }
      gerador.gera(ebook);

      A̶o̶F̶i̶n̶a̶l̶i̶z̶a̶r̶G̶e̶r̶a̶c̶a̶o̶.̶g̶e̶r̶o̶u̶(̶e̶b̶o̶o̶k̶,̶ ̶a̶c̶a̶o̶P̶o̶s̶G̶e̶r̶a̶c̶a̶o̶)̶;̶
      for (AoFinalizarGeracao finalizacao : finalizacoes) {  // inserido
        finalizacao.aposGeracao(ebook, acaoPosGeracao);  // inserido
      }

    }

  }
  ```

12. Para que o Spring consiga usar a Reflection API para gerenciar os `@Component`, abra para reflexão no `module-info.java` do módulo `cotuba-core`, os pacotes `cotuba.plugin`, `cotuba.md` e `cotuba.tema`.

  ####### cotuba-core/src/main/java/module-info.java

  ```java
  module cotuba.core {

    // código omitido...

    opens cotuba.plugin;
    opens cotuba.md;
    opens cotuba.tema;

  }
  ```

13. Teste a geração de EPUBs e PDFs pelo `cotuba-cli`, executando a classe `Main`. Deve funcionar!

## Exercício: Injetando a classe Cotuba no Cotuba Web

1. Para que a classe `Cotuba` possa ser injetada pelo Spring nas classes do módulo `cotuba-web`, anote com `@ComponentScan` a classe `Boot`, passando o pacote `cotuba` como parâmetro.

  ####### cotuba.web.Boot

  ```java
  @ComponentScan("cotuba") // inserido
  @SpringBootApplication
  public class Boot {

    public static void main(String[] args) {
      SpringApplication.run(Boot.class, args);
    }

  }
  ```

  Não esqueça do import:

  ####### cotuba.web.Boot

  ```java
  import org.springframework.context.annotation.ComponentScan;
  ```

2. Na classe `GeracaoDeLivros`, do pacote `cotuba.web.application`, remova a instanciação da classe `Cotuba` no método `geraLivro`. Ao invés disso, receba o `Cotuba` como parâmetro do construtor, armazenando-o em um atributo.

  ####### cotuba.web.application.GeracaoDeLivros

  ```java
  @Service
  public class GeracaoDeLivros {

    private final CadastroDeLivros livros;
    private final Cotuba cotuba; // inserido

    p̶u̶b̶l̶i̶c̶ ̶G̶e̶r̶a̶c̶a̶o̶D̶e̶L̶i̶v̶r̶o̶s̶(̶C̶a̶d̶a̶s̶t̶r̶o̶D̶e̶L̶i̶v̶r̶o̶s̶ ̶l̶i̶v̶r̶o̶s̶)̶ ̶{̶
    public GeracaoDeLivros(CadastroDeLivros livros, Cotuba cotuba) { // modificado
      this.livros = livros;
      this.cotuba = cotuba; // inserido
    }

    public Path geraLivro(Long id, FormatoEbook formato) {
      Livro livro = livros.detalha(id);

      C̶o̶t̶u̶b̶a̶ ̶c̶o̶t̶u̶b̶a̶ ̶=̶ ̶n̶e̶w̶ ̶C̶o̶t̶u̶b̶a̶(̶)̶;̶

      // código omitido...

      return parametros.getArquivoDeSaida();
    }

  }
  ```

3. Teste a geração de PDFs e EPUBs pelo Cotuba Web, acessando a URL a seguir. Deve funcionar!

  http://localhost:8080/
