# 3-1_all_in_one_state

**팀 이름 : 내 상태와 너의 상태.. 그 사이 어디..**

## [주제]

Redux, Zustand, Recoil, Jotai, MobX를 중심으로 한 상태 관리 라이브러리 분석

## [목표]

각 라이브러리를 심도있게 이해해보자

## [스터디 방법]

주차별 주제에 대해 공부해서 정리합니다 (1인당 1주제 선택)

매주 미팅전까지 공부하고 준비한 자료를 수요일 23:00까지 PR로 올립니다.

미팅전에 질문을 미리 작성하셔도 무방하고, 당일에 질문하셔도 좋습니다.

매주 목요일 19:30에 만나 공부한 내용을 공유하고 의견을 나눕니다

의견 공유의 방식은 “왜” 라는 관점을 가집니다.

## [주차별 계획]

아래 주차별 활동의 내용들은 제가 생각하는 각 상태에서 공부하면 좋을 주제 예시입니다 (좋은 주제있으면 공유해주세요~!)

변경의 여지가 많습니다. 많은 의견과 질문을 제시해주시면 감사하겠습니다!!

### 1주차 : React의 상태 관리 아키텍처

- Fiber 아키텍처에서의 상태 관리 메커니즘
- React의 내부 dispatcher 구현 분석
- **Preserving and Resetting State**
- React 18의 동시성 렌더링과 상태 관리

### 2주차: Zustand 심층 분석

- Vanilla Store와 React Store의 차이점과 내부 구현
- `create` vs `createStore`의 차이점 분석
- 상태 구독 메커니즘과 `subscribe` 메서드 구현 분석
- 불변성 관리와 immer 미들웨어 동작 원리
- 비동기 상태 관리 패턴 (async actions)
- useShallow 성능 평가/개선

### 3주차: Redux 미들웨어 아키텍처

- Redux Toolkit의 내부 구현과 최적화 전략
- `createSlice`와 `createReducer` 동작 원리
- RTK Query의 캐싱 메커니즘과 데이터 무효화 전략
- Redux Saga vs Redux Thunk 동작 원리 비교
- Redux의 상태 정규화 패턴과 구현
- `combineReducers`의 내부 구현 분석
- Selector 패턴과 메모이제이션 전략

### 4주차: Mobx 반응형 프로그래밍 모델

- MobX의 Proxy 기반 관찰 가능성 구현
- Transactions와 Actions의 자동 배칭
- `makeAutoObservable`vs `makeObservable` 차이점
- MobX의 Computed Values 캐싱 메커니즘
- Reactions(`autorun`, `reaction`, `when`)의 동작 원리
- MobX-State-Tree 아키텍처 분석
- MobX의 메모리 누수 방지 전략
- 상태 지속성과 하이드레이션 구현

### 5주차: Recoil의 비동기 셀렉터와 동시성 모델 / Jotai

- Recoil의 상태 의존성 그래프 구현
- Atom Effects와 상태 초기화 전략
- `atomFamily`와 `selectorFamily` 동작 원리
- 트랜잭션 관리와 상태 일관성 유지 방법
- Primitive Atoms vs Derived Atoms 구현 분석
- Jotai의 상태 하이드레이션 구현
- Atoms 간의 의존성 관리 방식
- Recoil vs Jotai 아키텍처 비교
