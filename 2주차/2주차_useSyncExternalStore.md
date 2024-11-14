# useSyncExternalStore (이지훈, 성지현)

외부 상태 관리 스토어와 React의 동시성 렌더링(Concurrent Rendering)을 안전하게 연동하기 위해 설계되었다.

<br>

## Tearing 현상

Tearing 현상은 React의 Concurrent 렌더링 환경에서 발생할 수 있는 UI 불일치 현상을 의미한다.(간혹 동일한 데이터를 보여주는 UI 컴포넌트들이 서로 다른 값을 표기하는 버그)

이 현상은 Concurrent mode에서 useEffect를 사용한 subscribe이 비동기적으로 처리되어, 렌더링이 중단될 수 있다.

이렇게되면 서로다른 사용자마다 서로다른 데이터를 보게됨(금융, 병원등 데이터 일관성이 중요할 때 치명적 오류)

이런 문제를 useSyncExternalStore로 해결할 수 있다.

## useSyncExternalStore에 대해서

```tsx
function Counter1() {
  const count = useSyncExternalStore(
    store.subscribe, // 구독 함수
    store.getValue // 값을 가져오는 함수
  );

  return <div>Count: {count}</div>;
}

function Counter2() {
  const count = useSyncExternalStore(store.subscribe, store.getValue);

  return <div>Count: {count}</div>;
}
```

변경하고나서는 상태 업데이트가 동기적으로 처리됨

React가 렌더링 도중 스토어 값이 변경되면 즉시 감지하고 모든 컴포넌트가 동일한 시점의 데이터를 보여준다.

이렇게 useSyncExternalStore을 이용해 외부 상태를 구독할 때 useEffect를 사용하는 방식의 한계와 상태 업데이트 동기화 문제를 해결함.

<br>

## 내부 동작코드 살펴보기

```tsx
const snapshot = useSyncExternalStore(
  subscribe: (onStoreChange: () => void) => () => void,
  getSnapshot: () => T,
  getServerSnapshot?: () => T
);

```

- subscribe

  - 스토어 구독을 설정하는 함수로 스토어 변경 시 호출될 콜백을 받는다.
  - 구독 해제 함수를 반환해야한다.
  - 렌더링 간에 안정적인 참조를 유지해야 한다.

- getSnapshot

  - 스토어의 현재 상태를 반환하는 함수다. 스토어가 변경되지 않았다면 동일한 값을 반환해야 한다.
  - 반환값은 불변(immutable)이어야 함

- getServerSnapshot (optional)

  - SSR 시 사용될 초기 상태를 반환하는 함수
  - 서버와 클라이언트 간 일관된 상태 유지에 사용

<br>

## useSyncExternalStore 동작 순서

1. 초기 렌더링 발생

   - getSnapshot으로 초기 값 획득 → inst 객체 생성 및 상태 초기화 → useLayoutEffect에서 동기화 확인 → useEffect에서 구독 설정

2. 업데이트 발생

   - 외부 스토어 변경 → handleStoreChange 호출 → 스냅샷 확인 → 필요시 리렌더링

3. 클린업

   - useEffect의 클린업 함수 실행 → 구독 해제

<br>

## 주요 특징

- 동기적 업데이트 : 외부 스토어의 변경사항은 항상 동기적으로 처리
- startTransition으로 래핑되어도 동기적 처리 유지
- React 상태 업데이트와의 일관성 보장

<br>

## 중요 포인트

- getSnapshot 결과는 반드시 캐시
  - React 에서는 Object.is로 비교한다.
  - 캐싱 전략을 사용해 메모리 사용량과 성능의 균현을 맞춰야한다.
- subscribe 함수는 안정적인 참조를 유지
- 상태는 불변성을 유지

<br>

## 깃헙 코드 뜯어보기

```typescript
function useSyncExternalStore(subscribe, getSnapshot) {
  const value = getSnapshot();
  const [{ inst }, forceUpdate] = useState({ inst: { value, getSnapshot } });

  useLayoutEffect(() => {
    inst.value = value;
    inst.getSnapshot = getSnapshot;

    if (checkIfSnapshotChanged(inst)) {
      forceUpdate({ inst });
    }
  }, [subscribe, value, getSnapshot]);

  useEffect(() => {
    if (checkIfSnapshotChanged(inst)) {
      forceUpdate({ inst });
    }

    const handleStoreChange = () => {
      if (checkIfSnapshotChanged(inst)) {
        forceUpdate({ inst });
      }
    };

    return subscribe(handleStoreChange);
  }, [subscribe]);

  return value;
}
```

### 초기에 useSyncExternalStore 설정

```typescript
function useSyncExternalStore(subscribe, getSnapshot) {
  const value = getSnapshot(); // 현재 스냅샵을 가져옴

  // 컴포넌트의 상태로 인스턴스 객체를 관리
  // useState의 초기값으로 inst 객체를 생성
  const [{ inst }, forceUpdate] = useState({
    inst: {
      value, // 현재 스냅샷 값
      getSnapshot, // 스냅샷을 가져오는 함수
    },
  });
}
```

렌더링 간에 지속적으로 참조해야하는 값들을 저장하기 위해서 inst 객체를 useState로 관리한다.

forceUpdate를 통해 컴포넌트의 리렌더링을 트리거할 수 있다.

### Layout Effect에서의 동기화(Tearing 방지를 위한 핵심 메커니즘)

```typescript
useLayoutEffect(() => {
  // inst 객체 업데이트
  inst.value = value;
  inst.getSnapshot = getSnapshot;

  // 스냅샷 변경 확인 및 리렌더링
  if (checkIfSnapshotChanged(inst)) {
    forceUpdate({ inst });
  }
}, [subscribe, value, getSnapshot]);
```

DOM 업데이트 전에 동기적으로 실행되어야 하기 때문에 useLayoutEffect를 사용한다.

checkIfSnapshotChanged 함수를 이용해 렌더링 과정에서 스냅샷이 변경되었는지 즉시 확인한다.

### Effect에서 구독 설정

```typescript
useEffect(() => {
  // 초기 스냅샷 변경 확인
  if (checkIfSnapshotChanged(inst)) {
    forceUpdate({ inst });
  }

  // 스토어 변경 핸들러
  const handleStoreChange = () => {
    if (checkIfSnapshotChanged(inst)) {
      forceUpdate({ inst });
    }
  };

  // 구독 설정 및 정리 함수 반환
  return subscribe(handleStoreChange);
}, [subscribe]);
```

스토어 변경 시 스냅샷 변경 확인 후 필요한 경우만 리렌더링하고 컴포넌트 언마운트 시 자동으로 구독 정리한다.

### 스냅샷 변경 확인 로직

```typescript
function checkIfSnapshotChanged(inst) {
  const latestGetSnapshot = inst.getSnapshot;
  const prevValue = inst.value;

  try {
    const nextValue = latestGetSnapshot();
    return !Object.is(prevValue, nextValue);
  } catch (error) {
    return true; // 에러 발생 시 변경된 것으로 간주
  }
}
```

Object.is를 활용해서 정확한 값 비교를 한다. 이 과정에서 에러 처리도 포함되어있고 불변성을 전제로 코드가 작성되어있다.

<br>

## 참고자료

- [React-v18-useSyncExternalStore](https://react.dev/blog/2022/03/29/react-v18#usesyncexternalstore)
- [useSyncExternalStore](https://ko.react.dev/reference/react/useSyncExternalStore)
- [useMutableSource → useSyncExternalStore](https://github.com/reactwg/react-18/discussions/86)
- [useSyncExternalStore- Github 코드](https://github.com/facebook/react/blob/main/packages/use-sync-external-store/src/useSyncExternalStoreShim.js)
- [useSyncExternalStoreShimClient.js의 핵심 로직](https://github.com/facebook/react/blob/main/packages/use-sync-external-store/src/useSyncExternalStoreShimClient.js)
