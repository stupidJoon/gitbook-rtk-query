# 타입스크립트와 사용하기

{% hint style="success" %}
**배우게 될 것들**

* 타입스크립트와 함께 다양한 RTK Query API를 사용하는 방법
{% endhint %}

## 소개

RTK Query는 Redux Toolkit 패키지의 나머지 부분과 마찬가지로 타입스크립트로 작성됐으며 API는 타입스크립트 애플리케이션에서 원활하게 사용할 수 있도록 설계되었습니다. 

이 페이지는 RTK Query의 API들을 Typescript와 함께 올바르게 사용하는 방법에 대해 자세히 설명합니다.

{% hint style="info" %}
**정보**

**저희는 RTK Query 최상의 결과를 위해서 타입스크립트 4.1+ 버전을 사용하는 것을 추천드립니다.** 

이 페이지에 설명되어 있지 않는 타입 관련 문제가 발생하는 경우 이슈를 열어주세요. 
{% endhint %}

## createApi

### auto-generated 리액트 Hooks 사용하기

리액트에서 RTK Query는 쿼리와 엔드포인트들을 위해 자동으로 생성되는 리액트 hooks인 createApi를 exports하는것에서부터 시작됩니다. 

타입스크립트 사용자를 위한 자동생성 리액트 hooks를 사용할려면 TS4.1+ 버전을 사용해야 합니다. 

{% tabs %}
{% tab title="TypeScript" %}
```typescript
// Need to use the React-specific entry point to allow generating React hooks
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react'
import type { Pokemon } from './types'

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

// Export hooks for usage in function components, which are
// auto-generated based on the defined endpoints
export const { useGetPokemonByNameQuery } = pokemonApi
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
// Need to use the React-specific entry point to allow generating React hooks
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

// Export hooks for usage in function components, which are
// auto-generated based on the defined endpoints
export const { useGetPokemonByNameQuery } = pokemonApi
```
{% endtab %}
{% endtabs %}

예전 TS 버전을 위해서 `api.endpoints.[endpointName].useQuery/useMutation`로 같을 hooks을 사용할 수 있습니다. 

{% tabs %}
{% tab title="TypeScript" %}
{% code title="api hooks 직접 접근하기" %}
```typescript
import { pokemonApi } from './pokemon'

const useGetPokemonByNameQuery = pokemonApi.endpoints.getPokemonByName.useQuery
```
{% endcode %}
{% endtab %}

{% tab title="JavaScript" %}
{% code title="api hooks 직접 접근하기" %}
```javascript
import { pokemonApi } from './pokemon'

const useGetPokemonByNameQuery = pokemonApi.endpoints.getPokemonByName.useQuery
```
{% endcode %}
{% endtab %}
{% endtabs %}

### baseQuery 작성하기

RTK Query에서 BaseQueryFn 타입을 export하면 커스텀 baseQuery를 작성할 수 있습니다. 

{% code title="Base Query signature" %}
```javascript
export type BaseQueryFn<
  Args = any,
  Result = unknown,
  Error = unknown,
  DefinitionExtraOptions = {},
  Meta = {}
> = (
  args: Args,
  api: BaseQueryApi,
  extraOptions: DefinitionExtraOptions
) => MaybePromise<QueryReturnValue<Result, Error, Meta>>

export interface BaseQueryApi {
  signal: AbortSignal
  dispatch: ThunkDispatch<any, any, any>
  getState: () => unknown
}

export type QueryReturnValue<T = unknown, E = unknown, M = unknown> =
  | {
      error: E
      data?: undefined
      meta?: M
    }
  | {
      error?: undefined
      data: T
      meta?: M
    }
```
{% endcode %}

BaseQueryFn 타입은 다음의 제너릭을 가집니다:

*  `Args` - 함수의 첫번째 파라미터 타입입니다. endpoint의 쿼리 프로퍼티에서 반환받은 결과가 여기에 전달됩니다. 
* Result - success 케이스일때 반환될 data 프로퍼티 타입입니다. 모든 query와 mutation들이 같은 타입을 반환하지 않는한 이 타입을 unknown으로 하고 밑에서처럼 각각의 타입을 지정하는걸 추천합니다. 
* Error - error 케이스일때 반환될 error 프로퍼티 타입입니다. 이 타입은 API 정의의 endpoints에서 사용되는 모든 queryFn에도 적용됩니다. 
* DefinitionExtraOptions - 함수의 세번째 파라미터 타입입니다. endpoint의 extraOption 프로퍼티 값이 여기에 전달됩니다. 
* Meta - baseQuery를 호출할때 반환될 수 있는 meta 프로퍼티의 타입입니다. meta 프로퍼티는 transformResponse의 두번째 인자로 접근 가능합니다. 

{% hint style="info" %}
**노트**

baseQuery에서 반환받은 meta 프로퍼티는 에러가 thorw된 경우에는 반환이 되지 않아서 잠재적으로 undefined일 수 있습니다. meta 프로퍼티에 접근할때는 optional chaining같은 방법으로 접근해야합니다. 
{% endhint %}

{% tabs %}
{% tab title="TypeScript" %}
{% code title="간단한 타입스크립트 baseQuery 예시" %}
```typescript
import { createApi, BaseQueryFn } from '@reduxjs/toolkit/query'

const simpleBaseQuery: BaseQueryFn<
  string, // Args
  unknown, // Result
  { reason: string }, // Error
  { shout?: boolean }, // DefinitionExtraOptions
  { timestamp: number } // Meta
> = (arg, api, extraOptions) => {
  // `arg` has the type `string`
  // `api` has the type `BaseQueryApi` (not configurable)
  // `extraOptions` has the type `{ shout?: boolean }

  const meta = { timestamp: Date.now() }

  if (arg === 'forceFail') {
    return {
      error: {
        reason: 'Intentionally requested to fail!',
        meta,
      },
    }
  }

  if (extraOptions.shout) {
    return { data: 'CONGRATULATIONS', meta }
  }

  return { data: 'congratulations', meta }
}

const api = createApi({
  baseQuery: simpleBaseQuery,
  endpoints: (builder) => ({
    getSupport: builder.query({
      query: () => 'support me',
      extraOptions: {
        shout: true,
      },
    }),
  }),
})
```
{% endcode %}
{% endtab %}

{% tab title="Javascript" %}
{% code title="간단한 타입스크립트 baseQuery 예시" %}
```javascript
import { createApi } from '@reduxjs/toolkit/query'

const simpleBaseQuery = (arg, api, extraOptions) => {
  // `arg` has the type `string`
  // `api` has the type `BaseQueryApi` (not configurable)
  // `extraOptions` has the type `{ shout?: boolean }

  const meta = { timestamp: Date.now() }

  if (arg === 'forceFail') {
    return {
      error: {
        reason: 'Intentionally requested to fail!',
        meta,
      },
    }
  }

  if (extraOptions.shout) {
    return { data: 'CONGRATULATIONS', meta }
  }

  return { data: 'congratulations', meta }
}

const api = createApi({
  baseQuery: simpleBaseQuery,
  endpoints: (builder) => ({
    getSupport: builder.query({
      query: () => 'support me',
      extraOptions: {
        shout: true,
      },
    }),
  }),
})
```
{% endcode %}
{% endtab %}
{% endtabs %}

### query와 mutation endpoints 작성하기

endpoints는 builder syntax를 이용해서 정의된 오브젝트입니다. query와 mutation endpoints는 제너릭 포맷인 &lt;ResultType, QueryArg&gt;으로 타입을 제공할 수 있습니다. 

* ResultType - 쿼리에서 반환되는 최종 데이터 타입, optional한 transformResponse를 factoring합니다. 

  * 만약 transformResponse가 없다면, success query가 이 타입을 대신 반환합니다. 
  * 만약 transformResponse가 있다면, 초기 쿼리가 반환하는 타입때문에 transformResponse의 input 타입이 있어야 합니다. transformResponse의 반환 타입을 ResultType과 같아야 합니다. 
  * 만약 QueryFn이 query대신 쓰인다면, success case일때 다음과 같은 형태의 값을 반환해야 합니다:

  ```javascript
  {
    data: ResultType
  }
  ```

* QueryArg - endpoint의 query 프로퍼티이거나 queryFn을 대신 사용하면  queryFn의 첫번째 파라미터의 타입입니다?.\( The type of the input that will be passed as the only parameter to the `query` property of the endpoint, or the first parameter of a `queryFn` property if used instead.\) 

{% tabs %}
{% tab title="TypeScript" %}
{% code title="타입스크립트로 endpoints 정의하기" %}
```typescript
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react'
interface Post {
  id: number
  name: string
}

const api = createApi({
  baseQuery: fetchBaseQuery({ baseUrl: '/' }),
  endpoints: (build) => ({
    //              ResultType  QueryArg
    //                    v       v
    getPost: build.query<Post, number>({
      // inferred as `number` from the `QueryArg` type
      //       v
      query: (id) => `post/${id}`,
      // An explicit type must be provided to the raw result that the query returns
      // when using `transformResponse`
      //                             v
      transformResponse: (rawResult: { result: { post: Post } }, meta) => {
        //                                                        ^
        // The optional `meta` property is available based on the type for the `baseQuery` used

        // The return value for `transformResponse` must match `ResultType`
        return rawResult.result.post
      },
    }),
  }),
})
```
{% endcode %}
{% endtab %}

{% tab title="Javascript" %}
{% code title="endpoints 정의하기" %}
```javascript
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react'

const api = createApi({
  baseQuery: fetchBaseQuery({ baseUrl: '/' }),
  endpoints: (build) => ({
    //              ResultType  QueryArg
    //                    v       v
    getPost: build.query({
      // inferred as `number` from the `QueryArg` type
      //       v
      query: (id) => `post/${id}`,
      // An explicit type must be provided to the raw result that the query returns
      // when using `transformResponse`
      //                             v
      transformResponse: (rawResult, meta) => {
        //                                                        ^
        // The optional `meta` property is available based on the type for the `baseQuery` used

        // The return value for `transformResponse` must match `ResultType`
        return rawResult.result.post
      },
    }),
  }),
})
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% hint style="info" %}
**노트**

queries와 mutations은 위의 방식대신 baseQuery를 통해 반환 타입을 정의할 수 있습니다. 그러나 모든 query와 mutation들이 같은 타입을 반환하지 않는 한 baseQuery의 반환 타입을 unkown으로 두는 것을 추천합니다. 
{% endhint %}

### queryFn 작성하기

query와 mutation endpoints 작성하기에서 설명한 것 처럼 queryFn은 해당 endpoint의 제너릭으로 결과와 매개변수 타입들을 받습니다. 

{% tabs %}
{% tab title="TypeScript" %}
```typescript
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react'
import { getRandomName } from './randomData'

interface Post {
  id: number
  name: string
}

const api = createApi({
  baseQuery: fetchBaseQuery({ baseUrl: '/' }),
  endpoints: (build) => ({
    //              ResultType  QueryArg
    //                    v       v
    getPost: build.query<Post, number>({
      // inferred as `number` from the `QueryArg` type
      //         v
      queryFn: (arg, queryApi, extraOptions, baseQuery) => {
        const post: Post = {
          id: arg,
          name: getRandomName(),
        }
        // For the success case, the return type for the `data` property
        // must match `ResultType`
        //              v
        return { data: post }
      },
    }),
  }),
})
```
{% endtab %}

{% tab title="Javascript" %}
```javascript
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react'
import { getRandomName } from './randomData'

const api = createApi({
  baseQuery: fetchBaseQuery({ baseUrl: '/' }),
  endpoints: (build) => ({
    //              ResultType  QueryArg
    //                    v       v
    getPost: build.query({
      // inferred as `number` from the `QueryArg` type
      //         v
      queryFn: (arg, queryApi, extraOptions, baseQuery) => {
        const post = {
          id: arg,
          name: getRandomName(),
        }
        // For the success case, the return type for the `data` property
        // must match `ResultType`
        //              v
        return { data: post }
      },
    }),
  }),
})
```
{% endtab %}
{% endtabs %}

queryFn이 항상 반환해야하는 error 타입은 createApi가 제공하는 baseQuery에 의해서 결정됩니다. 

fetchBaseQuery에서 error 타입은 다음과 같습니다:

{% code title="fetchBaseQuery error shape" %}
```typescript
{
  status: number
  data: any
}
```
{% endcode %}

queryFn을 사용한 위의 error 케이스와 fetchBaseQuery의 error 타입 예시는 다음과 같습니다:

{% tabs %}
{% tab title="TypeScript" %}
{% code title="fetchBaseQuery에서 queryFn error와 error 타입" %}
```typescript
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react'
import { getRandomName } from './randomData'

interface Post {
  id: number
  name: string
}

const api = createApi({
  baseQuery: fetchBaseQuery({ baseUrl: '/' }),
  endpoints: (build) => ({
    getPost: build.query<Post, number>({
      queryFn: (arg, queryApi, extraOptions, baseQuery) => {
        if (arg <= 0) {
          return {
            error: {
              status: 500,
              data: 'Invalid ID provided.',
            },
          }
        }
        const post: Post = {
          id: arg,
          name: getRandomName(),
        }
        return { data: post }
      },
    }),
  }),
})
```
{% endcode %}
{% endtab %}

{% tab title="Javascript" %}
{% code title="fetchBaseQuery에서 queryFn error와 error 타입" %}
```javascript
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react'
import { getRandomName } from './randomData'

const api = createApi({
  baseQuery: fetchBaseQuery({ baseUrl: '/' }),
  endpoints: (build) => ({
    getPost: build.query({
      queryFn: (arg, queryApi, extraOptions, baseQuery) => {
        if (arg <= 0) {
          return {
            error: {
              status: 500,
              data: 'Invalid ID provided.',
            },
          }
        }
        const post = {
          id: arg,
          name: getRandomName(),
        }
        return { data: post }
      },
    }),
  }),
})
```
{% endcode %}
{% endtab %}
{% endtabs %}



