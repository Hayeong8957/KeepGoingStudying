# useMemo

## useMemo를 사용하는 이유

> 성능 최적화
> useMemo는 계산 비용이 높은 함수의 결과값을 메모리에 저장(캐싱)하여, 동일한 입력값에 대해 함수를 재실행하지 않고 저장된 값을 재사용할 수 있도록 합니다. 이를 통해 불필요한 계산을 줄이고, 애플리케이션의 성능을 향상시킬 수 있습니다.

```JavaScript
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

여기서 computeExpensiveValue는 비용이 많이 드는 함수이며, [a, b]는 이 함수가 의존하는 값들입니다. a와 b가 변경될 때만 함수가 재실행되고, 그렇지 않다면 이전에 계산된 값을 재사용합니다.

## 동작 방식

- 초기 렌더링 시

컴포넌트가 처음 렌더링될 때, useMemo에 전달된 콜백 함수가 실행되고, 그 결과(여기서는 객체)가 메모리에 저장됩니다.

- 의존성 배열의 값이 변경될 때

의존성 배열에 포함된 값이 변경되면, useMemo에 전달된 콜백 함수가 다시 실행됩니다. 새로 계산된 객체가 다시 메모리에 저장되고, 이 값이 반환됩니다.

- 의존성 배열의 값이 변경되지 않을 때

의존성 배열의 값이 변경되지 않으면, useMemo는 메모리에 저장된 이전 객체를 반환합니다. 이는 새로운 객체를 생성하지 않고, 메모리에 저장된 객체를 재사용하는 것을 의미합니다.

## 주의할 점

useMemo를 사용할 때 주의할 점은, 메모리에 값이 저장되므로 메모리 사용량이 증가할 수 있다는 것입니다. 따라서, 정말로 필요한 곳에만 useMemo를 사용하는 것이 좋습니다.

# useMemo Definition from 모던 리액트 딥다이브

```jsx
import { useMemo } from 'react';

const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

**첫 번째 인수로는 어떠한 값을 반환하는 생성 함수를, 두 번째 인수로는 해당 함수가 의존하는 값의 배열을 전달**한다. useMemo는 렌더링 발생 시 의존성 배열의 값이 변경되지 않았으면 함수를 재실행하지 않고 이전에 기억해 둔 해당 값을 반환하고, 의존성 배열의 값이 변경됐다면 첫 번째 인수의 함수를 실행한 후에 그 값을 반환하고 그 값을 다시 기억해 둘 것이다. 이러한 메모이제이션은 단순히 값뿐만 아니라 컴포넌트도 가능하다.

- useMemo를 사용한 컴포넌트 메모이제이션

```jsx
function 복잡한_컴포넌트({ value }) {
  useEffect(() => {
    console.log('렌더링이요~');
  });
  return <span>{value + 1000}</span>;
}

function App() {
  const [value, setValue] = useState(10);
  const [, triggerRendering] = useState(false);

  // 컴포넌트의 props를 기준으로 컴포넌트 자체를 메모이제이션했다. -> useMemo로 컴포넌트 감쌀 수 있다.
  const 메모이제이션된_컴포넌트 = useMemo(
    () => <복잡한_컴포넌트 value={value} />,
    [value],
  );

  function handleChange(e) {
    setValue(Number(e.target.value));
  }

  function handleClick() {
    triggerRendering((prev) => !prev);
  }

  return (
    <>
      <input value={value} onChange={handleChange} />
      <button onClick={handleClick}>렌더링 발생!</button>
      {메모이제이션된_컴포넌트}
    </>
  );
}
```

useMemo로 컴포넌트도 감쌀 수 있다. 물론 React.memo를 쓰는 것이 더 현명하다.

triggerRendering으로 컴포넌트 렌더링을 강제로 발생시켰지만 메모이제이션된*컴포넌트는 리렌더링되지 않는 것을 확인할 수 있다.
메모이제이션된*컴포넌트는 의존성으로 선언된 value가 변경되지 않는 한 다시 계산되는 일은 없을 것이다.
useMemo등 메모이제이션을 활용하면 무거운 연산을 다시 수행하는 것을 막을 수 있다는 장점이 있다.

useMemo는 어떠한 값을 계산할 떄 해당 값을 연산하는 데 비용이 많이 든다면 사용해 봄 직하다. 그러나 **'비용이 많이 드는 연산'**이란 뭘까? 비용은 어떻게 측정할 수 있을까?
