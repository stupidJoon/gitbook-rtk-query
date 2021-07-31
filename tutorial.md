# RTK Query 시작하기

{% hint style="success" %}
**배우게 될 것들**

* Redux Toolkit의 RTK Query 데이터 패칭 기능을 어떻게 설정하고 사용하는지
{% endhint %}

{% hint style="info" %}
**배우기 위해 필요한 것들**

* [Redux의 용어와 개념](https://redux.js.org/tutorials/fundamentals/part-2-concepts-data-flow) 이해
{% endhint %}

## Introduction

Redux Toolkit Query 튜토리얼에 오신걸 환영합니다! **이 튜토리얼은 Redux Toolkit의 "RTK Query" 데이터 패칭 기능을 소개하고 올바르게 사용하는 방법에 대해서 간단하게 소개합니다.** 

RTK Query는 웹 애플리케이션에서 데이터를 로딩하는 흔한 케이스를 간단하게하는 진보된 데이터 패칭, 캐싱 툴입니다. RTK Query는 Redux Toolkit core의 위에서 작성되었고, RTK의 API들은 [`createSlice`](https://redux-toolkit.js.org/api/createSlice)와 [`createAsyncThunk`](https://redux-toolkit.js.org/api/createAsyncThunk)를 확장해서 만들어졌습니다. 

RTK Query는 `@reduxjs/toolkit` 패키지의 추가적인 애드온으로 포함되어져있습니다. Redux Toolkit을 사용해도 RTK Query Api를 사용하지 않아도 되지만 우리는 RTK Query의 데이터 패칭과 캐싱이 많은 사용자들에게 택을 가져다 줄것이라고 생각합니다

### How to Read This Tutorial

이 튜토리얼에서 우리는 React와 Redux Toolkit을 사용하는 걸 가정하지만, 다른 UI layers들과도 사용할 수 있습니다. 예시들은 애플리케이션 코드가 `src` 폴더에 있는 [전형적인 Create-React-App 폴더 구조](https://create-react-app.dev/docs/folder-structure)를 기반으로 하지만, 패턴들은 대부분의 사용하는 프로젝트나 폴더 구조에 적용할 수 있습니다.

## 스토어와 API 서비스 설정하기 <a id="setting-up-your-store-and-api-service"></a>

RTK Query가 어떻게 작동하는지 보기위해, 기본적인 사용 예시를 만들어 보겠습니다. 이 예시에서는 React를 사용하고 RTK Query에서 자동 생성된 리액트 hooks를 사용하는 것을 가정하겠습니다. 

### API 서비스 생성하기 <a id="create-an-api-service"></a>

첫째로, 공개 API인 [PokeAPI](https://pokeapi.co/)를 이용해서 서비스 정의를 생성하겠습니다.

{% tabs %}
{% tab title="TypeScript" %}
{% code title="src/services/pokemon.ts" %}
```typescript
// createApi를 import하기위해 React 엔트리 포인트 사용
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react'
import { Pokemon } from './types'

// base URL과 엔드포인트들로 서비스 정의
export const pokemonApi = createApi({
  reducerPath: 'pokemonApi',
  baseQuery: fetchBaseQuery({ baseUrl: 'https://pokeapi.co/api/v2/' }),
  endpoints: (builder) => ({
    getPokemonByName: builder.query<Pokemon, string>({
      query: (name) => `pokemon/${name}`,
    }),
  }),
})

// 정의된 엔드포인트에서 자동으로 생성된 훅을 함수형 컴포넌트에서 사용하기 위해 export
export const { useGetPokemonByNameQuery } = pokemonApi
```
{% endcode %}
{% endtab %}

{% tab title="Javascript" %}
{% code title="src/services/pokemon.js" %}
```javascript
// Need to use the React-specific entry point to import createApi
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react'

// Define a service using a base URL and expected endpoints
export const pokemonApi = createApi({
  reducerPath: 'pokemonApi',
  baseQuery: fetchBaseQuery({ baseUrl: 'https://pokeapi.co/api/v2/' }),
  endpoints: (builder) => ({
    getPokemonByName: builder.query({
      query: (name) => `pokemon/${name}`,
    }),
  }),
})

// Export hooks for usage in functional components, which are
// auto-generated based on the defined endpoints
export const { useGetPokemonByNameQuery } = pokemonApi
```
{% endcode %}
{% endtab %}
{% endtabs %}

RTK Query를 사용할때, 전체 API를 보통 한곳에 정의합니다. 이 점은 `swr`이나 `react-query`같은 라이브러리들과 가장 많이 다른 점일 것인데, 여기에는 여러가지 이유들이 있습니다. 저희의 관점에서는 여러개의 커스텀 hooks들이 다른 파일들에 있는 것 보다 한곳에 위치하는게 요청, 캐시 무효화, 공통 앱 설정을 관리하기가 더욱 쉽다고 생각합니다.

{% hint style="success" %}
**팁**

일반적으로, 애플리케이션에 필요한 베이스 URL당 하나의 API 슬라이스를 가져야 합니다. 예시로 만약 사이트에서 `/api/posts`와 `/api/users`에서 데이터를 가져와야 한다면 `/api`를 베이스 URL로 하는 하나의 API 슬라이스를 만들고 `posts`와 `users`로 엔드포인트를 나누어야 합니다. 이러면 endpoints와의 관계를 tag로 정의해서 [자동 데이터 리패칭](https://redux-toolkit.js.org/rtk-query/usage/automated-refetching) 기능을 효과적으로 활용할 수 있습니다. 

유지보수적 관점에서, 하나의 API 슬라이스에 엔드포인트들을 포함하면서 엔드포인트들을 여러개의 파일에 나누어 정의하고 싶을 수도 있습니다. [코드 스플리팅](https://redux-toolkit.js.org/rtk-query/usage/code-splitting)에서 어떻게 `injectEndpoints` 프로퍼티를 사용해서 여러 파일들에서 하나의 API 슬라이스로 API 엔드포인트를 주입할 수 있는지 알아보세요.
{% endhint %}

### 스토어에 서비스 추가하기 <a id="add-the-service-to-your-store"></a>

RTK Query는 리덕스 루트 리듀서에 추가해야하는 "슬라이스 리듀서"와 데이터 패칭을 위한 커스텀 미들웨어를 생성합니다. 둘 다 리덕스 스토어에 추가해야 합니다. 

{% tabs %}
{% tab title="TypeScript" %}
{% code title="src/store.ts" %}
```typescript
import { configureStore } from '@reduxjs/toolkit'
// Or from '@reduxjs/toolkit/query/react'
import { setupListeners } from '@reduxjs/toolkit/query'
import { pokemonApi } from './services/pokemon'

export const store = configureStore({
  reducer: {
    // 특정 top-level slice에서 생성된 리듀서를 추가
    [pokemonApi.reducerPath]: pokemonApi.reducer,
  },
  // 캐싱, 요청 취소, 폴링 등등 유용한 rtk-query의 기능들을 위한 api 미들웨어 추가
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(pokemonApi.middleware),
})

// 옵셔널, refetchOnFocus/refetchOnReconnect 기능을 위해 필요함
// setupListeners 문서를 참고 - 커스텀을 위한 옵셔널 콜백을 2번째 인자로 받음
setupListeners(store.dispatch)
```
{% endcode %}
{% endtab %}

{% tab title="Javascript" %}
{% code title="src/store.js" %}
```javascript
import { configureStore } from '@reduxjs/toolkit'
// Or from '@reduxjs/toolkit/query/react'
import { setupListeners } from '@reduxjs/toolkit/query'
import { pokemonApi } from './services/pokemon'

export const store = configureStore({
  reducer: {
    // Add the generated reducer as a specific top-level slice
    [pokemonApi.reducerPath]: pokemonApi.reducer,
  },
  // Adding the api middleware enables caching, invalidation, polling,
  // and other useful features of `rtk-query`.
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(pokemonApi.middleware),
})

// optional, but required for refetchOnFocus/refetchOnReconnect behaviors
// see `setupListeners` docs - takes an optional callback as the 2nd arg for customization
setupListeners(store.dispatch)
```
{% endcode %}
{% endtab %}
{% endtabs %}

### 애플리케이션을 Provider로 감싸기 <a id="wrap-your-application-with-the-provider"></a>

만약 애플리케이션을 Provider로 감싸지 않았다면, 리액트 애플리케이션 컴퍼넌트를 감싸는 리덕스 스토어의 표준 패턴을 사용하세요:

{% tabs %}
{% tab title="TypeScript" %}
{% code title="src/index.tsx" %}
```jsx
import * as React from 'react'
import { render } from 'react-dom'
import { Provider } from 'react-redux'

import App from './App'
import store from './app/store'

const rootElement = document.getElementById('root')
render(
  <Provider store={store}>
    <App />
  </Provider>,
  rootElement
)
```
{% endcode %}
{% endtab %}

{% tab title="Javascript" %}
{% code title="src/index.jsx" %}
```jsx
import * as React from 'react'
import { render } from 'react-dom'
import { Provider } from 'react-redux'

import App from './App'
import store from './app/store'

const rootElement = document.getElementById('root')
render(
  <Provider store={store}>
    <App />
  </Provider>,
  rootElement
)
```
{% endcode %}
{% endtab %}
{% endtabs %}

## 컴포넌트에서 쿼리 사용하기 <a id="use-the-query-in-a-component"></a>

서비스를 정의하면 hooks를 가져와서 요청을 생성할 수 있습니다.

{% tabs %}
{% tab title="TypeScript" %}
{% code title="src/App.tsx" %}
```jsx
import * as React from 'react'
import { useGetPokemonByNameQuery } from './services/pokemon'

export default function App() {
  // 자동으로 데이터를 패치하고 쿼리 값을 가져오는 쿼리 hook을 사용
  const { data, error, isLoading } = useGetPokemonByNameQuery('bulbasaur')
  // 각각의 hooks은 생성된 엔드포인트에서도 접근 가능함
  // const { data, error, isLoading } = pokemonApi.endpoints.getPokemonByName.useQuery('bulbasaur')

  return (
    <div className="App">
      {error ? (
        <>Oh no, there was an error</>
      ) : isLoading ? (
        <>Loading...</>
      ) : data ? (
        <>
          <h3>{data.species.name}</h3>
          <img src={data.sprites.front_shiny} alt={data.species.name} />
        </>
      ) : null}
    </div>
  )
}
```
{% endcode %}
{% endtab %}

{% tab title="Javascript" %}
{% code title="src/App.jsx" %}
```jsx
import * as React from 'react'
import { useGetPokemonByNameQuery } from './services/pokemon'

export default function App() {
  // Using a query hook automatically fetches data and returns query values
  const { data, error, isLoading } = useGetPokemonByNameQuery('bulbasaur')
  // Individual hooks are also accessible under the generated endpoints:
  // const { data, error, isLoading } = pokemonApi.endpoints.getPokemonByName.useQuery('bulbasaur')

  return (
    <div className="App">
      {error ? (
        <>Oh no, there was an error</>
      ) : isLoading ? (
        <>Loading...</>
      ) : data ? (
        <>
          <h3>{data.species.name}</h3>
          <img src={data.sprites.front_shiny} alt={data.species.name} />
        </>
      ) : null}
    </div>
  )
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

요청을 생성할 때 여러 방법으로 상태를 추적할 수 있습니다. `data`, `status`, `error`로 알맞는 UI를 렌더링할 수 있습니다. 또한 `useQuery`는 유틸리티 불리언 값인 `isLoading`, `isFetching`, `isSuccess`, `isError` 로 가장 최근의 요청에 대한 값을 제공합니다. 

#### 기본 예시

{% embed url="https://codesandbox.io/embed/github/reduxjs/redux-toolkit/tree/master/examples/query/react/basic?fontsize=9&hidenavigation=1&theme=dark" %}

흥미롭네요... 하지만 만약 여러개의 포켓몬을 한번에 보여주고 싶거나 여러개의 컴포넌트들이 같은 포켓몬을 불러오면 어쩌죠?

#### 고급 예시

RTK Query는 어떤 컴포넌트든 같은 쿼리를 구독하면 항상 같은 데이터를 사용할 수 있도록 보장합니다. RTK Query는 자동으로 중복 요청을 제거하기 때문에 in-flight 요청과 성능 최적화에 대해서 걱정할 필요가 없습니다. 아래의 샌드박스를 시작해보죠. 브라우저 데브툴의 네트워크 탭을 확인해보세요. 4개의 구독된 컴포넌트가 있음에도 3개의 요청을 볼 수 있습니다. `bulbasur`는 오직 하나만 요청하고 두개의 컴포넌트의 로딩 상태는 동기화되어있습니다. 재미를 위해, 드롭다운의 값을 `Off`에서 `1s`로 바꿔서 쿼리가 리렌더링할때 행동을 봅시다.

{% embed url="https://codesandbox.io/embed/github/reduxjs/redux-toolkit/tree/master/examples/query/react/advanced?fontsize=14&hidenavigation=1&theme=dark" %}



