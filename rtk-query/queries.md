# 쿼리

## 개요

이 부분은 RTK Query의 가장 일반적인 유즈케이스일것입니다. 쿼리는 어떤 데이터 패칭이든 수행할 수 있지만 일반적으로는 데이터를 가져오는 요청에만 사용하는 것을 추천합니다. 서버의 데이터를 변경하거나 캐시를 무효화할때는 뮤테이션을 사용해야 합니다. 

기본적으로 RTK Query는 axios같은 라이브러리같이 요청 헤더와 응답 파싱을 자동으로 처리하는 가벼운 fetch 래퍼인 fetchBaseQuery를 제공합니다. 만약 fetchBaseQuery가 요구사항을 만족하지 못한다면 쿼리 커스터마이징을 참고하세요. 

{% hint style="info" %}
**INFO**

환경에 따라 fetchBaseQuery나 fetch를 사용하는 경우 node-fetch나 cross-fetch로 fetch를 폴리필할 필요가 있습니다. 
{% endhint %}

hook 시그니쳐나 자세한 내용은 useQuery를 참고하세요. 

## 쿼리 엔드포인트 정의하기

쿼리 엔드포인트는 createApi의 endpoints에서 builder.query\(\) 메소드를 사용해서 정의합니다. 

쿼리 엔드포인트는 URL\(URL 쿼리 파라미터 포함\)을 반환하는 query 콜백이나 임의의 비동기 로직을 수행하고 결과를 반환하는 queryFn 콜백을 정의해야 합니다.

query 콜백에서 URL을 생성할때 추가적인 데이터가 필요하다면 하나의 매개변수를 작성해야 합니다. 만약 여러개의 매개변수를 전달해야 한다면 하나의 객체로 만들어서 전달해야 합니다. 

또한 쿼리 엔드포인트는 결과가 캐시되기 전에 응답 내용을 수정하고, "tags"를 정의해서 캐시 무효화를 식별하고, 캐시 항목이 추가 및 제거될 때 로직을 추가하기 위한 캐시 라이프사이클 콜백을 제공합니다. 

{% tabs %}
{% tab title="TypeScript" %}
{% code title="쿼리 엔드포인트 옵션 예제" %}
```typescript
// Or from '@reduxjs/toolkit/query/react'
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query'
import { Post } from './types'

const api = createApi({
  baseQuery: fetchBaseQuery({
    baseUrl: '/',
  }),
  tagTypes: ['Post'],
  endpoints: (build) => ({
    getPost: build.query<Post, number>({
      // note: an optional `queryFn` may be used in place of `query`
      query: (id) => ({ url: `post/${id}` }),
      // Pick out data and prevent nested properties in a hook or selector
      transformResponse: (response: { data: Post }) => response.data,
      providesTags: (result, error, id) => [{ type: 'Post', id }],
      // The 2nd parameter is the destructured `QueryLifecycleApi`
      async onQueryStarted(
        arg,
        {
          dispatch,
          getState,
          extra,
          requestId,
          queryFulfilled,
          getCacheEntry,
          updateCachedData,
        }
      ) {},
      // The 2nd parameter is the destructured `QueryCacheLifecycleApi`
      async onCacheEntryAdded(
        arg,
        {
          dispatch,
          getState,
          extra,
          requestId,
          cacheEntryRemoved,
          cacheDataLoaded,
          getCacheEntry,
          updateCachedData,
        }
      ) {},
    }),
  }),
})
```
{% endcode %}
{% endtab %}

{% tab title="JavaScript" %}
{% code title="쿼리 엔드포인트 옵션 예제" %}
```javascript
// Or from '@reduxjs/toolkit/query/react'
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query'

const api = createApi({
  baseQuery: fetchBaseQuery({
    baseUrl: '/',
  }),
  tagTypes: ['Post'],
  endpoints: (build) => ({
    getPost: build.query({
      // note: an optional `queryFn` may be used in place of `query`
      query: (id) => ({ url: `post/${id}` }),
      // Pick out data and prevent nested properties in a hook or selector
      transformResponse: (response) => response.data,
      providesTags: (result, error, id) => [{ type: 'Post', id }],
      // The 2nd parameter is the destructured `QueryLifecycleApi`
      async onQueryStarted(
        arg,
        {
          dispatch,
          getState,
          extra,
          requestId,
          queryFulfilled,
          getCacheEntry,
          updateCachedData,
        }
      ) {},
      // The 2nd parameter is the destructured `QueryCacheLifecycleApi`
      async onCacheEntryAdded(
        arg,
        {
          dispatch,
          getState,
          extra,
          requestId,
          cacheEntryRemoved,
          cacheDataLoaded,
          getCacheEntry,
          updateCachedData,
        }
      ) {},
    }),
  }),
})
```
{% endcode %}
{% endtab %}
{% endtabs %}



