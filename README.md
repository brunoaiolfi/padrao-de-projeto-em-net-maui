# Estrutura dos projetos em .NET MAUI

## Introdução
Primeiramente gostaria de dizer que o conteúdo a seguir foi elaborado com base no livro Arquitetura limpa, conversas entre os desenvolvedores e experiências em outros projetos.

O conteúdo a seguir não é regra talhada em pedra, sempre leve o cenário em que você se encontra em consideração.

## Como consumir este artigo

Iremos começar com um resumo geral da estrutura e posteriormente terá uma explicação detalhada de cada tópico. O resumo servirá como uma introdução, terá tudo o que você precisa saber para entender e se localizar em um projeto. Já a explicação detalhada será o que você precisa saber para desenvolver e concluir os seus WI!

## Arquitetura limpa, o que é e por que usar?

A arquitetura limpa, segundo Robert Martin, é uma maneira de separar as políticas, dos detalhes do seu software. As políticas de um software compõem todos os casos de uso, regras e modelos do seu negócio. Enquanto todo o resto seria os detalhes, desde a sua UI, o seu framework, banco de dados, a api que você consome, bibliotecas e tudo aquilo que é externo à aplicação. Desta forma você terá os componentes da regra de negócio mais estáveis e não vinculados a detalhes menores do código.

Então, se você precisar migrar de React (UI) para Angular, ou de MySQL para MongoDB, as regras de negócio (ex.: cálculo de impostos, fluxo de vendas) não são afetadas.

### Prós e contras da arquitetura limpa

<table>
    <thead>
        <tr>
            <th>Prós</th>
            <th>Contras</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Separação de responsabilidades claras (domínio, aplicação, infraestrutura)</td>
            <td>Adiciona uma complexidade maior ao código</td>
        </tr>
        <tr>
            <td>Facilidade de manutenção e testes devido ao baixo acoplamento</td>
            <td>Curva de aprendizado mais alta</td>
        </tr>
        <tr>
            <td>Alta reutilização e independência de frameworks</td>
            <td>Desenvolvimento inicial pode ser mais lento</td>
        </tr>
        <tr>
            <td>Alta testabilidade (domínio pode ser testado isoladamente)</td>
            <td>Estrutura pode parecer exagerada para projetos pequenos</td>
        </tr>
        <tr>
            <td>Alta flexibilidade e independência tecnológica</td>
            <td>Exige disciplina e boas práticas para manter a arquitetura limpa</td>
        </tr>
    </tbody>
</table>

Além de todos os pontos positivos desta abordagem, decidimos seguir por este método devido a familiaridade dos desenvolvedores. Facilitando assim a manutenção do código por várias partes.

## Como aplicar

Iremos separar o nosso código em três camadas principais, <b>a infra, a application e o domain.</b> 

### Recapitulação rápida sobre CCP

O princípio do fechamento comum (CCP) reúne componentes que mudam de forma semelhante num mesmo momento. Ou seja, iremos dividir o nosso código com base no que faz ele mudar. 

Por exemplo, em um aplicativo de entregas eu possuo chamadas para diversas apis, apis de georreferenciamento, apis de pagamento, apis de logística, como eu deveria separar as chamadas entre diferentes apis? Segundo o princípio de fechamento comum eu irei separar elas com o seguinte pensamento em mente, <i> se algo da api x mudar a api y será ou não afetada</i>. Se a resposta for sim então devemos manter as chamadas das duas apis juntas, seja na mesma classe, pasta ou função. Então neste cenário poderíamos separar as apis da seguinte forma:

```
/apis
  /logisticas  # Agrupa geolocation + transportadora X (mudam juntas)
  /pagamentos   # Lógica isolada
```

Recapitulado o CCP voltamos as camadas do nosso software.

### Infra

A camada de infra ou infraestrutura, será a camada responsável por conter a parte dos detalhes. Então será aqui que teremos as:

* <b>Apis</b> - As comunicações feitas com apis ficarão concentradas em classes chamadas de <b>Service.</b> Aqui o código deve ser separado por componentes respeitando o princípio de fechamento comum.

```
/Apis
  /Logisticas  # Agrupa geolocation + transportadora X (mudam juntas)
    /DTOS # Agrupa os dtos
    /Service # Agrupa as classes de service
  /Pagamentos   # Lógica isolada
    /DTOS
    /Service
```

* <b>Database</b> - Aqui iremos armazenar as nossas entidades do banco, nossas migrations e nossos repositórios além é claro da nossa classe de contexto do banco de dados. A classe contexto do nosso banco de dados será responsável por configurar o banco conforme as entidades e migrations, enquanto os nossos repositórios serão responsáveis por manipular os dados do nosso banco de dados.

```
/Database
  /Entities
  /Migrations
  /Repositories # Aqui o código será dívido com base no CCP
  DatabaseContext.cs
```

* <b>Implementacoes</b> - Aqui temos o código de implementação das bibliotecas, como a biblioteca de Conectividade (verifica a conexão com a internet), a biblioteca de geolocalização, de notificações, de permissões e etc... Uma classe de implementação pode conter mais de uma biblioteca.

* <b>Views</b> - Nas views teremos então as classes que serão utilizadas pela nossa UI. A UI será abordada futuramente e não ficará em nenhuma pasta das camadas principais.

### Domain

A nossa camada de domain ou domínio, será responsável por conter todas as políticas do nosso negócio. Aqui é de extrema importância o uso do CCP para termos um código mais coeso. A estrutura do código poderá ficar da seguinte forma:

```
/Domain
  /Pagamentos
    /Models # Contém a classe que representa o nosso objeto segundo o domínio
        PagamentoModel.cs
    /UseCases # Contém os casos de uso e as regras de negócio
        CriarPagamentoUseCase.cs
  /Logisticas
    /Models
    /UseCases
```

Aqui vale ressaltar que a sua <b>Model não é uma cópia do seu banco de dados e nem muito menos uma cópia de algum DTO</b>, ela deve refletir somente a sua regra de negócio e casos de uso.

### Application

Para facilitar o desenvolvimento e a comunicação entre ambos extremos da nossa arquitetura (Domain e Infra) teremos uma classe intermediária que irá unir os nossos casos de uso com os detalhes da nossa aplicação.

Por exemplo, no CriaPagamentoUseCase do domínio eu irei possuir um método que cuidará de criar um pagamento, sem se comunicar com nenhuma api e sem persistir os dados em um banco de dados, afinal a minha política não depende destes detalhes. Porém o meu aplicativo precisa se comunicar com uma api externa e precisa persistir esses dados em um banco de dados, para resolver este problema iremos criar a classe PagamentosAplic.

```
# pseudocódigo
PagamentosAplic (IPagamentoRepo PagamentoRepo, IPagamentoService PagamentoService, ICriarPagamentoUseCase CriarPagamentoUseCase) {

    CriaPagamento(IDadosDoPagamento DadosDoPagamento) {
        var Pagamento = CriarPagamentoUseCase(DadosDoPagamento) # Criei o pagamento conforme os meus casos de uso

        PagamentoRepo.Inserir(Pagamento) # Persisti os dados do meu pagamento 
        PagamentoService.Salvar(Pagamento) # Chama o service de pagamentos para salvar esse pagamento no backend
    }
}
```

Portanto a divisão da nossa estrutura fica mais ou menos assim:

<table>
    <thead>
        <tr>
            <th>Camada</th>
            <th>Responsabilidade</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><strong>Domain</strong></td>
            <td>Regras de negócio puras (modelos + casos de uso, sem dependências externas)</td>
        </tr>
        <tr>
            <td><strong>Application</strong></td>
            <td>Orquestra os casos de uso e aciona repositórios ou serviços externos (via interfaces)</td>
        </tr>
        <tr>
            <td><strong>Infrastructure</strong></td>
            <td>Implementa os detalhes técnicos como banco de dados, APIs externas, bibliotecas, etc.</td>
        </tr>
    </tbody>
</table>

## Views e ViewModels

Como dito anteriormente, a nossa UI se encaixa na nossa camada de INFRA portanto a mesmo pode ser substituída a qualquer momento. Porém no .NET Maui as coisas são um pouco diferentes, aqui o padrão é de MVVM, ou seja, eu possuo models, views e viewmodels.

Na arquitetura de MVVM as minhas models representam os dados e a lógica de negócios, a minha view seria a UI e por fim a viewmodel seria a ponte entre ambas.

<b>Não iremos seguir esta arquitetura.</b> O que faremos será um pouco diferente, iremos utilizar as nossas views e as facilidades que as viewmodels nos proporcionam para encaixar elas na nossa arquitetura. Desta forma iremos respeitar um princípio básico da arquitetura limpa, "Os detalhes não devem influenciar nas políticas". Mas como faremos isto?

Simples, iremos tratar a nossa UI de acordo com a sua classificação, ou seja, <b> um detalhe</b>. Portanto a nossa viewmodel se comunicará com o restante da infra e também com as applications, porém nunca se comunicará diretamente com o domain. Aqui a sua viewmodel não pode se comunicar diretamente com os repositórios e com os services, para evitar acoplamentos da ui com o restante. Acho que ficará melhor deixar essas comunicações contidas na application e a viewmodel pedir para a application esses dados. Desta forma teríamos um código mais centralizado.

Então o nosso fluxo ficará assim:

A <b>View</b> chama a sua <b>Viewmodel</b> que por sua vez chamará a <b>Application</b> que consumirá as regras e casos de uso da <b>Domain</b> e também poderá consumir os repositórios e os services.

Para não confundir os devs mais tradicionais de .NET Maui, as nossas views e viewmodels, apesar de se encaixarem na infra, ficarão forá da infra. Portanto a divisão da nossa estrutura fica mais ou menos assim:

<table>
    <thead>
        <tr>
            <th>Camada</th>
            <th>Responsabilidade</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><strong>Domain</strong></td>
            <td>Regras de negócio puras (modelos + casos de uso, sem dependências externas)</td>
        </tr>
        <tr>
            <td><strong>Application</strong></td>
            <td>Orquestra os casos de uso e aciona repositórios ou serviços externos (via interfaces)</td>
        </tr>
        <tr>
            <td><strong>Infra</strong></td>
            <td>Implementa os detalhes técnicos como banco de dados, APIs externas, bibliotecas, etc.</td>
        </tr>
        <tr>
            <td><strong>Views</strong></td>
            <td>Interface do usuário (XAML/MAUI), exibe dados e captura interações, sem lógica de própria</td>
        </tr>
        <tr>
            <td><strong>ViewModels</strong></td>
            <td>Prepara dados para a View, executa comandos e se comunica exclusivamente com a Application e com as implementações de bibliotecas</td>
        </tr>
    </tbody>
</table>

<h4>Agora como será a estrutura dessas classes, a comunicação entre as camadas, como vamos lidar com os erros, quais design patterns são interessantes para o nosso cenário e vários outros detalhes técnicos que você precisa saber para resolver seus primeiros WIs você verá a seguir.</h4>
