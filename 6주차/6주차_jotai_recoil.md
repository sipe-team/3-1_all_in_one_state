# jotai, recoil 과 함께하는 atomic 여행

## 1. Recoil의 RecoilRoot와 Jotai의 Provider 비교

### RecoilRoot와 Jotai Provider의 특징

|  | Recoil Root | Jotai Provider |
| --- | --- | --- |
| 특징 | - 복잡한 상태 관리 컨텍스트를 생성하며 atom 상태 추적, 트랜잭션 관리, 일괄 업데이트 기능 포함
- Suspense 경계를 자동으로 처리
- 광범위한 디버깅 기능과 개발 도구 통합 제공
- 구독, 트랜잭션, 그래프 기반 의존성 추적 관리
- 메모리 관리와 정리 메커니즘 내장 | - 기본 저장소 관리에 중점을 둔 단순한 구현
- Provider 없이도 작동하는 provider-less 모드 기본 지원
- 경량화된 컨텍스트 기반 저장소 관리
- 간단한 저장소 생성과 값 전파 |

#### 공통점

- Context API로 상태 전달
- store를 prop으로 받을 수 있음 (외부에서 생성된 store를 주입받을 수 있음)
- 컴포넌트 트리 구조의 유사성

#### 차이점

- store의 참조군이 다름
    - Recoil은 getNextStoreID() 함수로 고유 식별자를 생성해 여러 RecoilRoot 중첩을 방지하고 구분
    - Jotai는 useRef 훅으로 store 인스턴스를 유지
- 초기화
    - Recoil은 복잡한 초기화 로직과 atom 효과를 위한 initializeState prop 제공
    - Jotai는 직접적인 저장소 생성이나 기존 저장소 전달로 간단하게 초기화
- 중첩 동작
    - Recoil은 중첩된 root 동작을 제어하기 위한 override prop이 있음
    - Jotai는 특별한 설정 없이 여러 provider와의 중첩을 자연스럽게 지원

## 2. atom() 함수의 내부 구현과 상태 초기화 프로세스 & 업데이트 리렌더링 최적화 방식

### Recoil의 atom

- key(아톰 키), default(상태 초기값)을 전달하여 상태 선언
- 다른 atom의 상태값인 RecoilValue를 받아 파생 상태를 만들 수 있음
    - atom이 RecoilValue를 받으면 atomWithFallback을 호출하고, 내부에서 atom 함수를 재귀 호출하여 모든 atom 상태를 체이닝함

### Jotai의 atom

- key를 받지 않고, read와 write 값을 받음
    - read: atom의 초기값 또는 파생된 값을 반환하는 getter 함수
    - write: setter 함수
- Read-only, Write-only, Read-write atom 방식
- 내부적으로 keyCount를 증가시키며 key 생성
- config 객체를 만들어 초기화된 atom 값, read/write 설정
- atom에서 반환된 config 객체는 useAtom의 인자로 들어가며, [state, setState] 패턴으로 구조분해됨

### **Recoil의 전역상태 업데이트 최적화 방식**

- useRecoilValue로 atom 상태값을 구독
- 상태 업데이트 방식이 버전에 따라 다름
    - React 18+
        - useSyncExternalStore 훅 사용
        - RecoilRoot Provider의 store 객체와 동기화
        - 추가 React 렌더러(react-three-fiber 등) 미지원 시 TRANSITION_SUPPORT로 폴백
    - React 18 이전
        - useState + useEfect + forceUpdate 조합
        - 배치 업데이트 처리를 위한 추가 로직 포함
    - Concurrent Mode와 Transition API 지원
    - 성능 최적화된 상태 업데이트 처리

### **Jotai의 전역상태 업데이트 최적화 방식**

- useAtomValue로 atom 상태값을 구독
- useReducer 기반 상태 관리 방식
    - useSyncExternalStore 사용 x
        - 리액트 동시성모드에서 일시적 tearing은 감수하되, time-slicing (렌더링 우선순위조정) 성능 최적화를 우선하는 방식의 트레이드오프 전략

## 3. selector() 함수의 내부 동작 분석

### Recoil의 selector

#### 기본 구조

```tsx
function selector({
    key, // 고유 식별자
    get, // 읽기 함수
    set? // 선택적 쓰기 함수
}) {
    // selector의 핵심 로직
}
```

#### 의존성 추적 메커니즘

```tsx
function createSelector(config) {
  // 의존성 추적을 위한 내부 로직
  const dependencyMap = new Map();

  function trackDependencies(get) {
    // 현재 selector가 읽는 atom들을 추적
    const dependencies = [];

    // 프록시를 통한 의존성 추적
    const proxyGet = new Proxy(get, {
      apply: (target, thisArg, args) => {
        const [atom] = args;
        dependencies.push(atom);
        return target(...args);
      },
    });

    // selector의 계산 로직 실행
    config.get(proxyGet);

    return dependencies;
  }
}
```

#### 메모이제이션 전략

```tsx
function createMemoizedSelector(config) {
  let lastDependencies = null;
  let lastResult = null;

  return {
    read: (get) => {
      // 현재 의존성 추적
      const currentDependencies = trackDependencies(get);

      // 의존성 변경 여부 확인
      const dependenciesChanged =
        !lastDependencies ||
        currentDependencies.some(
          (dep, index) => dep !== lastDependencies[index]
        );

      // 캐시된 결과 또는 새로운 결과 반환
      if (!dependenciesChanged) {
        return lastResult;
      }

      // 새로운 결과 계산
      const newResult = config.get(get);

      // 상태 업데이트
      lastDependencies = currentDependencies;
      lastResult = newResult;

      return newResult;
    },
  };
}
```

#### 비동기 selector 처리

```tsx
function asyncSelector({
    key,
    get: async (get) => {
        // 비동기 작업 처리
        // Recoil은 내부적으로 Suspense와 연동
        const dependencyValue = get(someAtom);
        const result = await fetchData(dependencyValue);
        return result;
    }
})
```

### Jotai의 selector

#### 기본 구조

```tsx
function atom(read, write?) {
  return {
    read: (get) => {
      // 다른 atom의 값을 읽어올 수 있는 get 함수
      // 의존성 추적 및 메모이제이션 로직 포함
    },
    write: (get, set, value) => {
      // 쓰기 작업 구현 (선택적)
    },
  };
}
```

#### 의존성 추적 메커니즘

- dependency tracking system
- selector가 다른 atom을 읽을 때마다 해당 의존성을 추적
- 의존하는 atom 값이 바뀌면 selector도 다시 계산

#### 메모이제이션 전략

```tsx
function createMemoizedSelector(computeFn) {
  let lastDependencies = null;
  let lastResult = null;

  return (get) => {
    // 현재 의존하는 atom들의 값 추적
    const currentDependencies = trackDependencies(get);

    // 의존성이 변경되지 않았다면 캐시된 결과 반환
    if (lastDependencies && areEqual(lastDependencies, currentDependencies)) {
      return lastResult;
    }

    // 새로운 결과 계산
    const newResult = computeFn(get);

    // 상태 업데이트
    lastDependencies = currentDependencies;
    lastResult = newResult;

    return newResult;
  };
}
```

#### 비동기 selector 처리

```tsx
const asyncSelector = atom(async (get) => {
  // 비동기 작업 처리 메커니즘
  const dependencyValue = get(someAtom);
  const result = await fetchSomeData(dependencyValue);
  return result;
});
```

## 4. atom 구독 메커니즘 & 상태 변경 감지와 전파 과정

### 구독 메커니즘

- atom이라는 상태 단위 각각 subscribers 목록 관리
- 컴포넌트에서 atom을 사용하는 것 = atom의 구독자가 됨
    - 구독 로직은 Context API로 구현됨

### Recoil의 상태 변경 감지와 전파 과정

- `useRecoilState`로 store에 atom을 등록하며, 해당 컴포넌트를 구독자로 등록함 → 의존성 그래프 구축
- 상태 변경 시, 의존성 그래프를 통해 영향받는 selector를 다시 계산
- 변경된 atom/selector로 구독하는 컴포넌트들에게 알림 → 리렌더링 발생

### Jotai의 상태 변경 감지와 전파 과정

- 컴포넌트에서 atom 호출 시 내부적으로 store에 등록, 현재 컴포넌트는 구독자가 됨
- 상태 변경 시, 해당 상태를 구독하는 컴포넌트에 리렌더링 발생

## 5. atomWithDefault와 loadable 패턴 , 비동기

### Recoil

```jsx
import { atom, selector } from "recoil";
import { fetchTodos } from "../api/todoApi";

// 필터링 로직 (파생 상태)
export const filteredTodosSelector = selector({
  key: "filteredTodosSelector",
  get: ({ get }) => {
    const data = await fetchTodos();
    return data;
  },
});

// hooks/useTodos.js
import { useRecoilValueLoadable, useSetRecoilState } from "recoil";
import {  filteredTodosSelector } from "../store/todoStore";

export const useTodos = () => {
  const todosLoadable = useRecoilValueLoadable(filteredTodosSelector);

  // Loadable 상태를 사용하기 쉬운 형태로 변환
  const state = {
    isLoading: todosLoadable.state === "loading",
    isError: todosLoadable.state === "hasError",
    error: todosLoadable.state === "hasError" ? todosLoadable.contents : null,
    data: todosLoadable.state === "hasValue" ? todosLoadable.contents : null,
  };

  return {
    ...state,
  };
};
```

- 비동기 처리시 string으로 정의된 파생 상태를 useRecoilValueLoadable로 가져옴

### Jotai

```jsx
import { loadable } from "jotai/utils";
import { atom, useAtom } from "jotai";

const fetchMessage = () =>
  new Promise<{ data: string }>((resolve) => {
    setTimeout(() => {
      resolve({ data: "안녕하세요, Jotai!" });
    }, 3000);
  });

const fetchUser = () =>
  new Promise<{ name: string; age: number }>((resolve) => {
    setTimeout(() => {
      resolve({ name: "김철수", age: 25 });
    }, 2000);
  });

const messageAtom = atom(async () => {
  const data = await fetchMessage();
  return data;
});

const userAtom = atom(async () => {
  const data = await fetchUser();
  return data;
});

const messageAtomLodable = loadable(messageAtom);
const userAtomLodable = loadable(userAtom);

const JotaiAsyncTest = () => {
  const [messageData] = useAtom(messageAtomLodable);
  const [userData] = useAtom(userAtomLodable);

  return (
    <div>
      {/* 메시지 데이터 렌더링 */}
      <div>
        <h3>메시지 상태:</h3>
        {messageData.state === "loading" && <div>메시지 로딩중...</div>}
        {messageData.state === "hasError" && (
          <div>에러: {messageData.error}</div>
        )}
        {messageData.state === "hasData" && (
          <div>메시지: {JSON.stringify(messageData.data)}</div>
        )}
      </div>

      {/* 사용자 데이터 렌더링 */}
      <div>
        <h3>사용자 상태:</h3>
        {userData.state === "loading" && <div>사용자 정보 로딩중...</div>}
        {userData.state === "hasError" && <div>에러: {userData.error}</div>}
        {userData.state === "hasData" && (
          <div>
            <p>이름: {userData.data.name}</p>
            <p>나이: {userData.data.age}</p>
          </div>
        )}
      </div>
    </div>
  );
};

export default JotaiAsyncTest;
```

- `jotai/utils`에 `loadable` 유틸로 비슷한 역할 수행

## 6. atomFamily 함수 구현 분석

### atomFamily의 기본 구조와 필요성

- 동일한 형태의 상태를 여러 개 관리해야 하는 경우 유용
    - ex. 할 일의 완료 상태 (할일1의 완료 상태, 할일2의 완료상태 … )
- 각 항목이 독립적인 상태를 가지고, 필요한 상태만 구독하여 관련있는 컴포넌트에만 영향을 줌

### 핵심 구현 메커니즘

- 파라미터 기반 캐싱
    - 파라미터를 기반으로 atom 생성
    - 이미 만들어진 atom이 있다면 재사용
- 자동 정리 메커니즘
    - atom이 더 이상 필요 없을 때 자동 제거
    - 캐시에서도 자동 삭제되어 메모리 누수 방지
    - 클린업은 컴포넌트의 언마운트와 연계됨
- 선택적 구독 시스템
    - 컴포넌트는 필요한 atom만 구독하여 불필요한 리렌더링 방지
- 파라미터별로 다른 메모리 관리 정책

### real world 사례, 이점

- 파라미터 기반 캐싱과 독립적인 atom 관리 방식은 동시성 렌더링 환경에서 발생할 수 있는 문제를 해결함
- 성능 최적화
- 유연한 상태 관리

## 7. Recoil의 메모리 누수 문제

### 실험 방법

- [초기 상태 힙 스냅샷1]
- add user 버튼을 클릭해 사용자 추가x100회 → atom(atomFamily)을 생성
- [힙 스냅샷2]
- remove all user 버튼을 클릭해 사용자 제거
- [힙 스냅샷3]

→ 위 스냅샷을 비교함

- 1, 2 비교: 메모리 증가량 확인
- 2, 3 비교: 메모리 해제 여부 확인

### 결과

| 단계 | 힙 크기 | 메모리 변화 설명 |
| --- | --- | --- |
| 초기 상태 | 3.5MB | 애플리케이션 시작 시 기본 메모리 사용  |
| 100명의 사용자  | 45.7MB | 각 사용자 추가 시 약 0.5 MB 증가 |
| 전체 사용자 제거 후 | 44.8MB | 수동으로 가비지 컬렉터 동작시켜도 메모리 해제 미미  |
- 사용자를 추가하면서 힙 메모리가 증가하며, 모두 제거한 후에는 노드는 제거되지만 메모리가 해제되지 않음
    
    → 메모리 누수 지점
    

### 메모리 누수는 왜 발생하는가?

- `retainedBy`의 기본값이 `recoilRoot`로 설정되어 있고, SPA 환경에서 RecoilRoot가 거의 언마운트되지 않는 특성 때문에 메모리 누수가 발생