- title : Programação Funcional e Javascript
- description : Diminuindo a Dor de Cabeça no Frontend
- author : Lucas Reis
- theme : night
- transition : slide

***

- data-background : linear-gradient(to bottom right, #FF5E3A, #FF2A68)

### Functional Javascript:
## Diminuindo a Dor de Cabeça no Frontend
#### Lucas Reis
@iamlucasreis

***

### Quem sou eu?

* Líder do time de Frontend do Checkout das marcas **Americanas, Submarino, Shoptime e Soubarato**
* Time composto por 5 desenvolvedores + 1 Product Owner + 1 part-time Quality Assurance
* 3 projetos *white label*: Checkout + Login + Finance
* ES5, Babel, Angular, React, Grunt, Gulp, Brunch...
* + ou - 100 mil linhas de javascript

***

### O que você pensa quando pensa em 100 mil linhas de javascript?

***

<img src="images/thisisfine.png" alt="This Is Fine" style="width: 800px;"/>

***

- data-background : linear-gradient(to bottom right, #1D77EF, #81F3FD)

Quais os problemas?

## Confiabilidade + Segurança

***

## Confiabilidade
Meu código faz o que eu quero que ele faça

***

## Segurança
Ausência de acidentes que causam prejuízos inesperados e inaceitáveis

***

- data-background : linear-gradient(to bottom right, #FF5E3A, #FF2A68)

Vamos começar a atacar esses problemas num dos níveis mais fundamentais da programação:

## Function Design

***

## Existem apenas dois tipos de inputs e outputs para qualquer função

***

Primeiro tipo:
## Convencionais

```js
const sum = (n, m) => n + m
```

Os parâmetros são o *input* e o retorno é o *output* da função.

E só.

***

Segundo tipo:
## Escondidos

```js
function processNext() {
  let msg = InboxQueue.getNext()

  transformMessage(msg);

  let response;

  if (msg.type === 'TYPE_A') {
    response = axios(url, { params: msg })
  } else {
    let params = { some: 'parameter' }
    response = axios(url, { params })
  }

  response.then(() => processed = true)
}
```

***

Problemas de um código com muitos inputs e outputs escondidos:

## Difícil de entender
## Difícil de testar

***

<img src="images/hidden.gif" alt="Fear Of Hidden" style="width: 800px;"/>

Ou seja, medo.

***

- data-background : linear-gradient(to bottom right, #1D77EF, #81F3FD)

### Vamos refatorar!

***

```js
// ...
let msg = InboxQueue.getNext()
// ...
```
:
```js
const getNext = queue => queue.getNext()
```

```js
// simple test:
it('should pop from queue', () => {
  const fakeQueue = { getNext: () => 'something' }

  const result = getNext(fakeQueue)

  assert.equal(result, 'something')
})
```
*"pratos limpos"*

---

```js
function processNext() {
  let msg = InboxQueue.getNext()

  transformMessage(msg);

  let response;

  if (msg.type === 'TYPE_A') {
    response = axios(url, { params: msg })
  } else {
    let params = { some: 'parameter' }
    response = axios(url, { params })
  }

  response.then(() => processed = true)
}
```

***

```js
// ...
transformMessage(msg);
// ...
```
:
```js
const pureTransform = msg => {
  let newMsg = { ...msg }
  transformMessage(newMsg);
  return newMsg
}
```

"*se uma árvore cai na floresta...*"

---

```js
function processNext() {
  let msg = InboxQueue.getNext()

  transformMessage(msg);

  let response;

  if (msg.type === 'TYPE_A') {
    response = axios(url, { params: msg })
  } else {
    let params = { some: 'parameter' }
    response = axios(url, { params })
  }

  response.then(() => processed = true)
}
```

***

```js
// ...
let response;

if (msg.type === 'TYPE_A') {
  response = axios(url, { params: msg })
} else {
  let params = { some: 'parameter' }
  response = axios(url, { params })
}
// ...
```
:
```js
const buildParams = msg => msg.type === 'TYPE_A'
  ? { params: msg}
  : { params: { some: 'parameter' } }
```

*"joga no simples*"

---

```js
function processNext() {
  let msg = InboxQueue.getNext()

  transformMessage(msg);

  let response;

  if (msg.type === 'TYPE_A') {
    response = axios(url, { params: msg })
  } else {
    let params = { some: 'parameter' }
    response = axios(url, { params })
  }

  response.then(() => processed = true)
}
```

***

```js
const apiRequest = (requestFn, url) =>
  params => requestFn(url, params)
```

*"me engana que eu gosto"*

---

```js
function processNext() {
  let msg = InboxQueue.getNext()

  transformMessage(msg);

  let response;

  if (msg.type === 'TYPE_A') {
    response = axios(url, { params: msg })
  } else {
    let params = { some: 'parameter' }
    response = axios(url, { params })
  }

  response.then(() => processed = true)
}
```

***

Facilmente conseguimos fazer injeção de dependências:

```js
// no código da aplicação:
const getItems = apiRequest(axios, 'http://my-url.com')
const items = getItems({ tag: 'some-tag' })
```
```js
// no código de teste:
const fakeAxios = (url, params) => {
  assert.ok(url === 'http://my-url.com')
  assert.ok(params.tag === 'some-tag')
  return Promise.resolve({ status: 200 })
}

const getItems = apiRequest(fakeAxios, 'http://my-url.com')
const items = getItems({ tag: 'some-tag' })
item.then(res => assert.ok(res.status === 200))
```

---

```js
function processNext() {
  let msg = InboxQueue.getNext()

  transformMessage(msg);

  let response;

  if (msg.type === 'TYPE_A') {
    response = axios(url, { params: msg })
  } else {
    let params = { some: 'parameter' }
    response = axios(url, { params })
  }

  response.then(() => processed = true)
}
```

***

### Versão final mais simples!

```js
const processNext = (requestFn, url) =>
  queue => {
    const msg = getNext(queue)
    const transformed = pureTransformMessage(msg)
    const pars = buildParams(transformed)
    const response = apiRequest(requestFn, url)(pars)
    return response.then(() => true)
  }
```

```js
const productionProcessNext =
  processNext(axios, 'http://my-url.com')

processed = productionProcessNext(InboxQueue)
```

---

```js
function processNext() {
  let msg = InboxQueue.getNext()

  transformMessage(msg);

  let response;

  if (msg.type === 'TYPE_A') {
    response = axios(url, { params: msg })
  } else {
    let params = { some: 'parameter' }
    response = axios(url, { params })
  }

  response.then(() => processed = true)
}
```

***

### Alternativa: composição de funções

```js
const then = fn => promise => promise.then(fn)

const always = x => () => x
```

```js
const processNext = (requestFn, url) =>
  queue => compose(
    then(always(true))
    apiRequest(requestFn, url),
    buildParams,
    pureTransformMessage,
    getNext
  )(queue)
```

***

```js
it('should work!', function () {

  const requestFn = (requestFn, url) =>
    params => Promise.resolve({ status: 200 })

  const queue =
    { getNext: () => { some: 'message' } }

  const processNextFn =
    processNext(requestFn, 'http://my-url.com')

  assert.ok(processNextFn(queue))
})
```

***

![Happy](images/happy.gif)

Felicidade!

***

- data-background : linear-gradient(to bottom right, #FF5E3A, #FF2A68)

Mais um benefício:

### Funções feitas dessa maneira evoluem mais facilmente

***

*Precisamos saber se os customers têm o mesmo nome*

```js
const hasSameName = (customerA, customerB) =>
  customerA.name === customerB.name
```

***

*Precisamos saber também se os customers também têm o mesmo telefone*

```js
const hasSameName = (customerA, customerB) =>
  customerA.name === customerB.name

const hasSamePhone = (customerA, customerB) =>
  customerA.phone === customerB.phone
```

***

```js
const hasSameProp = (property, customerA, customerB) =>
  customerA[property] === customerB[property]

const hasSameName = (customerA, customerB) =>
  hasSameProp('name', customerA, customerB)

const hasSamePhone = (customerA, customerB) =>
  hasSameProp('phone', customerA, customerB)
```

***

Podemos melhorar **hasSameProp**:

```js
const prop = (property, obj) =>
  typeof obj === 'object'
    ? obj[property]
    : undefined

const hasSameProp = (property, customerA, customerB) =>
  prop(property, customerA) === prop(property, customerB)
```

Mais segurança!

***

**Curry** torna o código mais conciso:

```js
import curry from 'curry'

const hasSameProp = curry((property, customerA, customerB) =>
  prop(property, customerA) === prop(property, customerB))

const hasSameName = hasSameProp('name')

const hasSamePhone = hasSameProp('phone')
```

***

*Também precisamos saber se tem os mesmos endereços...*

```js
import curry from 'curry'
import deepEqual from 'deep-equal'

const shallowEqual = (x, y) => x === y

const hasSameProp = curry((eqFn, property, customerA, customerB) => {
  const propA = prop(property, customerA)
  const propB = prop(property, customerB)
  return eqFn(propA, propB)
}

const hasSameName = hasSameProp(shallowEqual, 'name')

const hasSamePhone = hasSameProp(shallowEqual, 'phone')

const hasSameAddresses = hasSameProp(deepEqual, 'addresses')
```

***

*...e também se eles gastaram a mesma quantidade em compras...?* (1/2)

```js
const totalSpent = customer => compose(
  sum,
  map(prop('total')),
  propOr([], 'orders')
)(customer)

const hasSameValues = curry((eqFn, customerFn, customerA, customerB) => {
  const valueA = customerFn(customerA)
  const valueB = customerFn(customerB)
  return eqFn(valueA, valueB)
}
```

***

*...e também se eles gastaram a mesma quantidade em compras...?* (2/2)

```js
const hasSameName =
  hasSameValues(shallowEqual, prop('name'))

const hasSamePhone =
  hasSameValues(shallowEqual, prop('phone'))

const hasSameAddresses =
  hasSameValues(deepEqual, prop('addresses'))

const hasSameTotal =
  hasSameValues(shallowEqual, totalSpent)
```

***

- data-background : linear-gradient(to bottom right, #1D77EF, #81F3FD)

Conclusão:

## Programação funcional torna sua vida mais simples!

***

<img src="images/simple.gif" alt="Simple" style="width: 800px;"/>

***

Eu quero me beneficar das vantagens da programação funcional!

### Ótimo!

***

- data-background : linear-gradient(to bottom right, #FF5E3A, #FF2A68)

Continuando no mundo Javascript:

### Redux

* Um objeto como estado da aplicação
* Interações do usuário disparam *actions*
* Uma função pura chamada *reducer* recebe a action e o estado anterior e gera um novo estado
* Tem um ecossistema gigante, está virando o novo padrão

***

### Redux

![Lion O](images/liono.jpg)

***

- data-background : linear-gradient(to bottom right, #1D77EF, #81F3FD)

Continuando no mundo Javascript:

### CycleJS

* Utiliza RxJS como base
* Interações do usuário são um *stream* de dados
* A UI é uma função pura desses streams
* O RxJS é um paradigma extremamente poderoso, que facilita trabalhar com concorrência, e é portado para várias linguagens

***

### CycleJS

![Tygra](images/tygra.jpg)

***

- data-background : linear-gradient(to bottom right, #FF5E3A, #FF2A68)

Saindo do mundo Javascript...

### ClojureScript

* Linguagem super simples, fácil de entender
* Imutabilidade por default
* Macros dão uma flexibilidade para a linguagem, que permanece evoluindo
* Possui boa parte dos frameworks que influenciaram o mundo Javascript hoje: Om, Reagent e Quiescent

***

### ClojureScript

```cljs
(def process-next [request-fn url queue]
  (-> queue
      get-next
      pureTransformMessage
      build-params
      (api-request request-fn url))
```

***

### ClojureScript

![Cheetara](images/cheetara.jpg)

***

- data-background : linear-gradient(to bottom right, #1D77EF, #81F3FD)

Saindo do mundo Javascript...

### Elm

* Toda função é pura, sempre. Nem adianta tentar
* Compilação muito poderosa, ajuda muito o desenvolvedor - *não há erros em runtime!!!*
* É uma linguagem *e* um framework, que também influenciou o mundo Javascript de hoje

***

### Elm

```elm
processNext requestFn url queue =
  queue
    |> getNext
    |> pureTransformMessage
    |> buildParams
    |> apiRequest requestFn url
```

***

### Elm

![Panthro](images/panthro.jpg)

***

Ok, mas eu quero continuar trabalhando com orientação a objetos e acho que programação funcional é só modinha hipster!

***

### Orientação a objetos

![Snarf](images/snarf.jpg)

**;)**

***

### Muito obrigado!

lucas.reis@b2wdigital.com

@iamlucasmreis

http://lucasmreis.github.io/blog/

https://github.com/lucasmreis

***

<img src="images/trabalhe-conosco.png" alt="Trabalhe Conosco" style="width: 800px;"/>

***

Referências

* Confiabilidade + Segurança: **Nancy Leveson** - https://mitpress.mit.edu/books/engineering-safer-world
* Dois tipos de inputs e outputs: **Kris Jenkins** - http://blog.jenkster.com/2015/12/what-is-functional-programming.html
* Difícil de entender, difícil de testar: **Ben Moseley + Peter Marks** - http://shaffner.us/cs/papers/tarpit.pdf
* Isolar side-effects: **Brian Lonsdorf + Joe Nelson** - https://frontendmasters.com/courses/functional-javascript/
