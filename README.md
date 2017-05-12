# Metodologia projetos React
Uma metodologia que estou desenvolvendo com base em aprendizados nos diversos projetos que realizei.

As estratégias seguidas neste modelo também são fruto de muitas pesquisas e intenso uso de bibliotecas modernas, seguindo todas as melhores práticas de desenvolvimento web moderno.

Os passos a seguir fazem parte de um ciclo de desenvolvimento e podem ser aplicados de uma vez para o projeto todo ou subdivindo o projeto em pequenas entregas (iterações)

# 1. Componentes Visuais

## 1.1 Divisão de componentes visuais (stateless/dumb components)
  Tendo em mãos os wireframes que contenham todos os elementos e funcionalidades que serão desenvolvidos nesta iteração, iremos realizar o trabalho de dividir tudo em componentes visuais.
  É nesta hora que vamos estabelecer componentes como:
  - SearchInput
  - TodoList
  - TodoListItem
  - UserAvatar
  - FavoriteStar
  - etc...

  Procuraremos estabelecer um nível de granularidade que evite completamente repetições (DRY) mas ao mesmo tempo tenha foco em praticidade.
  Por exemplo, nosso input de busca pode ter diversos elementos (um input *(1)*, um icone de search *(2)*, um chip com a quantidade de resultados *(3)* e um botão de cancel *(4)*)
  Cada um destes (*1*, *2*, *3*, *4*), apesar de bem granulares, merecem ser um componente. Com isto, conseguiremos reutilizar estes mesmos elementos em outros contextos, caso seja preciso.
  Mas não deixaremos de criar o componente SearchInput, que é um componente que desenha e organiza os subcomponente *1*, *2*, *3*, *4*,
  nos proporcionando praticidade quando formos de fato construir nosso sistema de busca.

## 1.2 Stateless!
  Todos estes componentes deverão ser stateless. Para garantir isto, iremos utilizar a sintaxe React Functional Stateless Component:
  ```js
    const SearchInput = () => (
      ...
    )
  ```
  Isto irá nos proporcionar algumas vantagens:
  - Iremos garantir que os componentes desenvolvidos nesta etapa não terão estado próprio, nos forçando a guardar o estado na nossa Store do Redux (single source of truth)
  - A sintaxe é menos verbosa que ES6 classes
  - Componentes stateless são mais fáceis de serem compreendidos e utilizados

## 1.2 Props
  É neste momento também que estabeleceremo, olhando para nossas funcionalidades e wireframes, quais Props cada stateless component irá receber.
  Neste momento, não economize em props: Absolutamente *todos*
  os parâmetros que descrevem o estado do componente e os eventos que ele pode emitir devem ser levados
  em consideração. Até mesmo estados "superficiais" como por exemplo Hover (Veremos nos próximos tópicos como otimizá-los)
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
  - Servirão de documentação para o componente. Quando revisitarmos este componente no futuro ou se outro membro do time quiser usá-lo, ficará fácil de entender o que ele precisa receber e o que faz.
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
  - Sempre iremos seguir esta ordem na desestruturação do objeto e declaração das propTypes:
   Primeiro propriedades de estado do componente e depois callbacks (funções)

  ### 1.2.2 defaultProps
  *Todas* as props que não forem required devem especificar uma defaultProp
  Utilize este recurso para tornar mais prático o uso de seus componentes
  ```js
  SearchInput.propTypes = {
      resultsCount: 0
  }
  ```
---

# 2. Storybook
  Até o momento, não escrevemos o código dos componentes propriamente ditos, apenas estabelecemos
  a divisão lógica entre os diversos componentes e quais parâmetros cada um pode receber.
  Contudo, já temos em mãos como esperamos que o componente se comporte quando submetido a diferentes parâmetros
  (pois temos em mãos os wireframes), então é uma ótima oportunidade para aplicar o `TDD`
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
   - Ambiente rápido de desenvolvimento (HMR)
   - Você poderá testar se seus componentes estão se comportando como esperado
   - Poderá alternar entre os diferentes casos de uso de maneira rápida
   - Irá gerar um catálogo de componentes que será útil para você no futuro e para sua equipe
   - Irá garantir que o componente está chamando corretamente as funções (emitindo os eventos necessários e com os parâmetros corretos)

Daqui pra baixo é rascunho:
## Layout
 - Analisar layout e definir componentes visuais
 - Definir props dos componentes para eles assumirem os estados

## Storybook
 -> TDD: Escreva as estórias e teste visualmente os componentes
 -> Escrever os compoenntes

 ## Desenhar o estado
  - Escrever JSOn que representa o seu state
  - Entender quais são os estados que podem ser guardados localmente (não importam pro resto do app, ex: hover)
  - Nujnca guardar estado computado


## Routes
  - Criar rotas e paginas

## Container
  mapStateToProps: {
    result : state.a * state.b
  }
  mapDispatchToProps: {

  }
  - Estados derivados: usar memoization (por exemplo lib reselect)
  - ORGASMO!
