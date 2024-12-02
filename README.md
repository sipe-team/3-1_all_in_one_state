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

## 1주차 : 목표설정 & 친해지기

[1주차 스터디 기록](1주차/1주차_정리.md)을 정리해보았습니다.

- 서로의 경험 공유
- 목표 수립
- 맛있는 음식먹기
- 규칙 정하기

## 2주차 : React 상태 관리 아키텍쳐

2주차 스터디 내용을 정리해보았습니다. 세부적인 것은 하위 주제들을 확인해주세요!

- [공통주제] : **상태가 변화하는 과정을 Deep하게 공부해보기**

  - [이지훈](2주차/이지훈/2주차_이지훈_공통.md)
  - [류지예](2주차/류지예/2주차_류지예_공통.md)
  - [심미진](2주차/심미진/2주차_심미진_공통.md)
  - [조명근](2주차/조명근/2주차_조명근_공통.md)
  - [최여진](2주차/최여진/2주차_최여진_공통.md)

- [1번 React 19에서 상태의 개념과 동작원리가 변하는가?? (심미진)](2주차/심미진/2주차_심미진_개인.md)
- 2번 useSyncExternalStore, external store에 대해 공부하기
  - [성지현](2주차/성지현/2주차_성지현_개인.md)
  - [이지훈](2주차/이지훈/2주차_이지훈_개인.md)
- 3번 동시성과 병렬성이 리엑트에서 어떻게 해석되는가?
  - [조명근](2주차/조명근/2주차_조명근_개인.md)
  - [류지예](2주차/류지예/2주차_류지예_개인.md)
- 4번 Fiber(파이버) 아키텍처에서의 상태 관리 메커니즘
  - [심현준](2주차/심현준/2주차_심현준_개인.md)
  - [최여진](2주차/최여진/2주차_최여진_개인.md)

### 2주차 스터디 내용 정리

- [React 상태가 변화는 과정](2주차/2주차_공통주제_최종정리.md)
- [동시성, 병렬성](2주차/2주차_동시성&병렬성.md)
- [React 19에서 상태의 개념과 동작원리 변화](2주차/2주차_React19.md)
- [Fiber Architecture](2주차/2주차_React-Fiber-Architecture.md)
- [useSyncExternalStore](2주차/2주차_useSyncExternalStore.md)

## 3주차 : Zustand 심층 분석

- [공통주제] : **Zustand는 왜 ProviderLess?**
  - [이지훈](https://hooninedev.com/240818/)
  - [류지예](https://github.com/sipe-team/3-1_all_in_one_state/blob/main/3%EC%A3%BC%EC%B0%A8/%EB%A5%98%EC%A7%80%EC%98%88/3%EC%A3%BC%EC%B0%A8_%EB%A5%98%EC%A7%80%EC%98%88_%EA%B3%B5%ED%86%B5.md)
  - [성지현](https://github.com/sipe-team/3-1_all_in_one_state/blob/main/3%EC%A3%BC%EC%B0%A8/%EC%84%B1%EC%A7%80%ED%98%84/3%EC%A3%BC%EC%B0%A8_%EC%84%B1%EC%A7%80%ED%98%84_%EA%B3%B5%ED%86%B5.md)
  - [조명근](https://github.com/sipe-team/3-1_all_in_one_state/blob/main/3%EC%A3%BC%EC%B0%A8/%EC%A1%B0%EB%AA%85%EA%B7%BC/3%EC%A3%BC%EC%B0%A8_%EC%A1%B0%EB%AA%85%EA%B7%BC_%EA%B3%B5%ED%86%B5%EA%B3%BC%EC%A0%9C.md)
  - [최여진](https://github.com/sipe-team/3-1_all_in_one_state/blob/main/3%EC%A3%BC%EC%B0%A8/%EC%B5%9C%EC%97%AC%EC%A7%84/3%EC%A3%BC%EC%B0%A8_%EC%B5%9C%EC%97%AC%EC%A7%84_%EA%B3%B5%ED%86%B5.md)

- [1번 Vanilla Store와 React Store의 차이점과 내부 구현 (류지예)](/3주차/류지예/3주차_류지예_개인.md)
- [2번 `create` vs `createStore`의 차이점 분석 (심현준)](/3주차/심현준/3주차_심현준_개인.md)
- [3번 상태 구독 메커니즘과 `subscribe` 메서드 구현 분석 (성지현)](/3주차/성지현/3주차_성지현_개인.md)
- [4번 불변성 관리와 immer 미들웨어 동작 원리 / valtio (이지훈)](/3주차/이지훈/3주차_이지훈_개인.md)
- [5번 비동기 상태 관리 패턴 (async actions) (조명근)](/3주차/조명근/3주차_조명근_개인.md)
- [6번 useShallow 성능 평가/개선(최여진)](/3주차/최여진/3주차_최여진_개인.md)

### 3주차 스터디 내용 정리

[3주차 총정리](/3주차/3주차_zustand.md)를 해보았습니다! 자세한 내용은 하위 링크를 확인해주세요!

## 4주차: Redux 미들웨어 아키텍처

- [공통주제] : 리덕스 붐이 다시 올까? (왜 안 쓰려고 하는지? , 왜 방대하게 설계되었고, 다른 상태 관리 라이브러리로 넘어가는가?)
  - [류지예](https://github.com/sipe-team/3-1_all_in_one_state/blob/main/4%EC%A3%BC%EC%B0%A8/%EB%A5%98%EC%A7%80%EC%98%88/4%EC%A3%BC%EC%B0%A8_%EB%A5%98%EC%A7%80%EC%98%88_%EA%B3%B5%ED%86%B5.md)
  - [심미진](https://github.com/sipe-team/3-1_all_in_one_state/blob/main/4%EC%A3%BC%EC%B0%A8/%EC%8B%AC%EB%AF%B8%EC%A7%84/4%EC%A3%BC%EC%B0%A8_%EC%8B%AC%EB%AF%B8%EC%A7%84_%EA%B3%B5%ED%86%B5.md)
  - [심현준](https://github.com/sipe-team/3-1_all_in_one_state/blob/main/4%EC%A3%BC%EC%B0%A8/%EC%8B%AC%ED%98%84%EC%A4%80/4%EC%A3%BC%EC%B0%A8_%EC%8B%AC%ED%98%84%EC%A4%80_%EA%B3%B5%ED%86%B5.md)
  - [조명근](https://github.com/sipe-team/3-1_all_in_one_state/blob/main/4%EC%A3%BC%EC%B0%A8/%EC%A1%B0%EB%AA%85%EA%B7%BC/4%EC%A3%BC%EC%B0%A8_%EC%A1%B0%EB%AA%85%EA%B7%BC_%EA%B3%B5%ED%86%B5.md)

- [1번 Redux가 전역 상태 관리의 시초인데 Redux의 구조와 다른 라이브러리의 구조 비교, 인터페이스 비교 및 왜 옮겨가는지 (이지훈)](https://github.com/sipe-team/3-1_all_in_one_state/blob/main/4%EC%A3%BC%EC%B0%A8/%EC%9D%B4%EC%A7%80%ED%9B%88/4%EC%A3%BC%EC%B0%A8_%EC%9D%B4%EC%A7%80%ED%9B%88_%EA%B3%B5%ED%86%B5.md)
- [2번 Redux Toolkit의 내부 구현과 최적화 전략 (심미진)](https://github.com/sipe-team/3-1_all_in_one_state/blob/main/4%EC%A3%BC%EC%B0%A8/%EC%8B%AC%EB%AF%B8%EC%A7%84/4%EC%A3%BC%EC%B0%A8_%EC%8B%AC%EB%AF%B8%EC%A7%84_%EA%B0%9C%EC%9D%B8.md)
- [3번 Redux의 미들웨어가 해결하려고 했던 것들은 다른 라이브러리는 어떻게 해결하려 했을까? (조명근)](https://github.com/sipe-team/3-1_all_in_one_state/blob/main/4%EC%A3%BC%EC%B0%A8/%EC%A1%B0%EB%AA%85%EA%B7%BC/4%EC%A3%BC%EC%B0%A8_%EC%A1%B0%EB%AA%85%EA%B7%BC_%EA%B0%9C%EC%9D%B8.md)
- [4번 각 상태 관리 라이브러리가 Redux 미들웨어를 사용하던데 어느면에서 호환성이 좋은가? (최여진)](https://github.com/sipe-team/3-1_all_in_one_state/blob/main/4%EC%A3%BC%EC%B0%A8/4%EC%A3%BC%EC%B0%A8_redux_%EA%B0%9C%EB%B3%84.md)
- [5번 `createSlice`와 `createReducer` 동작 원리 (심현준)](https://github.com/sipe-team/3-1_all_in_one_state/blob/main/4%EC%A3%BC%EC%B0%A8/%EC%8B%AC%ED%98%84%EC%A4%80/4%EC%A3%BC%EC%B0%A8_%EC%8B%AC%ED%98%84%EC%A4%80_%EA%B0%9C%EC%9D%B8.md)
- [6번 `combineReducers`의 내부 구현 분석 (성지현)](https://github.com/sipe-team/3-1_all_in_one_state/blob/main/4%EC%A3%BC%EC%B0%A8/%EC%84%B1%EC%A7%80%ED%98%84/4%EC%A3%BC%EC%B0%A8_%EC%84%B1%EC%A7%80%ED%98%84.md)
- [7번 Redux의 상태 정규화 패턴과 구현 (류지예)](https://github.com/sipe-team/3-1_all_in_one_state/blob/main/4%EC%A3%BC%EC%B0%A8/%EB%A5%98%EC%A7%80%EC%98%88/4%EC%A3%BC%EC%B0%A8_%EB%A5%98%EC%A7%80%EC%98%88_%EA%B0%9C%EC%9D%B8.md)

### 4주차 스터디 내용 정리

[4주차 총정리](https://github.com/sipe-team/3-1_all_in_one_state/blob/main/4%EC%A3%BC%EC%B0%A8/4%EC%A3%BC%EC%B0%A8_redux_%EA%B0%9C%EB%B3%84.md)를 해보았습니다! 자세한 내용은 하위 링크를 확인해주세요!

## 5주차: Tanstack-Query 각자 주제 정해서 공부해보기

- [1번 2024년 TanStack Query 의 큰 Issue, Discussion, PR 에 대해서 소개 (이지훈)](https://github.com/sipe-team/3-1_all_in_one_state/blob/main/5%EC%A3%BC%EC%B0%A8/%EC%9D%B4%EC%A7%80%ED%9B%88/5%EC%A3%BC%EC%B0%A8_%EC%9D%B4%EC%A7%80%ED%9B%88.md)
- [2번 Query Key 설계 전략과 컨벤션 (심미진)](https://github.com/sipe-team/3-1_all_in_one_state/blob/main/5%EC%A3%BC%EC%B0%A8/5%EC%A3%BC%EC%B0%A8_%EC%8B%AC%EB%AF%B8%EC%A7%84_%EA%B0%9C%EC%9D%B8.md)
- [3번 캐시 무효화(invalidation) 전략 (조명근)](https://github.com/sipe-team/3-1_all_in_one_state/blob/main/5%EC%A3%BC%EC%B0%A8/5%EC%A3%BC%EC%B0%A8_%EC%A1%B0%EB%AA%85%EA%B7%BC_%EA%B0%9C%EC%9D%B8.md)
- [4번 next.js 에서 tanstack query 를 써야할까? 쓴다면 어떻게 써야할까? (최여진)](https://github.com/sipe-team/3-1_all_in_one_state/blob/main/5%EC%A3%BC%EC%B0%A8/%EC%B5%9C%EC%97%AC%EC%A7%84/5%EC%A3%BC%EC%B0%A8_%EC%B5%9C%EC%97%AC%EC%A7%84_%EA%B0%9C%EC%9D%B8.md)
- [5번 prefetchQuery & dehydrate  (심현준)](https://github.com/sipe-team/3-1_all_in_one_state/blob/main/5%EC%A3%BC%EC%B0%A8/%EC%8B%AC%ED%98%84%EC%A4%80/5%EC%A3%BC%EC%B0%A8_%EC%8B%AC%ED%98%84%EC%A4%80_%EA%B0%9C%EC%9D%B8.md)
- [6번 staleTime vs cacheTime (성지현)](https://github.com/sipe-team/3-1_all_in_one_state/blob/main/5%EC%A3%BC%EC%B0%A8/%EC%84%B1%EC%A7%80%ED%98%84/5%EC%A3%BC%EC%B0%A8_%EC%84%B1%EC%A7%80%ED%98%84_%EA%B0%9C%EC%9D%B8.md)
- [7번 prefetching과 infinite query (류지예)](https://github.com/sipe-team/3-1_all_in_one_state/blob/main/5%EC%A3%BC%EC%B0%A8/%EB%A5%98%EC%A7%80%EC%98%88/5%EC%A3%BC%EC%B0%A8_%EB%A5%98%EC%A7%80%EC%98%88_%EA%B0%9C%EC%9D%B8.md)
