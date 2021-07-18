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

RTK Query는 웹 애플리케이션에서 데이터를 로딩하는 흔한 케이스를 간단하게하는 진보된 데이터 패칭, 캐싱 툴입니다. RTK Query는 Redux Toolkit core의 위에서 작성되었고, RTK의 API들은 createSlice와 createAsyncThunk를 확장해서 만들어졌습니다. 

RTK Query는 @reduxjs/toolkit 패키지의 추가적인 애드온으로 포함되어져있습니다. Redux Toolkit을 사용해도 RTK Query Api를 사용하지 않아도 되지만 우리는 RTK Query의 데이터 패칭과 캐싱이 많은 사용자들에게 이득이라고 생각합니다. 

### How to Read This Tutorial

이 튜토리얼에서 우리는 React와 Redux Toolkit을 사용하는 걸 가정하지만, 다른 UI layers들과도 사용할 수 있습니다. 예시들은 애플리케이션 코드가 src 폴더에 있는 전형적인 Create-React-App 폴더 구조를 기반으로 하지만, 패턴들은 대부분의 사용하는 프로젝트나 폴더 구조에 적용할 수 있습니다.

## 스토어와 API 서비스 설정하기

RTK Query가 어떻게 작동하는지 보기위해, 기본적인 사용 예시를 만들어 보겠습니다. 이 예시에서는 React를 사용하고 RTK Query에서 자동 생성된 리액트 hooks를 사용하는 것을 가정하겠습니다. 

### API 서비스 생성하기

첫째로, 공개 API인 PokeAPI를 이용해서 서비스 정의를 생성하겠습니다.

{% tabs %}
{% tab title="TypeScript" %}
{% code title="src/services/pokemon.ts" %}
```typescript
// Need to use the React-specific entry point to import createApi
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react'
import { Pokemon } from './types'

// Define a service using a base URL and expected endpoints
export const pokemonApi = createApi({
  reducerPath: 'pokemonApi',
  baseQuery: fetchBaseQuery({ baseUrl: 'https://pokeapi.co/api/v2/' }),
  endpoints: (builder) => ({
    getPokemonByName: builder.query<Pokemon, string>({
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

RTK Query를 사용할때, 전체 API를 보통 한곳에 정의합니다. 이 점은 swr이나 react-query같은 라이브러리들과 가장 많이 다른 점일 것인데, 여기에는 여러가지 이유들이 있습니다. 저희의 관점에서는 여러개의 커스텀 hooks들이 다른 파일들에 있는 것 보다 하나의 중심에 위치하는게 요청 관찰, 캐시 무효화, 설정에서 훨씬 쉽다고 생각합니다. 

{% hint style="success" %}
**팁**

일반적으로, 애플리케이션에 필요한 베이스 URL당 하나의 API 슬라이스를 가져야 합니다. 예시로 만약 사이트에서 /api/posts와 /api/users에서 데이터를 가져와야 한다면 /api를 베이스 URL로 하는 하나의 API 슬라이스를 만들고 posts와 users로 엔드포인트를 나누어야 합니다. 이러면 endpoints와의 관계를 tag로 정의해서 자동 데이터 리패칭이라는 효과적인 장점을 얻을 수 있습니다. 

유지보수적 관점에서, 하나의 API 슬라이스에 엔드포인트들을 포함하면서 엔드포인트들을 여러개의 파일에 나누어 정의하고 싶을 수도 있습니다. 코드 스플리팅에서 어떻게 injectEndpoints 프로퍼티를 사용해서 여러 파일들에서 하나의 API 슬라이스로 API 엔드포인트를 주입할 수 있는지 알아보세요.
{% endhint %}

### 스토어에 서비스 추가하기

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

### 애플리케이션을 Provider로 감싸기

만약 애플리케이션을 Provider로 감싸지 않았다면, 리액트 애플리케이션 컴퍼넌트를 감싸는 리덕스 스토어의 표준 패턴을 사용하세요:

{% tabs %}
{% tab title="TypeScript" %}
{% code title="src/idnex.tsx" %}
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
{% code title="src/store.jsx" %}
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

### 

