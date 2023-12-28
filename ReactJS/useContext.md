# useContext

## Context란?

props drilling 극복하기 위해 등장한 개념이 Context이고, Context를 사용하면, 명시적인 props 전달 없이도 선언한 하위 컴포넌트 모두에서 자유롭게 원하는 값을 사용할 수 있다.

## Context를 함수형 컴포넌트에서 사용할 수 있게 해주는 useContext훅

```tsx
const Context = createContext<{ hello: string } | undefined>();

function 부모컴포넌트() {
  return (
    <>
      <Context.Provider value={{ hello: 'react' }}>
        <Context.Provider value={{ hello: 'javascript' }}>
          <자식컴포넌트 />
        </Context.Provider>
      </Context.Provider>
    </>
  );
}

function 자식컴포넌트() {
  const value = useContext(Context);

  // react가 아닌 Javascript가 반환된다.
  return <>{value ? value.hello : ''}</>;
}
```

useContext는 상위 컴포넌트에서 만들어진 Context를 함수형 컴포넌트에서 사용할 수 있도록 만들어진 훅이다. useContext를 사용하면 상위 컴포넌트 어딘가에서 선언된 `<Context.Provider />`에서 제공한 값을 사용할 수 있게 된다. 만약 여러 개의 Provider가 있다면 가장 가까운 Provider의 값을 가져오게 된다. 예제에서는 가까운 콘텍스트 값인 javascript가 반환된다.

## useContext custom hook

useContext로 원하는 값을 얻으려고 했지만 정작 컴포넌트가 실행될 때 이 콘텍스트가 존재하지 않아 예상치 못하게 에러가 발생한 경험이 종종 있을 것이다.

이러한 에러를 방지하려면 useContext 내부에서 해당 콘텍스트가 존재하는 환경인지, 즉 콘텍스트가 한 번이라도 초기화되어 값을 내려주고 있는지 확인해 보면 된다.

```tsx
import { PropsWithChildren, createContext, useContext } from 'react';

const MyContext = createContext<{ hello: string } | undefined>(undefined);

function ContextProvider({
  children,
  text,
}: PropsWithChildren<{ text: string }>) {
  return (
    <MyContext.Provider value={{ hello: text }}>{children}</MyContext.Provider>
  );
}

function useMyContext() {
  const context = useContext(MyContext);

  if (context === undefined) {
    throw new Error('useMyContext는 ContextProvider 내부에서만 사용가능함');
  }

  return context;
}

function 자식_컴포넌트() {
  // 타입이 명확히 설정돼 있어서 굳이 undefined 체크를 하지 않아도 된다.
  // 이 컴포넌트가 Provider 하위에 없다면 에러가 발생할 것이다.

  const { hello } = useMyContext();

  return <>{hello}</>;
}

function 부모_컴포넌트() {
  return (
    <>
      <ContextProvider text='react'>
        <자식_컴포넌트 />
      </ContextProvider>
    </>
  );
}
```

다수의 Provider와 useContext를 사용할 때, 특히 타입스크립트를 사용하고 있다면 위와 같이 별도 함수로 감싸서 사용하는 것이 좋다.
타입 추론에도 유용하고, 상위에 Provider가 없는 경우에도 사전에 쉽게 에러를 찾을 수 있다.

## useContext를 사용할 때 주의할 점

useContext를 함수형 컴포넌트 내부에서 사용할 때는 항상 컴포넌트 재활용이 어려워진다는 점을 염두에 둬야 한다.
useContext가 선언돼 있으면 Provider에 의존성을 가지고 있는 셈이 되므로 아무데서나 재활용하기에는 어려운 컴포넌트가 된다.
해당 함수형 컴포넌트가 Provider하위에 있지 않은 상태로 UseContext를 사용한다면 예기치 못한 작동 방식이 만들어진다.

즉, useContext가 있는 컴포넌트는 그 순간부터 눈으로는 직접 보이지도 않을 수 있는 Provider와의 의존성을 갖게 되는 셈이다.

이러한 상황을 방지하려면 useContext를 사용하는 컴포넌트를 최대한 작게 하거나 재사용되지 않을 만한 컴포넌트에서 사용해야 한다.
콘텍스트가 많아질수록 루트 컴포넌트는 더 많은 콘텍스트로 둘러싸일 것이고 해당 props를 다수의 컴포넌트에서 사용할 수 있게끔 해야 하므로 불필요하게 리소스가 낭비된다. 따라서 컨텍스트가 미치는 범위는 필요한 환경에서 최대한 좁게 만들어야 한다.

콘텍스트는 상태를 주입해 주는 API다. 상태 관리 라이브러리가 되기 위해서는 최소한 다음 두 가지 조건을 만존해야 한다.

> 1. 어떠한 상태를 기반으로 다른 상태를 만들어 낼 수 있어야 한다.
> 2. 필요에 따라 이러한 상태 변화를 최적화할 수 있어야 한다.

콘텍스트는 둘 중 어느 것도 하지 못한다. 단순히 props 값을 하위로 전달해 줄 뿐, useContext를 사용한다고 해서 렌더링이 최적화되지는 않는다.

```tsx
import {
  PropsWithChildren,
  createContext,
  useContext,
  useEffect,
  useState,
} from 'react';

const MyContext = createContext<{ hello: string } | undefined>(undefined);

function ContextProvider({
  children,
  text,
}: PropsWithChildren<{ text: string }>) {
  return (
    <MyContext.Provider value={{ hello: text }}>{children}</MyContext.Provider>
  );
}

function useMyContext() {
  const context = useContext(MyContext);

  if (context === undefined) {
    throw new Error('useMyContext는 ContextProvider 내부에서만 사용가능함');
  }

  return context;
}

function 손자_컴포넌트() {
  const { hello } = useMyContext();

  useEffect(() => {
    console.log('렌더링 손자_컴포넌트');
  });
  // text가 변경되는 부모_컴포넌트와 이를 사용하는 손자_컴포넌트만 렌더링 될 것 같지?
  // 응 아니야~ 사실은 컴포넌트 트리 전체가 렌더링 되고 있다.

  return <h3>{hello}</h3>;
}

function 자식_컴포넌트() {
  useEffect(() => {
    console.log('렌더링 자식_컴포넌트');
  });

  return <손자_컴포넌트 />;
}

function 부모_컴포넌트() {
  const [text, setText] = useState('');

  function handleChange(e: ChangeEvent<HTMLInputElement>) {
    setText(e.target.value);
  }

  useEffect(() => {
    console.log('렌더링 부모_컴포넌트');
  });

  return (
    <>
      <ContextProvider text='react'>
        <input value={text} onChange={handleChange} />
        <자식_컴포넌트 />
      </ContextProvider>
    </>
  );
}
```

손자 컴포넌트, 자식 컴포넌트, 부모 컴포넌트 전체가 리렌더링 되고 있다.
부모 컴포넌트가 렌더링되면 하위 컴포넌트는 모두 리렌더되는 상황이 이 사용 예제에 해당한다.

> useContext는 상태를 관리하는 마법이 아니라 단순히 상태를 주입할 뿐 그 이상의 기능도 그 이하도 아니다.<br/>
> useContext로 상태 주입을 최적화했다면 반드시 Provider의 값이 변경될 때 어떤식으로 렌더링되는지 눈여겨봐야한다.

### 위 예제 최적화 하려면 어떻게 할까?

자식\_컴포넌트가 렌더링 되지 않게 막으려면 React.memo를 쓰면 된다.
memo는 Props 변화가 없으면 리렌더링 되지 않고 계속해서 같은 결과물을 반환할 것이다.

```tsx
const 자식_컴포넌트 = memo(() => {
  useEffect(() => {
    console.log('렌더링 자식_컴포넌트');
  });

  return <손자_컴포넌트 />;
});
```
