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

## 리액트 hooks과 함께 쿼리 사용하기

리액트 hooks를 사용한다면 RTK Query는 추가적인 기능을 제공합니다. 가장 큰 이점은 hooks을 통해 백그라운드 패칭에 대한 불리언 값을 손쉽게 사용할 수 있습니다. 

hooks는 서비스의 endpoint에서 자동으로 생성됩니다. getPost: builder.query\(\) 엔드포인트는 useGetPostQuery hook을 생성합니다. 

### Hook의 종류

다음과 같은 5개의 쿼리와 관련된 hooks이 있습니다:

* useQuery
  * useQuerySubscription과 useQueryState를 구성하는 기본 hook입니다. 엔드포인트에서 자동으로 데이터를 패치하고, 캐시된 데이터를 컴포넌트에 구독하며, 리덕스 스토어에서 요청 상태와 캐시된 데이터를 가져옵니다. 
* useQuerySubscription
  * refetch 함수를 반환하고 모든 hook 옵션을 받습니다. 엔드포인트에서 자동으로 데이터를 패치하고 컴포넌트에 캐시된 데이터를 구독합니다. 
* useQueryState
  * 쿼리 상태를 반환하고 skip과 selectFromResult를 받습니다. 리덕스 스토어에서는 요청 상태와 캐시된 데이터를 읽습니다. 
* useLazyQuery
  * fetch 함수, 쿼리 결과, 마지막 promise 정보가 포함된 튜플을 반환합니다. useQuery와 비슷하지만 언제 데이터 패칭을 하는지 수동으로 제어합니다. 
* useLazyQuerySubscription
  * fetch 함수와 마지막 promise 정보가 있는 튜플을 반환합니다. useQuerySubscription과 비슷하지만 언제 데이터 패칭을 하는지 수동으로 제어합니다. 

예시에서는 useGetPostQuery같은 표준 useQuery 기반 hooks를 사용합니다. 그러나 다른 hooks들도 특정한 사례에 사용할 수 있습니다. 

### Hook의 옵션

쿼리 hook은 두개의 파라미터를 받습니다. \(queryArg?, queryOptions?\)

queryArg 파라미터는 query 콜백에 전달되어 URL을 생성합니다. 

{% hint style="warning" %}
**주의**

queryArg 파라미터는 내부적으로 useEffect의 의존성 배열로 전달됩니다. RTK Query는 파라미터에 대해 shallowEquals로 차이를 알아내지만 깊게 중첩된 값을 전달한 경우는 직접 파라미터를 안정적으로 유지해야 합니다\(e.g. with useMemo\).
{% endhint %}

queryOptions 객체는 데이터 패칭을 제어하는데 사용되는 추가적인 파라미터를 받습니다. 

* skip - 해당 렌더에 대해 실행 중인 쿼리를 '스킵'할 수 있습니다. 기본값은 false입니다. 
* pollingInterval - 지정한 간격\(ms\)에 따라 자동으로 쿼리를 리패치할 수 있습니다. 기본값은 0\(비활성화\)입니다. 
* selectFromResult - 훅에서 반환되는 결과값을 변경하고, 변경된 결과값을 렌더에 최적화할 수 있습니다. 
* [refetchOnMountOrArgChange](https://redux-toolkit.js.org/rtk-query/api/createApi#refetchonmountorargchange) - 마운트시 항상 쿼리를 강제로 리패치할 수 있습니다\(true인 경우\). 동일한 캐시에 대한 마지막 쿼리 이후 충분한 시간이 경과한 경우 쿼리를 강제로 리패치할 수 있습니다\(number인 경우\). 기본값은 false입니다. 
* refetchOnFocus - 브라우저 창에 포커스를 다시 가질때 쿼리를 강제로 리패치할 수 있습니다. 기본값은 false입니다. 
* refetchOnReconnect - 네트워크가 다시 연결되었을때 쿼리를 강제로 리패치할 수 있습니다. 기본값은 false입니다. 

{% hint style="info" %}
**정보**

모든 리패치와 관련된 옵션들은 createApi에서 설정된 값을 덮어씁니다. 
{% endhint %}

### 자주 사용되는 Hook 반환값



