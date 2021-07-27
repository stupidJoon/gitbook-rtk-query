# RTK Query 개요

{% hint style="success" %}
**배우게 될 것들**

* RTK Query가 무엇이고 어떤 문제를 해결해 주는지
* RTK Query에 포함되어있는 API들
* 기본 RTK Query 사용법
{% endhint %}



**RTK Query**는 강력한 data fetching, caching 툴입니다. 웹 애플리케이션에서 데이터를 가져오는 단순한 상황을 간단하게 만들어서 **data fetching과 caching 로직을 스스로 작성할 필요가 없도록 만들어졌습니다.**

RTK Query는 **Redux Toolkit 패키지에 포함된 선택적인 추가기능이며** Redux Toolkit에 있는 다른 API와 같이 제공됩니다.

## 동기 <a id="motivation"></a>

웹에서 데이터를 보여주려면 일반적으로 서버에서 데이터를 가져와야 합니다. 또한 해당 데이터를 업데이트하고, 그 업데이트 내용을 서버에 보내고, 클라이언트에 캐시 된 데이터를 서버와 동기화를 유지해야 합니다. 이는 오늘날의 애플리케이션에서 사용되는 다른 동작들을 구현해야 하기 때문에 더욱 복잡해집니다.

* 스피너 UI의 로딩 상태관리
* 동일한 데이터에 대한 중복 요청 방지
* UI가 느리지 않도록 업데이트 최적화
* UI와 상호작용할 때 캐시 상태 관리

Redux core는 개발자에게 실제 로직을 작성하도록 항상 최소한으로 만들었습니다. 즉, Redux는 이러한 유스케이스들을 해결하기 위해 돕는 기능들을 포함하지 않습니다. Redux 공식문서에서 [로딩 상태를 추적하고 결과를 요청하는 요청 라이프사이클](https://redux.js.org/tutorials/fundamentals/part-7-standard-patterns#async-request-status)을 알려줬으며, [Redux Toolkit의 `createAsyncThunk` API](https://redux-toolkit.js.org/api/createAsyncThunk)는 그런 전형적인 패턴을 추상화하도록 만들어졌습니다. 그러나 사용자들을 여전히 로딩 상태나 캐시 된 데이터를 관리하기 위해 많은 리듀서 로직을 작성해야 합니다. 

지난 몇 년 동안 React 커뮤니티는 **데이터 패칭과 캐싱은 상태 관리와 전혀 다른 문제**인지를 깨달았습니다. Redux와 같은 상태 관리 라이브러리를 사용해서 데이터를 캐시 할 수 있지만, 실제 사례에서는 데이터 패칭을 위한 툴을 사용할 가치가 충분히 있습니다.

RTK Query는 Apollo Client, React Query, Urql, SWR과 같은 데이터 패칭을 위한 라이브러리들에서 영감을 얻었지만, API 설계에 고유한 접근 방식을 추가했습니다.

* 데이터 패칭과 캐싱 로직은 Redux Toolkit의 `createSlice`와 `createAsyncThunk` API 위에서 동작합니다.
* Redux Toolkit은 UI 독립적이기 때문에 RTK Query의 기능들은 모든 UI 계층에서 사용할 수 있습니다.
* API 엔드포인트는 인자로부터 쿼리 파리미터를 생성하고 캐싱을 위해 응답을 변환하는 방법을 포함해서 미리 정의됩니다.
* RTK Query는 데이터 패칭 프로세스를 캡슐화해서 `data`와 `isLoading`필드를 컴포넌트에게 제공하고, 컴트가 mount, unmount시 캐시 된 데이터의 라이프타임을 관리하는 React hook을 제공합니다.
* RTK Query는 초기 데이터를 패칭후 웹소켓 메시지를 통해 캐시 업데이트 스트리밍 같은 상황을 위한 "cache entry lifecycle"이라는 옵션을 제공합니다.
* 우리는 OpenAPI와 GraphQL 스키마 API 슬라이스 코드 예시를 제공합니다.
* 마지막으로, RTK Query는 완전히 타입스크립트로 작성되었으며, 완벽한 타입스크립트 경험을 제공합니다. 

## 포함된 항목  <a id="whats-included"></a>

### APIs

RTK Query는 Redux Toolkit 패키지에 포함되어있습니다. 아래의 두 가지 코드로부터 시작할 수 있습니다.

```typescript
import { createApi } from '@reduxjs/toolkit/query'

/* React-specific entry point that automatically generates
   hooks corresponding to the defined endpoints */
import { createApi } from '@reduxjs/toolkit/query/react'
```

RTK Query는 이러한 API들을 포함합니다:

* [`createApi()`](https://redux-toolkit.js.org/rtk-query/api/createApi): RTK Query 기능의 코어입니다. 데이터를 패치하고 변환하는 설정을 포함해서 엔드포인트들에서 어떻게 데이터를 패치하는지 정의할 수 있습니다. 대부분의 케이스에서는 베이스 URL당 하나의 API 슬라이스를 사용해야 합니다.
* [`fetchBaseQuery()`](https://redux-toolkit.js.org/rtk-query/api/fetchBaseQuery) 간단한 요청을 위한 [`fetch`](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)의 래퍼입니다. 대부분의 사용자에게 `createApi`의 `baseQuery`로 권장합니다.
* [`<ApiProvider />`](https://redux-toolkit.js.org/rtk-query/api/ApiProvider): **Redux 스토어를 가지고 있지 않다면** `Provider`로 사용 가능합니다.
* [`setupListeners()`](https://redux-toolkit.js.org/rtk-query/api/setupListeners): `refetchOnMount`와 `refetchOnReconnect`기능을 위해 사용되는 유틸리티입니다.

### 번들 사이즈 <a id="bundle-size"></a>

RTK Query는 고정된 크기를 앱의 번들 사이즈에 추가합니다. RTK Query가 Redux Toolkit과 React-Redux위에서 빌드하기 때문에 얼마나 그것들을 이용하고 있는지에 따라서 사이즈가 달라집니다. 예상되는 min+gzip 번들 사이즈는:

* 만약 RTK를 이미 사용 중이면: RTK Query ~9kb, hooks ~2kb
* 만약 RTK를 사용 중이지 않는다면:
  * Without React: RTK+dependencies+RTK Query 17kb
  * With React: 19kb + React-Redux\(peer dependency\)

엔드포인트 정의를 추가하면 오직 `endpoint` 내의 코드의 크기를 추가하며, 일반적으로 몇 바이트에 불과합니다.

RTK Query의 번들 사이즈는 직접 작성한 데이터 패칭 로직을 제거하면 대부분의 애플리케이션의 사이즈가 개선됩니다.

## 기본 사용법 <a id="basic-usage"></a>

### API Slice를 생성하기 <a id="create-an-api-slice"></a>

RTK Query는 Redux Toolkit 패키지에 포함되어있습니다. 아래의 두 가지 코드로부터 시작할 수 있습니다.

```typescript
import { createApi } from '@reduxjs/toolkit/query'

/* React-specific entry point that automatically generates
   hooks corresponding to the defined endpoints */
import { createApi } from '@reduxjs/toolkit/query/react'
```

React에서 전형적인 사용법은 `createApi`를 import하고 서버의 베이스 URL과 우리가 접근하고 싶은 엔드포인트들의 리스트들인 "API slice"를 정의하는 것에서부터 시작합니다.

```typescript
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

### Store 설정하기 <a id="configure-the-store"></a>

또한 "API slice"는 자동으로 생성된 Redux slice 리듀서와 커스텀 미들웨어를 포함합니다. 둘 다 리덕스 스토어에 추가되어야 합니다:

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

### 컴포넌트에서 Hooks 사용하기 <a id="use-hooks-in-components"></a>

API slice에서 자동 생성된 리액트 hooks를 import하고 필요한 파라미터와 함께 컴포넌트에서 hooks를 호출합니다. RTK Query는 자동으로 컴포넌트가 마운트될때 데이터를 패치하고, 파라미터가 바뀔 때 다시 패치를 해서 `{data, isFetching}` 값들을 제공하고 컴포넌트를 리렌더합니다.

```typescript
import * as React from 'react'
import { useGetPokemonByNameQuery } from './services/pokemon'

export default function App() {
  // 자동으로 데이터를 패치하고 쿼리 값을 가져오는 쿼리 hook을 사용
  const { data, error, isLoading } = useGetPokemonByNameQuery('bulbasaur')
  // 각각의 hooks은 생성된 엔드포인트에서도 접근 가능함
  // const { data, error, isLoading } = pokemonApi.endpoints.getPokemonByName.useQuery('bulbasaur')
  
  // 데이터와 로딩 상태에 따라 UI를 렌더링
}
```

## 추가 정보 <a id="further-information"></a>

\*\*\*\*[**RTK Query 빠른 시작 튜토리얼**](https://redux-toolkit.js.org/tutorials/rtk-query/)에서 어떻게 RTK Query를 Redux Toolkit을 사용하는 프로젝트에서 추가하고, "API slice"를 엔드포인트 정의와 함깨 설정하고, 어떻게 자동으로 생성된 리액트 hooks를 컴퍼트에서 사용하는지 예제와 함께 설명합니다.

\*\*\*\*[**RTK Query 사용 가이드 섹션**](https://redux-toolkit.js.org/rtk-query/usage/queries)에서는 [데이터 쿼링](https://redux-toolkit.js.org/rtk-query/usage/queries), [뮤테이션을 이용해 서버에게 업데이트 보내기](https://redux-toolkit.js.org/rtk-query/usage/mutations), [캐시 업데이트를 스트리밍하기](https://redux-toolkit.js.org/rtk-query/usage/streaming-updates) 같은 토픽을 설명합니다. 

\*\*\*\*[**예시 페이지**](https://redux-toolkit.js.org/rtk-query/usage/examples)에서는 [GraphQL과 함께 쿼리 만들기](https://redux-toolkit.js.org/rtk-query/usage/examples#react-with-graphql), [인증](https://redux-toolkit.js.org/rtk-query/usage/examples#authentication) 그리고 [Svelte같은 다른 UI 라이브러리와 함께 RTK Query를 사용하기](https://redux-toolkit.js.org/rtk-query/usage/examples#svelte) 같은 주제를 보여주는 실행 가능한 CodeSandboxes를 가지고 있습니다.

