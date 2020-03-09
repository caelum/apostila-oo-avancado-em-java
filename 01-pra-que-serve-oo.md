# Para que serve OO?

## Modelagem e Dependências

Você aprendeu Orientação a Objetos (OO).

Entendeu classes, objetos, atributos, métodos, herança, polimorfismo, interfaces.

Aprendeu algumas soluções comuns para problemas recorrentes estudando _Design Patterns_.

Mas como e quando usar OO?

Sem dúvida, a resposta tem a ver com organizar o código, minimizando o impacto de mudanças no médio/longo prazo.

No post [The Principles of OOD](http://butunclebob.com/ArticleS.UncleBob.PrinciplesOfOod) (MARTIN, 2005), Robert Cecil Martin, o famoso Uncle Bob, descreve duas abordagens complementares no uso de OO:

- criar um **modelo de domínio** no seu código
- gerenciar as **dependências** do seu código

### OO como ferramenta de modelagem do domínio

OO é uma ótima ferramenta para representar, em código, os conceitos do problema que estamos resolvendo. É importante selecionar entidades de negócio e criar um modelo de domínio que as "traduza" para uma linguagem de programação.

Um bom _domain model_ é o foco de metodologias e técnicas como:

- Feature-Driven Development
- Class-Responsibility-Collaboration (CRC)
- _Domain-Driven Design_

### OO como ferramenta de gerenciamento de dependências

Mas OO também é uma ótima maneira de evitar código "amarrado" demais, controlando as dependências e minimizando o acoplamento. Um bom código OO, com as dependências administradas com cuidado, leva a mais flexibilidade, robustez e possibilidade de reuso.

Dependências bem gerenciadas são o foco de técnicas como:

- GRASP Patterns/Principles
- Design Patterns
- Dependency Injection
- **Princípios SOLID.**

## Os princípios SOLID

Nosso foco nesse curso é aprofundar no estudo dos princípios S. O. L. I. D. de Orientação a Objetos, um acrônimo cunhado por Uncle Bob.

Os 5 princípios **S.O.L.I.D.** são:

- _**S**ingle Responsibility Principle (SRP)_: Uma classe deve ter um, e apenas um, motivo para ser modificada.
- _**O**pen/Closed Principle (OCP)_: Deve ser possível estender o comportamento de uma classe sem modificá-la.
- _**L**iskov Substitution Principle (LSP)_: Subtipos devem ser substituíveis por seus tipos base.
- _**I**nterface Segregation Principle (ISP)_: Clientes não devem ser obrigados a depender de métodos que eles não usam.
- _**D**ependency Inversion Principle (DIP)_: Módulos de alto nível não devem depender de módulos de baixo nível. Ambos devem depender de abstrações. Abstrações não devem depender de detalhes. Detalhes devem depender de abstrações.

> _Esses princípios expõe os aspectos de gerenciamento de dependências do Design Orientado a Objetos (OOD), em oposição aos aspectos de conceitualização e modelagem. Isso não quer dizer que OO é uma ferramenta ruim para conceitualização do domínio do problema ou que não é um bom veículo para criar modelos. Certamente, muitas pessoas extraem valor desses aspectos de OO. Os princípios, porém, focam bastante no gerenciamento de dependências._
>
>Uncle Bob, no post [The Principles of OOD](http://butunclebob.com/ArticleS.UncleBob.PrinciplesOfOod) (MARTIN, 2005) - Tradução livre

### O que são princípios?

Uncle Bob indica no artigo [Getting a SOLID start](https://sites.google.com/site/unclebobconsultingllc/getting-a-solid-start) (MARTIN, 2009) que os princípios não são _check lists_, nem leis ou regras.

São bons conselhos vindos do senso comum de gente experiente, coletados em projetos reais ao longo do tempo.

Não significa que sempre funcionam ou que sempre devem ser seguidos.

## Os princípios de coesão e acoplamento de módulos

Uncle Bob também descreve alguns princípios relacionados a módulos.

### Os princípios de coesão de módulos

Os 3 primeiros princípios de módulos descrevem como devemos organizar o conteúdo de um módulo. Ou seja, são sobre **coesão**:

- _Release Reuse Equivalency Principle (REP)_: A granularidade de reuso é a granularidade de entrega.
- _Common Closure Principle (CCP)_: Classes que são modificadas juntas devem estar no mesmo módulo.
- _Common Reuse Principle (CRP)_: Classes que são usadas juntas devem estar no mesmo módulo.

### Os princípios de acoplamento de módulos

Há mais 3 princípios de módulos, focados no **acoplamento** e em métricas para avaliar a estrutura de módulos de um sistema:

- _Acyclic Dependencies Principle (ADP)_: O grafo de dependências dos módulos não deve ter ciclos.
- _Stable Dependencies Principle (SDP)_: Dependa na direção da estabilidade.
- _Stable Abstractions Principle (SAP)_: Abstração traz estabilidade.

## Para saber mais: A história dos princípios SOLID

### Os dez (ou onze) mandamentos de OO

Robert Cecil Martin, o famoso Uncle Bob, listou os seus 10 (na verdade, 11) mandamentos de OO, no grupo Usenet [comp.object](https://groups.google.com/forum/?hl=en#!topic/comp.object/WICPDcXAMG8) (MARTIN, 1995):

_Se eu tivesse que escrever mandamentos, esses seriam os candidatos:_

1. _Entidades de software (classes, módulos, etc) devem ser abertas para extensão, mas fechadas para modificação. (o Open/Closed Principle - Bertrand Meyer)_

2. _Classes derivadas devem ser usáveis através da interface da classe base sem a necessidade do usuário saber a diferença. (o Liskov Substitution Principle)_

3. _Detalhes devem depender de abstrações. Abstrações não devem depender de detalhes. (o Dependency Inversion Principle)_

4. _A granularidade do reuso é a mesma que a granularidade da entrega (release). Apenas componentes que são entregues através de um tracking system podem ser efetivamente reusados._

5. _As classes de um componente entregue devem compartilhar um closure comum. Isto é, se uma classe precisar ser modificada, todos as outras provavelmente precisarão ser modificadas. O que afeta um, afeta todos._

6. _As classes de um componente entregue devem ser reusadas juntas. Isto é, é impossível separar os componentes uns dos outros para que sejam menos que o todo._

7. _A estrutura de dependências dos componentes entregues deve ser um grafo acíclico direcionado (DAG). Não podem existir ciclos._

8. _Dependências entre componentes entregues devem ser na direção da estabilidade. O componente que recebe a dependência deve ser mais estável que o que usa a dependência._

9. _Quando mais estável um componente entregue é, mais deve ser consistir de classes abstratas. Um componente completamente estável deve consistir de nada mais que classes abstratas._

10. _Onde possível, use patterns comprovados para resolver problemas de design._

11. _Ao atravessar entre dois paradigmas diferentes, construa uma camada de interfaces que os separe. Não polua um lado com um paradigma do outro._

(Tradução livre)

### Os princípios tomam forma

Em 1996, fez uma série de artigos na revista _C++ Report_ sobre o que chamou de **princípios**:

- _Open-Closed Principle_ (MARTIN, 1996a)
- _Liskov Substitution Principle_ (MARTIN, 1996b)
- _Dependency Inversion Principle_ (MARTIN, 1996c)
- _Interface Segregation Principle_ (MARTIN, 1996d)
- _Granularity_ (MARTIN, 1996e)
- _Stability_ (MARTIN, 1997)

Em 2002, no livro [Agile Software Development: Principles, Patterns, and Practices](https://www.amazon.com.br/Software-Development-Principles-Patterns-Practices/dp/0135974445) (MARTIN, 2002), Uncle Bob adiciona o _Single Responsibility Principle_.

Em 2004, em colaboração com Michael Feathers, reordena os princípios e cunha o acrônimo _S.O.L.I.D._.

Em 2006, foi lançada uma versão em C# do livro original, com os mesmos princípios: [Agile Principles, Patterns, and Practices in C#](https://www.amazon.com.br/Agile-Principles-Patterns-Practices-C/dp/0131857258) (MARTIN, 2006).
