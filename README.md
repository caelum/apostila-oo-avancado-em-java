# Apostila do curso [Práticas de Design e Arquitetura de Código para Aplicações Java](https://www.caelum.com.br/curso-design-arquitetura-de-aplicacoes-java) da Caelum

Para mais informações sobre o curso, acesse: https://www.caelum.com.br/curso-design-arquitetura-de-aplicacoes-java

## Licença

Esta obra está licenciada com uma Licença [Creative Commons Atribuição-NãoComercial-SemDerivações 4.0 Internacional](http://creativecommons.org/licenses/by-nc-nd/4.0/).

![](https://i.creativecommons.org/l/by-nc-nd/4.0/88x31.png)

## Ementa

### Para que serve OO?
- Modelagem e Dependências
- Os princípios SOLID
- Os princípios de coesão e acoplamento de módulos
- Para saber mais: A história dos princípios SOLID

### Conhecendo o Cotuba
- Exercício: gerando um ebook
- Exercício: abrindo o código do Cotuba no Eclipse
- Exercício: gerando um ebook pelo Eclipse
- Um pouco sobre a implementação do Cotuba
- Para saber mais: a sintaxe do Markdown

### Single Responsibility Principle: classes coesas
- Uma classe com muitos motivos para ser modificada
- O Princípio da Responsabilidade Única
- Responsabilidades, Coesão e SRP no Cotuba
- Exercício: uma classe para ler as opções de linha de comando
- Exercício: uma classe para gerar PDF
- Exercício: uma classe para gerar EPUB
- Refatorar é bom pra aprender OO
- Não se repita
- Duplicação no Cotuba
- Exercício: extraindo duplicação para uma classe renderizadora de Markdown
- Um domain model para o Cotuba
- Exercício: um domain model simples
- Exercício: usando o domain model no renderizador
- Exercício: usando o domain model na classe principal e nos geradores de ebook
- MVC: um molde de distribuição de responsabilidades
- MVC pra linha de comando?
- Exercício: separando a Aplicação do Controller no Cotuba
- Exercício: organizando os pacotes
- Responsabilidades no nível de métodos
- Exercício opcional: métodos responsáveis
- Para saber mais: GRASP
- Para saber mais: Design Simples

### Dependency Inversion Principle: dependências gerenciadas
- Uma classe com muitas dependências
- Acoplamento, Estabilidade e Volatilidade
- Abstrações e inversão das dependências
- Regras de Negócio x Detalhes
- Código de alto nível e baixo nível
- O Princípio da Inversão de Dependências
- Toda interface é uma abstração de alto nível?
- DIP e a Arquitetura em 3 camadas
- Regra da Dependência
- Acoplamento, Volatilidade, Abstrações e DIP no Cotuba
- Exercício: invertendo as dependências da classe Cotuba
- Design Pattern: Factory, gerenciando dependências ao criar objetos
- Exercício: usando Factory para as dependências da classe Cotuba
- Nomeando as implementações
- Exercício: melhorando o nome das implementações
- Abstrações mais perto de quem as usa
- Exercício: movendo as abstrações do Cotuba
- Uma classe para os parâmetros
- Exercício: simplificando parâmetros
- Ih! Ferimos o DIP...
- Exercício: mais uma interface para inverter as dependências

### Open/Closed Principle: objetos flexíveis
- Em busca da abstração perfeita
- Flexibilidade
- Design Pattern: Command, abstraindo operações
- Variações Protegidas
- O Princípio Aberto/Fechado
- OCP, boas abstrações e o Cotuba
- Exercício: uma abstração melhor para a geração de ebooks
- Exercício: removendo interfaces desnecessárias
- Design Pattern: Strategy, diferentes maneiras de fazer a mesma coisa
- Exercício: simplificando os nomes das implementações
- Uma Factory inteligente
- Exercício: implementando uma Factory inteligente
- Polimorfismo
- A campanha Anti-IF
- Exercício: Até mais, condicionais!

### Temas e Plugins
- Aplicando CSS no ebook
- Exercício: uma classe para aplicar temas
- Explorando o design: responsabilidades e dependências
- Exercício: aplicando o tema
- A necessidade de plugins
- Pontos de extensão por meio de interfaces
- Exercício: um plugin para o Cotuba
- Exercício: uma implementação do plugin
- Ligando os pontos (de extensão) com a Service Loader API
- Exercício: ligando a SPI com o Service Provider
- Exercício: carregando o service provider
- Exercício: testando o plugin de tema
- Para saber mais: Alguns usos da Service Loader API na plataforma Java

### Um plugin de estatísticas
- Um plugin menos específico
- Exercício: mais um plugin no Cotuba
- Um problema no plugin de tema
- Exercício: corrigindo o problema do tema
- Um plugin para calcular estatísticas do ebook
- Exercício: iniciando a implementação do plugin de estatísticas
- Exercício: testando o plugin de estatísticas

### Liskov Substitution Principle: herança do jeito certo
- Continuando a implementação das estatísticas
- Exercício: tentando contar as palavras
- Contando palavras
- Exercício: implementando a contagem de palavras
- Palavras em ordem alfabética
- Exercício: ordenando as palavras
- Herdando inutilidades
- OO não é só herança
- O Princípio da Substituição de Liskov
- Favorecendo composição à herança
- Exercício: composição na contagem de palavras
- A forte intimidade entre uma (classe) filha e sua mãe
- Quando usar herança?
- Para saber mais: o uso incorreto de herança na plataforma Java
- Qual a necessidade de um método se não há nada pra ser feito?

### Interface Segregation Principle: clientes separados, interfaces separadas
- Uma interface, dois usos
- O Princípio da Segregação de Interfaces
- Exercício: uma interface separada para temas
- Exercício: uma interface separada para a finalização da geração do ebook
- Separando classes, não apenas interfaces
- Interfaces específicas para o cliente no Cotuba
- Exercício: protegendo o domain model com interfaces
- Desafio (opcional): generalizando o plugin de tema

### Um pouco de imutabilidade e encapsulamento
- Violar o ISP ou o DIP?
- Classes sem setters
- Em direção à imutabilidade
- Imutabilidade
- Exercício: voltando atrás na segregação de interfaces
- Exercício: um domain model imutável
- Usando objetos imutáveis
- Design Pattern: Builder
- Exercício: um builder para os capítulos
- Código invejoso x Código tímido
- Encapsulamento
- A Lei de Deméter e o lema "Diga, não pergunte"
- Maximizando o uso do this
- Quebra de encapsulamento no plugin de estatísticas
- Design Pattern: Iterator
- Iteração externa x iteração interna
- Exercício: iteração interna para favorecer o encapsulamento
- MVC ferido
- Um objeto para impressão
- Uma abstração de uma ação com o Command Pattern
- Uma abordagem mais funcional
- Usando uma interface funcional pré-definida
- Exercício opcional: respeitando o MVC com uma abordagem funcional

### Princípios de Coesão e Acoplamento de Módulos
- O que são módulos?
- JARs
- Porque modularizar?
- O que é reuso?
- Princípios de coesão de módulos
- O Princípio da Equivalência entre Entrega e Reuso
- O Princípio do Agrupamento Comum
- O Princípio do Reuso Comum
- A tensão entre os princípios de coesão de módulos
- Módulos, componentes ou pacotes?
- Modularizando o Cotuba
- Módulos Maven
- Exercício: criando um projeto Maven multi-módulos
- Exercício: criando um módulo para a aplicação
- Exercício: criando um módulo para a linha de comando
- Princípios de acoplamento de módulos
- O Princípio das Dependências Acíclicas
- O Princípio das Dependências Estáveis
- O Princípio das Abstrações Estáveis
- Exercício: um módulo para a geração de PDFs
- Exercício: um módulo para a geração de EPUBs
- Exercício: quebrando ciclos através de plugins
- Métrica: instabilidade
- Métrica: abstratividade

### Além da linha de comando
- Uma interface Web para o Cotuba
- Exercício: uma UI Web
- Entendendo o código do Cotuba Web
- Exercício: implementando o download de ebooks
- Exercício: preparando para chamar o Cotuba
- Chamando o Cotuba a partir da Web
- Quando nosso design não antecipa as mudanças
- Mudanças inesperadas e design incremental
- Exercício: mais abstração, mais flexibilidade
- Exercício: ajustando a linha de comando
- Exercício: gerando ebooks a partir do Cotuba Web

### Arquitetura Hexagonal
- Por uma arquitetura centrada no domínio
- Hexágonos (ou Conectores & Adaptadores)
- Arquitetura Hexagonal no Cotuba
- Arquitetura de Plugins
- Barreiras Arquiteturais
- Clean Architecture
- Outras arquiteturas semelhantes
- Exercício: Discussão
- Para saber mais: Microsserviços e o Monólito Modular

### Apêndice: Módulos com o Java Platform Module System (JPMS)
- A falta de encapsulamento dos módulos no Java
- Exercício: acessando o que não deveríamos
- O problema do Classpath
- JPMS, um sistema de módulos para o Java
- A JDK modularizada
- Usando a JDK modularizada com o Classpath
- Exercício: Instalando o Java 10 com o SDKMAN!
- Exercício: Configurando o Java 10 no Eclipse
- Exercício: Atualizando os módulos do Cotuba para o Java 10
- Erros ao usar o JAXB
- Exercício: Corrigindo erro de JAXB no Cotuba Web
- Encapsulando os módulos do Cotuba com o JPMS
- Módulos automáticos
- Exercício: Migrando o módulo Cotuba Core para o JPMS
- Exercício: Migrando o módulo Cotuba CLI para o JPMS
- Declarando SPIs e Service Providers com JPMS
- Exercício: Adequando o Service Loader ao JPMS
- Exercício: migrando os geradores de ebooks para o JPMS
- O Modulepath
- Exercício: usando o Modulepath no Cotuba CLI

### Apêndice: Injeção de Dependências com Spring
- Exercício: gerenciando o Cotuba CLI com Spring
- Exercício: abrindo o módulo Cotuba CLI para Reflection
- Exercício: gerenciando o Cotuba Core com Spring
- Exercício: Injetando a classe Cotuba no Cotuba Web

### Apêndice: Referências
