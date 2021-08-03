# 다른 라이브러리들과 비교

**RTK Query는 다른 데이터 패칭 라이브러리들에서 많은 영향을 받았습니다.** [Redux core 라이브러리가 Flux나 Elm같은 라이브러리들에서 영향을 받았던것처럼](https://redux.js.org/understanding/history-and-design/prior-art), RTK Query는 React Query, SWR, Apollo, Urql같은 라이브러리에서 많이 사용되는 API 설계 패턴과 개념을 기반으로 합니다. RTK Query는 기초부터 만들어졌지만, 리덕스의 강점과 능력에 맞게 다른 라이브러리의 우수한 개념들을 사용하고자 노력했습니다. 

저희는 위의 라이브러리들이 모두 우수하다고 생각합니다. 만약 그중 하나를 사용중이다면, 개발자가 겪는 여러 문제들을 해결할 수 있게 도와준다고 생각합니다. 이 페이지 정보들의 목적은 **기능, 구현, 접근, API 디자인들에서 어떤 차이점**이 있는지 알려주는 것 입니다. 이 페이지의 목표는 X가 Y보다 좋다고 주장하기 보다는 **정보에 입각해서 결정을 돕고 장단점들을 이해하는 것 입니다.** 

## 언제 RTK Query를 사용해야 하나요? <a id="when-should-you-use-rtk-query"></a>

보통 RTK Query를 사용하는 이유들로는:

* 이미 Redux 어플리케이션을 가지고 있고 이미 작성된 데이터 패칭 로직을 단순화시키기 위해서
* Redux DevTools를 이용해서 상태의 히스토리 변화를 보고싶어서
* Redux 생태계에 RTK Query 기능을 통합시키고싶어서
* 애플리케이션 로직이 React밖에서 동작해야해서

### 특별한 기능들 <a id="unique-capabilities"></a>

RTK Query는 고려할 가치가 있는 몇몇의 특별한 API 디자인 측면들과 기능들을 가지고 있습니다.

* React Query와 SWR에서, 보통 여러곳에서 hooks를 정의하고 어디서나 hooks를 사용할 수 있습니다. RTK Query에서는 하나의 "API slice"와 여러개의 엔드포인트들을 한곳에서 관리해야합니다. 이를 통해 자동으로 쿼리 무효화/리패칭 작업들이 긴밀하게 통합될 수 있습니다. 
* RTK Query는 보통 리덕스 액션처럼 요청이 처리되기때문에 모든 액션들은 Redux DevTools에서 볼 수 있습니다. 추가적으로 모든 요청들은 Redux 리듀서에서 접근가능해 필요하다면 글로벌 상태를 손쉽게 업데이트할 수 있습니다 \([예시](https://github.com/reduxjs/redux-toolkit/issues/958#issuecomment-809570419)\). [엔드포인트 matcher 기능](https://redux-toolkit.js.org/rtk-query/api/created-api/endpoints#matchers)을 통해 리듀서에서 캐시와 관련된 작업을 할 수 있습니다. 
* Redux처럼 RTK Query의 주요한 기능은 UI 독립적이며 다른 UI 레이어에서도 사용할 수 있습니다. 
* 미들웨어에서 엔티티 무효화나 이미 존재하는 쿼리 데이터 패치작업\(`util.updateQueryData`를 통해\)을 손쉽게 할 수 있습니다. 
* RTK Query는 웹소켓으로 부터 받은 초기 패치 데이터 업데이트같은 [스트리밍 캐시 업데이트](https://redux-toolkit.js.org/rtk-query/usage/streaming-updates)를 지원하고 [업데이트 최적화](https://redux-toolkit.js.org/rtk-query/usage/optimistic-updates)도 기본적으로 지원합니다. 
* RTK Query는 작고 유연한 fetch 래퍼인 [`fetchBaseQuery`](https://redux-toolkit.js.org/rtk-query/api/fetchBaseQuery)를 제공합니다. 또한 `axios`, `redaxios` 같은 [다른 클라이언트와 바꾸기도](https://redux-toolkit.js.org/rtk-query/usage/customizing-queries) 매우 쉽습니다. 
* RTK Query는 OpenAPI 스펙과 GraphQL 스키마를 지원하는 [\(현재 실험적인\)코드 생성 기능](https://github.com/rtk-incubator/rtk-query-codegen)을 통해 작성된 클라이언트 API를 제공합니다.

## Tradeoffs

### 캐시 정규화, 중복제거 미지원 <a id="no-normalized-or-deduplicated-cache"></a>

RTK Query는 **여러 요청에서 동일한 항목에 대한 캐시 중복제거를 의도적으로 구현하지 않았습니다.** 그 이유로는 여러가지가 있는데: 

* 완전히 정규화된 쿼리간 공유 캐시는 해결하기 어려운 문제입니다. 
* 저희는 그 문제를 해결하기위한 시간, 자원, 흥미가 충분하지 않습니다. 
* 많은 상황에서 간단하게 데이터를 다시 패칭하는게 유효하지 않은 작업을 해결하는 쉽고 간단한 방법입니다. 
* 최소한 RTK Query는 여러 사람들의 힘든 부분인 "데이터 패칭"같은 일반적인 사례를 해결하는 데 도움을 줄 수 있습니다. 

### 번들 사이즈 <a id="bundle-size"></a>

RTK Query는 고정된 번들 사이즈를 앱에 추가합니다. RTK Query가 Redux Toolkit과 React-Redux 위에서 동작하기 때문에 추가된 크기는 얼마나 그것들을 사용하고 있는지에 따라서 달라집니다. 예상되는 min+gzip 번들 사이즈는: 

* 만약 RTK를 이미 사용 중이면: RTK Query ~9kb, hooks ~2kb
* 만약 RTK를 사용 중이지 않는다면:
  * Without React: RTK+dependencies+RTK Query 17kb
  * With React: 19kb + React-Redux\(peer dependency\)

엔드포인트 정의를 추가하면 오직 `endpoint` 내의 코드의 크기를 추가하며, 일반적으로 몇 바이트에 불과합니다.

RTK Query의 번들 사이즈는 직접 작성한 데이터 패칭 로직을 제거하면 대부분의 애플리케이션의 사이즈가 의미있게 개선됩니다.

## 기능들 비교하기 <a id="comparing-feature-sets"></a>

다른 라이브러리들을 비교해서 기능의 공통점과 차이점을 파악할 수 있습니다.

{% hint style="info" %}
**정보**

이 비교 테이블은 최대한 정확하고 편견이 없도록 노력했습니다. 이러한 라이브러리를 사용하고 정보가 개선되야한다고 생각하는 경우, 언제든지 [이슈를 열어](https://github.com/reduxjs/redux-toolkit/issues/new) 변경을 제안하세요
{% endhint %}

| Feature | rtk-query | react-query | apollo | urql |
| :--- | :--- | :--- | :--- | :--- |
| **Supported Protocols** | any, REST included | any, none included | GraphQL | GraphQL |
| **API Definition** | declarative | on use, declarative | GraphQL schema | GraphQL schema |
| **Cache by** | endpoint + serialized arguments | user-defined query-key | type/id | type/id? |
| **Invalidation Strategy + Refetching** | declarative, by type and/or type/id | manual by cache key | automatic cache updates on per-entity level, manual query invalidation by cache key | declarative, by type OR automatic cache updates on per-entity level, manual query invalidation by cache key |
| **Polling** | yes | yes | yes | yes |
| **Parallel queries** | yes | yes | yes | yes |
| **Dependent queries** | yes | yes | yes | yes |
| **Skip queries** | yes | yes | yes | yes |
| **Lagged queries** | yes | yes | no | ? |
| **Auto garbage collection** | yes | yes | no | ? |
| **Normalized caching** | no | no | yes | yes |
| **Infinite scrolling** | TODO | yes | requires manual code | ? |
| **Prefetching** | yes | yes | yes | yes? |
| **Retrying** | yes | yes | requires manual code | ? |
| **Optimistic updates** | can update cache by hand | can update cache by hand | `optimisticResponse` | ? |
| **Manual cache manipulation** | yes | yes | yes | yes |
| **Platforms** | hooks for React, everywhere Redux works | hooks for React | various | various |

## 추가 정보 <a id="further-information"></a>

* [React Query "Comparison" 페이지](https://react-query.tanstack.com/comparison)는 추가적인 세부 기능 비교 표와 기능에 대한 설명이 있습니다. 
* Urql 메인테이너인 Phil Pluckthun의 ["normalized cache"가 무엇이고 Urql의 캐시가 어떻게 작동하는지 완벽한 설명](https://kitten.sh/graphql-normalized-caching)을 작성했습니다. 
* [RTK Query의 "Cache Behavior" 페이지](https://redux-toolkit.js.org/rtk-query/usage/cache-behavior#tradeoffs)에 왜 RTK Query가 normalized cache를 구현하지 않았는지 상세한 설명이 있습니다. 

