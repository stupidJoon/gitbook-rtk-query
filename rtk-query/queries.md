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

쿼리 hook는 쿼리 요청에 대한 최신 데이터와 현재 요청 상태가 포함된 객체를 반환합니다. 다음은 자주 사용되는 속성들입니다. 모든 반환된 속성들은 useQuery를 참조하세요.

* data - 반환된 결과값입니다. 
* error - 에러 결과값입니다. 
* isUninitialized - true일때 쿼리가 아직 시작하지 않았음을 나타냅니다. 
* isLoading - true일때 쿼리가 처음 로딩중이고 아직 데이터가 없다는걸 나타냅니다. 이는 처음 요청에만 해당하며 이후 요청에는 해당되지 않습니다. 
* isFetching - true일때 쿼리가 현재 패칭중이지만 이전 요청의 데이터가 있을 수 있음을 나타냅니다. 이는 처음 요청과 이후 요청 모두에 해당합니다. 
* isSuccess - true일때 쿼리의 요청이 성공했고 데이터가 있음을 나타냅니다. 
* isError - true일때 쿼리의 error 상태임을 나타냅니다. 
* refetch - 쿼리를 강제 리패치 시키는 함수입니다. 

대부분의 상황에서 UI를 렌더링하기위해 data와 isLoading 또는 isFetching이면 충분할 것 입니다. 

### 쿼리 hook 사용 예시

다음은 postDetail 컴포넌트 예시입니다: 

{% code title="예시" %}
```jsx
export const PostDetail = ({ id }: { id: string }) => {
  const { data: post, isFetching, isLoading } = useGetPostQuery(id, {
    pollingInterval: 3000,
    refetchOnMountOrArgChange: true,
    skip: false,
  })

  if (isLoading) return <div>Loading...</div>
  if (!post) return <div>Missing post!</div>

  return (
    <div>
      {post.name} {isFetching ? '...refetching' : ''}
    </div>
  )
}
```
{% endcode %}

이 컴포넌트의 방식에는 다음과 같은 몇 가지 특성이 있습니다. 

* 초기 로드시 'Loading...'만 표시합니다. 
  * 초기 로드는 쿼리가 pending상태이고 캐시에 데이터가 없는 상태입니다. 
* polling 인터벌에 의해 다시 요청을 보낼때 post name에 '...refetching'을 추가합니다. 
* 사용자가 PostDetail을 없앴다가 허용된 시간 내에 다시 생성하면 캐시된 결과를 즉시 제공하고 polling이 진행될 것 입니다. 

### 쿼리 로딩 상태

createApi에서 자동 생성된 리액트 hooks는 주어진 쿼리에 대한 현재 상태를 제공합니다. \(Derived booleans are preferred for the generated React hooks as opposed to a status flag, as the derived booleans are able to provide a greater amount of detail which would not be possible with a single status flag, as multiple statuses may be true at a given time \(such as isFetching and isSuccess\).\)???

쿼리 엔드포인트에서 RTK Query는 좀 더 유연하게 정보들을 제공하기 위해서 isLoading과 isFetching을 구분합니다. 

* isLoading은 처음으로 실행중인 쿼리를 의미합니다. 이 시간동안에는 데이터를 사용할 수 없습니다. 
* isFetching은 엔드포인트와 쿼리 매개변수로 실행중인 쿼리를 의미하지만, 반드시 처음으로 실행중인 쿼리는 아닙니다. 이전 요청의 쿼리와 매개변수로 데이터를 사용할 수 있습니다. 

이러한 구분을 통해서 UI에 더 많은 제어권을 가질 수 있습니다. 예를 들어 isLoading은 처음 로딩하는 동안에 스켈레톤을 보여줄 수 있고 isFetching은 페이지를 변경하거나 데이터를 무효화해서 리패칭할때 이전 데이터를 회색으로 보여줄 수 있습니다. 

{% code title="쿼리 로딩 상태일때 UI 관리하기" %}
```jsx
import { Skeleton } from './Skeleton'
import { useGetPostsQuery } from './api'

function App() {
  const { data = [], isLoading, isFetching, isError } = useGetPostsQuery()

  if (isError) return <div>An error has occurred!</div>

  if (isLoading) return <Skeleton />

  return (
    <div className={isFetching ? 'posts--disabled' : ''}>
      {data.map((post) => (
        <Post
          key={post.id}
          id={post.id}
          name={post.name}
          disabled={isFetching}
        />
      ))}
    </div>
  )
```
{% endcode %}

### 쿼리 캐시 keys

쿼리를 수행할 때 RTK Query는 요청 파라미터를 자동으로 직렬화하고 요청에 대한 queryCacheKey를 내부적으로 생성합니다. 이후 동일한 쿼리의 모든 요청들은 원본 데이터에 의해서 중복 제거되고 컴포넌트에서 패치가 일어났을때 업데이트를 공유합니다. 

### 쿼리 result에서 필요한 데이터 가져오기

부모 컴포넌트가 쿼리를 구독하고있고 자식 컴포넌트가 쿼리에서 데이터를 가져오는 경우가 있을 수 있습니다. 대부분의 경우에서는 이미 결과를 가지고 있을때 getItemById 형식의 추가적인 요청을 하고싶지 않을 것 입니다. 

selectFromResult를 사용하면 쿼리 결과에서 특정한 부분을 선택해서 가져올 수 있습니다. 이 기능을 사용하면 선택된 항목의 기본 데이터가 변경되지 않는 한 컴포넌트는 리렌더링이 이루어지지 않을 것 입니다. 선택된 항목이 원본 항목보다 더 큰 경우에는 같은 데이터를 가지도록 무시합니다. 

{% code title="selectFromResult를 사용해서 하나의 값을 가져오기" %}
```jsx
function PostsList() {
  const { data: posts } = api.useGetPostsQuery()

  return (
    <ul>
      {posts?.data?.map((post) => (
        <PostById key={post.id} id={post.id} />
      ))}
    </ul>
  )
}

function PostById({ id }: { id: number }) {
  // Will select the post with the given id, and will only rerender if the given posts data changes
  const { post } = api.useGetPostsQuery(undefined, {
    selectFromResult: ({ data }) => ({
      post: data?.find((post) => post.id === id),
    }),
  })

  return <li>{post?.name}</li>
}
```
{% endcode %}

### 불필요한 요청 피하기

기본값으로는, 이미 존재하는 쿼리를 새로운 컴포넌트에서 추가하면 요청이 수행되지 않습니다. 

경우에 따라서는 이 행동을 스킵하고 강제로 리패치하는 상황이 있을 수 있습니다. 이런 경우일때는 hook에서 반환된 refetch 함수를 실행하면 됩니다. 

{% hint style="info" %}
**정보**

리액트 hooks를 사용하고 있지 않다면, refetch를 다음과 같은 방식으로 접근할 수 있습니다:

```javascript
const { status, data, error, refetch } = dispatch(
  pokemonApi.endpoints.getPokemon.initiate('bulbasaur')
)
```
{% endhint %}



