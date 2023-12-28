# useLayoutEffect(feat. useEffect)

## useEffect와 useLayoutEffect의 차이점에 대해 말씀해주세요.

> **💡 면접을 위한 quick answer :**
>
> - Render tree 구성: DOM Tree를 구성하기 위해 각 엘리먼트의 스타일 속성을 계산하는 과정
> - paint: 실제 스크린에 layout을 표시하고 업데이트 하는 과정
>
> - **useEffect**
>   - render와 paint 된 후 실행됩니다. 비동기적으로 실행됩니다.
>   - paint된 후 실행되기 때문에, useEffect 내부에 dom에 영향을 주는 코드가 있을 경우 사용자 입장에서는 화면 깜빡임을 보게 됩니다.
>   - 데이터 패칭, 상태 업데이트 등과 같은 대부분의 부수 효과(side-effects)를 처리하기에 적합합니다.
> - **useLayoutEffect**
>   - 컴포넌트들이 render된 후 실행되며 그 이후에 paint가 됩니다. 동기적으로 실행됩니다.
>   - paint가 되기 전에 실행되기 때문에 dom을 조작하는 코드가 존재하더라도 사용자는 깜빡임을 경험하지 않습니다.
>   - 로직이 복잡할 경우 사용자가 레이아웃을 보는데까지 시간이 오래 걸린다는 단점
>   - DOM의 크기나 스타일을 직접 조작해야 하는 경우나, 화면 깜빡임을 방지해야 하는 경우에 사용됩니다.

## useEffect와 useLayoutEffect의 사용법 비교

```jsx
function App() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    console.log('useEffect', count);
  }, [count]);

  useLayoutEffect(() => {
    console.log('useLayoutEffect', count);
  }, [count]);

  function handleClick() {
    setCount((prev) => prev + 1);
  }

  return (
    <>
      <h1>{count}</h1>
      <button onClick={handleClick}>+</button>
    </>
  );
}
```

useEffect와 useLayoutEffect를 사용한 예제 코드 모두 동일한 모습으로 작동하는 것처럼 보인다.

여기서 useLayoutEffect를 이해하기 위한 중요한 사실은 **모든 DOM의 변경 후에 useLayoutEffect의 콜백 함수 실행이 동기적으로 발생**한다는 점이다. 여기서 말하는 DOM 변경이란 렌더링이지, 브라우저에 실제로 해당 변경 사항이 반영되는 시점을 의미하는 것은 아니다.

즉, 실행 순서는 다음과 같다.

```
1. 리액트가 DOM을 업데이트
2. useLayoutEffect를 실행
3. 브라우저에 변경 사항을 반영
4. useEffect를 실행
```

동기적으로 발생한다는 것은 리액트는 useLayoutEffect의 실행이 종료될 때까지 기다린 다음에 화면을 그린다는 것을 의미한다.
즉, 리액트 컴포넌트는 useLayoutEffect가 완료될 때까지 기다리기 때문에 컴포넌트가 잠시 동안 일시 중지되는 것과 같은 일이 발생하게 된다. 따라서 이러한 작동 방식으로 인해 웹 애플리케이션 성능에 문제가 발생할 수 있다.

## 그럼 언제 useLayoutEffect를 사용하는 것이 좋을까?

useLayoutEffect의 특징상 **DOM은 계산됐지만 이것이 화면에 반영되기 전에 하고 싶은 작업이 있을 때**와 같이 반드시 필요할 때만 사용하는 것이 좋다.
특정 요소에 따라 DOM요소를 기반으로 한 애니메이션, 스크롤 위치를 제어하는 등 화면에 반영되기 전에 하고 싶은 작업에 UseLayoutEffect를 사용한다면 UseEffect를 사용했을 때보다 훨씬 더 자연스러운 사용자 경험을 제공할 수 있다.

구체적인 예시로는

```
- data fetch
- event handler
- state reset
```

등의 작업은 항상 useEffect를 사용하되

```jsx
import React, { useLayoutEffect, useRef, useState } from 'react';

export default function Example() {
  const [width, setWidth] = useState(0);
  const boxRef = useRef(null);

  useLayoutEffect(() => {
    // boxRef로 참조된 <div>요소의 너비를 측정하고, 이 값을 상태 변수 width에 저장
    // useLayoutEffect를 사용하면 화면에 어떤 변경 사항이 보이기 전에 DOM을 조작할 수 있기에
    // 사용자가 중간 단계를 보지 않도록 할 수 있다.
    if (boxRef.current) {
      setWidth(boxRef.current.getBoundingClientRect().width);
    }
  }, []); // 의존성 배열이 비어 있으므로 컴포넌트 마운트 시에만 실행

  return (
    <div>
      <div ref={boxRef}>이 요소의 너비를 측정합니다.</div>
      <p>측정된 너비: {width}px</p>
    </div>
  );
}
```

DOM 업데이트가 화면에 반영되기 전에 동기적으로 실행되는 **레이아웃에 관련된 계산이나 DOM 조작**예시에 사용될 수 있다.
위의 예제는 컴포넌트가 화면에 마운트된 후 바로 특정 DOM요소의 크기를 계산하는 예제이다.
