# useImperativeHandle

useImperativeHandle을 이해하기 위해서는 먼저 React.forwardRef에 대해 알아야 한다.

## forwardRef

ref는 useRef에서 반환한 객체로, 리액트 컴포넌트의 props인 ref에 넣어 HTMLElement에 접근하는 용도로 흔히 사용된다. key와 마찬가지로 ref도 리액트에서 컴포넌트의 props로 사용할 수 있는 예약어로서 별도로 선언돼 있지 않아도 사용할 수 있다. 리액트는 ref를 특별하게 인식하여, 이를 통해 생성된 참조를 해당 컴포넌트나 DOM 요소에 연결한다. 이 과정은 내부적으로 자동으로 이루어지기 때문에 개발자가 별도로 'ref'를 선언할 필요가 없다.

만일 이러한 ref는 상위 컴포넌트에서 하위 컴포넌트로 전달하고 싶다면 어떻게 해야 할까? 즉, 상위 컴포넌트에서는 접근하고 싶은 ref가 있지만 이를 직접 props로 넣어 사용할 수 없을 때는 어떻게 해야 할까? 우리가 알고 있는 단순한 ref와 props에 대한 상식으로 이 문제를 해결한다면 다음과 같이 짤 수도 있다.

- 문제 발생 코드

```jsx
import React, { useRef } from 'react';

function Input({ ref }) {
  return <input type='text' ref={ref} />;
}

function Field() {
  const inputRef = useRef(null);

  function handleFocus() {
    inputRef.current.focus();
  }

  return (
    <>
      <Input ref={inputRef} />
      <button onClick={handleFocus}>입력란 포커스</button>
    </>
  );
}
```

```
Warning: Input: `ref` is not a prop. Trying to access it will result in `undefined` being returned. If you need to access the same value within the child component, you should pass it as a different prop. (https://reactjs.org/link/special-props)
    at Input (https://nqxh4.csb.app/src/Field.jsx:18:18)
    at Field (https://nqxh4.csb.app/src/Field.jsx:30:36)
    at div
    at App
```

리액트에서 ref는 props로 쓸 수 없다는 경고문과 함께 접근을 시도할 경우 undefined를 반환한다고 돼 있다. 그래서 다른 props를 사용해야한다는 것인데, 가이드대로 ref 대신에 ref2와 같이 다른 이름의 prop을 사용하도록 `<Input/>`을 수정하여 문제를 해결할 수 있다.

- 다른 이름의 props 사용하기

```jsx
import React, { useRef } from 'react';

function Input({ parentRef }) {
  return <input type='text' ref={parentRef} />;
}

function Field() {
  const inputRef = useRef(null);

  function handleFocus() {
    inputRef.current.focus();
  }

  return (
    <>
      <Input parentRef={inputRef} />
      <button onClick={handleFocus}>입력란 포커스</button>
    </>
  );
}
```

이러한 방식은 앞선 예제와 다르게 잘 작동하는 것으로 보인다. 그리고 이는 클래스형 컴포넌트나 함수형 컴포넌트에서도 동일하게 작동한다. forwardRef는 방금 작성한 코드와 동일한 작업을 하는 리액트 API이다.

- `forwardRef`사용하기

```jsx
import React, { useRef, forwardRef } from 'react';

const Input = forwardRef((props, ref) => {
  return <input type='text' ref={ref} />;
});

function Field() {
  const inputRef = useRef(null);

  function handleFocus() {
    inputRef.current.focus();
  }

  return (
    <>
      <Input ref={inputRef} />
      <button onClick={handleFocus}>입력란 포커스</button>
    </>
  );
}
```

forwardRef가 탄생한 배경은 ref를 전달하는 데 있어서 일관성을 제공하기 위해서다. 어떤 props명으로 전달할지 모르고, 이에 대한 완전한 네이밍의 자유가 주어진 props 보다는 forwardRef를 사용하면 좀 더 확실하게 ref를 전달할 것임을 예측할 수 있고, 또 사용하는 쪽에서도 확실히 안정적으로 받아서 사용할 수 있다.

**ref를 받고자 하는 컴포넌트를 forwardRef로 감싸고, 두 번째 인수로 ref를 전달받는다.**
부모 컴포넌트에서는 동일하게 props.ref를 통해 ref를 넘겨주면 된다.

## useImperativeHandle

useImperativeHandle은 부모에게서 넘겨받은 ref를 원하는 대로 수정할 수 있는 훅이다.

```jsx
import React, {
  useState,
  useRef,
  forwardRef,
  useImperativeHandle,
} from 'react';

const Input = forwardRef((props, ref) => {
  // useImperativeHandle을 사용하면 ref의 동작을 추가로 정의할 수 있다.
  useImperativeHandle(
    ref,
    () => ({
      alert: () => alert(props.value),
    }),
    [props.value],
  ); // useEffect의 deps와 같다.

  return <input type='text' ref={ref} />;
});

function Field() {
  const inputRef = useRef(null);
  const [text, setText] = useState('');

  function handleClick() {
    // inputRef에 추가한 alert라는 동작을 사용할 수 있다.
    inputRef.current.alert();
  }

  function handleChange(e) {
    setText(e.target.value);
  }
  return (
    <>
      <Input ref={inputRef} value={text} onChange={handleChange} />
      <button onClick={handleClick}>입력란 포커스</button>
    </>
  );
}
```

예시에서 Input 컴포넌트는 forwardRef를 사용해 부모 컴포넌트로부터 ref를 전달받는다. 그리고 useImperativeHandle을 사용하여 이 ref에 alert라는 새로운 함수를 추가한다. 이 함수는 부모 컴포넌트에서 호출할 수 있으며, Input 컴포넌트의 props.value값을 알림으로 보여주는 역할을 한다.

원래 ref는 {current: <HTMLElement>}와 같은 형태로 HTMLElement만 주입할 수 있는 객체였다. 그러나 여기서는 전달받은 ref에다 useImperativeHandle 훅을 사용해 추가적인 동작을 정의했다. 이로써 부모는단순히 HTMLElement 뿐만 아니라 자식 컴포넌트에서 새롭게 설정한 객체의 키와 값에 대해서도 접근할 수 있게 됐다. useImperativeHandle을 사용하면 이 ref의 값에 원하는 값이나 액션을 정의할 수 있다.
