# React Fiber 아키텍처 상태관리 메커니즘 (심현준, 최여진)

## 1. Fiber Architecture 개념과 등장 배경

### 기존 React의 한계

React 16 버전 이전에는 동기식 스택 기반의 렌더링 엔진을 사용했으며, 다음과 같은 주요 한계가 있었다.

- **렌더링 작업의 원자성**

  - 한번 시작된 렌더링은 중단 불가능
  - 전체 컴포넌트 트리를 순차적으로 처리
  - 우선순위 조정 불가능

- **성능 문제**
  - 브라우저의 프레임 처리 제약 (16ms)
  - 메인 스레드 블로킹으로 인한 UI 응답성 저하
  - 대규모 업데이트 시 프레임 드롭 현상

위 문제를 해결하기 위한 새로운 아키텍처가 논의되었고 2016년에 [react-fiber-architecture](https://github.com/acdlite/react-fiber-architecture)이 등장하게 되었다.

위 문서만 이해해도 훌륭할 정도로 잘 정리되었으니 확인해보면 좋겠다.

<br>

### Fiber 아키텍처의 도입 목적

- 작업 분할 (Work Splitting)을 통해 렌더링 작업을 작은 단위로 분할해 중단/재개가 가능한 구조

- 중요한 업데이트 우선 처리를 통해 사용자 경험 최적화

- 효율적인 리소스 활용을 이용해 여러 렌더링 작업의 병렬 처리(동시성 처리)

<br>

## 2. Fiber의 핵심 개념

Fiber는 다음 세 가지 관점에서 이해할 수 있다.

1. **아키텍처 관점**

   - React의 새로운 재조정(reconciliation) 엔진
   - 렌더링 작업을 작은 단위로 쪼개는 메커니즘

2. **데이터 구조 관점**

   - 컴포넌트의 정보를 담는 객체
   - 가상 스택 프레임으로서의 역할

3. **작업 단위 관점**
   - 수행해야 할 작업의 최소 단위
   - 우선순위와 상태를 포함

<br>

### 주요 특징

- **비동기식 렌더링**

  - 작업을 여러 프레임에 걸쳐 분산
  - 브라우저의 idle 타임 활용

- **양방향 링크드 리스트**
  - 효율적인 트리 순회
  - 작업 중단/재개 지원

## 3. Fiber의 내부 구조

```typescript
type Fiber = {
  // 식별 정보
  tag: WorkTag; // 컴포넌트 유형
  key: null | string; // 리스트 렌더링용 key
  elementType: any; // 엘리먼트 타입
  type: any; // 컴포넌트 타입

  // 트리 구조
  return: Fiber | null; // 부모 Fiber
  child: Fiber | null; // 첫 자식 Fiber
  sibling: Fiber | null; // 다음 형제 Fiber
  index: number; // 형제들 사이에서의 인덱스

  // 상태 관리
  pendingProps: any; // 새로운 props
  memoizedProps: any; // 이전 props
  updateQueue: UpdateQueue; // 업데이트 큐
  memoizedState: any; // 이전 상태

  // 의존성
  dependencies: Dependencies | null;

  // 효과
  flags: Flags; // 사이드 이펙트 플래그
  subtreeFlags: Flags; // 하위 트리 플래그
  deletions: Array<Fiber> | null; // 삭제될 노드들

  // 더블 버퍼링
  alternate: Fiber | null; // 대체 Fiber

  // 디버깅
  _debugOwner?: Fiber | null;
  _debugSource?: Source | null;
  _debugIsCurrentlyTiming?: boolean;
};
```

### 3.2 Hook의 내부 구조

```typescript
type Hook = {
  memoizedState: any; // 현재 상태
  baseState: any; // 기본 상태
  baseQueue: Update<any> | null; // 기본 업데이트 큐
  queue: UpdateQueue<any> | null; // 현재 업데이트 큐
  next: Hook | null; // 다음 훅
};

type UpdateQueue<State> = {
  pending: Update<State> | null;
  interleaved: Update<State> | null;
  lanes: Lanes;
  dispatch: ((action: any) => any) | null;
  lastRenderedReducer: ((state: State, action: any) => State) | null;
  lastRenderedState: State | null;
};
```

## 4. 렌더링 과정

렌더링 과정은 렌더링 단계 (Render Phase)와 커밋 단계 (Commit Phase)로 구성되어있다.

그 중 렌더링 단계는 beginWork,completeWork를 이용해 업데이트와 마운트를 핸들링한다.

- beginWork

  ```typescript
  function beginWork(
    current: Fiber | null,
    workInProgress: Fiber,
    renderLanes: Lanes
  ): Fiber | null {
    if (current !== null) {
      // 업데이트 로직
    } else {
      // 마운트 로직
    }
  }
  ```

- completeWork
  ```typescript
  function completeWork(
    current: Fiber | null,
    workInProgress: Fiber,
    renderLanes: Lanes
  ): Fiber | null {
    // DOM 업데이트 준비
    // 이벤트 리스너 설정
    // 스타일 계산 등
  }
  ```

커밋 단계는 커밋준비, DOM 업데이트, 정리 작업으로 구성되어있다.

커밋 준비 단계에서는 변경사항 수집과 사이드 이펙트 정렬을 하고, DOM 업데이트 단계에서는 실제 DOM 변경 적용과 적용을 위한 레이아웃 계산을 다룬다.

정리 작업에는 이전 트리 구조를 정리하고, 메모리 정리, 참조 정리등을 통해 다음 단계를 준비한다.

<br>

## 5. 상태 관리 메커니즘

상태 관리 메커니즘은 더블 버퍼링, 우선순위 기반 업데이트, 배치 처리 등을 통해 효율적인 상태 관리를 실현하며, 동시에 예측 가능한 상태 변화와 에러 복구를 보장하는 설계 시스템이다.

주요 특징은 아래와 같은 것들이 있다.

- 연결 리스트 기반의 훅 관리

- 우선순위 기반 업데이트 처리

- 배치 처리를 통한 성능 최적화

- 견고한 에러 처리 메커니즘

- 메모이제이션을 통한 불필요한 재계산 방지

### 더블 버퍼링

```typescript
function createWorkInProgress(current: Fiber, pendingProps: any): Fiber {
  let workInProgress = current.alternate;

  if (workInProgress === null) {
    // 새로운 Fiber 생성
    workInProgress = createFiber(
      current.tag,
      pendingProps,
      current.key,
      current.mode
    );

    workInProgress.elementType = current.elementType;
    workInProgress.type = current.type;
    workInProgress.stateNode = current.stateNode;

    workInProgress.alternate = current;
    current.alternate = workInProgress;
  } else {
    // 기존 Fiber 재사용
    workInProgress.pendingProps = pendingProps;
    workInProgress.type = current.type;
    workInProgress.flags = NoFlags;
    workInProgress.subtreeFlags = NoFlags;
  }

  return workInProgress;
}
```

### 상태 업데이트 큐

```typescript
function enqueueUpdate<State>(fiber: Fiber, update: Update<State>) {
  const updateQueue = fiber.updateQueue;
  const pending = updateQueue.pending;

  if (pending === null) {
    // 새로운 업데이트 큐 생성
    update.next = update;
  } else {
    // 기존 큐에 업데이트 추가
    update.next = pending.next;
    pending.next = update;
  }
  updateQueue.pending = update;
}
```

<br>

## 6. 작업 우선순위와 스케줄링

우선순위 레벨은 아래와 같은 구조로 되어있다.

```typescript
export const priorities = {
  ImmediatePriority: 1,
  UserBlockingPriority: 2,
  NormalPriority: 3,
  LowPriority: 4,
  IdlePriority: 5,
};
```

스케줄링을 통해 작업 분류, 타임슬라이스 할당을 처리한다.

1. 작업 분류

   - 동기 작업 (Sync)
   - 비동기 작업 (Async)
   - 지연 가능 작업 (Deferred)

2. 타임슬라이스 할당
   - 각 우선순위별 최대 실행 시간 설정
   - 시간 초과 시 작업 중단

<br>

## 7. 성능 최적화

### 메모이제이션

```typescript
function memoizeValue<T>(nextValue: T, deps: Array<mixed> | void | null): T {
  if (deps !== null && deps !== undefined) {
    const prevDeps: Array<mixed> | null = prevState[0];
    if (areHookInputsEqual(deps, prevDeps)) {
      return prevState[1];
    }
  }
  return nextValue;
}
```

<br>

### 작업 재사용

- **불필요한 렌더링 방지**

  - shouldComponentUpdate
  - React.memo
  - useMemo/useCallback

- **컴포넌트 분할**
  - 적절한 크기로 컴포넌트 분리
  - 렌더링 최적화를 위한 구조화

<br>

### 성능 모니터링

```typescript
// 개발 모드에서 성능 측정
if (__DEV__) {
  const renderStartTime = performance.now();
  try {
    return renderRoot(root, lanes, options);
  } finally {
    const renderEndTime = performance.now();
    const renderDuration = renderEndTime - renderStartTime;
    logRenderDuration(renderDuration);
  }
}
```

<br>

## 참고자료

- [React Fiber Architecture](https://github.com/acdlite/react-fiber-architecture) - Andrew Clark (React 코어 팀)
- [Naver D2 - React 파이버 아키텍처 분석](https://d2.naver.com/helloworld/2690975)
- [React Fiber와 상태 관리](https://maystar8956.tistory.com/206)
- [JSer.dev - Fiber Tree Structure](https://jser.dev/react/2022/01/16/fiber-traversal-in-react/)
