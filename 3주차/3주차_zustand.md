# Zustand 심층 분석

## Zustand는 왜 Provider가 필요없을까?

> **"Zustand는 왜 Provider가 필요없을까?"** 에 대한 개인 자료는 
[이지훈, ](https://hooninedev.com/240818/)
[류지예, ](https://github.com/sipe-team/3-1_all_in_one_state/blob/main/3%EC%A3%BC%EC%B0%A8/%EB%A5%98%EC%A7%80%EC%98%88/3%EC%A3%BC%EC%B0%A8_%EB%A5%98%EC%A7%80%EC%98%88_%EA%B3%B5%ED%86%B5.md)
[성지현, ](https://github.com/sipe-team/3-1_all_in_one_state/blob/main/3%EC%A3%BC%EC%B0%A8/%EC%84%B1%EC%A7%80%ED%98%84/3%EC%A3%BC%EC%B0%A8_%EC%84%B1%EC%A7%80%ED%98%84_%EA%B3%B5%ED%86%B5.md)
[조명근, ](https://github.com/sipe-team/3-1_all_in_one_state/blob/main/3%EC%A3%BC%EC%B0%A8/%EC%A1%B0%EB%AA%85%EA%B7%BC/3%EC%A3%BC%EC%B0%A8_%EC%A1%B0%EB%AA%85%EA%B7%BC_%EA%B3%B5%ED%86%B5%EA%B3%BC%EC%A0%9C.md)
[최여진 ](https://github.com/sipe-team/3-1_all_in_one_state/blob/main/3%EC%A3%BC%EC%B0%A8/%EC%B5%9C%EC%97%AC%EC%A7%84/3%EC%A3%BC%EC%B0%A8_%EC%B5%9C%EC%97%AC%EC%A7%84_%EA%B3%B5%ED%86%B5.md)
을 확인해주세요


Zustand는 Provider 필요 없이 사용할 수 있는 형태의 전역 상태관리 라이브러리이다.  
어떻게 Provider가 없이도 전역 상태관리를 할 수 있는지, 어떻게 Zustand가 구성되어 있는지 알아보자.

<br>

### 1. 클로저를 통한 상태 캡슐화

Zustand는 자바스크립트 클로저를 활용하여 안전하고 격리된 상태 컨테이너를 생성한다.

`state`와 `listeners`는 함수 스코프(클로저) 내부에 캡슐화되어 외부에서 직접 접근이 불가능하다.

그렇기에 외부에서의 상태 조작은 오직 아래 3개의 인터페이스를 통해서만 가능하다.
  - `setState`: 상태 업데이트
  - `getState`: 현재 상태 조회
  - `subscribe`: 상태 변경 감지를 위한 리스너 등록

이러한 클로저 기반 아키텍처는 스레드 안전한 상태 관리와 무단 수정 방지를 보장한다.

<br>

### 2. 옵저버 패턴을 통한 반응형 업데이트

React의 Context 기반 전파 대신, Zustand는 옵저버 패턴을 구현한다.

`Set` 자료구조를 사용하여 중복 없이 리스너 관리하고, `Object.is`를 사용한 얕은 비교로 변경을 감지한다.

그리고 상태 변경 시 등록된 모든 리스너에게 알림을 전파한다.

이 접근 방식은 Context Provider의 트리 기반 전파를 제거하여 **컴포넌트 리렌더링 감소, 불필요한 컴포넌트 트리 순회를 피하여 성능 향상,Provider context 상속 없이 간단한 상태 업데이트를 가능**하게 만든다.

<br>

### 3. 자바스크립트 모듈 시스템 활용

Zustand는 자바스크립트 모듈 시스템을 효율적으로 활용하여 전역 상태를 공유한다.

스토어 인스턴스는 자바스크립트 모듈 스코프 내에서 관리한다.

모듈은 본질적으로 싱글톤이다. 첫 번째 import에서 스토어 인스턴스 생성하고, 이후 import에서는 동일한 인스턴스 재사용한다.

이러한 모듈 레벨 싱글톤 패턴으로 애플리케이션 전체에서 일관된 상태 보장하고, Provider 없이 전역 상태 접근이 가능하다.

<br>

### 4. 상태 불변성 관리

Zustand는 아래와 같이 강력한 불변성 처리를 제공한다.

- `Object.assign` 또는 전개 연산자를 사용한 상태 객체 복제
- 유연한 변경 패턴을 허용하면서도 불변 상태 업데이트 유지
- 미들웨어를 통한 불변 및 "변경 가능한" 업데이트 스타일 모두 지원
- 중첩된 상태 업데이트를 위한 내장 유틸리티 제공

<br>


### 5. React 통합 및 동기화

Zustand의 아키텍처는 핵심 상태 관리를 React 통합과 분리한다.

- 핵심 기능은 바닐라 자바스크립트로 구현되어 프레임워크에 독립적이다.
- React 통합은 `useSyncExternalStore` 훅을 통해 이루어지는데, React의 렌더링 라이프사이클과 일관된 상태 동기화 보장, Zustand와 React 사이의 업데이트를 적절히 조정하여 티어링 이슈 방지, 특정 상태 변경에만 구독하여 리렌더링 최적화가 이루어진다.

<br>


## Zustand는 어떻게 데이터를 저장하고 관리할까?

[류지예 - Vanilla Store와 React Store의 차이점과 내부 구현](https://github.com/sipe-team/3-1_all_in_one_state/blob/main/3%EC%A3%BC%EC%B0%A8/%EB%A5%98%EC%A7%80%EC%98%88/3%EC%A3%BC%EC%B0%A8_%EB%A5%98%EC%A7%80%EC%98%88_%EA%B0%9C%EC%9D%B8.md)  
[성지현 - 상태 구독 메커니즘과 subscribe 메서드 구현 분석](https://github.com/sipe-team/3-1_all_in_one_state/blob/main/3%EC%A3%BC%EC%B0%A8/%EC%84%B1%EC%A7%80%ED%98%84/3%EC%A3%BC%EC%B0%A8_%EC%84%B1%EC%A7%80%ED%98%84_%EA%B0%9C%EC%9D%B8.md)  
[심현준 - create VS createStore 차이점 분석](https://github.com/sipe-team/3-1_all_in_one_state/blob/main/3%EC%A3%BC%EC%B0%A8/%EC%8B%AC%ED%98%84%EC%A4%80/3%EC%A3%BC%EC%B0%A8_%EC%8B%AC%ED%98%84%EC%A4%80_%EA%B0%9C%EC%9D%B8.md)  
[최여진 - Zustand useShallow 제대로 쓰기](https://github.com/sipe-team/3-1_all_in_one_state/blob/main/3%EC%A3%BC%EC%B0%A8/%EC%B5%9C%EC%97%AC%EC%A7%84/3%EC%A3%BC%EC%B0%A8_%EC%B5%9C%EC%97%AC%EC%A7%84_%EA%B0%9C%EC%9D%B8.md)

Zustand는 Vanilla Store, React Store 두 가지 형태의 스토어를 제공한다.  

각 스토어가 가지는 특징들과 어떻게 상태를 관리할 수 있는지 정리해보자.

### Vanilla Store

> 자세한 코드는 [vanilla.ts](https://github.com/pmndrs/zustand/blob/main/src/vanilla.ts) 를 확인

React 독립적으로 동작하는 순수 Javascript 기반 상태 관리

```ts
// Vanilla Store - 명령형 API
const store = createStore((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
}));

// 명시적인 상태 접근과 변경
const count = store.getState().count;
store.setState({ count: count + 1 });
```

createStore는 Javascript로 구성되어 있기 때문에, React 상태관리 시스템 바깥에서 전역 스토어를 초기화해서 사용하고 싶은 경우 또는 커스터마이징이 필요한 경우에 용이하다.

-  서버사이드렌더링시, 서버에서 fetch한 데이터 기반으로 initialState를 초기화해주고, 이렇게 만든 Store객체를 Context Provider value로 제공하여, Context API와 useStore훅을 함께 사용하는 방식으로 전역상태를 제공
  
<br>

### React Store

> 자세한 코드는 [react.ts](https://github.com/pmndrs/zustand/blob/main/src/react.ts) 를 확인

React 상태관리 시스템 내부에서 동작할 수 있도록 `useSyncExternalStore로` vanilla Store를 감싼 상태관리

```ts
// React Store - 선언적 API
const useStore = create((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
}));

// 컴포넌트 내부에서의 자연스러운 사용
function Counter() {
  const { count, increment } = useStore();
  const selectedCount = useStore((state) => state.count); //select
  return <button onClick={increment}>{count}</button>;
}
```

선택적으로 데이터를 가저올 수 있는 `selector` 개념으로 불필요한 렌더링을 막을 수 있다.  

`useSyncExternalStore`로 React 상태관리 시스템에서 동작할 수 있도록 했고 createStore로 만든 Store에 바인딩된 새로운 useStore훅을 반환한다.

그렇다면 어떤 메커니즘으로 내부에서 상태를 구독하는걸까?  

Vanilla Store를 React Store로 만드는 함수는 아래와 같다.

```ts
// 현재 getState의 리턴값은 count와 increment 함수를 가지고 있는 객체라고 가정한다.
export function useStore(api, selector = identity) {
  const slice = React.useSyncExternalStore(
    api.subscribe, // api의 subscribe 함수가 전달된다.
    () => selector(api.getState()),
    () => selector(api.getInitialState())
  );
  React.useDebugValue(slice);
  // useStore의 리턴값은 스토어의 저장된 state로 selector 함수를 호출한 결과값이다. (참고: useSyncExternalStore는 스토어의 상태를 리턴한다.)
  return slice;
}

const createImpl = (createState) => {
  const api = createStore(createState); // 스토어에 이니셜 상태를 저장하고, 상태를 다루는 api를 받게되는 과정

  const useBoundStore = (selector) => useStore(api, selector);

  Object.assign(useBoundStore, api);

  return useBoundStore;
};
```

`useSyncExternalStore`의 간단 구현 예시이다.

```ts
import { useEffect, useState, useRef } from "react";

function useSyncExternalStore(subscribe, getSnapshot) {
  // 현재 스토어의 스냅샷을 상태로 보관
  const [snapshot, setSnapshot] = useState(getSnapshot);

  // 스냅샷과 이전 상태를 비교하기 위해 참조 변수 사용
  const snapshotRef = useRef(snapshot);

  useEffect(() => {
    // 상태가 변경될 때마다 새로운 스냅샷을 업데이트하는 함수
    const checkForUpdates = () => {
      const newSnapshot = getSnapshot();
      if (snapshotRef.current !== newSnapshot) {
        snapshotRef.current = newSnapshot;
        setSnapshot(newSnapshot);
      }
    };

    // 구독을 시작하고 컴포넌트가 마운트될 때 구독 해제 함수 반환
    const unsubscribe = subscribe(checkForUpdates);
    return unsubscribe;
  }, [subscribe, getSnapshot]);

  return snapshot;
}
```

### 상태 구독 매커니즘

1. 컴포넌트 마운트

2. `useCountStore를` 호출

3. `useSyncExternalStore`를 호출

4. 스토어에 저장된 상태의 스냅샷을 초깃값으로 `useState`로 저장

    - 스토어에 저장된 상태의 스냅샷을 가져와서, `useSyncExternalStore` 내부에서 `useState`로 관리하는 상태와 비교하고, 다르다면 `setState`를 호출하는 함수를 `subscribe` 호출 시 콜백으로 줌

   - `subscribe`를 호출하는 로직은 `useEffect`의 콜백해서 실행 (즉, `return unsubsribe` 하면 언마운트 시 클린업 구독해제 가능)

5. 컴포넌트가 구독하는 상태가 변경되는데 `api.setState` 호출하는 것을 의미한다.
     - `setState` 내부에서는 `listeners Set`을 순회하며 모두 실행하는 로직이 있음
    -  즉, 스토어에 저장된 상태의 스냅샷을 가져와서, `useSyncExternalStore` 내부에서 `useState`로 관리하는 상태와 비교하고, 다르다면 `setState`를 호출하는 함수 가 호출됨
    ```ts
    export const setState = (partial, replace) => {
      const nextState = typeof partial === "function" ? partial(state) : partial;
      if (!Object.is(nextState, state)) {
        const previousState = state;
        state =
          replace || typeof nextState !== "object" || nextState === null
            ? nextState
            : Object.assign({}, state, nextState);
        listeners.forEach((listener) => listener(state, previousState));
      }
    };
    ```

6. `setState`가 호출됐으므로 해당 컴포넌트는 리렌더링 대상이 된다.



## Zustand를 사용한 상태 렌더링

react store를 사용했을 때 어떻게 성능 최적화를 시킬 수 있을지에 대해서 알아보자.

```ts
// Bad: 구조 분해 할당
const Component = () => {
  const { name, company } = useUserStore();
  console.log("rerender!");
  return (
    <div>
      {name}: {company}
    </div>
  );
};

// age만 변경
useUserStore.increaseAge();

// name을 사용하지 않는데도 리렌더링 발생! ("rerender!" 출력)
```

위 예시처럼 구조분해 할당을 사용해서 selector없이 호출하게 되면 store전체를 구독하게 된다.

 ame, company가 아닌 다른 값이 변경되어도 의도치 않은 리렌더링이 발생하게 된다.

이런 경우는 selector를 사용해서 필요한 값만 선택해서 가져오자.

```ts
const Component = () => {
  const name = useUserStore((state) => state.name);
  return <div>{name}</div>;
};

useUserStore.increaseAge();
```

Selector가 아닌 구조분해 할당으로 여러 state를 가져와서 효율적으로 관리하려면 어떻게 해야할까?  

이럴 때 `useShallow`로 의도치않은 리렌더링을 막아보자.

[공식 문서](https://github.com/pmndrs/zustand?tab=readme-ov-file#selecting-multiple-state-slices)에서는 useShallow를 사용하여 여러 state를 얻거나, 단일 상태를 구독하는 커스텀 훅을 재사용 할것을 권장하고 있다.

```ts
// 1️⃣ 컴포넌트에서 useShallow로 여러 상태 구독
const Component = () => {
  const { name, company } = useUserStore(
    useShallow((state) => ({
      name: state.name,
      company: state.company,
    }))
  );
  return (
    <div>
      {name}: {company}
    </div>
  );
};

// 2️⃣ 단일 상태를 추출하는 커스텀 훅 생성
export const useName = () => useUserStore((state) => state.name);
export const useCompany = () => useUserStore((state) => state.company);

const Component = () => {
  // 컴포넌트에서 selector 작성 로직이 제거됨
  const name = useName();
  const company = useCompany();
  return (
    <div>
      {name}: {company}
    </div>
  );
};
```

`useShallow`는 3가지 비교 방식을 사용한다.  

단, 얕은 비교만 수행하게 되므로 중첩된 객체의 깊은 변화는 감지할 수 없다.

```ts
// 1. Object.is 를 사용한 얕은 비교
if (Object.is(valueA, valueB)) {
  return true;
}

if (
  typeof valueA !== "object" ||
  valueA === null ||
  typeof valueB !== "object" ||
  valueB === null
) {
  return false;
}

// 2. 객체 비교
const compareEntries = (
  valueA: { entries(): Iterable<[unknown, unknown]> },
  valueB: { entries(): Iterable<[unknown, unknown]> }
) => {
  // Map 인스턴스이면 그대로 사용, 일반 객체의 경우 `Object.entries()`로 얻은 key-value 쌍을 Map으로 변환
  const mapA = valueA instanceof Map ? valueA : new Map(valueA.entries());
  const mapB = valueB instanceof Map ? valueB : new Map(valueB.entries());

  // 객체의 크기(프로퍼티 개수)가 다르면 false
  if (mapA.size !== mapB.size) {
    return false;
  }

  // 각 프로퍼티의 값을 Object.is로 비교
  for (const [key, value] of mapA) {
    if (!Object.is(value, mapB.get(key))) {
      return false;
    }
  }
  return true;
};

// 3. 배열과 이터러블 객체 비교
const compareIterables = (
  valueA: Iterable<unknown>,
  valueB: Iterable<unknown>
) => {
  // 각각의 Iterator 가져오기
  const iteratorA = valueA[Symbol.iterator]();
  const iteratorB = valueB[Symbol.iterator]();

  // next 메서드로 순차적으로 값에 접근
  let nextA = iteratorA.next();
  let nextB = iteratorB.next();

  // 두 Iterator를 동시에 순회하면서 Object.is로 값 비교
  while (!nextA.done && !nextB.done) {
    if (!Object.is(nextA.value, nextB.value)) {
      return false;
    }
    nextA = iteratorA.next();
    nextB = iteratorB.next();
  }

  // 둘 다 순회가 끝났는지 확인
  return !!nextA.done && !!nextB.done;
};
```

## Zustand의 불변성 및 비동기 데이터 관리

[이지훈 - zustand불변성(Immutability)과 valtio에 대해서](https://github.com/sipe-team/3-1_all_in_one_state/blob/main/3%EC%A3%BC%EC%B0%A8/%EC%9D%B4%EC%A7%80%ED%9B%88/3%EC%A3%BC%EC%B0%A8_%EC%9D%B4%EC%A7%80%ED%9B%88_%EA%B0%9C%EC%9D%B8.md)  
[조명근 - Zustand에서의 비동기 상태 관리 패턴 (async actions)](https://github.com/sipe-team/3-1_all_in_one_state/blob/main/3%EC%A3%BC%EC%B0%A8/%EC%A1%B0%EB%AA%85%EA%B7%BC/3%EC%A3%BC%EC%B0%A8_%EC%A1%B0%EB%AA%85%EA%B7%BC_%EA%B0%9C%EC%9D%B8.md)

Zustand는 Client 상태관리에 적합하다. Server 상태가 필요한건 react-query 등 다른 서버 상태관리 툴을 참고하자.

Client, Server 상태 관리를 Zustand로 같이 한다고 했을 때, 어떻게 불변성을 보장할 수 있을까?

불변성을 보장할 때, immer는 좋은 선택지 중 하나가 될 수 있다.  
하지만 immer는 완전한 불변성을 보장할까?

이 관점은 DX를 포함한 관점에 가깝다. immer의 철학과 원리만 보았을 때, 객체에서 관리되는 property 각각의 불변성을 보장하는건 가능하지만, 만약 3depth의 object 구조를 가지고 있다면? 불변성을 완전히 보장하기에는 코드가 복잡해지고 확장성이 떨어질 것 이다. 

zustand에서 미들웨어에서는 1차원 object의 불변성만 보장하기 때문에 깊은 object 구조에 대한 immer의 불변성은 보장하기 어렵다.

```ts
// zustand immer 미들웨어의 내부 동작 코드
const immerImpl: ImmerImpl = (initializer) => (set, get, store) => {
  type T = ReturnType<typeof initializer>;

  store.setState = (updater, replace, ...a) => {
    const nextState = (
      typeof updater === "function" ? produce(updater as any) : updater
    ) as ((s: T) => T) | T | Partial<T>; // 1차원 객체 property의 불변성만 보장한다.

    return set(nextState, replace as any, ...a);
  };

  return initializer(store.setState, get, store);
};

//-----------------------------------

// 사용 예시
import create from "zustand";
import { immer } from "zustand/middleware/immer";

const useStore = create(
  immer((set) => ({
    users: [],
    addUser: (user) =>
      set((state) => {
        state.users.push(user);
      }),
  }))
);
```

이런 문제때문에 Poimandres 집단에서는 valtio라는 새로운 불변성 보장 오픈소스를 만들었다.

valtio와 Immer모두 proxy기반이지만 조금 다르다.  

immer는 상태 업데이트시에 일시적으로 proxy를 생성하지만, valito는 상태를 처음 만들 때 부터 proxy로 래핑하고 지속적으로 유지한다.  

결과적으로 깊은 depth의 객체도 좋은 DX를 유지하면서 불변성을 유지하기에 용이하다.  

자동적으로 valtio가 객체의 불변성을 보장하는건 아니지만, zustand의 set 처럼 특정 메서드를 통하지 않더라도 Proxy를 사용해 개발자가 직접 property를 수정해도 리렌더링이 일어날 수 있도록 한다. (깊은 Object구조의 단일 데이터 변경에 용이하다.)

```ts
import React from "react";
import { proxy, useSnapshot } from "valtio";

const state = proxy({
  user: {
    profile: {
      name: "Alice",
      age: 25,
    },
  },
});

function App() {
  const snap = useSnapshot(state); // 상태 스냅샷 가져오기

  return (
    <div>
      <h1>{snap.user.profile.name}</h1>
      <button onClick={() => (state.user.profile.name = "Bob")}>
        Change Name
      </button>
    </div>
  );
}

export default App;
```
