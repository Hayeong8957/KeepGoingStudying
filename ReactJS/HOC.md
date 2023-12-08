# HOC 패턴

React는 교차 관심사를 처리하고 코드의 재사용성과 구성성을 증진하기 위해 다양한 패턴을 제공합니다. 그 중 고차 컴포넌트(HOC)와 렌더 Props가 있습니다.

리액트에서 고차 컴포넌트(Higher-Order Component, HOC)는 **컴포넌트를 인자로 받아 추가적인 props나 변경된 동작을 가진 새로운 컴포넌트를 반환**하는 함수입니다. 고차 컴포넌트는 함수형 프로그래밍에서 가져온 패턴입니다. HOC는 컴포넌트 간에 코드를 재사용, 로직 공유, 상태 추상화 및 props 조작 등을 할 수 있게 해주는 고급 기술입니다.

## 코드 재사용성과 구성성을 위한 HOC 생성

```tsx
function withExtraInfo(WrappedComponent) {
  return function (props) {
    return <WrappedComponent extraInfo='Important Info' {...props} />;
  };
}

// 일반 컴포넌트
function MyComponent(props) {
  return <div>{props.extraInfo}</div>;
}

// 고차 컴포넌트로 감싸진 컴포넌트
const EnhancedComponent = withExtraInfo(MyComponent);
```

이 예시에서 withExtraInfo는 고차 컴포넌트입니다. 이 함수는 WrappedComponent를 인자로 받고, 새로운 컴포넌트를 반환합니다. 반환된 컴포넌트는 extraInfo라는 추가적인 prop을 받게 됩니다.

## 주의사항

- HOC를 사용할 때는 원래 컴포넌트의 props를 그대로 전달해야 합니다({...props} 사용). 그렇지 않으면, 원래 컴포넌트가 기대하는 props를 받지 못할 수 있습니다.

- HOC 내부에서 컴포넌트의 생명주기 메서드나 상태를 직접적으로 조작하는 것은 피해야 합니다.

- HOC의 목적은 컴포넌트에 기능을 추가하는 것이지, 컴포넌트의 기본 동작을 변경하는 것이 아닙니다.

- 컴포넌트의 이름을 명확하게 해주는 것이 좋습니다. 디버깅 시에 HOC로 감싸진 컴포넌트를 쉽게 식별할 수 있도록 도와줍니다.

## 예시

newList를 React-query를 사용해 받고, 이를 Context API를 통해 전역에 관리하고, 커스텀 컴포넌트를 사용해 다른 컴포넌트들에게 제공하는 코드

### Context 생성 및 설정

```JSX
import React, { createContext, useContext } from 'react';
import { useQuery } from 'react-query';

// News 데이터를 위한 Context 생성
const NewsContext = createContext();

// News 데이터를 제공하는 Provider 컴포넌트
function NewsProvider({ children }) {
  const { data: newsList, isLoading, error } = useQuery('news', fetchNewsList);

  // fetchNewsList는 newsList를 가져오는 함수 (예: API 호출)
  async function fetchNewsList() {
    // API 호출 로직
  }

  return (
    <NewsContext.Provider value={{ newsList, isLoading, error }}>
      {children}
    </NewsContext.Provider>
  );
}

// Context를 사용하기 쉽게 하는 Custom Hook
function useNews() {
  return useContext(NewsContext);
}

export { NewsProvider, useNews };
```

### 고차 컴포넌트(HOC) 생성

```JSX
// 고차 컴포넌트 정의
function withNewsContext(WrappedComponent) {
  return function(props) {
    return (
      <NewsProvider>
        <WrappedComponent {...props} />
      </NewsProvider>
    );
  };
}

export default withNewsContext;
```

### HOC 사용 예

```JSX
import withNewsContext from './withNewsContext';

function MyComponent() {
  const { newsList, isLoading, error } = useNews();

  // 컴포넌트 로직
  return (
    <div>
      {/* newsList를 사용하여 뉴스 목록 렌더링 */}
    </div>
  );
}

// HOC를 사용하여 강화된 컴포넌트
const EnhancedComponent = withNewsContext(MyComponent);

export default EnhancedComponent;

```

이 코드에서 withNewsContext HOC는 MyComponent 컴포넌트를 감싸고, NewsProvider를 통해 newsList, isLoading, error 데이터를 MyComponent에 제공합니다. 이로써, MyComponent는 뉴스 데이터를 필요로 하는 모든 곳에서 재사용할 수 있으며, 데이터 로딩 및 오류 처리 로직을 중앙에서 관리할 수 있습니다.
