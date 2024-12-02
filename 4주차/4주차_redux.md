# Redux의 구현체 분석과 현대적 미들웨어 패턴

## 1. Redux의 핵심 구현체

### 1.1 createReducer와 createSlice 동작 원리

> 자세한 내용은 [심현준 - createReducer , createSlice 동작 원리](https://github.com/sipe-team/3-1_all_in_one_state/blob/main/4%EC%A3%BC%EC%B0%A8/%EC%8B%AC%ED%98%84%EC%A4%80/4%EC%A3%BC%EC%B0%A8_%EC%8B%AC%ED%98%84%EC%A4%80_%EA%B0%9C%EC%9D%B8.md#createreducer--createslice-%EB%8F%99%EC%9E%91-%EC%9B%90%EB%A6%AC)에 대한 내용을 참고해주세요.

Redux를 사용해본 개발자라면 액션 타입 정의, 액션 생성자 함수 작성, 리듀서 구현 등 반복적인 코드 작성에 피로를 느꼈을 것입니다. Redux Toolkit은 이러한 문제를 해결하기 위해 등장했습니다.

#### Redux 생태계의 구성
- **redux**: vanilla 환경에서 사용 가능한 코어 라이브러리
- **react-redux**: Redux를 React에서 사용할 수 있게 해주는 바인딩
  - useSelector, useDispatch 제공
- **@reduxjs/toolkit**: Redux + React-Redux + Redux-Thunk + 보일러플레이트 개선
  - redux-thunk: 간단한 사이드이펙트 처리용 미들웨어
  - redux-saga: 복잡한 사이드이펙트 처리용 미들웨어 (generator 문법 사용)

#### createReducer의 개선사항
1. 기존 Redux 방식
    ```typescript
    // 1. 액션 타입 정의
    const INCREMENT = "counter/increment";
    const DECREMENT = "counter/decrement";
    const ADD_BY_AMOUNT = "counter/addByAmount";

    // 2. 액션 생성자 함수 정의
    const increment = () => ({ type: INCREMENT });
    const decrement = () => ({ type: DECREMENT });
    const addByAmount = amount => ({
      type: ADD_BY_AMOUNT,
      payload: amount
    });

    // 3. 리듀서 함수 구현
    const counterReducer = (state = { value: 0 }, action) => {
      switch (action.type) {
        case INCREMENT:
          return { value: state.value + 1 };
        case DECREMENT:
          return { value: state.value - 1 };
        case ADD_BY_AMOUNT:
          return { value: state.value + action.payload };
        default:
          return state;
      }
    };
    ```
    - 액션 타입 변수 선언 및 액션 생성자 함수 작성
    - action에 대해 switch-case 문으로 분기 처리를 하여 **반복적인 보일러플레이트**가 발생
    - 직접적인 불변성 처리를 필요로 하여 **불변성 관리의 어려움** 

2. createReducer를 사용한 방식
    ```typescript
    const counterReducer = createReducer({ value: 0 }, builder => {
      builder
        .addCase("counter/increment", state => {
          state.value += 1;  // Immer 덕분에 직접 수정 가능
        })
        .addCase("counter/decrement", state => {
          state.value -= 1;
        });
    });
    ```

    주요 개선점으로는 ...
    - builder 객체 문법으로 switch-case 제거
    - 내부적으로 Immer를 통한 간편한 불변성 관리

#### createSlice를 통한 통합적 접근

createReducer에서 한 단계 더 나아가 createSlice는 관련 로직을 하나의 "조각"으로 상태를 관리합니다.

```typescript
const counterSlice = createSlice({
  name: 'counter',
  initialState: { value: 0 },
  reducers: {
    increment(state) {
      state.value++;
    },
    decrement(state) {
      state.value--;
    },
    incrementByAmount(state, action: PayloadAction<number>) {
      state.value += action.payload;
    },
  },
});

export const { increment, decrement, incrementByAmount } = counterSlice.actions;
```

`createSlice`를 통해 개선된 점은 ...
- 액션 타입, 생성자, 리듀서를 한 번에 정의
- 도메인별로 관련 로직을 하나의 "슬라이스"로 관리
- 액션 자동 생성

### 1.2 상태 트리 통합

> 자세한 내용은 [성지현 - combineReducers 의 내부 구현 분석 ](https://github.com/sipe-team/3-1_all_in_one_state/blob/main/4%EC%A3%BC%EC%B0%A8/%EC%84%B1%EC%A7%80%ED%98%84/4%EC%A3%BC%EC%B0%A8_%EC%84%B1%EC%A7%80%ED%98%84.md)에 대한 내용을 참고해주세요.

Redux는 단일 스토어 원칙을 따르지만, 실제 애플리케이션에서는 여러 개의 리듀서로 상태를 관리하게 됩니다. 이러한 경우에 사용하는 `combineReducers`는 여러 개의 리듀서를 하나의 루트 리듀서로 통합하여 단일 스토어로 관리합니다.

```typescript
// combineReducers 사용 사례
const rootReducer = combineReducers({
  todos: todosReducer,
  filters: filtersReducer,
});
```

실제 내부 구현을 분석하면 다음과 같습니다.

```typescript
export default function combineReducers(reducers) {
  const reducerKeys = Object.keys(reducers);
  const finalReducers = {};
  
  // 리듀서 검증 및 필터링
  for (let i = 0; i < reducerKeys.length; i++) {
    const key = reducerKeys[i];
    // 리듀서가 함수일 경우 finalReducers에 추가 (리듀서는 함수여야 함)
    if (typeof reducers[key] === 'function') {
      finalReducers[key] = reducers[key];
    }
  }

  // 2. 초기 상태 검증
  let shapeAssertionError;
  try {
    assertReducerShape(finalReducers);
  } catch (e) {
    // 리듀서의 형태가 잘못되었을 경우 에러 처리
    shapeAssertionError = e;
  }
  
  // 3. 통합 리듀서 반환
  return function combination(state = {}, action) {
    let hasChanged = false;
    const nextState = {};
    
    // finalReducers에 담긴 리듀서들을 순회하며 state를 갱신
    for (let i = 0; i < finalReducerKeys.length; i++) {
      const key = finalReducerKeys[i];  // state 이름
      const reducer = finalReducers[key]; // state에 대한 리듀서
      const previousStateForKey = state[key]; // 스토어에 저장된 state 값
      const nextStateForKey = reducer(previousStateForKey, action); // 해당 state에 대한 리듀서 실행 결과
      
      nextState[key] = nextStateForKey; // 상태 업데이트 (스토어에 갱신된 state 저장)
      hasChanged = hasChanged || nextStateForKey !== previousStateForKey; // 상태 변경 여부 확인
    }
    
    return hasChanged ? nextState : state;
  };
}
```

`createStore`로 단일 스토어를 구성할 때에는 리듀서를 통합하기 위해 `combineReducers`를 호출해야 합니다.

## 2. RTK와 현대적 상태 관리

### 2.1 Redux Toolkit의 내부 구현과 최적화 전략

> 자세한 내용은 [심미진 - Redux Toolkit의 내부 구현과 최적화 전략](https://github.com/sipe-team/3-1_all_in_one_state/blob/main/4%EC%A3%BC%EC%B0%A8/%EC%8B%AC%EB%AF%B8%EC%A7%84/4%EC%A3%BC%EC%B0%A8_%EC%8B%AC%EB%AF%B8%EC%A7%84_%EA%B0%9C%EC%9D%B8.md)에 대한 내용을 참고해주세요.

Redux Toolkit은 Redux의 여러 문제점들을 해결하기 위해 등장했습니다.
- **Redux**: 순수 자바스크립트 환경의 상태 관리자
- **React-redux**: Redux를 React에서 사용하기 위한 바인딩
- **Redux Toolkit**: Redux 사용성 개선을 위한 공식 도구 모음

#### RTK가 해결하고자 한 문제들
- Redux의 복잡한 초기 설정
- 불변성 관리의 어려움
- 과도한 보일러플레이트 코드

#### 7가지 주요 API와 그 역할

1. **configureStore**: 스토어 설정 자동화
    ```typescript
    const store = configureStore({
      reducer: rootReducer,
      middleware: (getDefaultMiddleware) => 
        getDefaultMiddleware().concat(logger)
    });
    ```
    - Redux DevTools 자동 설정
    - redux-thunk 기본 포함
    - 개발 모드 미들웨어 자동 구성

2. **createAction & createReducer**: 액션과 리듀서 생성 간소화
    ```typescript
    // createAction으로 간단한 리듀서 함수 생성
    const todosReducer = createReducer((state = []), (builder) => {
      builder.addCase("UPDATE_VALUE", (state, action) => {
        const { someId, someValue } = action.payload;

        state.first.second[someId].fourth = someValue;
      });
    });
    ```

3. **createAction** : 액션 타입과 생성자 함수를 하나로 통합
    ```typescript
    // 기존 방식의 복잡성
    const INCREMENT = "counter/increment";
    const increment = (amount) => ({
      type: INCREMENT,
      payload: amount,
    });

    // RTK의 간단한 방식
    const increment = createAction<number>('counter/increment');
    ```

4. **createSlice**: 도메인별 상태 관리
    ```typescript
    const alertSlice = createSlice({
      name: "todos",
      initialState,
      reducers: {},
      extraReducers: (builder) => {},
    });
    ```

5. **createAsyncThunk**: : `createAction` 비동기 버전을 위해 제안됨
    ```typescript
    const fetchUserById = createAsyncThunk(
      'users/fetchById',
      async (userId: string) => {
        const response = await api.fetchUser(userId);
        return response.data;
      }
    );
    ```

6. **createSelector**: Redux 스토어 상태 데이터 선택

7. **createEntityAdapter**: 정규화된 상태 관리

### 2.2 상태 정규화와 데이터 관리

> 자세한 내용은 [류지예 - Redux의 상태 정규화 패턴과 구현](https://github.com/sipe-team/3-1_all_in_one_state/blob/main/4%EC%A3%BC%EC%B0%A8/%EB%A5%98%EC%A7%80%EC%98%88/4%EC%A3%BC%EC%B0%A8_%EB%A5%98%EC%A7%80%EC%98%88_%EA%B0%9C%EC%9D%B8.md)에 대한 내용을 참고해주세요.

#### 상태 정규화의 필요성
```typescript
// 중첩된 상태 구조의 예
const state = {
  posts: [
    {
      id: 1,
      title: "Redux 시작하기",
      author: {
        id: 1,
        name: "김철수",
        posts: [1, 2, 3]  // 데이터 중복
      },
      comments: [/* 중첩된 댓글 데이터 */]
    }
  ]
};
```
중첩된 데이터 구조의 문제점들은 ...
1. 데이터 업데이트의 복잡성
2. 불필요한 리렌더링
3. 상태 일관성 유지의 어려움

#### createEntityAdapter를 통한 해결
```typescript
const usersAdapter = createEntityAdapter<User>({
  selectId: (user) => user.id,
  sortComparer: (a, b) => a.name.localeCompare(b.name)
});

// createSlice와 함께 사용하여 상태 정규화와 업데이트를 간단하게 통합
const usersSlice = createSlice({
  name: 'users',
  initialState: usersAdapter.getInitialState(),
  reducers: {
    userAdded: usersAdapter.addOne,
    userUpdated: usersAdapter.updateOne,
  }
});
```

1. 표준화된 상태 구조
   - 표준화된 상태 구조와 액세스 패턴

2. 자동화된 CRUD 작업
   - addOne/addMany
   - updateOne/updateMany
   - removeOne/removeMany

3. 편리한 데이터 접근
   ```typescript
   const {
     selectAll,
     selectById,
     selectIds
   } = postsAdapter.getSelectors();
   ```

이러한 RTK의 기능들은 Redux의 강력함은 유지하면서, 개발자 경험을 크게 개선했습니다.

## 3. 미들웨어를 통한 문제 해결

### 3.1 Redux 미들웨어의 역할

> 자세한 내용은 [조명근 - Redux의 미들웨어가 해결하려고 했던 것들은 다른 라이브러리는 어떻게 해결하려 했을까?](https://github.com/sipe-team/3-1_all_in_one_state/blob/main/4%EC%A3%BC%EC%B0%A8/%EC%A1%B0%EB%AA%85%EA%B7%BC/4%EC%A3%BC%EC%B0%A8_%EC%A1%B0%EB%AA%85%EA%B7%BC_%EA%B0%9C%EC%9D%B8.md)에 대한 내용을 참고해주세요.

Redux의 미들웨어는 dispatch와 reducer 사이에서 동작하는 함수로, Action이 Store에 도달하기 전에 중간에서 특정 작업을 수행합니다. 이는 주로 관심사 분리를 위해 도입되었습니다.

![redux 동작](https://cdn.frontoverflow.com/document/first-met-redux/images/chapter_02/redux_data_flow.gif)

#### 미들웨어가 필요한 이유

미들웨어 없이 비동기 작업을 처리하면 다음과 같은 문제가 발생합니다:

```javascript
const App = () => {
  const { data, loading, error } = useSelector((state) => state);
  const dispatch = useDispatch();

  const fetchData = async () => {
    dispatch({ type: FETCH_START });
    try {
      const response = await fetch("https://api.example.com/data");
      const result = await response.json();
      dispatch({ type: FETCH_SUCCESS, payload: result });
    } catch (err) {
      dispatch({ type: FETCH_ERROR, payload: err.message });
    }
  };

  return (/* ... */);
};
```

위와 같은 코드의 문제점은 ...
1. 관심사 분리 부족
2. 코드 중복 발생

#### 미들웨어를 통한 해결
```javascript
// API 미들웨어 예시
const apiMiddleware = store => next => action => {
  if (action.type !== 'FETCH_USER') return next(action);

  store.dispatch({ type: 'SET_LOADING', payload: true });

  return api.fetchUser(action.payload)
    .then(response => {
      store.dispatch({ type: 'FETCH_SUCCESS', payload: response.data });
    })
    .catch(error => {
      store.dispatch({ type: 'FETCH_ERROR', payload: error });
    })
    .finally(() => {
      store.dispatch({ type: 'SET_LOADING', payload: false });
    });
};
```

### 3.2 현대 라이브러리의 미들웨어 접근 방식

> 자세한 내용은 [최여진 - 각 상태 관리 라이브러리가 Redux 미들웨어를 사용하던데 어느면에서 호환성이 좋은가?](https://github.com/sipe-team/3-1_all_in_one_state/blob/main/4%EC%A3%BC%EC%B0%A8/%EC%B5%9C%EC%97%AC%EC%A7%84/4%EC%A3%BC%EC%B0%A8_%EC%B5%9C%EC%97%AC%EC%A7%84_%EA%B0%9C%EC%9D%B8.md)에 대한 내용을 참고해주세요.

#### Redux의 미들웨어 패턴
```javascript
// 기존 상태 변화 흐름
action → reducer ⇢ state 변경

// 미들웨어를 사용한 상태 변화 흐름
action → middleware1 → middleware2 → reducer ⇢ state 변경
```
Redux의 미들웨어는 상태가 변경되기 전에 여러 가지 작업을 단계적으로 처리할 수 있게 해주는 기능입니다.

Redux의 미들웨어 도입으로 ...
1. 하나의 역할을 담당하여 상태 변화를 단계적으로 처리
2. 코드의 재사용성과 유지보수성을 높이고, 
3. 관심사를 분리할 수 있습니다.

각 라이브러리는 이러한 Redux의 미들웨어 패턴에서 영감을 받아 자신만의 방식으로 발전시켰습니다.

#### Zustand
```typescript
const useStore = create(
  devtools(
    persist(
      (set) => ({
        count: 0,
        increase: () => set(state => ({ count: state.count + 1 }))
      })
    )
  )
);
```
- 함수 합성 방법을 통한 미들웨어 체이닝 지원
- devtools, persist, immer 등 내장 미들웨어 제공

#### Jotai
```typescript
const countAtom = atomWithDevtools(
  atomWithStorage('count', 0),
  'count'
);
```

- `atomWithStorage`, `atomWithDevtools` 등의 내장 유틸을 제공
- 유틸을 조합하여 체이닝 방식으로 미들웨어 적용
- 유틸리티 기반의 접근

#### Recoil
```typescript
const loggingEffect = ({ setSelf, onSet }) => {
  onSet((newValue, oldValue) => {
    console.log('State changed:', { oldValue, newValue });
  });
};

const persistEffect = ({ setSelf, onSet }) => {
  const savedValue = localStorage.getItem('count');
  if (savedValue != null) {
    setSelf(JSON.parse(savedValue));
  }

  onSet((newValue) => {
    localStorage.setItem('count', JSON.stringify(newValue));
  });
};

const countState = atom({
  key: 'count',
  default: 0,
  effects: [loggingEffect, persistEffect]
});
```

- Atom Effects를 통한 사이드 이펙트 처리
- Atom 단위로 효과를 적용할 수 있음
- 상태 변화를 한 번에 처리하기 어려움

## 4. Redux의 현재: 변화하는 상태관리 트렌드

> 자세한 내용은 [이지훈 - 전역 상태 관리의 시초인 Redux의 구조와 다른 라이브러리의 구조와 인터페이스 비교 및 왜 옮겨가는가?](https://github.com/sipe-team/3-1_all_in_one_state/blob/main/4%EC%A3%BC%EC%B0%A8/%EC%9D%B4%EC%A7%80%ED%9B%88/4%EC%A3%BC%EC%B0%A8_%EC%9D%B4%EC%A7%80%ED%9B%88_%EA%B3%B5%ED%86%B5.md)에 대한 내용을 참고해주세요.

Redux는 강력한 기능과 명확한 패턴을 제시했지만, 최근 개발자들은 다른 상태관리 라이브러리들을 선택하는 경향을 보이고 있습니다. 그 이유를 간단히 살펴보겠습니다.

### 보일러플레이트와 러닝 커브

- 보일러플레이트
   - Redux를 설정하고 사용하려면 많은 상용구 코드가 필요
   - Action/Action Creator/Reducer/Selector/Store/MiddleWare 등을 정의해야 함
   - HOC, useSelector, useDispatch 등의 사용이 필요

- 러닝커브
   - 학습 곡선이 가파르며 불변성, 순수 함수, 미들웨어 등의 고급 개념을 이해해야 함
    - 불변성, 순수 함수, 미들웨어 등의 제약이 있음
    - 미들웨어 체인의 동작 방식에 대한 이해도가 필요
    - Reducer, selector를 활용한 함수형 프로그래밍 지식이 필요

### 번들 크기와 메모리
- Zustand: ~1KB vs Redux + React-Redux: ~11KB
- 초기 메모리 사용량도 Redux가 현대 라이브러리들보다 2-5배 더 많음

### React의 새로운 패러다임
React 18의 동시성 모드, Suspense 등 새로운 기능들과의 통합이 현대 라이브러리들에서 더 자연스럽게 이루어지고 있습니다.
