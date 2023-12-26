# useCallback

> **💡 면접을 위한 quick answer :** useMemo는 Memoization된 '값'을 반환하는 함수이고, useCallback은 Memoization된 '값'이 아닌 '함수'를 반환하는 함수이다. useMemo는 함수를 실행해서 그 실행 값을 반환하지만, useCallback은 함수 자체를 반환하는 것이다.

useMemo가 값을 기억했다면, useCallback은 인수로 넘겨받은 콜백 자체를 기억한다. 쉽게 말해 useCallback은 특정 함수를 새로 만들지 않고 다시 재사용한다는 의미다.

```jsx
const 자식컴포넌트 = memo(({ name, value, onChange }) => {
  // 렌더링이 수행되는지 확인하기 위해 넣었다.
  useEffect(() => {
    console.log('rendering!', name)
  })

  return (
    <>
      <h1>{name} {value ? '켜짐' : '꺼짐'}</h1>
      <button onClick={onChange}>toggle</button>
    </h1>
  )
})

function App() {
  const [status1, setStatus1] = useState(false)
  const [status2, setStatus2] = useState(false)

  const toggle1 = () => {
    setStatus1(!status1)
  }

  const toggle2 = () => {
    setStatus2(!status2)
  }

  return(
    <>
      <자식컴포넌트 name="1" value={status1} onChange={toggle1}/>
      <자식컴포넌트 name="2" value={status2} onChange={toggle2}/>
    </>
  )
}
```

memo를 사용해서 컴포넌트를 메모이제이션했지만 App의 자식 컴포넌트 전체가 렌더링되고 있다.
정상적인 흐름이라면 하나의 value변경이 다른 컴포넌트에 영향을 미쳐서는 안 되고, 클릭할 때마다 하나의 컴포넌트만 렌더링 되어야 한다. 그러나 어느 한 버튼을 클릭하면 클릭한 컴포넌트 외에도 클릭하지 않은 컴포넌트도 렌더링되는 것을 알 수 있다. 그 이유는 state값이 바뀌면서 App 컴포넌트가 리렌더링되고, 그때마다 매번 onChange로 넘기는 함수가 재생성되고 있기 때문이다.

값의 메모이제이션을 위해 useMemo를 사용했다면, 함수의 메모이제이션을 위해 사용하는 것이 useCallback이다.
useCallback의 첫 번째 인수로 함수를, 두 번째 인수로 의존성 배열을 집어 넣으면 useMemo와 마찬가지로 의존성 배열이 변경되지 않는 한 함수를 재생성하지 않는다.

## Preact에서의 useCallback 구현

기본적으로 useCallback은 useMemo를 사용해서 구현할 수 있다.

```jsx
export function useCallback(callback, args) {
  currentHook = 8;
  return useMemo(() => callback, args);
}
```

## useMemo & useCallback 동일한 역할 in code

```jsx
import { useState, useCallback, useMemo } from 'react'

export default function App() {
  const [counter, setCounter] = useState(0)

  /**
   * 아래 두 함수의 작동은 동일하다.
   *
   */
  const handleClick1 = useCallback(() => {
    setCounter((prev) = > prev + 1)
  }, [])

  const handleClick2 = useMemo(() => {
    return () => setCounter((prev) = > prev + 1)
  }, [])

  return (
    <>
      <h1>{counter}</h1>
      <button onClick={handleClick1}>+</button>
      <button onClick={handleClick2}>+</button>
    </>
  )
}
```

useMemo는 값 자체를 메모이제이션하는 용도이기에 반환문으로 함수 선언문을 반환해야 한다.
