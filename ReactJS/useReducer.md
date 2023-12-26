# useReducer

> state값을 변경하는 시나리오를 제한적으로 두고 이에 대한 변경을 빠르게 확인할 수 있게끔 하는 것이 useReducer의 목적이다.

useState의 심화 버전으로 볼 수 있다. useState와 비슷한 형태를 띠지만 좀 더 복잡한 상태값을 미리 정의해 놓은 시나리오에 따라 관리할 수 있다.

- 반환값은 useState와 동일하게 길이가 2인 배열이다.

  - `state` : 현재 useReducer가 가지고 있는 값을 의미한다. useState와 마찬가지로 배열을 반환하는데, 동일하게 첫 번째 요소가 이 값이다.
  - `dispatcher` : state를 업데이트하는 함수, useReducer가 반환하는 배열의 두 번째 요소다. setState는 단순히 값을 넘겨주지만 여기서는 action을 넘겨준다는 점이 다르다. 이 action은 state를 변경할 수 있는 액션을 의미한다.

- useState의 인수와 달리 2개에서 3개의 인수를 필요로 한다.
  - `reducer` : useReducer의 기본 action을 정의하는 함수다. 이 reducer는 useReducer의 첫 번째 인수로 넘겨주어야 한다.
  - `initialState` : 두 번째 인수로, useReducer의 초깃값을 의미한다.
  - `init` : useState의 인수로 함수를 넘겨줄 때처럼 초깃값을 지연해서 생성시키고 싶을 때 사용하는 함수다. 이 함수는 필수값이 아니며, 만약 여기에 인수로 넘겨주는 함수가 존재한다면 useState와 동일하게 게으른 초기화가 일어나며 initialState를 인수로 init함수가 실행된다.

```jsx
// useReducer가 사용할 state를 정의
type State = {
  count: number,
};

// state의 변화를 발생시킬 action의 타입과 넘겨줄 값(payload)을 정의
// 꼭 type과 payload라는 네이밍을 지킬 필요도 없으며, 굳이 객체일 필요도 없다.
// 다만 이러한 네이밍이 가장 널리 쓰인다.
type Action = { type: 'up' | 'down' | 'reset', payload?: State };

// 무거운 연산이 포함된 게으른 초기화 함수
// 이 함수가 없다면 두 번째 인수로 넘겨받은 기본값을 사용하게 될 것이다.
// 게으른 초기화 함수를 넣어줌으로써 useState에 함수를 넣은 것과 같은 동일한 이점을 누릴 수 있고, 추가로 state에 대한 초기화가 필요할 때 reducer에서 이를 재사용할 수 있다는 장점도 있다.
function init(count: State): State {
  // count: State를 받아서 초깃값을 어떻게 정의할지 연산하면 된다.
  return count;
}

// 초깃값
const initialState: State = { count: 0 };

// 앞서 선언한 state와 action을 기반으로 state가 어떻게 변경될지 정의
function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'up':
      return { count: state.count + 1 };
    case 'down':
      return { count: state.count - 1 > 0 ? state.count - 1 : 0 };
    case 'reset':
      return init(action.payload || { count: 0 });
    default:
      throw new Error(`Unexpected action type ${action.type}`);
  }
}

export default function App() {
  const [state, dispatcher] = useReducer(reducer, initialState, init);

  function handleUpButtonClick() {
    dispatcher({ type: 'up' });
  }

  function handleDownButtonClick() {
    dispatcher({ type: 'down' });
  }

  function handleResetButtonClick() {
    dispatcher({ type: 'reset', payload: { count: 1 } });
  }

  return (
    <div className='App'>
      <h1>{state.count}</h1>
      <button onClick={handleUpButtonClick}>+</button>
      <button onClick={handleDownButtonClick}>-</button>
      <button onClick={handleResetButtonClick}>reset</button>
    </div>
  );
}
```

복잡한 형태의 state를 사전에 정의된 dispatcher로만 수정할 수 있게 만들어줌으로써 state값에 대한 접근은 컴포넌트 내에서만 가능하게 하고, 이를 업데이트하는 방법에 대한 상세 정의는 컴포넌트 밖에다 둔 다음, state의 업데이트를 미리 정의해 둔 dispatcher로만 제한하는 것이다.

여러 개의 state를 관리하는 것보다 때로는 성격이 비슷한 여러 개의 state를 묶어 useReducer로 관리하는 편이 더 효율적일 수도 있다. state를 사용하는 로직과 이를 관리하는 비즈니스 로직을 분리할 수 있어 state를 관리하기 쉬워진다.

## Preact가 구현한 useState

Preact의 useState 코드를 살펴보면 useReducer로 구현돼 있는 것을 알 수 있다.

```jsx
/**
 * @param {import('./index').StateUpdater<any>} [initialState]
 */
export function useState(initialState) {
  currentHook = 1;
  return useReducer(invokeOrReturn, initialState);
}
```

그렇다면 useState는 useReducer로 어떻게 구현할 수 있을까? 먼저 첫 번째 인수는 값을 업데이트하는 함수이거나 값 그 자체여야 한다.

```jsx
function reducer(prevState, newState) {
  return typeof newState === 'function' ? newState(prevState) : newState;
}
```

두 번째 인수는 초깃값이기 때문에 별다른 처리를 할 필요가 없다. 세 번째 값은 이 두 번째 값을 기반으로 한 게으른 초기화를 하는 함수다. 이것 역시 함수라면 실행해 값을 반환해야 한다.

```jsx
function init(initialArg: Initializer) {
  return typeof initialArg === 'function' ? initialArg() : initialArg;
}
```

위 두 함수를 모두 useReducer에서 사용하면 다음과 같이 useState의 작동을 흉내 낼 수 있다.

```jsx
function useState(initialArg) {
  return useReducer(reducer, initialArg, init);
}
```

## useState로 구현한 useReducer

useReducer를 useState로 구현할 수 있다.

```jsx
const useReducer = (reducer, initialArg, init) => {
  const [state, setState] = useState(
    // 초기화 함수가 있으면 초깃값과 초기화 함수를 실행하고,
    // 그렇지 않으면 초깃값을 넣는다.
    init ? () => init(initialArg) : initialArg,
  );

  // 값을 업데이트하는 dispatch를 넣어준다
  const dispatch = useCallback(
    (action) => setState((prev) => reducer(prev, action)),
    [reducer],
  );

  // 이 값을 메모이제이션한다.
  return useMemo(() => [state, dispatch], [state, dispatch]);
};
```

useReducer나 useState나 세부 작동과 쓰임에만 차이가 있을 뿐, 클로저를 활용해 값을 가둬서 state를 관리한다는 사실에는 변함이 없다.
