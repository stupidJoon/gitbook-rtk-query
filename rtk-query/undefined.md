# 뮤테이션

## 개요

뮤테이션은 서버에게 데이터 업데이트를 전달하고 로컬에서 변화된 값을 적용시킬때 사용됩니다. 또한 뮤테이션은 캐시 무효화와 강제 리패칭을 할 수 있습니다. 

### 뮤테이션 엔드포인트 정의하기

뮤테이션 엔드포인트는 createApi의 endpoints에서 객체를 반환하고 builder.mutation\(\)메소드를 사용해서 정의합니다. 

뮤테이션 엔드포인트는 URL\(URL 파라미터 포함\)을 생성하는 query 콜백이거나 임의의 비동기 로직을 실행후 결과를 반환하는 queryFn 콜백이여야 합니다. query 콜백은 URL, 사용할 HTTP 메소드, 요청 바디를 포함하는 객체를 반환할 수 있습니다. 

query 콜백이 URL을 생성하기 위해서 추가 데이터가 필요한 경우 하나의 파라미터만 작성해야 합니다. 여러 데이터들이 필요한 경우 단일 객체로 포맷된 파라미터로 전달해야 합니다. 

또한 뮤테이션 엔드포인트는 결과가 캐시되기 전에 응답 내용을 수정하고 캐시 무효화를 식별하기 위한 "tags"를 정의하며 캐시 항목이 추가 및 제거될 때 추가 로직을 위한 캐시 수명주기 콜백을 제공합니다. 

{% tabs %}
{% tab title="Typescript" %}
{% code title="뮤테이션 엔드포인트 옵션 예제" %}
```typescript
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query'
import { Post } from './types'

const api = createApi({
  baseQuery: fetchBaseQuery({
    baseUrl: '/',
  }),
  tagTypes: ['Post'],
  endpoints: (build) => ({
    updatePost: build.mutation<Post, Partial<Post> & Pick<Post, 'id'>>({
      // note: an optional `queryFn` may be used in place of `query`
      query: ({ id, ...patch }) => ({
        url: `post/${id}`,
        method: 'PATCH',
        body: patch,
      }),
      // Pick out data and prevent nested properties in a hook or selector
      transformResponse: (response: { data: Post }) => response.data,
      invalidatesTags: ['Post'],
      // onQueryStarted is useful for optimistic updates
      // The 2nd parameter is the destructured `MutationLifecycleApi`
      async onQueryStarted(
        arg,
        { dispatch, getState, queryFulfilled, requestId, extra, getCacheEntry }
      ) {},
      // The 2nd parameter is the destructured `MutationCacheLifecycleApi`
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
        }
      ) {},
    }),
  }),
})
```
{% endcode %}
{% endtab %}

{% tab title="Javascript" %}
{% code title="뮤테이션 엔드포인트 옵션 예제" %}
```javascript
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query'

const api = createApi({
  baseQuery: fetchBaseQuery({
    baseUrl: '/',
  }),
  tagTypes: ['Post'],
  endpoints: (build) => ({
    updatePost: build.mutation({
      // note: an optional `queryFn` may be used in place of `query`
      query: ({ id, ...patch }) => ({
        url: `post/${id}`,
        method: 'PATCH',
        body: patch,
      }),
      // Pick out data and prevent nested properties in a hook or selector
      transformResponse: (response) => response.data,
      invalidatesTags: ['Post'],
      // onQueryStarted is useful for optimistic updates
      // The 2nd parameter is the destructured `MutationLifecycleApi`
      async onQueryStarted(
        arg,
        { dispatch, getState, queryFulfilled, requestId, extra, getCacheEntry }
      ) {},
      // The 2nd parameter is the destructured `MutationCacheLifecycleApi`
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
        }
      ) {},
    }),
  }),
})
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% hint style="info" %}
**정보**

onQueryStarted 메소드는 optimistic updates에 사용될 수 있습니다. 
{% endhint %}

