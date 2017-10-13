# Metodologia para desenvolvimento de projetos em React (web e native)
Uma metodologia que estou desenvolvendo com base em aprendizados nos diversos projetos que realizei.

As estratégias seguidas neste modelo também são fruto de muitas pesquisas e se baseiam no uso de bibliotecas modernas, seguindo todas as melhores práticas de desenvolvimento web moderno.

Os passos a seguir fazem parte de um ciclo de desenvolvimento e podem ser aplicados de uma vez para o projeto todo ou subdivindo o projeto em pequenas entregas (iterações)

# Bibliotecas utilizadas
- [React](https://facebook.github.io/react/ "React") - Construção da interface gráfica
- [Redux](http://redux.js.org/ "Redux") - Gerenciamento de estado
- [GraphQL](http://graphql.org/learn/ "GraphQL") - Uma linguagem para realizar consultas em grafos
- [Apollo](http://dev.apollodata.com/react/ "Apollo") - Client de GraphQL
- [Immutable](https://facebook.github.io/immutable-js/ "Immutable") - Estruturas de dados imutáveis
- [Reselect](https://github.com/reactjs/reselect "Reselect") - Function memoization
- [Styled Components](https://styled-components.com/ "Styled Components") - Estilização de componentes React
- [Recompose](https://github.com/acdlite/recompose "Recompose") - Utilitários para React funcional
- [Redux Actions](https://github.com/acdlite/redux-actions "Redux Actions") - Utilitários para lidar com Standard Flux Actions

# Overview
Seguiremos uma série de passos tendo como objetivo o desenvolvimento de uma determinada funcionalidade para nosso software.
A imagem a seguir mostra uma visão macro do processo. Nela, assumimos que a equipe está utilizando o SCRUM como framework
de desenvolvimento de projeto.

Esta metodologia poderia ser aplicada em outros modelos, desde que tenhamos de antemão uma lista de tarefas/funcionalidades a serem desenvolvidas nesta iteração.

![Overview](https://raw.githubusercontent.com/rafaelcorreiapoli/react-metodologia/master/images/Overview.png)

# 1. Definição dos componentes visuais
  Tendo em mãos os wireframes/mockups que contenham todos os elementos (e a representação de suas funcionalidades) que serão desenvolvidos nesta iteração, iremos realizar a divisão deles em componentes visuais.
  É nesta hora que iremos estabelecer componentes como:
  - SearchInput
  - TodoList
  - TodoListItem
  - UserAvatar
  - FavoriteStar
  - etc...

  Procuraremos estabelecer um nível de granularidade que evite completamente repetições (DRY) mas ao mesmo tempo tenha foco em praticidade.
  Vamos assumir que, nos wireframes recebidos da equipe de Design, temos o seguinte componente

  ![Overview](https://raw.githubusercontent.com/rafaelcorreiapoli/react-metodologia/master/images/SearchInput.png)

  Conseguimos extrair desta imagem os seguintes sub-componentes:
  - Icone de search **(1)**
  - Input **(2)**
  - Chip com a quantidade de resultados **(3)**
  - Botão de cancel **(4)**

  Cada um destes (**1**, **2**, **3**, **4**), apesar de bem granulares, merecem ser um componente.
  Com isto, conseguiremos reutilizar estes mesmos elementos em outros contextos, caso seja preciso.
  Mas não deixaremos de criar o componente SearchInput, que é um componente que desenha e organiza os subcomponente **1**, **2**, **3**, **4**, nos proporcionando praticidade quando formos de fato construir nosso sistema de busca.

## 1.2 Stateless
  Todos os componentes nesta fase **deverão** ser stateless. Para garantir isto, iremos utilizar a sintaxe para declarar componentes com funções (React Functional Stateless Component):
  ```js
    const SearchInput = () => (
      ...
    )
  ```
  Isto irá nos proporcionar algumas vantagens:
  - Iremos garantir que os componentes desenvolvidos nesta etapa não terão estado próprio, nos forçando a guardar o estado na nossa Store do Redux (single source of truth).
  - A sintaxe é menos verbosa que ES6 classes.
  - Componentes stateless são mais fáceis de serem compreendidos e utilizados.

## 1.2 Props
  É neste momento também que estabeleceremos, olhando para nossos wireframes/mockups, quais Props cada stateless component irá receber.

  Neste momento, não economize em props: Absolutamente **todos**
  os parâmetros que descrevem o estado do componente e os eventos que ele pode emitir devem ser levados
  em consideração.

  Até mesmo estados "superficiais" como por exemplo *hover* (Veremos nos mais adiante como otimizá-los)

  ```js
    const SearchInput = ({
        // estado
        text,
        resultsCount,
        hovered,
        resultsCount,
        // eventos
        onChangeText,
        onClickCancel,
        onHover,
    }) => (
      ...
    )
  ```

### 1.2.1 propTypes
  É de extrema importância que sejam escritos os PropTypes. Eles nos ajudaram em diversos pontos:
  - Servirão de documentação para o componente. Quando revisitarmos este componente no futuro ou se outro membro do time quiser utilizá-lo, ficará fácil de entender o que ele precisa receber e o que faz.
  - Irão garantir que estamos passando os dados corretamente, tornando mais fácil o diagnóstico de problemas.
  - No futuro, utilizaremos estas props para gerar uma documentação oficial do componente com react-docgen

  ```js
  import React, { PropTypes } from 'react'
  const SearchInput = ({
      // estado
      text,
      resultsCount,
      hovered,

      // callbacks
      onChangeText,
      onClickCancel,
      onHover,
  }) => (
    ...
  )
  const { string, number, bool, func } = PropTypes
  SearchInput.propTypes = {
      text: string.isRequired,
      resultsCount: number,
      hovered: bool.isRequired,

      onChangeText: func.isRequired,
      onClickCancel: func.isRequired,
      onHover: func.isRequired
  }
  ```
  *OBS: Sempre iremos seguir esta ordem na desestruturação do objeto e declaração das propTypes:
   Primeiro propriedades de estado do componente e depois callbacks (funções)*

### 1.2.2 defaultProps
  **Todas** as props que não forem required devem especificar uma defaultProp
  Utilize este recurso para tornar mais prático o uso de seus componentes
  ```js
  SearchInput.defaultProps = {
      resultsCount: 0
  }
  ```
---

# 2. Storybook
  Até o momento, não escrevemos o código dos componentes propriamente ditos, apenas estabelecemos
  a divisão lógica entre os diversos componentes e quais parâmetros cada um pode receber.

  Contudo, já temos em mãos como esperamos que o componente se comporte quando submetido a diferentes parâmetros
  (pois temos em mãos os wireframes/mockups), então é uma ótima oportunidade para aplicar o `TDD`

  ## 2.1 Test Drive Development (TDD) para interface gráfica
  Começaremos escrevendo todas as estórias do nosso componente:

  ```js
  const actionHandlers = {
      onChangeText: action('onChangeText'),
      onClickCancel: action('onClickCancel'),
      onHover: action('onHover')
  }
  storiesOf('SearchInput', module)
    .add('With some text, no results', () => (
      <SearchInput
          text="Como os dinoss"
          resultsCount={0}
          hovered={false}
          {...actionHandlers}
      />
    ))
    .add('No text with some results', () => (
      <SearchInput
          text=""
          resultsCount={10}
          hovered={false}
          {...actionHandlers}
      />
    ))
    .add('With some text and hovered', () => (
      <SearchInput
          text="Como os dinoss"
          resultsCount={0}
          hovered
          {...actionHandlers}
      />
    ))
    .add('With some text and results', () => (
      <SearchInput
          text="Como os dinoss"
          resultsCount={10}
          hovered={false}
          {...actionHandlers}
      />
    ))
  ```
  Repare que, como no momento não estamos interessados em alterar o estado devido à emissão de eventos, estamos passando a função retornada pela chamada do `action(String)`,
  que apenas logará no console do Storybook que a função foi chamada e quais parâmetros e ela foram passados.

  É claro que, devido às diferentes combinações de props, o número de estórias irá crescer muito.
  Se preocupe em cobrir os casos mais comuns e que, pelo menos visualmente, representem todos os estados que o componente pode assumir.
  Lembre-se também de alguns casos que podem gerar problemas
  (Por exemplo a estória #3, onde não há nada nada busca mas existem resultados)

  As vantagens de se usar o storybook são inúmeras:
   - Ambiente rápido de desenvolvimento (HMR).
   - Você poderá testar se seus componentes estão se comportando como esperado.
   - Poderá alternar entre os diferentes casos de uso de maneira rápida.
   - Irá gerar um catálogo de componentes que será útil para você no futuro e para sua equipe.
   - Irá garantir que o componente está chamando corretamente as funções (emitindo os eventos necessários e com os parâmetros corretos).

Neste momento, iremos de fato escrever nosso componente, mas antes, vamos falar sobre estilização.

  ## 2.2 Estilização

  Há diversas formas de fazer estilização com React, mas para esta metodologia iremos utilizar a biblioteca `styled-components`
  Primeiro, escreva a **estrutura** do seu componente, sem se preocupar com css:

  ```js
  import Input from '@components/InputWrapper'
  import SearchIcon from '@components/SearchIcon'
  import ResultsCountChip from '@components/ResultsCountChip'
  import styled from 'styled-components'

  const Container = styled.div`
    /*styles aqui */
  `
  const InputWrapper = styled.div`
    /*styles aqui */
  `

  const CancelButton = styled.button`
    /*styles aqui */
  `
  const SearchInput = ({
      // estado
      text,
      resultsCount,
      hovered,
      resultsCount,
      // eventos
      onChangeText,
      onClickCancel,
      onHover,
      onMouseLeave,
  }) => (
    <Container onMouseEnter={onHover} onMouseLeave={onMouseLeave}>
      <SearchIcon active={hovered} />
      <InputWrapper>
        <Input value={text} onChange={e => onChangeText(e.target.value)} />
      </InputWrapper>
      <ResultsCountChip count={resultsCount}/>
      <CancelButton onClick={onClickCancel} resultsCount={resultsCount} />
    </Container>
  )
  ```

  Repare que `Container` e `InputWrapper` são componentes locais, criados utilizado o hoc `Styled` (styled.div é apenas um alias para Styled('div'))
  A partir de agora é só estilizar. Propriedades de css que são calculadas a partir de props são facilmente implementadas utilizando interpolação de strings:
  ```js
  const CancelButton = styled.button`
    height: 50px;
    width: 200px;
    background-color: ${props => props.resultsCount > 0  ? '#F00' : '#0F0'};
  `
  ```

  O único caso em que iremos declarar classes diretamente no css é caso precisemos utilizar o ReactCSSTransitionGroup, que depende de classes globais.
  Fora isso, todo o estilo estará escopado por componente, fazendo com que ele se torne verdadeiramente um componente autosuficiente, que irá se comportar da mesma maneira independente do contexto em que for renderizado.

  Algumas vantagens em utilizar o styled-components
   - É muito mais fácil calcular estilos condicionais desta maneira do que com classes.
   - O estilo fica escopado para o componente, evitando estilos globais.
   - Diferentemente de Inline-Styles, é possível utilizar outros recursos de css como media-queries e pseudo-seletores.

  E como reaproveitaremos estilos?

  Diferentemente do CSS convencional, **não criaremos estilos globais** para reaproveitá-los.
  O que faremos é o reaproveitamnete de Components
  Vamos supor que todos nossos componentes de texto tenham estas propriedades

  ```
   color: red;
   font-size: 16px;
   font-family: Nunito Sans;
  ```

  Ao invez de criarmos uma classe global ou mesmo aplicar estes
  estilos em um escopo ainda mais genérico (body, por exemplo), iremos criar um componente Text

  ```jsx
   const Text = styled.span`
     color: #FFF;
     font-size: 16px;
     font-family: Nunito Sans;
     `
  ```

  E sempre que precisarmos destas características, importaremos o componente `Text` e o utilizaremos.
  E caso precisemos criar componentes que tem como base as características de `Text`,
  porém com customizações? É simples

  ```jsx
   const Title = styled(Text)`
     font-weight: 900;
     font-size: 26px;
    `
  ```

# 3. Routes e Pages
  Neste momento teremos todos os componentes visuais criados, testados e documentados,
  apenas esperando para receber os dados e funções vindas dos containers.

  Porém há alguns passos antes de chegarmos no que propriamente vai ditar a lógica da nossa aplicação
  Nesta fase, criaremos as rotas envolvidas nesta iteração utilizando o `react-router v4`.
  Criaremos também uma outra categoria de componentes: as `Pages`

  Nossas routes irão sempre desenhar Pages, e por mais que a Page tenha apenas um componente (um Container, por exemplo), é importante criar esta camada.

  A vantagem é que, criando esta interface entre o container e a rota, não estaremos acoplando o nosso container com como a rota trata os params da rota, por exemplo
  ```js
  const SearchPage = ({
    match: {
      params: {
        categoryId
      }
    }
  }) => {
    return (
      <SearchContainer
        categoryIdFilter={categoryId}
      />
    )
  }
  ```
  Neste exemplo nosso `SearchContainer` recebe o categoryId, e no nosso caso de uso este categoryId está na rota (por exemplo `/search/:categoryId`)
  O papel do `SearchPage`, neste caso, é ler do match.params (específico do nosso Router) o categoryId e repassar para o componente.

  Sendo assim, nosso 'SearchContainer' não está atrelado a nosso Router e pode ser utilizado em outros contextos, por exemplo com o categoryId vindo de outra fonte de dados


# 4. State shape
  Neste passo, iremos analisar as funcionalidades e casos de uso para determinar o **mínimo** que devemos guardar na nossa store para representar o estado da aplicação.

  Por que **mínimo** ? Não devemos guardar valores computados, pois estes devem ser calculados e repassados para os componentes nos Containers. Veremos mais sobre este assunto no tópico sobre Containers.

  Mas, para exemplificar, vamos imaginar que estamos fazendo uma calculadora que mostra o resultado da multiplicação de dois números, o número **a** e o número **b**
  O resultado da multiplicação, que chamaremos de  **c**, não precisa e não deve ser guardado na nossa store, pois ele é um valor computado a partir de outros valores,
  no caso, **c = a * b**.

  Nesta fase, não devemos nos preocupar com o estado referente à dados buscados do backend (por exemplo os posts do usuário) pois, como veremos a seguir, este tipo
  de estado será gerenciado pelo nosso client de GraphQL.
  Pense apenas em estados locais, como por exemplo qual filtro está selecionado, itens em um carrinho de compras, formulários*, etc

  (\*) *Caso você opte por utilizar o `redux-form` (recomendado), você não precisará se preocupar com estado referente à forms.*

  Lembra-se quando conversamos sobre estados efêmeros, que não importam para o resto da aplicação? Neste momento, tendo uma visão mais geral do nosso State shape, é a hora
  de decidir quais partes do estado da nossa aplicação podem ser gerenciados localmente no componente e não na store do redux.

  Por exemplo, se um componente muda de cor
  ao sofrer um hover e nenhum outro componente do nosso app precisa ficar sabendo disto, encontramos um bom candidato a ser transferido para um estado local.
  Porém não queremos modificar nossos componentes stateless criados na primeira fase (transformando-os em classes, capazes de ter estado).

  Para resolver este problema, utilizaremos a biblioteca `recompose`, criando `enhancers` que darão capacidade a nossos componentes de terem um estado

  ```js
  import { withState, withHandlers, compose } from 'recompose'

  const withHoverState = withState('hovered', 'setHovered', false)
  const withHoverHandlers = ({
    onMouseOver: ({ setHover }) => () => setHover(true)
    onMouseOut: ({ setHover }) => () => setHover(false)
  })
  const enhanceWithHover = compose(withHoverState, withHoverHandlers)
  const SearchInputWithHover = enhanceWithHover(SearchInput)
  ```
  Pronto, agora nosso componente `SearchInputWithHover` é um `SearchInput` mas com capacidades de gerenciar e modificar seu estado em relação ao hover (:


# 5. GraphQL
  Esta é a parte mais fascinante e satisfatória da metodologia, onde veremos nossos componentes recebendo dados reais vindos por exemplo do nosso backend
  Utilizaremos o `Apollo` como client de GraphQL.

  ## 5.1 Queries
  Vamos assumir que recebemos da equipe de design o seguinte componente:
  ![Overview](https://raw.githubusercontent.com/rafaelcorreiapoli/react-metodologia/master/images/TodoList.png)

  Provavelmente, os componentes desta lista (TodoList) que desenhamos na primeira fase estão esperando um array de objetos Todo.
  É bem fácil popular a nossa lista com um client de GraphQL
  ```js
  const query = gql`
    query todos {
      todos {
        id
        name
        done
        tag {
          color
        }
      }
    }
  `

  const ListWithData = graphql(query)(TodoList) //*
  ```
  OBS: *O componente ListWithData é considerado um `Container`, mas não é um `Container` completo para nossa aplicação pois ele não interage com o estado local.

  No tópico sobre containers, iremos completar a metade que falta deste componente para que ele possa, através do connect do redux, receber props que estão guardadas
  no estado local (mapStateToProps) e poder dispachar ações que modifiquem o estado local (mapDispatchToProps)*

  O ideal é que o componente esteja esperando exatamente o mesmo formato de dados que está descrito na nossa schema de GraphQL.

  Caso precise tratar os dados antes de passá-los para o componente, utilize as `queryOptions` do Apollo.
  Por exemplo, supondo que nosso componente TechnologiesList e TechnologiesListItem esperem que a cor esteja na raiz do objeto de tecnologia
  (eles não sabem da existência da key `tag`), podemos tratar os dados da seguinte maneira:
  ```js
  const queryOptions = {
    props: ({ ownProps, data: { loading, todos, refetch } }) => ({
      loading,
      refetch,
      todos: todos && todos.map(todo => ({
        ...todo,
        color: todo.tag.color,
      }))
    }),
  }
  ```

  É recomendado também o uso de `fragments` para evitar repetição. *(Mais sobre este tópico no futuro)*

  O `Apollo` será responsável por trazer para o client todos os dados que necessários e que estejam necessariamente no backend (não são dados de estado local,
  como por exemplo qual filtro está selecionado).

  ## 5.2 Mutations
  Todas as operações (create, remove, update, etc) que precisem ser feitas no servidor serão disparadas pelo client
  através de mutations

  *Mais sobre este assunto em breve...*




  Algumas vantagens de usar GraphQL + Apollo ao invés de modelos mais convenciais como por exemplo REST:
  - Você pode consultar os dados da maneira que quiser, especificando as relations e quais campos quer de cada objeto
  - Caching
  - É declarativo, consistente com todo o resto do nosso ambiente
  - Do lado do backend, não é necessário criar novos endpoints para construção de diferentes visualizações no front
  - Temos uma documentação completa do que pode ser explorado no nosso backend (ferramentas como GraphiQL irão ajudar muito)
  - etc...


# 6. Actions e Reducers

O `Redux`, apesar suas inúmeras vantagens, é muito verboso.
O que podemos fazer para melhorá-lo neste aspecto?

Iremos utilizar a biblioteca `redux-actions` para nos ajudar com os `actionCreators` e `reducers`
Para entender melhor sua utilização, basta ler a [Documentação](https://redux-actions.js.org/docs/introduction/Motivation.html)

*Em construção...*

# 7. Testes
Para escrever nossos testes, recomendo a utilização do [Jest](https://facebook.github.io/jest/docs/en/getting-started.html).

Mas antes, qual a importância de escrever bons testes para o desenvolvimento do projeto?
O motivo mais óbvio é a garantia de que nosso software funciona, mas existem outras vantagens um pouco menos perceptíveis

- Novos integrantes da equipe poderão começar a contribuir para um projeto já existente com mais confiança e velocidade, pois eles terão a garantia (quase sempre) de que seu código não irá quebrar nada em produção.
- Os testes servem como documentação do comportamento de nosso software. Tanto para novos integrantes da equipe quanto para os integrantes que ficaram algum tempo distante de certas partes do código.
- Permitem refatorações de partes do código com mais velocidade (pois você consegue identificar exatamente o que não funciona e aonde arrumar) e confiança (pois você sabe que, se após a refatoração os testes continuarem passando, tudo funciona novamente como esperado)

Por estes e outros motivos, é extremamente importante que seja investido o tempo necessário para implementação de testes em todas as fases do projeto.

Os arquivos de testes devem ser colocados próximos aos arquivos sendo testados, por exemplo:
`src/modules/todos/reducer.js` -> `src/modules/todos/__tests__/reducer.spec.js`
*Em construção...*

### 7.1 Testes unitários
Para testar nossos `reducers`, `selectors` e `actionCreators`, iremos utilizars o [redux-testkit](https://github.com/wix/redux-testkit) para nos ajudar. Ele irá reduzir consideravelmente o boilerplate necessário para testar essa camada da aplicação.
*Em construção...*

### 7.2 Testes de UI
Iremos utilizar o [enzyme](https://github.com/airbnb/enzyme) e o [enzyme-to-json](https://github.com/adriantoine/enzyme-to-json)
*Em construção...*

# 8. Selectors
Os selectors são as funções responsáveis por extrair os dados da nossa store e transformá-los de alguma maneira que é mais interessante para a nossa View.
Vamos imaginar que temos a seguinte estrutura de dados na nossa store:
```js
{
  todos: {
    todo1: {
      text: 'This is the first todo',
      done: false
    },
    todo2: {
      text: 'This is the second todo',
      done: true
    }
  }
}
```

Porém, nosso dumb component `TodosList` precisa receber um `Array` de ToDos para serem renderizados, e não um mapa.
Vamos criar um selector que fará a transformação dos dados de maneira apropriada:
```js
const getTodosMap = state => state.todos
```

```js
const getTodosArrayFromMap = todosMap => Object.keys(todosMap).map(todoKey => ({
  id: todoKey,
  ...todosMap[todoKey]
}))
```

Agora, podemos obter o `Array` de ToDos a partir do `state`:

```js
const todosArray = getTodosArrayFromMap(getTodosMap(state))
```

E esta função será utilizada em nossos containers, que serão explicados com mais detalhe no próximo capítulo.

Mas antes, vamos pensar um pouco sobre um assunto importante: Performance.

Toda vez que alguma mudança de estado causar a rerenderização deste componente (o Container, que usa o selector), a função getTodosArray será chamada novamente e irá consumir os recursos computacionais necessários para sua execução, mesmo que o subset do state utilizado por esta função não tenha mudado e que o resultado da computação seja o mesmo.

Muitas vezes, não temos como evitar a rerenderização do componente, porém, podemos otimizar a computação da função através da [memoization](https://en.wikipedia.org/wiki/Memoization).

A idéia básica é, dada uma função pura (para o mesmo input X temos o mesmo output Y, sem side-effects), podemos fazer um cache do resultado Y, evitando a recomputação da função para um mesmo input X.

O caso descrito acima é um exemplo de onde podemos aplicar este conceito e temos uma biblioteca pronta para implementar memoization em JS:
[reselect](https://github.com/reactjs/reselect)

Com ela, nosso selector ficaria assim:


```js
const getTodosArray = createSelector(
  getTodosMap,
  todosMap => getTodosArrayFromMap(todosMap)
)
```

```js
const todosArray = getTodosArray(state)
```

Agora, caso a função `getTodosMap` retorne um mesmo valor para dois valores de `state`, a função `getTodosArrayFromMap` será chamada apenas uma vez (para a primeira computação, após isto o valor será cacheado e retornado diretamente, em uma espécie de short-circuit)



# 9. Containers
*Em construção...*

# 10. Extras
  ## 10.1. Plugins para o Atom

  - [Nuclide](https://nuclide.io/ "Nuclide")
  - [autocomplete-modules](https://atom.io/packages/autocomplete-modules "autocomplete-modules")
  - [file-icons](https://atom.io/packages/file-icons "file-icons")
  - [linter-eslint](https://atom.io/packages/linter-eslint "linter-eslint")
  - [language-graphql](https://atom.io/packages/language-graphql "language-graphql")
  - [language-babel](https://atom.io/packages/language-babel "language-babel")

  ## 10.2. Plugins para Babel
  - [babel-plugin-module-resolver](https://github.com/tleunen/babel-plugin-module-resolver)

  ## 10.3 Plugins para Webpack
  - [webpack-bundle-analyzer](https://www.npmjs.com/package/webpack-bundle-analyzer)

  ## 10.4 Produtividade
  - [Toggl](https://toggl.com/) - Marcar horas trabalhadas (instalar plugin para o Chrome)
  - [Waffle.io](https://waffle.io/) - Kanban board integrado com o github

  ## 10.5 Deploy
  - [surge.sh](http://surge.sh/)
  - [now](https://zeit.co/now)

  ## 10.6 Plugins para o Chrome
  - [Apollo Client Developer Tools](https://chrome.google.com/webstore/detail/apollo-client-developer-t/jdkknkkbebbapilgoeccciglkfbmbnfm)
  - [React Developer Tools](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi)
  - [React Perf](https://chrome.google.com/webstore/detail/react-perf/hacmcodfllhbnekmghgdlplbdnahmhmm)
  - [Redux Devtools](https://github.com/zalmoxisus/redux-devtools-extension)
  - [Wappalyzer](https://wappalyzer.com/)
  - [What Font?](http://www.chengyinliu.com/whatfont.html)
  - [ColorZilla](https://chrome.google.com/webstore/detail/colorzilla/bhlhnicpbhignbdhedgjhgdocnmhomnp)
  - [CSSViewer](https://chrome.google.com/webstore/detail/cssviewer/ggfgijbpiheegefliciemofobhmofgce)

  ## 10.7 Snippets
    - [Atom Snippets](https://github.com/rafaelcorreiapoli/atom-snippets)
