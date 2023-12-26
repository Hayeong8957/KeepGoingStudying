# useRef

### useRef가 뭐에요?

- 돔의 레퍼런스를 담는 용도가 주된 사용법이라고 설명한다. 또 렌더링 결과에 직접적인 영향을 미치지 않는 값을 저장하기 위해 사용된다.

- ref의 유용한 점은 ref의 값을 아무리 변경해도 컴포넌트는 다시 렌더링 되지 않는다.
  즉 state대신 ref를 사용한다면 불필요한 렌더링을 막을 수 있다.

### 리렌더링과 useRef

- react의 리렌더링 조건은 1) 자신의 state가 변경될 때, 2) 부모 컴포넌트로 받은 props가 변경될 때, 3) 부모 컴포넌트가 렌더링될 때 세가지이다.
  - 렌더링의 의미는 내부의 변수들이 담고 있는 기존에 저장한 값들이 초기화되고, 함수 로직이 재실행됨을 의미한다.
  - 하지만 렌더링 되더라도 기존에 저장한 값들을 그대로 보존해야 하는 경우가 있다. 이때 useRef를 사용하는 것이다.

### const로 컴포넌트 내 선언해 useRef를 하지 왜 useRef를 사용해요?

const로 선언된 값은 컴포넌트가 렌더링 될 때마다 초기화된다.<br/>
useRef를 사용하면 컴포넌트가 재렌더링 되어도 값이 유지된다.<br/>
useRef는 변경 가능한 값을 유지할 수 있게 해줍니다. 값이 변경되어도 컴포넌트가 재렌더링 되지 않습니다.<br/>

### useRef의 반환 타입

```TypeScript
// 'current'프로퍼티가 변경가능함 -> 'current'의 값이 변할 수 있다.
interface MutableRefObject<T> {
        current: T;
}

// 'current'프로퍼티가 읽기 전용임 -> 'current'는 T타입의 값이거나 null일 수 있다..
interface RefObject<T> {
        readonly current: T | null;
}
```

useRef는 `.current`프로퍼티를 가지고 있는 하나의 새로운 객체를 생성하며 그저 초기값을 저장할 뿐이다.
useRef는 `.current`프로퍼티에 변경 가능한 값을 담고 있는 상자와 같다.

```TypeScript
// 1)
// 인자 값이 있을 때, 인자 타입과 제네릭 타입이 일치하는 경우
// 초기값이 T를 받음 -> MutableRefObject<T>를 반환 -> 반환된 ref객체의 'current'프로퍼티 변경 가능
function useRef<T>(initialValue: T): MutableRefObject<T>;

// 2)
// 인자 값이 있을 때, 인자 타입이 null을 허용하는 경우
// 초기값이 T이거나 null일 수 있다 -> RefObject<T>를 반환 -> 'current'는 읽기 전용
// => 주로 DOM요소에 대한 참조를 만들 때 사용
function useRef<T>(initialValue: T|null): RefObject<T>;

// 3)
//제네릭 타입이 undefined인 경우
// 초기값이 undefined이다 -> 인자 받지 않고, 반환되는 ref객체의 'current'는 'T'타입의 값이거나 'undefined'가 됨
function useRef<T = undefined>(): MutableRefObject<T | undefined>;
```

이처럼 다양한 상황에 맞게 참조를 생성하고 관리할 수 있도록 한다.

#### 1. `MutableRefObject<T>`를 반환하는 경우

이 경우는 변경 가능한 값을 저장하기 위해 사용된다.
아래의 예시는 컴포넌트의 렌더링 사이클과 독립적인 카운터를 유지하는 예시이다.

```TypeScript
import React, { useRef, useEffect } from 'react';

function TimerComponent(): JSX.Element {
  const count = useRef<number>(0); // 초기 값 0으로 설정, 타입은 number

  useEffect(() => {
    const interval = setInterval(() => {
      count.current += 1;
      console.log(`타이머 카운트: ${count.current}`); // 타이머 카운트: 1, 2, 3, ...
    }, 1000);

    return () => clearInterval(interval); // 컴포넌트 언마운트 시 인터벌 제거
  }, []);

  return <div>타이머 카운트를 콘솔에서 확인하세요.</div>;
}
```

이 예시에서 'count'는 렌더링 사이에도 값이 유지되며, 컴포넌트가 재렌더링 되지 않고도 그 값이 변경될 수 있다.

#### 2. `RefObject<T>`를 반환하는 경우

이 경우는 주로 DOM 요소에 대한 참조를 만들 때 사용된다.
예를 들어, 특정 DOM요소에 직접 접근해야 할 때 사용할 수 있다.
모달 overlay 클릭 시 모달 닫히게 할 때나 무한스크롤 시 하단 부분을 DOM요소에 대한 참조를 할 때 사용할 수 있다.

```TypeScript
import React, { useRef, useEffect } from 'react';

function FocusInputComponent(): JSX.Element {
  const inputEl = useRef<HTMLInputElement>(null); // 타입은 HTMLInputElement 또는 null

  useEffect(() => {
    if(inputEl.current) inputEl.current.focus(); // 컴포넌트가 마운트되면 input 요소에 포커스
  }, []);

  return <input ref={inputEl} type="text" />;
}
```

이 예시에서 'inputEl'은 'input'요소에 대한 참조를 저장하며, 컴포넌트가 마운트될 때 해당 요소에 자동으로 포커스를 맞춘다.

- **`initialValue`가 null일 수 밖에 없지만 값을 수정해야할 때**

위의 코드를 focus()가 아닌 값을 전달하게 코드를 변경하면 어떻게 될까?

```TypeScript
import React, { useRef, useEffect } from 'react';

function FocusInputComponent(): JSX.Element {
  const inputEl = useRef<HTMLInputElement>(null); // 타입은 HTMLInputElement 또는 null

  useEffect(() => {
    if(inputEl.current) inputEl.current = "focus"; // 컴포넌트가 마운트되면 input 요소에 focus입력되게 => ❗️Error : 읽기 전용 속성이므로 'current'에 할당할 수 없습니다.
  }, []);

  return <input ref={inputEl} type="text" />;
}
```

RefObject는 readonly로서 수정이 불가능하다.
값을 변경하고 싶다면 다음과 같이 코드를 변경하면 된다.

```TypeScript
  useEffect(() => {
    if(inputEl.current) inputEl.current!.value = "focus";
  }, []);
```

이렇게 코드를 작성하면 정상 작동한다.

> `current`프로퍼티가 객체라는 사실에 초점을 맞춘다.<br/> `current`프로퍼티 자체는 수정이 불가하지만 `current`의 하위 프로퍼티 `value`는 수정이 가능하다.<br/>
> useRef의 `initialValue`가 null일 수 밖에 없고, 값을 수정해야한다면 하위 프로퍼티를 사용하면 오류없이 정상 작동 시킬 수 있다.<br/> `current`프로퍼티만 읽기 전용으로, `current`프로퍼티의 하위 프로퍼티는 수정 가능하다. readonly는 얕은 객체 복사이기 때문이다.

#### 3. 제네릭 타입이 `undefined`인 경우

이 경우는 초기값을 지정하지 않고, 추후에 값을 설정하거나 업데이트할 때 사용된다.

```TypeScript
import React, { useRef, useEffect } from 'react';

function ArbitraryComponent(): JSX.Element {
  const arbitraryRef = useRef<string | undefined>(); // 초기값 없음, 타입은 string 또는 undefined

  useEffect(() => {
    arbitraryRef.current = "임의의 값";
    console.log(arbitraryRef.current); // "임의의 값"
  }, []);

  return <div>임의의 값을 콘솔에서 확인하세요.</div>;
}
```

이 예시에서 arbitraryRef는 초기화되지 않았으나, 나중에 useEffect 내에서 값이 할당된다. 이렇게 useRef는 초기화되지 않은 상태로 시작해도 되며, 나중에 필요에 따라 값을 할당할 수 있다.

# useRef Definition from 모던 리액트 딥다이브

- useRef는 반환값인 객체 내부에 있는 current로 값에 접근 또는 변경할 수 있다.
- useRef는 그 값이 변하더라도 렌더링을 발생시키지 않는다.

useRef가 왜 필요한지 고민해보자. 렌더링에 영향을 미치지 않는 고정된 값을 관리하기 위해서 useRef를 사용한다면 useRef를 사용하지 않고 그냥 함수 외부에서 값을 선언해서 관리하는 것도 동일한 기능을 수행할 수도 있지 않을까?

```jsx
let value = 0;

function Component() {
  function handleClick() {
    value += 1;
  }

  // ...
}
```

컴포넌트가 실행되어 렌더링되지 않았음에도 value라는 값이 기본적으로 존재하게 된다. 이는 메모리에 불필요한 값을 갖게 하는 악영향을 미친다.
만약 Component가 여러 번 생성된다면 각 컴포넌트에서 가리키는 값이 모두 value로 동일하다. 컴포넌트가 초기화되는 지점이 다르더라도 하나의 값을 봐야 하는 경우라면 유효할 수도 있지만 대부분의 경우에는 컴포넌트 인스턴스 하나당 하나의 값을 필요로 하는 것이 일반적이다.

useRef는 앞서 언급한 두 가지 문제를 모두 극복할 수 있는 리액트식 접근법이다. 컴포넌트가 렌더링될 때만 생성되며, 컴포넌트 인스턴스가 여러 개라도 각각 별개의 값을 바라본다.

## useRef를 사용한 DOM 접근 예제

```jsx
function RefComponent() {
  const inputRef = useRef()

  // 이때는 미처 렌더링이 실행되기 전(반환되기 전)이므로 undefined를 반환한다.
  console.log(inputRef.current) // undefined

  useEffect(() => {
    console.log(inputRef.current) // <input type="text"></input>
  }, [inputRef])

  return <input ref={inputRef} type="text">
}
```

useRef는 최초에 넘겨받은 기본값을 가지고 있다. 한 가지 명심할 것은 useRef의 최초 기본값은 return 문에 정의해 둔 DOM이 아니고 useRef()로 넘겨받은 인수라는 것이다. useRef가 선언된 당시에는 아직 컴포넌트가 렌더링되기 전이라 return 으로 컴포넌트의 DOM이 반환되기 전이므로 undefined다.

useRef를 사용할 수 있는 유용한 경우는 렌더링을 발생시키지 않고 원하는 상태값을 저장할 수 있다는 특징을 활용해 useState의 이전 값을 저장하는 usePrevious() 같은 훅을 구현할 때다.

```jsx
function usePrevious(value) {
  const ref = useRef();
  useEffect(() => {
    ref.current = value;
  }, [value]); // value가 변경되면 그 값을 ref에 넣어둔다.
  return ref.current;
}

function SomeComponent() {
  const [counter, setCounter] = useState(0);
  const previousCounter = usePrevious(counter);

  function handleClick() {
    setCounter((prev) => prev + 1);
  }

  // 0 (undefined)
  // 1, 0
  // 2, 1
  // 3, 2
  return (
    <button onClick={handleClick}>
      {counter} {previousCounter}
    </button>
  );
}
```

개발자가 원하는 시점의 값을 렌더링에 영향을 미치지 않고 보관해 두고 싶다면 useRef를 사용하는 것이 좋다.

## Preact에서 useRef 구현

```jsx
export function useRef(initialValue) {
  currentHook = 5;
  return useMemo(() => ({ current: initialValue }), []);
}
```

값이 변경돼도 렌더링되면 안된다는 점, 실제 값은 {current: value}와 같은 객체 형태로 있다는 점을 떠올려보자. 렌더링에 영향을 미치면 안 되기 때문에 useMemo에 의도적으로 빈 배열을 선언해 뒀고, 이는 각 렌더링마다 동일한 객체를 가리키는 결과를 낳을 것이다.
자바스크립트의 특징 중 객체의 값을 변경해도 객체를 가리키는 주소가 변경되지 않는다는 것을 떠올리면 useMemo로 useRef를 구현할 수 있다.
