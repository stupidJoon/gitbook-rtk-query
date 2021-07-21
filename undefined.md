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

이 페이지에 설명되어 있지 않는 타입 관련 문제가 발생하는 경우 [이슈를 열어주세요](https://github.com/reduxjs/redux-toolkit/issues?q=is%3Aissue+is%3Aopen+sort%3Aupdated-desc). 
{% endhint %}

## `createApi`

### auto-generated 리액트 Hooks 사용하기

리액트에서 RTK Query는 쿼리와 엔드포인트들을 위해 각각의 query와 mutation [`endpoints`](https://redux-toolkit.js.org/rtk-query/api/createApi#endpoints)에서 자동으로 생성되는 리액트 hooks인 [`createApi`](https://redux-toolkit.js.org/rtk-query/api/createApi)를 exports하는것에서부터 시작됩니다. 

타입스크립트 사용자를 위한 자동생성 리액트 hooks를 사용할려면 **TS4.1+ 버전을 사용해야 합니다.** 

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

### `baseQuery` 작성하기

RTK Query에서 `BaseQueryFn` 타입을 export하면 커스텀 [`baseQuery`](https://redux-toolkit.js.org/rtk-query/api/createApi#basequery)를 작성할 수 있습니다. 

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

`BaseQueryFn` 타입은 다음의 제너릭을 가집니다:

*  `Args` - 함수의 첫번째 파라미터 타입입니다. endpoint의 [`query`](https://redux-toolkit.js.org/rtk-query/api/createApi#query) 프로퍼티에서 반환받은 결과가 여기에 전달됩니다. 
* `Result` - success 케이스일때 반환될 `data` 프로퍼티 타입입니다. 모든 query와 mutation들이 같은 타입을 반환하지 않는한 이 타입을 `unknown`으로 하고 [밑에서처럼](https://redux-toolkit.js.org/rtk-query/usage-with-typescript#typing-query-and-mutation-endpoints) 각각의 타입을 지정하는걸 추천합니다. 
* `Error` - error 케이스일때 반환될 `error` 프로퍼티 타입입니다. 이 타입은 API 정의의 endpoints에서 사용되는 모든 [`queryFn`](https://redux-toolkit.js.org/rtk-query/usage-with-typescript#typing-a-queryfn)에도 적용됩니다. 
* `DefinitionExtraOptions` - 함수의 세번째 파라미터 타입입니다. endpoint의 [`extraOption`](https://redux-toolkit.js.org/rtk-query/api/createApi#extraoptions) 프로퍼티 값이 여기에 전달됩니다. 
* `Meta` - `baseQuery`를 호출할때 반환될 수 있는 `meta` 프로퍼티의 타입입니다. `meta` 프로퍼티는 [`transformResponse`](https://redux-toolkit.js.org/rtk-query/api/createApi#transformresponse)의 두번째 인자로 접근 가능합니다. 

{% hint style="info" %}
**노트**

`baseQuery`에서 반환받은 `meta` 프로퍼티는 에러가 `thorw`된 경우에는 반환이 되지 않아서 잠재적으로 undefined일 수 있습니다. `meta` 프로퍼티에 접근할때는 optional chaining같은 방법으로 접근해야합니다. 
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

### query와 mutation `endpoints` 작성하기

`endpoints`는 builder syntax를 이용해서 정의된 오브젝트입니다. `query`와 `mutation` endpoints는 제너릭 포맷인 `<ResultType, QueryArg>`으로 타입을 제공할 수 있습니다. 

* `ResultType` - 쿼리에서 반환되는 최종 데이터 타입, optional한 [`transformResponse`](https://redux-toolkit.js.org/rtk-query/api/createApi#transformresponse)를 factoring합니다. 

  * 만약 `transformResponse`가 없다면, success query가 이 타입을 대신 반환합니다. 
  * 만약 `transformResponse`가 있다면, 초기 쿼리가 반환하는 타입때문에 `transformResponse`의 input 타입이 있어야 합니다. `transformResponse`의 반환 타입을 `ResultType`과 같아야 합니다. 
  * 만약 `QueryFn`이 `query`대신 쓰인다면, success case일때 다음과 같은 형태의 값을 반환해야 합니다:

  ```javascript
  {
    data: ResultType
  }
  ```

* `QueryArg` - endpoint의 `query` 프로퍼티이거나 `queryFn`을 대신 사용하면  queryFn의 첫번째 파라미터의 타입입니다?.\( The type of the input that will be passed as the only parameter to the `query` property of the endpoint, or the first parameter of a `queryFn` property if used instead.\) 

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

`queries`와 `mutations`은 위의 방식대신 `baseQuery`를 통해 반환 타입을 정의할 수 있습니다. 그러나 모든 query와 mutation들이 같은 타입을 반환하지 않는 한 `baseQuery`의 반환 타입을 `unkown`으로 두는 것을 추천합니다. 
{% endhint %}

### `queryFn` 작성하기

[query와 mutation endpoints 작성하기](https://redux-toolkit.js.org/rtk-query/usage-with-typescript#typing-query-and-mutation-endpoints)에서 설명한 것 처럼 `queryFn`은 해당 endpoint의 제너릭으로 결과와 매개변수 타입들을 받습니다. 

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

`queryFn`이 항상 반환해야하는 error 타입은 `createApi`가 제공하는 [`baseQuery`](https://redux-toolkit.js.org/rtk-query/usage-with-typescript#typing-a-basequery)에 의해서 결정됩니다. 

[`fetchBaseQuery`](https://redux-toolkit.js.org/rtk-query/api/fetchBaseQuery)에서 error 타입은 다음과 같습니다:

{% code title="fetchBaseQuery error shape" %}
```typescript
{
  status: number
  data: any
}
```
{% endcode %}

`queryFn`을 사용한 위의 error 케이스와 `fetchBaseQuery`의 error 타입 예시는 다음과 같습니다:

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

`baseQuery`를 추가하지 않고 `queryFn`만 각각의 endpoint에서 사용하고 싶을때, RTK Query는 `queryFn`이 반환할 error 타입을 쉽게 정할 수 있는 함수인 `fakeBaseQuery`를 제공합니다. 

{% tabs %}
{% tab title="TypeScript" %}
{% code title="모든 endpoints에서 baseQuery 없이 사용하기" %}
```typescript
import { createApi, fakeBaseQuery } from '@reduxjs/toolkit/query'

type CustomErrorType = { reason: 'too cold' | 'too hot' }

const api = createApi({
  // This type will be used as the error type for all `queryFn` functions provided
  //                              v
  baseQuery: fakeBaseQuery<CustomErrorType>(),
  endpoints: (build) => ({
    eatPorridge: build.query<'just right', 1 | 2 | 3>({
      queryFn(seat) {
        if (seat === 1) {
          return { error: { reason: 'too cold' } }
        }

        if (seat === 2) {
          return { error: { reason: 'too hot' } }
        }

        return { data: 'just right' }
      },
    }),
    microwaveHotPocket: build.query<'delicious!', number>({
      queryFn(duration) {
        if (duration < 110) {
          return { error: { reason: 'too cold' } }
        }
        if (duration > 140) {
          return { error: { reason: 'too hot' } }
        }

        return { data: 'delicious!' }
      },
    }),
  }),
})
```
{% endcode %}
{% endtab %}

{% tab title="Javascript" %}
{% code title="모든 endpoints에서 baseQuery 없이 사용하기" %}
```javascript
import { createApi, fakeBaseQuery } from '@reduxjs/toolkit/query'

const api = createApi({
  // This type will be used as the error type for all `queryFn` functions provided
  //                              v
  baseQuery: fakeBaseQuery(),
  endpoints: (build) => ({
    eatPorridge: build.query({
      queryFn(seat) {
        if (seat === 1) {
          return { error: { reason: 'too cold' } }
        }

        if (seat === 2) {
          return { error: { reason: 'too hot' } }
        }

        return { data: 'just right' }
      },
    }),
    microwaveHotPocket: build.query({
      queryFn(duration) {
        if (duration < 110) {
          return { error: { reason: 'too cold' } }
        }
        if (duration > 140) {
          return { error: { reason: 'too hot' } }
        }

        return { data: 'delicious!' }
      },
    }),
  }),
})
```
{% endcode %}
{% endtab %}
{% endtabs %}

### `providesTags`/`invalidatesTags` 작성하기

RTK Query는 만료된 데이터를 [자동으로 re-fetching](https://redux-toolkit.js.org/rtk-query/usage/automated-refetching)하는 캐시 태그 무효화 시스템을 제공합니다. 

함수를 사용할때 endpoint에서 `providesTags`와 `invalidatesTags` 프로퍼티는 다음과 같은 매개변수를 받습니다. 

* result: `ResultType` \| `undefined` - 쿼리가 success 케이스일때 반환되는 결과값입니다. 이 타입은 [endpoint에 제공된](https://redux-toolkit.js.org/rtk-query/usage-with-typescript#typing-query-and-mutation-endpoints) `ResultType`과 동일합니다. 쿼리가 error 케이스일때는 undefined 일 것 입니다. 
* error: `ErrorType` \| `undefined` - 쿼리가 error 케이스일때 반환되는 에러 값입니다. 이 타입은 [api의 baseQuery에 제공된](https://redux-toolkit.js.org/rtk-query/usage-with-typescript#typing-a-basequery) `Error`값과 동일합니다. 쿼리가 success 케이스일때는 `undefined` 일 것 입니다.  
* arg: `QueryArg` - 쿼리가 호출됐을때 `query` 프로퍼티로 제공되는 값입니다. 이 타입은 [endpoint에 제공된](https://redux-toolkit.js.org/rtk-query/usage-with-typescript#typing-query-and-mutation-endpoints) `QueryArg`과 동일합니다. 

`providesTags`의 권장 사용 사례는 쿼리가 엔티티 ID 또는 '리스트' ID 태그가 있는 항목들을 반환하는 쿼리일때 입니다\([Advanced Invalidation with abstract tag IDs](https://redux-toolkit.js.org/rtk-query/usage/automated-refetching#advanced-invalidation-with-abstract-tag-ids)\).

This is often written by spreading the result of mapping the received data into an array, as well as an additional item in the array for the `'LIST'` ID tag. When spreading the mapped array, by default, TypeScript will broaden the `type` property to `string`. As the tag `type` must correspond to one of the string literals provided to the [`tagTypes`](https://redux-toolkit.js.org/rtk-query/api/createApi#tagtypes) property of the api, the broad `string` type will not satisfy TypeScript. In order to alleviate this, the tag `type` can be cast `as const` to prevent the type being broadened to `string`.

{% tabs %}
{% tab title="TypeScript" %}
{% code title="providesTags TypeScript example" %}
```typescript
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react'
interface Post {
  id: number
  name: string
}
type PostsResponse = Post[]

const api = createApi({
  baseQuery: fetchBaseQuery({ baseUrl: '/' }),
  tagTypes: ['Posts'],
  endpoints: (build) => ({
    getPosts: build.query<PostsResponse, void>({
      query: () => 'posts',
      providesTags: (result) =>
        result
          ? [
              ...result.map(({ id }) => ({ type: 'Posts' as const, id })),
              { type: 'Posts', id: 'LIST' },
            ]
          : [{ type: 'Posts', id: 'LIST' }],
    }),
  }),
})
```
{% endcode %}
{% endtab %}

{% tab title="JavaScript" %}
{% code title="providesTags TypeScript example" %}
```javascript
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react'

const api = createApi({
  baseQuery: fetchBaseQuery({ baseUrl: '/' }),
  tagTypes: ['Posts'],
  endpoints: (build) => ({
    getPosts: build.query({
      query: () => 'posts',
      providesTags: (result) =>
        result
          ? [
              ...result.map(({ id }) => ({ type: 'Posts', id })),
              { type: 'Posts', id: 'LIST' },
            ]
          : [{ type: 'Posts', id: 'LIST' }],
    }),
  }),
})
```
{% endcode %}
{% endtab %}
{% endtabs %}

## Skipping queries with TypeScript using `skipToken`

RTK Query provides the ability to conditionally skip queries from automatically running using the `skip` parameter as part of query hook options \(see [Conditional Fetching](https://redux-toolkit.js.org/rtk-query/usage/conditional-fetching)\).

TypeScript users may find that they encounter invalid type scenarios when a query argument is typed to not be `undefined`, and they attempt to `skip` the query when an argument would not be valid.

{% tabs %}
{% tab title="TypeScript" %}
{% code title="API definition" %}
```typescript
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react'
import { Post } from './types'

export const api = createApi({
  baseQuery: fetchBaseQuery({ baseUrl: '/' }),
  endpoints: (build) => ({
    // Query argument is required to be `number`, and can't be `undefined`
    //                            V
    getPost: build.query<Post, number>({
      query: (id) => `post/${id}`,
    }),
  }),
})

export const { useGetPostQuery } = api
```
{% endcode %}
{% endtab %}

{% tab title="JavaScript" %}
{% code title="API definition" %}
```javascript
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react'

export const api = createApi({
  baseQuery: fetchBaseQuery({ baseUrl: '/' }),
  endpoints: (build) => ({
    // Query argument is required to be `number`, and can't be `undefined`
    //                            V
    getPost: build.query({
      query: (id) => `post/${id}`,
    }),
  }),
})

export const { useGetPostQuery } = api
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="TypeScript" %}
{% code title="Using skip in a component" %}
```typescript
import { useGetPostQuery } from './api'

function MaybePost({ id }: { id?: number }) {
  // This will produce a typescript error:
  // Argument of type 'number | undefined' is not assignable to parameter of type 'number | unique symbol'.
  // Type 'undefined' is not assignable to type 'number | unique symbol'.

  // @ts-expect-error id passed must be a number, but we don't call it when it isn't a number
  const { data } = useGetPostQuery(id, { skip: !id })

  return <div>...</div>
}
```
{% endcode %}
{% endtab %}

{% tab title="JavaScript" %}
{% code title="Using skip in a component" %}
```javascript
import { useGetPostQuery } from './api'

function MaybePost({ id }: { id?: number }) {
  // This will produce a typescript error:
  // Argument of type 'number | undefined' is not assignable to parameter of type 'number | unique symbol'.
  // Type 'undefined' is not assignable to type 'number | unique symbol'.

  // @ts-expect-error id passed must be a number, but we don't call it when it isn't a number
  const { data } = useGetPostQuery(id, { skip: !id })

  return <div>...</div>
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

While you might be able to convince yourself that the query won't be called unless the `id` arg is a `number` at the time, TypeScript won't be convinced so easily.

 RTK Query provides a `skipToken` export which can be used as an alternative to the `skip` option in order to skip queries, while remaining type-safe. When `skipToken` is passed as the query argument to `useQuery`, `useQueryState` or `useQuerySubscription`, it provides the same effect as setting `skip: true` in the query options, while also being a valid argument in scenarios where the `arg` might be undefined otherwise.

{% code title="Using skipToken in a component" %}
```typescript
import { skipToken } from '@reduxjs/toolkit/query/react'
import { useGetPostQuery } from './api'

function MaybePost({ id }: { id?: number }) {
  // When `id` is nullish, we will still skip the query.
  // TypeScript is also happy that the query will only ever be called with a `number` now
  const { data } = useGetPostQuery(id ?? skipToken)

  return <div>...</div>
}
```
{% endcode %}

