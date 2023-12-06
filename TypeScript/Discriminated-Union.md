# Discriminated Union

고급 타입 시스템의 한 부분으로, 유니온 타입(Union Types)내의 서로 다른 타입들을 쉽게 구별할 수 있게 해주는 패턴입니다.
이 패턴은 "태그된 유니온(Tagged Union)" 또는 "대수적 타입(Algebraic Data Types)"이라고도 불립니다.

Discriminated Union을 사용하는 주요 이유는 유니온 타입의 다양한 변형을 구별하여, 타입 안전성을 강화하고 오류 가능성을 줄이는 것입니다. 각 변형은 일반적으로 공통된 리터럴 타입 속성(일반적으로 kind나 type 같은)을 가지며, 이 속성을 사용하여 유니온 내의 구체적인 타입을 구별합니다.

## 작성 방법

1. 공통 속성 정의

모든 타입에 공통적인 리터럴 타입 속성을 정의합니다.

2. 유니온 타입 생성

이 공통 속성을 가진 타입들을 하나의 유니온 타입으로 결합합니다.

3. 타입 가드 사용

타입스크립트의 타입 가드 기능을 사용하여, 유니온 타입의 구체적인 타입을 구별하고 접근합니다.

## 예시

```TypeScript
// 각 타입에 공통 속성(type)을 정의
interface Circle {
  type: "circle";
  radius: number;
}

interface Square {
  type: "square";
  sideLength: number;
}

// 유니온 타입 생성
type Shape = Circle | Square;

// 함수에서 타입 가드를 사용하여 구별
function getArea(shape: Shape): number {
  switch (shape.type) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.sideLength ** 2;
  }
}
```

이 예시에서 Circle과 Square 인터페이스는 공통 속성 type을 가지고 있으며, Shape 유니온 타입을 통해 결합됩니다.

getArea 함수는 shape의 type 속성을 확인하여 적절한 타입에 따라 처리합니다.

이러한 방식으로 Discriminated Union을 사용하면 코드의 안정성을 높이고, 타입에 따른 별도의 처리를 보다 명확하게 구현할 수 있습니다.
