# Jotai

Recoil의 atom모델에 영감을 받아 만들어진 상태 관리 라이브러리다. Jotai는 상향식(bottom-up) 접근법을 취하고 있다고 나와있는데, 이는 리덕스와 같이 하나의 큰 상태를 애플리케이션에 내려주는 방식이 아니라, 작은 단위의 상태를 위로 전파할 수 있는 구조를 취하고 있음을 의미한다.

**리엑트 Context의 문제점인 불필요한 리렌더링이 일어난다는 문제를 해결하고자 설계**돼 있으며, 추가적으로 개발자들이 메모이제이션이나 최적화를 거치지 않아도 리렌더링이 발생되지 않도록 설계돼 있다.

## atom

> Jotai v1.8.3

최소 단위의 상태를 의미한다. atom 하나만으로도 상태를 만들 수도, 또 이에 파생된 상태를 만들 수도 있다.

```jsx
const counterAtom = atom(0);
```

이렇게 만든 counterAtom에는 다음과 같은 정보가 담긴다.

```jsx
console.log(counterAtom);
// ...
// {
//   init: 0,
//   read: (get) => get(config),
//   write: (get, set, update) =>
//     set(config, typeof update === 'function' ? update(get(config)) : update)
// }
```

이 atom은 어떤 구조를 가지고 있을까? Jotai의 내부 atom 구현을 살펴보자.

- Jotai의 atom 코드

```jsx
// Value(atom), Update, Result 세 가지 타입을 받음
export function atom<Value, Update, Result extends void | Promise<void>>(
  read: Value | Read<Value>,      // -> atom의 값을 읽는 방법을 정의
  write?: Write<Update, Result>,  // -> 값을 업데이트 하는 방법을 정의
) {
  const key = `atom${++keyCount}` // 디버깅이나 로딩 목적으로 사용되는 key

  const config = {                // atom의 설정을 담고 있음
    toString: () => key,
  } as WritableAtom<Value, Update, Result> & { init?: Value }
  // init: atom의 초기값을 설정
  // read: atom의 현재 값을 읽는 함수이다. 함수 형태의 'read'를 전달받으면 그대로 사용하고, 아니면 get(config)를 통해 값을 반환한다.
  // write: atom의 값을 업데이트하는 함수이다. 전달된 write가 있으면 그것을 사용하고, 없으면 기본적인 업데이트 로직을 제공한다.

  if (typeof read === 'function') {
    config.read = read as Read<Value>
  } else {
    config.init = read
    config.read = (get) => get(config)
    config.write = (get, set, update) =>
      set(config, typeof update === 'function' ? update(get(config)) : update)
  }

  if (write) {
    config.write = write
  }
  return config
}
```

각 atom을 생성할 때마다 고유한 key를 필요로 했던 Recoil과는 다르게, Jotai는 atom을 생성할 때 별도의 key를 넘겨주지 않아도 된다. atom 내부에는 key라는 변수가 존재하긴 하지만 외부에서 받는 값은 아니며 단순히 toString()을 위한 용도로 한정돼 있다. 그리고 config라는 객체를 반환하는데, 이 config에는 초깃값을 의미하는 init, 값을 가져오는 read, 값을 설정하는 write만 존재한다. 즉, Jotai에서의 atom에 따로 상태를 저장하고 있지 않다. 이 상태는 useAtomValue에 있다.

## useAtomValue

- Jotai의 useAtomValue 구현

```jsx
export function useAtomValue<Value>(
  atom: Atom<Value>,     // 읽고자 하는 atom을 지정
  scope?: Scope          // atom이 속한 범위를 지정하는 선택적 파라미터이다. 이는 주로 atom 의 범위를 제한.
): Awaited<Value> {
  const ScopeContext = getScopeContext(scope)     // 현재 컴포넌트의 범위
  const scopeContainer = useContext(ScopeContext) // 현재 컴포넌트에 관련된 정보
  const { s: store, v: versionFromProvider } = scopeContainer

  const getAtomValue = (version?: VersionObject) => {
    // This call to READ_ATOM is the place where derived atoms will actually be
    // recomputed if needed.
    // 주어진 version에 따라 atom의 현재 상태를 store에서 가져옴.
    // atom의 상태에 오류가 있거나 아직 결정되지 않은 경우에 대한 처리도 포함
    const atomState = store[READ_ATOM](atom, version)
    if (__DEV__ && !atomState.y) {
      throw new Error('should not be invalidated')
    }
    if ('e' in atomState) {
      throw atomState.e // read error
    }
    if ('p' in atomState) {
      throw atomState.p // read promise
    }
    if ('v' in atomState) {
      return atomState.v as Awaited<Value>
    }
    throw new Error('no atom value')
  }

  // Pull the atoms's state from the store into React state.
  // useReducer 사용해 atom의 값과 버전을 React 컴포넌트의 상태로 관리
  // rerenderIfChanged는 atom의 상태가 변경되었을 때 컴포넌트를 다시 렌더링하도록 함
  const [[version, valueFromReducer, atomFromReducer], rerenderIfChanged] =
    useReducer<
      Reducer<
        readonly [VersionObject | undefined, Awaited<Value>, Atom<Value>],
        VersionObject | undefined
      >,
      VersionObject | undefined
    >(
      (prev, nextVersion) => {
        const nextValue = getAtomValue(nextVersion)
        if (Object.is(prev[1], nextValue) && prev[2] === atom) {
          return prev // bail out
        }
        return [nextVersion, nextValue, atom]
      },
      versionFromProvider,
      (initialVersion) => {
        const initialValue = getAtomValue(initialVersion)
        return [initialVersion, initialValue, atom]
      }
    )

  let value = valueFromReducer
  if (atomFromReducer !== atom) {
    rerenderIfChanged(version)
    value = getAtomValue(version)
  }

  // useEffect를 통해 atom의 상태가 변경되었을 때 컴포넌트가 적절히 리렌더링되도록 구독 로직을 설정
  // 이는 atom의 값이 변경될 때마다 컴포넌트가 최신 상태를 반영하도록 함
  useEffect(() => {
    const { v: versionFromProvider } = scopeContainer
    if (versionFromProvider) {
      store[COMMIT_ATOM](atom, versionFromProvider)
    }
    // Call `rerenderIfChanged` whenever this atom is invalidated. Note
    // that derived atoms may not be recomputed yet.
    const unsubscribe = store[SUBSCRIBE_ATOM](
      atom,
      rerenderIfChanged,
      versionFromProvider
    )
    rerenderIfChanged(versionFromProvider)
    return unsubscribe
  }, [store, atom, scopeContainer])

  useEffect(() => {
    store[COMMIT_ATOM](atom, version)
  })

  useDebugValue(value)
  return value    // 마지막으로 atom의 현재 값을 반환
}
```

- **useReducer**

useReducer에서 반환하는 상태값은 3가지로 [versiom, valueFromReducer, atomFromReducer]인데, 첫 번째는 store의 버전, 두 번째는 atom에서 get을 수행했을 때 반환되는 값, 세 번째는 atom 그 자체를 의미한다.

- **컴포넌트 루트 레벨에서 Context의 존재**

컴포넌트 루트 레벨에서 Context가 존재하지 않아도 되는데, Context가 없다면 앞선 예제에서처럼 Provider가 없는 형태로 기본 스토어를 루트에 생성하고 이를 활용해 값을 저장하기 때문이다. 물론 Jotai에서 export하는 Provider를 사용한다면 앞선 예제에서 여러 개의 Provider를 관리했던 것처럼 각 Provider별로 atom값을 관리할 수도 있다.

- **atom의 값은 Store에 존재**

store에 atom 객체 그 자체를 키로 활용해 값을 저장한다. 이러한 방식을 위해 WeakMap이라고 하는, 자바스크립트에서 객체만을 키로 가질 수 있는 독특한 방식의 Map을 활용해 recoil과는 다르게 별도의 Key를 받지 않아도 스토어에 값을 저장할 수 있다.

- **rerenderIfChanged**

리렌더링을 일으키기 위해 사용하는 rerenderIfChanged가 일어나는 경우는

1. 넘겨받은 atom이 Reducer를 통해 스토어에 있는 atom과 달라지는 경우
2. subscribe를 수행하고 있다가 어디선가 이 값이 달라지는 경우다

이러한 로직 덕분에 atom 값이 어디서 변경되더라도 useAtomValue로 값을 사용하는 쪽에서는 언제든 최신 값의 atom을 사용해 렌더링할 수 있게 된다.

## useAtom

useAtom은 useState와 동일한 형태의 배열을 반환한다. 첫 번째로는 atom의 현재 값을 나타내는 useAtom value훅의 결과를 반환하며, 두 번재로는 UseSetAtom 훅을 반환하는데, 이 훅은 atom을 수정할 수 있는 기능을 제공한다.

- useAtom 구현부

```Jsx
export function useAtom<Value, Update, Result extends void | Promise<void>>(
  atom: WritableAtom<Value, Update, Result>,
  scope?: Scope
): [Awaited<Value>, SetAtom<Update, Result>]

export function useAtom<Value>(
  atom: Atom<Value>,
  scope?: Scope
): [Awaited<Value>, never]

export function useAtom<Value, Update, Result extends void | Promise<void>>(
  atom: Atom<Value> | WritableAtom<Value, Update, Result>,
  scope?: Scope
) {
  if ('scope' in atom) {
    console.warn(
      'atom.scope is deprecated. Please do useAtom(atom, scope) instead.'
    )
    scope = (atom as { scope: Scope }).scope
  }
  return [
    useAtomValue(atom, scope),
    // We do wrong type assertion here, which results in throwing an error.
    useSetAtom(atom as WritableAtom<Value, Update, Result>, scope),
  ]
}
```

- useSetAtom 구현부

```jsx
export function useSetAtom<Value, Update, Result extends void | Promise<void>>(
  atom: WritableAtom<Value, Update, Result>,
  scope?: Scope
): SetAtom<Update, Result> {
  const ScopeContext = getScopeContext(scope)
  const { s: store, w: versionedWrite } = useContext(ScopeContext)
  const setAtom = useCallback(
    (update: Update) => {
      if (__DEV__ && !('write' in atom)) {
        // useAtom can pass non writable atom with wrong type assertion,
        // so we should check here.
        throw new Error('not writable atom')
      }
      const write = (version?: VersionObject) =>
        store[WRITE_ATOM](atom, update, version)
      return versionedWrite ? versionedWrite(write) : write()
    },
    [store, versionedWrite, atom]
  )
  return setAtom as SetAtom<Update, Result>
}
```

setAtom으로 명명돼 있는 콜백 함수 내부에서 사용하고 있는 write함수를 살펴보면, write함수는 스토어에서 해당 atom을 찾아 직접 값을 업데이트하는 것을 볼 수 있다. 그리고 스토어에서 새로운 값을 작성한 이우에는 해당 값의 변화에 대해 알고 있어야 하는 listener함수를 실행해 값의 변화가 있음을 전파하고, 사용하는 쪽에서 리렌더링이 수행되게 한다.

## 간단한 사용법

다음 코드는 Jotai에서 간단한 상태를 선언하고, 만들어진 상태로부터 파생된 상태를 사용하는 예제이다.

```jsx
import { atom, useAtom, useAtomValue } from 'jotai';

const counterState = atom(0); // Jotai의 atom함수를 사용하여 정의된 상태

function Counter() {
  const [_, setCount] = useAtom(counterState); // useAtom함수 사용하여 CounterState atom에 접근
  // useAtom은 atom의 현태 값을 읽고, 그 값을 업데이트할 수 있는 함수를 제공

  function handleButtonClick() {
    setCount((count) => count + 1);
  }

  return (
    <>
      ㄴ<button onClick={handleButtonClick}>+</button>
    </>
  );
}

const isBiggerThan10 = atom((get) => get(counterState) > 10); // counterState의 값을 기반으로 산출되는 파생상태
// get함수를 사용하여 counterState의 현재 값을 가져오고 이 값이 10보다 큰지 여부 파악

function Count() {
  // useAtomValue 훅을 사용하여 counterState와 isBiggerThan10 atom의 값을 읽음
  const count = useAtomValue(counterState);
  const biggerThan10 = useAtomValue(isBiggerThan10);

  return (
    <>
      <h2>{count}</h2>
      <p>count is bigger than 10: {JSON.stringify(biggerThan10)}</p>
    </>
  );
}

export default function App() {
  return (
    <>
      <Counter />
      <Count />
    </>
  );
}
```

먼저 Jotai에서 상태를 선언하기 위해서는 atom이라는 API를 사용하는데, 이 API는 리액트의 useState와는 다르게 컴포넌트 외부에서도 선언할 수 있다는 장점이 있다. 또한 atom은 값뿐만 아니라 함수를 인수로 받을 수 있는데, 이러한 특징을 활용해 다른 atom의 값으로부터 파생된 atom 을 만들 수도 있다. 그리고 이 atom은 컴포넌트 내부에서 useAtom을 활용해 useState와 비슷하게 사용하거나 useAtomValue를 통해 Getter만 가져올 수 있다.

## Recoil과 차이점

- Recoil

  - atom에선 각 상태값이 모두 별도의 키를 필요로 하기 때문에 이 키를 별도로 관리해야 한다.
  - atom에서 파생된 값을 만들기 위해서 selector가 필요하다.

- Jotai

  - 키를 추상화해 사용자가 키를 관리할 필요가 없다. Jotai가 별도의 문자열 키가 없이도 각 값들을 관리할 수 있는 것은 객체의 참조를 통해 값을 관리하기 때문이다. 객체의 참조를 WeakMap에 보관해 해당 객체 자체가 변경되지 않는 한 별도의 키가 없이도 객체의 참조를 통해 값을 관리할 수 있다.
  - selector없이도 Atom만으로 atom값에서 또 다른 파생된 상태를 만들 수 있다.
  - 타입스크립트로 작성돼 있어 타입을 잘 지원한다.
