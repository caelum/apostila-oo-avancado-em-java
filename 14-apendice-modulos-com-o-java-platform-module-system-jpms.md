# Apêndice: Módulos com o Java Platform Module System (JPMS)



## A falta de encapsulamento dos módulos no Java

Os módulos da plataforma Java são JARs, conforme estudado em capítulos anteriores.

E o que há dentro de JARs? Arquivos `.class` e de configuração. Os arquivos `.class`, que contém tipos como classe, interfaces, enums e anotações , são definidos em diretórios, os pacotes.

Um tipo tem apenas dois níveis de modificadores de acesso:

- o padrão, também conhecido como _package private_, acessível apenas no pacote em que o tipo está definido
- `public`, tornando acessível a tipos de qualquer outro pacote

Em projetos em que você participou, já viu alguma classe ser definida sem o modificador `public`? É muito raro!

Classes ou outros tipos raramente são definidas com o modificador de acesso padrão. Uma classe criada por uma IDE, em geral, já é definida como pública.

Em um projeto modularizado, isso causa problemas: parte das classes deveriam ser detalhes internos de um módulo. Mas se todas as classes são públicas, podem ser acessadas por qualquer outro módulo. Com essa brecha for aberta, é grande a tentação de usar um módulo de maneiras indevidas.

## Exercício: acessando o que não deveríamos

1. Teste a falta de encapsulamento do módulo `cotuba-core` instanciando as classes `CapituloBuilder`, `RenderizadorMDParaHTMLComCommonMark` e `AplicadorTema` na classe `Main` do `cotuba.cli`.

  ####### cotuba.cli.Main

  ```java
  public class Main {

    public static void main(String[] args) {

      CapituloBuilder builder = new CapituloBuilder(); // inserido
      RenderizadorMDParaHTMLComCommonMark md = new RenderizadorMDParaHTMLComCommonMark();  // inserido
      AplicadorTema tema = new AplicadorTema();  // inserido

      // restante do código...

    }

  }
  ```

  Faça os imports:

  ####### cotuba.cli.Main

  ```java
  import cotuba.domain.builder.CapituloBuilder;
  import cotuba.md.RenderizadorMDParaHTMLComCommonMark;
  import cotuba.tema.AplicadorTema;
  ```

  Isso deveria ser possível?

## O problema do Classpath

No módulo `cotuba-cli`, podemos instanciar detalhes internos do módulo `cotuba-core`, já que há uma dependência do módulo de linha de comando em relação ao módulo de negócio declarada no `pom.xml`.

O módulo `cotuba-cli` poderia também usar dependências transitivas: as dependências dos módulos dos quais `cotuba-cli` depende. Portanto, poderíamos utilizar o `Parser` ou o `HtmlRenderer` da biblioteca CommonMark, que é uma dependência do módulo `cotuba-core`

As dependências declaradas no `pom.xml` influenciam na compilação. O módulo `cotuba-core` não poderia usar classes do Apache Commons CLI diretamente no código sem apresentar erros ao compilar, já que essa biblioteca não está entre suas dependências transitivas: `cotuba-core` não depende de `cotuba-cli`, mas o contrário.

Porém, qualquer classe pública de qualquer módulo presente no Classpath pode ser manipulada pela Reflection API em tempo de execução (_runtime_). E, como discutimos anteriormente, a grande maioria das classes é pública.

O Classpath é a lista de módulos (JARs) que podem ser utilizados na compilação e na execução. É definido pela opção `-cp` dos comando `javac` e `java`.

No script de execução definido no módulo `cotuba-cli`, usamos o Classpath para indicar a pasta `libs` como fonte dos JARs usados na aplicação.

####### cotuba-cli/src/scripts/cotuba.sh

```sh
java -cp "libs/*" cotuba.Main "$@"
```

A possibilidade de manipular, pela Reflection API, qualquer classe disponível acontece o Classpath é somente uma lista de classes, sem qualquer referência a seu JAR de origem ou de quais outros JARs dependem.

Com o Classpath, classes e outros tipos públicos, acessíveis por todo o JAR de origem também serão acessíveis a classes de quaisquer outros JARs.

A ausência de algo que represente JAR de origem e suas dependências no Classpath enfraquece o encapsulamento em uma aplicação Java modularizada.

### Outros problemas do Classpath

Nicolai Parlog, no livro [The Java Module System](https://www.manning.com/books/the-java-module-system) (PARLOG, 2018), elenca os seguintes problemas no Classpath:

- _Encapsulamento fraco entre JARs_: conforme estudamos, o Classpath é uma grande lista de classes. As classes públicas são visíveis por quaisquer outras. Não é possível criar uma funcionalidade visível dentro de todos os pacotes de um JAR, mas não fora dele.
- _Ausência de representação das dependências entre JARs_: não há como um JAR declarar de quais outros JARs ele depende apenas com o Classpath.
- _Ausência de checagens automáticas de segurança_: o encapsulamento fraco dos JARs permite que código malicioso acesse e manipule funcionalidade crítica. Porém, é possível implementar manualmente checagens de segurança.
- _Sombreamento de classes com o mesmo nome_: no caso de JARs que definem duas classes com o mesmo nome, apenas uma delas é tornada disponível. E não é possível saber qual.
- _Conflitos entre versões diferentes do mesmo JAR_: duas versões do mesmo JAR no Classpath levam a comportamentos imprevisíveis.
- _JRE é rígida_: não é possível disponibilizar no Classpath um subconjunto das bibliotecas padrão do Java. Muitas classes não utilizadas ficam acessíveis.
- _Performance ruim no startup_: apesar dos class loaders serem _lazy_, carregando classes só no seu primeiro uso, muitas classes já são carregadas ao iniciar uma aplicação.
- _Class loading complexo_

## JPMS, um sistema de módulos para o Java

A partir do Java 9, o Java inclui um sistema de módulos bastante poderoso: o Java Platform Module System (JPMS).

Um módulo JPMS é um JAR que define:

- um nome único para o módulo
- dependências a outros módulos
- pacotes exportados, cujos tipos são acessíveis por outros módulos

Para definir essas informações, o código fonte deve definir um arquivo `module-info.java`, com a seguinte estrutura:

```java
module meu.modulo {

  exports br.com.empresa.pacote;

  requires outro.modulo;

}
```

Um módulo JPMS deverá ter o `module-info.class` no diretório raiz de seu JAR.

Com um módulo JPMS, conseguimos definir **encapsulamento no nível de pacotes**, escolhendo quais pacotes são ou não exportados e, em consequência, acessíveis por outros módulos.

## A JDK modularizada

Um dos grandes avanços do JPMS, disponível a partir da JDK 9, foi a modularização da própria plataforma Java.

O JPMS é resultado do projeto _Jigsaw_, criado em 2009, no início do desenvolvimento da JDK 7.

Antes do Java 9, todo o código das bibliotecas padrão da JDK ficava em apenas no módulo de _runtime_: o `rt.jar`.

O estudo inicial do projeto _Jigsaw_ agrupou o código já existente da JDK em diferentes módulos. Por exemplo, foi identificado um módulo base, que conteria pacotes fundamentais como o `java.lang` e `java.io`; um módulo desktop, com as bibliotecas Swing, AWT; além de módulos para APIs como Java Logging, JMX, JNDI.

A análise das dependências entre os módulos identificados na JDK 7 levou à descoberta de ciclos como: o módulo base depende de Logging que depende de JMX que depende de JNDI que depende de desktop que, por sua vez, depende do base.

![Emaranhado de dependências na JDK 7](imagens/cap14-apendice-modulos-com-o-java-platform-module-system-jpms/dependencias-entre-modulos-jdk7-b65.png)

O código da JDK 8 foi reorganizado para que não houvessem ciclos e dependências indevidas, mesmo que ainda sem um sistema de módulos propriamente dito. Os pacotes que pertenceriam ao módulo base não teriam mais dependências a nenhum outro módulo.

Na JDK 9, foram definidos módulos JPMS para cada parte do código da JDK:

- `java.base`, contendo código de pacotes como `java.lang`, `java.math`, `java.text`, `java.io`, `java.net` e `java.nio`.
- `java.logging`, com a Java Logging API.
- `java.management`, com a Java Managing Extensions (JMX) API.
- `java.naming`, com a Java Naming and Directory Interface (JNDI) API.
- `java.desktop`, contendo código de bibliotecas como Swing, AWT, 2D.

As dependências entre os módulos JPMS da JDK foram organizadas de maneira bem cuidadosa.

![Dependências organizadas na JDK 9](imagens/cap14-apendice-modulos-com-o-java-platform-module-system-jpms/dependencias-entre-modulos-jdk9.png)

Antes da JDK 9, toda aplicação teria disponível todos os pacotes de todas as bibliotecas do Java. Não era possível depender de menos que a totalidade da JDK.

A partir da JDK 9, aplicações modularizadas com JPMS podem escolher, no arquivo `module-info.java`, de quais módulos da JDK dependerão.

## Usando a JDK modularizada com o Classpath

O Classpath é apenas uma lista de classes. O conceito de módulos JPMS que encapsulam pacotes e dependem de outros módulos não existe para o Classpath.

Por outro lado, a partir da versão 9, a JDK é definida em termos de módulos JPMS.

Será que, ao atualizar para a JDK 9, somos obrigados a definir o arquivo `module-info.java` para cada JAR, incluindo o de bibliotecas? A migração de _todos_ os módulos iria requerer um trabalho imenso! Impossível de ser feita na prática!

Como conciliar o Classpath com a JDK modularizada?

Para permitir uma migração mais suave, foi criado o conceito do **unnamed module** (o módulo sem nome), que disponibiliza para o Classpath, módulos JPMS.

Não bastaria que ficasse disponível apenas o módulo `java.base`, já que aplicações podem depender de código de outros módulos, como o `java.logging` ou o `java.desktop`.

Por isso, o _unnamed module_ depende automaticamente do módulo `java.se`, que agrega boa parte dos módulos que estariam disponíveis em versões anteriores, não modularizadas, do Java. São parte do módulo `java.se`, os seguintes módulos:

- `java.base`
- `java.compiler`
- `java.datatransfer`
- `java.desktop`
- `java.instrument`
- `java.logging`
- `java.management`
- `java.management.rmi`
- `java.naming`
- `java.prefs`
- `java.rmi`
- `java.scripting`
- `java.security.jgss`
- `java.security.sasl`
- `java.sql`
- `java.sql.rowset`
- `java.xml`
- `java.xml.crypto`

Alguns módulos não são parte do módulo agregador `java.se`. Entre eles, módulos com detalhes específicos da JDK e módulos referentes ao JavaFX.

## Exercício: Instalando o Java 10 com o SDKMAN!

1. Abra um Terminal e digite os comandos a seguir para baixar e executar o script de instalação da ferramenta _SDKMAN!_:

  ```sh
  curl -s "https://get.sdkman.io" | bash
  source "$HOME/.sdkman/bin/sdkman-init.sh"
  ```

2. Use o _SDKMAN!_ para instalar o Java 10.0.2 (OpenJDK):

  ```sh
  sdk install java 10.0.2-open
  source ~/.bashrc
  ```

  Ao final da instalação, execute o seguinte comando:

  ```sh
  java -version
  ```

  Deve ser apresentado um resultado parecido com o seguinte:

  ```txt
  openjdk version "10.0.2" 2018-07-17
  OpenJDK Runtime Environment 18.3 (build 10.0.2+13)
  OpenJDK 64-Bit Server VM 18.3 (build 10.0.2+13, mixed mode)
  ```

## Exercício: Configurando o Java 10 no Eclipse

1. Acesse _Windows > Preferences_ e, então, _Java > Installed JREs_. Clique em _Add..._.

2. Em _JRE Type_, escolha _Standard VM_ e clique em _Next_.

3. Em _JRE Home_, defina `/home/<usuario-do-curso>/.sdkman/candidates/java/11.0.2-open`.

  Não esqueça de trocar `<usuario-do-curso>` pelo valor correto.

  Clique em Finish.

## Exercício: Atualizando os módulos do Cotuba para o Java 10

1. No `pom.xml` do supermódulo `cotuba`, modifique as propriedades de compilação do Maven para a versão 10.

  ####### cotuba/pom.xml

  ```xml
  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>10</maven.compiler.source> 
    <maven.compiler.target>10</maven.compiler.target> 
  </properties>
  ```

2. Ainda no `pom.xml` do supermódulo `cotuba`, defina uma versão atualizada do plugin de compilação do Maven:

  ```xml
  <build>

    <plugins>

      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.8.0</version>
      </plugin>

    </plugins>

  </build>
  ```

3. Botão direito no projeto `cotuba` > _Maven_ > _Update Project..._. Selecione todos os submódulos do Cotuba e clique em OK.

  Depois da atualização, todos os submódulos devem ter a versão _JavaSE-10_ em _JRE System Library_.

4. Teste a geração de PDF e EPUB pelo módulo `cotuba-cli`. Deve funcionar!

5. Teste o `cotuba-web`, subindo um servidor por meio da classe `Boot` do pacote `cotuba.web`.

  Ih! Ao executar o Cotuba Web, deve ocorrer uma exceção com a seguir:

  ```txt
  org.springframework.beans.factory.BeanCreationException:
    Error creating bean with name 'entityManagerFactory' defined in class path resource
    [org/springframework/boot/autoconfigure/orm/jpa/HibernateJpaConfiguration.class]:
    Invocation of init method failed;
    nested exception is java.lang.NoClassDefFoundError: javax/xml/bind/JAXBException

  at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory[...]
  ...
  at org.springframework.boot.SpringApplication.run(SpringApplication.java:1265)
    [spring-boot-2.0.5.RELEASE.jar:2.0.5.RELEASE]
  at cotuba.web.Boot.main(Boot.java:10) [classes/:na]
  at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:na]
  ...
  at org.springframework.boot.devtools.restart.RestartLauncher.run(RestartLauncher.java:49)
    [spring-boot-devtools-2.0.5.RELEASE.jar:2.0.5.RELEASE]

  Caused by: java.lang.NoClassDefFoundError: javax/xml/bind/JAXBException

  at org.hibernate.boot.spi.XmlMappingBinderAccess.<init>(XmlMappingBinderAccess.java:43) 
    ~[hibernate-core-5.2.17.Final.jar:5.2.17.Final]
  ...
  at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory[...]
  ... 21 common frames omitted

  Caused by: java.lang.ClassNotFoundException: javax.xml.bind.JAXBException

  ...
  ... 32 common frames omitted
  ```

## Erros ao usar o JAXB

A exceção que ocorreu ao tentarmos executar o Cotuba Web tem a seguinte causa raiz:

```txt
Caused by: java.lang.ClassNotFoundException: javax.xml.bind.JAXBException
```

Ou seja, a classe `JAXBException` não foi encontrada.

Perceba, pelo _stack trace_, que o módulo que ocasionou o erro foi o `hibernate-core`.

Esse é um erro muito comum ao migrarmos para a JDK 9 ou superior.

A biblioteca Java Architecture for XML Binding (JAXB) permite transformar objetos Java em XML e vice-versa. Parte do Java EE 5, foi embutida na própria JDK a partir da versão 6.

Além do JAXB, a JDK 6 incluiu APIs e implementações de outras tecnologias Java EE: Java API for XML-Based Web Services (JAX-WS), JavaBeans Activation Framework (JAF) e Common Annotations. A motivação dessas inclusões era prover na própria JDK um solução completa para criação de Web Services SOAP.

Em versões anteriores da JDK, foi incluído um subconjunto da Java Transaction API (JTA), também parte da especificação Java EE. Além dessas, a JDK incluía desde as versões mais antigas, código relacionado à tecnologia CORBA.

A JEP 320 marcou para remoção da JDK as APIs relacionadas à especificação Java EE e à tecnologia CORBA.

Nas JDK 9 e 10, essas bibliotecas foram marcadas como _deprecated_, sendo acessíveis com configurações específicas. A partir da JDK 11, seriam removidas definitivamente.

### Uma solução paliativa para a falta do JAXB

Para que o JAXB fique disponível no Classpath (ou unnamed module) das JDKs 9 e 10, podemos adicionar o módulo `java.xml.bind` usando a seguinte opção na compilação e na execução:

```sh
javac --add-modules java.xml.bind Main.java
```

```sh
java --add-modules java.xml.bind Main
```

Se quisermos disponibilizar todas as APIs do Java EE que estão _deprecated_, basta adicionar o módulo `java.se.ee`.

Porém, essa solução só funcionaria nas JDKs 9 e 10. Ao atualizarmos para a JDK 11, não teríamos nenhum desses módulos disponíveis na JDK.

### A solução correta para a falta do JAXB

Para corrigirmos o problema de maneira mais robusta, fazendo com que funcione com a JDK 11 e posteriores, temos que incluir a API e uma implementação do JAXB como dependências.

Em um projeto Maven, basta adicionar as seguintes dependências ao `pom.xml`:

```xml
<dependency>
  <groupId>javax.xml.bind</groupId>
  <artifactId>jaxb-api</artifactId>
  <version>2.2.11</version>
</dependency>
<dependency>
  <groupId>com.sun.xml.bind</groupId>
  <artifactId>jaxb-core</artifactId>
  <version>2.2.11</version>
</dependency>
<dependency>
  <groupId>com.sun.xml.bind</groupId>
  <artifactId>jaxb-impl</artifactId>
  <version>2.2.11</version>
</dependency>
```

Em um projeto Spring Boot como o Cotuba Web, basta definir a API JAXB, sem a versão. O Spring Boot cuida das versões e demais dependências.

```xml
<dependency>
  <groupId>javax.xml.bind</groupId>
  <artifactId>jaxb-api</artifactId>
</dependency>
```

## Exercício: Corrigindo erro de JAXB no Cotuba Web

1. No `pom.xml` do módulo `cotuba-web`, adicione como dependências a API do JAXB, que é usada pelo Hibernate e não está mais disponível a partir do Java 9.

  Não é preciso declarar uma versão, pois já é declarada pelo Spring Boot.

  ####### cotuba-web/pom.xml

  ```xml
  <dependencies>

    

    <dependency>
      <groupId>javax.xml.bind</groupId>
      <artifactId>jaxb-api</artifactId>
    </dependency>

  </dependencies>
  ```

2. Rode novamente a classe `Boot` do pacote `cotuba.web`. O servidor deve ser iniciado com sucesso!

  Teste o Cotuba Web através da URL:

  http://localhost:8080/

  Não deixe de gerar PDF e EPUB.

## Encapsulando os módulos do Cotuba com o JPMS

Depois de conseguirmos executar o Cotuba CLI e o Cotuba Web com o Java 10, seria interessante usarmos o JPMS para termos encapsulamento no nível de pacotes, escolhendo quais podem ou não ser acessador por outros pacotes.

Para isso, criaríamos arquivos `module-info.java`, definindo um nome para o módulo, quais pacotes serão exportados e quais outros módulos são dependências.

Para o módulo `cotuba.core`, teríamos:

```java
module cotuba.core {

  exports cotuba.application;
  exports cotuba.plugin;
  exports cotuba.domain;

}
```

_Observação: o nome de um módulo JPMS só poder conter caracteres alfanuméricos e pontos. Não pode conter hifens._

Já para o módulo `cotuba.cli`, que depende do módulo `cotuba.core`, teríamos:

```java
module cotuba.cli {

  requires cotuba.core;

}
```

O `requires` no módulo `cotuba.cli` indica que há uma dependência em relação ao módulo `cotuba.core`.

## Módulos automáticos

Mas e quanto às outras dependências, como as bibliotecas CommonMark e Jsoup para o módulo `cotuba.core` e a Apache Commons CLI para o `cotuba.cli`?

Nenhuma dessas dependências é modularizada com JPMS, ou seja, nenhuma tem o arquivo `module-info.class` em seu JAR.

O que colocar no `requires`?

Para permitir que JARs modularizados dependam de JARs não modularizados, suavizando o esforço de migração, foi criado o conceito dos **Automatic Modules** (módulos automáticos) no JPMS.

Um _Automatic Module_:

- tem seu nome derivado do JAR, trocando hifens por pontos e removendo a versão.
- são exportados todos os seus pacotes.
- são declarados como dependências todos os outros módulos.

Uma biblioteca não modularizada pode influenciar o nome de seu _Automatic Module_ definindo a propriedade `Automatic-Module-Name` no arquivo `META-INF/MANIFEST.MF`.

Uma maneira de descobrir o nome do _Automatic Module_ criado a partir de um JAR é executar o comando abaixo:

```sh
jar --file jsoup-1.11.2.jar --describe-module
```

Como resposta, obteríamos:

```txt
No module descriptor found. Derived automatic module.

jsoup@1.11.2 automatic
requires java.base mandated
contains org.jsoup
contains org.jsoup.helper
contains org.jsoup.internal
contains org.jsoup.nodes
contains org.jsoup.parser
contains org.jsoup.safety
contains org.jsoup.select
```

Para as bibliotecas usadas nos módulos `cotuba-core`, `cotuba-pdf`, `cotuba-epub` e `cotuba-cli`, teríamos os seguintes _Automatic Modules_:

- `jsoup` para `jsoup-1.11.2.jar`, conforme vimos anteriormente
- `org.commonmark` para `commonmark-0.11.0.jar`, definido pela propriedade `Automatic-Module-Name`
- `html2pdf` para `html2pdf-2.0.0.jar`, parte do iText pdfHTML
- `kernel` para `kernel-7.1.0.jar`, parte do iText pdfHTML
- `layout` para `layout-7.1.0.jar`, parte do iText pdfHTML
- `io` para `io-7.1.0.jar`, parte do iText pdfHTML
- `epublib.core` para `epublib-core-3.1.jar`
- `commons.cli` para `commons-cli-1.4.jar`

## Exercício: Migrando o módulo Cotuba Core para o JPMS

1. Clique com o botão direito no projeto `cotuba-core` e vá em _Configure > Create module-info.java_.

  Na próxima tela, digite `cotuba.core` em _Module name_ e clique em _Create_.

  Será criado um arquivo `module-info.java` no _source folder_ que exporta todos os pacotes e declara as bibliotecas CommonMark e JSoup como dependências. O arquivo terá um conteúdo parecido com:

  ####### cotuba-core/src/main/java/module-info.java

  ```java
  module cotuba.core {
    exports cotuba.application;
    exports cotuba.domain.builder;
    exports cotuba.plugin;
    exports cotuba.tema;
    exports cotuba.domain;
    exports cotuba.md;

    requires jsoup;
    requires org.commonmark;
  }
  ```

  Ignore o _warning_ na declaração da dependência ao módulo `jsoup`.

2. Remova os `exports` dos pacotes do `cotuba.domain.builder`, `cotuba.tema` e `cotuba.md`.

  ####### cotuba-core/src/main/java/module-info.java

  ```java
  module cotuba.core {

    exports cotuba.application;
    e̶x̶p̶o̶r̶t̶s̶ ̶c̶o̶t̶u̶b̶a̶.̶d̶o̶m̶a̶i̶n̶.̶b̶u̶i̶l̶d̶e̶r̶;̶
    exports cotuba.plugin;
    e̶x̶p̶o̶r̶t̶s̶ ̶c̶o̶t̶u̶b̶a̶.̶t̶e̶m̶a̶;̶
    exports cotuba.domain;
    e̶x̶p̶o̶r̶t̶s̶ ̶c̶o̶t̶u̶b̶a̶.̶m̶d̶;̶

    requires jsoup;
    requires org.commonmark;

  }
  ```

3. Abra a classe `Main` do módulo `cotuba-cli`. Observe os erros de compilação nos imports e nos pontos em que instanciamos  `CapituloBuilder`, `RenderizadorMDParaHTMLComCommonMark` e `AplicadorTema`. As mensagens dos erros nos imports são parecidas com _The type is not accessible_.

  Remove o código código a seguir:

  ####### cotuba.cli.Main

  ```java
  public class Main {

    public static void main(String[] args) {

      C̶a̶p̶i̶t̶u̶l̶o̶B̶u̶i̶l̶d̶e̶r̶ ̶b̶u̶i̶l̶d̶e̶r̶ ̶=̶ ̶n̶e̶w̶ ̶C̶a̶p̶i̶t̶u̶l̶o̶B̶u̶i̶l̶d̶e̶r̶(̶)̶;̶
      R̶e̶n̶d̶e̶r̶i̶z̶a̶d̶o̶r̶M̶D̶P̶a̶r̶a̶H̶T̶M̶L̶C̶o̶m̶C̶o̶m̶m̶o̶n̶M̶a̶r̶k̶ ̶m̶d̶ ̶=̶ ̶n̶e̶w̶ ̶R̶e̶n̶d̶e̶r̶i̶z̶a̶d̶o̶r̶M̶D̶P̶a̶r̶a̶H̶T̶M̶L̶C̶o̶m̶C̶o̶m̶m̶o̶n̶M̶a̶r̶k̶(̶)̶;̶
      A̶p̶l̶i̶c̶a̶d̶o̶r̶T̶e̶m̶a̶ ̶t̶e̶m̶a̶ ̶=̶ ̶n̶e̶w̶ ̶A̶p̶l̶i̶c̶a̶d̶o̶r̶T̶e̶m̶a̶(̶)̶;̶

      // restante do código...

    }

  }
  ```

  Remova também os imports:

  ####### cotuba.cli.Main

  ```java
  i̶m̶p̶o̶r̶t̶ ̶c̶o̶t̶u̶b̶a̶.̶d̶o̶m̶a̶i̶n̶.̶b̶u̶i̶l̶d̶e̶r̶.̶C̶a̶p̶i̶t̶u̶l̶o̶B̶u̶i̶l̶d̶e̶r̶;̶
  i̶m̶p̶o̶r̶t̶ ̶c̶o̶t̶u̶b̶a̶.̶m̶d̶.̶R̶e̶n̶d̶e̶r̶i̶z̶a̶d̶o̶r̶M̶D̶P̶a̶r̶a̶H̶T̶M̶L̶C̶o̶m̶C̶o̶m̶m̶o̶n̶M̶a̶r̶k̶;̶
  i̶m̶p̶o̶r̶t̶ ̶c̶o̶t̶u̶b̶a̶.̶t̶e̶m̶a̶.̶A̶p̶l̶i̶c̶a̶d̶o̶r̶T̶e̶m̶a̶;̶
  ```

  Pronto! Agora tudo deve voltar a ser compilado sem erros.

## Exercício: Migrando o módulo Cotuba CLI para o JPMS

1. Clique com o botão direito no projeto `cotuba-cli` e vá em _Configure > Create module-info.java_.

  Na próxima tela, digite `cotuba.cli` em _Module name_ e clique em _Create_.

  Será criado um arquivo `module-info.java` no _source folder_ que exporta o pacote `cotuba.cli` e declara a biblioteca Apache Commons CLI e o módulo `cotuba-core` como dependências. O arquivo terá um conteúdo parecido com:

  ####### cotuba-cli/src/main/java/module-info.java

  ```java
  module cotuba.cli {
    exports cotuba.cli;

    requires commons.cli;
    requires cotuba.core;
  }
  ```

  Ignore o _warning_ na declaração da dependência ao módulo `commons.cli`.

2. Podemos remover o `exports`, ja que nenhum outro módulo usará o `cotuba.cli`:

  ####### cotuba-cli/src/main/java/module-info.java

  ```java
  module cotuba.cli {
    e̶x̶p̶o̶r̶t̶s̶ ̶c̶o̶t̶u̶b̶a̶.̶c̶l̶i̶;̶

    requires commons.cli;
    requires cotuba.core;
  }
  ```

3. Execute a classe `Main` do módulo `cotuba-cli`, disparando a geração de um PDF ou EPUB. Deve ser lançada a seguinte exceção:

  ```txt
  Exception in thread "main" java.util.ServiceConfigurationError:
    cotuba.plugin.Tema:
      module cotuba.core does not declare `uses`
  ...
  at java.base/java.util.ServiceLoader.load(ServiceLoader.java:1691)
  at cotuba.core/cotuba.plugin.Tema.listaDeTemas(Tema.java:12)
  ...
  ```

## Declarando SPIs e Service Providers com JPMS

A exceção `java.util.ServiceConfigurationError`, ocorre porque um módulo JPMS precisa declarar explicitamente as interfaces que são Service Provider Interface (SPI), cujas implementações poderão ser carregadas com um Service Loader.

Repare na mensagem de erro:

```txt
module cotuba.core does not declare `uses`
```

Devemos declarar o uso de uma SPI no `module-info.java`, por meio da palavra-chave `uses`:

```java
module cotuba.core {

  // ...

  uses cotuba.plugin.GeradorEbook;
}
```

Já um Service Provider, que implementa uma SPI, não precisa mais do arquivo com o nome da SPI no `META-INF/services`. Basta usarmos, no `module-info.java` as palavras-chave `provides` com o nome da SPI e `with` com o nome da implementação:

```java
module cotuba.pdf {

  // ...

  provides cotuba.plugin.GeradorEbook with GeradorPDF;

}
```

## Exercício: Adequando o Service Loader ao JPMS

1. No arquivo `module-info.java` do módulo `cotuba-core`, declare as SPIs utilizadas.

  ####### cotuba-core/src/main/java/module-info.java

  ```java
  module cotuba.core {

    // código omitido...

    uses cotuba.plugin.Tema; // inserido
    uses cotuba.plugin.AoFinalizarGeracao; // inserido
    uses cotuba.plugin.GeradorEbook; // inserido
  }
  ```

2. Teste novamente a geração de PDFs e EPUBs. Deve funcionar!

## Exercício: migrando os geradores de ebooks para o JPMS

1. Clique com o botão direito no projeto `cotuba-pdf` e vá em _Configure > Create module-info.java_.

  Na próxima tela, digite `cotuba.pdf` em _Module name_ e clique em _Create_.

  Será criado um arquivo `module-info.java` no _source folder_ que exporta o pacote `cotuba.pdf` e declara módulos da biblioteca iText e o módulo `cotuba-core` como dependências. O arquivo terá um conteúdo parecido com:

  ####### cotuba-pdf/src/main/java/module-info.java

  ```java
  module cotuba.pdf {
    exports cotuba.pdf;

    requires cotuba.core;
    requires html2pdf;
    requires io;
    requires kernel;
    requires layout;
  }
  ```

  Ignore os _warnings_ nas declarações de dependência aos módulos `html2pdf`, `io`, `kernel` e `layout`.

2. Configure o arquivo `module-info.java` do módulo `cotuba-pdf` para que declare a classe `GeradorPDF` como Service Provider da SPI `GeradorEbook`:

  ####### cotuba-pdf/src/main/java/module-info.java

  ```java
  import cotuba.pdf.GeradorPDF; // inserido

  module cotuba.pdf {

    // código omitido...

    provides cotuba.plugin.GeradorEbook with GeradorPDF; // inserido

  }
  ```

  Mantenha o arquivo `META-INF/services/cotuba.plugin.GeradorEbook` do módulo `cotuba-pdf`. Esse arquivo ainda será usado quando usarmos o módulo `cotuba-pdf` no Classpath.

3. Repita os passos acima para o módulo `cotuba-epub`.

## O Modulepath

Se executarmos um JAR modularizado com JPMS usando o Classpath, não teríamos as vantagens de encapsulamento em nível de pacotes, e nenhuma das outras vantagens mencionadas anteriormente.

As restrições do JPMS só são reforçadas se usarmos o Modulepath.

Para definir o Modulepath com diretórios que contém JARs modularizados, devemos usar a opção `--module-path` nos comandos de compilação (`javac`) e execução (`java`).

A compilação feita pelo Maven já cuida desses detalhes.

Ao executarmos com o comando `java`, devemos informar na opção `--module` qual é o módulo e a classe que contém o método `main`:

```sh
java --module-path libs --module cotuba.cli/cotuba.cli.Main "$@"
```

## Exercício: usando o Modulepath no Cotuba CLI

1. Altere o arquivo `cotuba.sh` do módulo `cotuba-cli`, trocando o uso de Classpath, a opção `-cp`, pelo Modulepath, a opção `--module-path`:

  ####### cotuba-cli/src/scripts/cotuba.sh

  ```sh
  #!/bin/bash
  j̶a̶v̶a̶ ̶-̶c̶p̶ ̶"̶l̶i̶b̶s̶/̶*̶"̶ ̶c̶o̶t̶u̶b̶a̶.̶c̶l̶i̶.̶M̶a̶i̶n̶ ̶"̶$̶@̶"̶
  java --module-path libs --module cotuba.cli/cotuba.cli.Main "$@"
  ```

2. Abra um Terminal e faça o build do projeto:

  ```sh
  cd ~/modulos/cotuba
  mvn clean install
  ```

  O build de todos os módulos deve ser realizado com sucesso.
  
  Será mostrado algo como o seguinte:

  ```txt
  [INFO] ------------------------------------------------------------------------
  [INFO] Reactor Summary:
  [INFO]
  [INFO] cotuba 0.0.1-SNAPSHOT .............................. SUCCESS [  0.408 s]
  [INFO] cotuba-core ........................................ SUCCESS [  2.030 s]
  [INFO] cotuba-pdf ......................................... SUCCESS [  0.391 s]
  [INFO] cotuba-epub ........................................ SUCCESS [  2.944 s]
  [INFO] cotuba-cli ......................................... SUCCESS [  3.179 s]
  [INFO] cotuba-web 0.0.1-SNAPSHOT .......................... SUCCESS [  0.942 s]
  [INFO] ------------------------------------------------------------------------
  [INFO] BUILD SUCCESS
  [INFO] ------------------------------------------------------------------------
  [INFO] Total time: 10.230 s
  [INFO] Finished at: 2018-10-24T16:03:42-03:00
  [INFO] ------------------------------------------------------------------------
  ```

3. Descompacte o ZIP do `cotuba-cli` no Desktop.

  Não deixe de remover o conteúdo anterior do diretório `Desktop/libs`.

  ```sh
  rm -rf ~/Desktop/libs
  unzip -o cotuba-cli/target/cotuba-*-distribution.zip -d ~/Desktop
  ```

4. Vá até o Desktop e teste a geração de um PDF:

  ```sh
  cd ~/Desktop
  ./cotuba.sh -d ~/cotuba/exemplo -f pdf
  ```

  Teste também a geração de um EPUB. Deve funcionar!


