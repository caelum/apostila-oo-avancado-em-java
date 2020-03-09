# Conhecendo o Cotuba

O Cotuba é um gerador de ebooks que converte arquivos escritos no formato Markdown (`.md`), em ebooks nos formatos EPUB (`.epub`) e PDF (`.pdf`).

Cada arquivo `.md` do diretório do livro, em ordem alfabética, será transformado em um capítulo do ebook.

## Exercício: gerando um ebook

1. Abra um Terminal e baixe o Cotuba usando o Git:

  ```sh
  git clone https://github.com/caelum/cotuba.git
  ```

2. Entre na pasta do projeto:

  ```sh
  cd cotuba
  ```

3. Faça o build do projeto usando o Maven:

  ```sh
  mvn install
  ```

4. Descompacte o `.zip` gerado para o seu Desktop com o comando:

  ```sh
  unzip -o target/cotuba-*-distribution.zip -d ~/Desktop
  ```

5. Vá até o seu Desktop:

  ```sh
  cd ~/Desktop
  ```

6. O Cotuba já vem com um livro de exemplo. Para gerar um PDF desse livro, faça:

  ```sh
  ./cotuba.sh -d ~/cotuba/exemplo -f pdf
  ```

  Verifique se está tudo certo com o PDF gerado.

7. Também é possível gerar um EPUB desse livro:

  ```sh
  ./cotuba.sh -d ~/cotuba/exemplo -f epub
  ```

  Verifique se o EPUB está OK.

## Exercício: abrindo o código do Cotuba no Eclipse

1. No Eclipse, vá em _File > Import... > Maven > Existing Maven Projects_ e clique em _Next_.
2. Em _Root Directory_, aponte para o diretório onde você baixou o Cotuba.
  Ou seja, preencha com `/home/<usuario-do-curso>/cotuba`.
  Não esqueça de trocar `<usuario-do-curso>` pelo nome de usuário do curso.
  Clique em _OK_.
3. Clique em _Finish_ e seu projeto será importado.
4. Abra a classe `Main`, do pacote `cotuba`.

## Exercício: gerando um ebook pelo Eclipse

1. Clique com o botão direito na classe `Main` e vá em _Run As > Run Configurations..._.
2. Dê duplo clique em _Java Application_ para adicionar uma nova configuração.
3. Na aba _Arguments_, coloque `-d /home/<usuario-do-curso>/cotuba/exemplo -f pdf` em _Program Arguments_. Não esqueça de trocar `<usuario-do-curso>` pelo nome de usuário do curso.
4. Clique em _Run_.
5. Dê _Refresh_ no projeto e veja se o arquivo `book.pdf` foi gerado.
6. Crie uma outra _Run Configuration_ que gera um EPUB.

## Um pouco sobre a implementação do Cotuba

O Cotuba foi implementado em Java 8 e usa o Maven como ferramenta de build.

A estrutura do código do Cotuba é a seguinte:

```txt
cotuba
├── pom.xml
│
└── src
    ├── assembly
    │   └── distribution.xml
    ├── main
    │   └── java
    │       └── cotuba
    │           └── Main.java
    │
    └── scripts
        └── cotuba.sh
```

As seguintes bibliotecas, declaradas como dependências no `pom.xml`, são usadas:

- _Apache Commons CLI_ para as opções de linha de comando. Detalhes em: https://commons.apache.org/proper/commons-cli/
- _CommonMark Java_ para renderizar arquivos `.md` para HTML. Detalhes em: https://github.com/atlassian/commonmark-java
- _iText pdfHTML_ para gerar os arquivos `.pdf`. Detalhes em: https://github.com/itext/i7j-pdfhtml/
- _Epublib_ para gerar os arquivos `.epub`. Detalhes em: https://github.com/psiegman/epublib

O `.zip`, que é o "entregável" do Cotuba, é gerado pelo _Maven Assembly Plugin_. Mais em: http://maven.apache.org/plugins/maven-assembly-plugin/

A configuração do _Assembly Plugin_ está em `src/assembly/distribution.xml`. Nesse XML, é descrito que o `.zip` conterá os JARs de todas as bibliotecas e o arquivo `src/script/cotuba.sh`.

O código do Cotuba possui apenas uma classe: a classe `Main` do pacote `cotuba`, que define o método `main` e possui cerca de 250 linhas de código.

É um código pequeno e relativamente simples. Mas será que esse código está fácil de ser mantido?

## Para saber mais: a sintaxe do Markdown

Markdown é uma linguagem de marcação com uma sintaxe simples. Foi criada por John Gruber e Aaron Swartz em 2004. Tem como objetivo ter uma marcação enxuta, sendo o mais próxima o possível de texto puro e podendo ser facilmente convertida para HTML.

Alguns detalhes da sintaxe:

### Títulos

`# Título` se transforma em  `<h1>Título</h1>`

`## Título` se transforma em `<h2>Título</h2>`

`### Título` se transforma em `<h3>Título</h3>`

`#### Título` se transforma em `<h4>Título</h4>`

`##### Título` se transforma em `<h5>Título</h5>`

`###### Título` se transforma em `<h6>Título</h6>`

### Itálico

`_texto_`

se transforma em

`<em>texto</em>`

### Negrito

`**texto**`

se transforma em

`<strong>texto</strong>`

### Links

`[Caelum](https://www.caelum.com.br)`

se transforma em

`<a href="https://www.caelum.com.br">Caelum</a>`

### Citações

`> Ser ou não ser, eis a questão`

se transforma em

`<blockquote>Ser ou não ser, eis a questão</blockquote>`

### Código inline

``A classe `String` é imutável``

se transforma em

`A classe <code>String</code> é imutável`

### Listas

```markdown
- modelagem
- dependências
```

se transforma em

```html
<ul>
  <li>modelagem</li>
  <li>dependências</li>
</ul>
```

### Listas ordenadas

```markdown
1. modelagem
2. dependências
```

se transforma em

```html
<ol>
  <li>modelagem</li>
  <li>dependências</li>
</ol>
```

### Blocos de código

````` ``
```
public class Pessoa {
  private String nome;
}
```
`` ``` ``

se transforma em

```html
<pre><code>
public class Pessoa {
  private String nome;
}
</code></pre>
```
