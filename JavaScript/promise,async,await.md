### 1초마다 1씩 늘어나는 함수를 작성하는데, Promise, async/await을 다 사용해서 구현해주세요

```JavaScript
function delay(time) {
  return new Promise(resolve => setTimeout(resolve, time))
  // Promise 생성자 함수 사용 법을 몰랐음
  // resolve, reject라는 두 개의 인수를 받음
  // resolve -> Promise가 성공적으로 완료도었을 때 호출
  // reject -> Promise가 실패했을 때 호출 -> 여기선 노필요
  // setTimeout에 resolve를 직접 넘겨주고 있음 
  // -> 지정된 시간 경과 후 resolve가 자동 호출하여 fullfill 상태로 변경하게 함
}

async function printNumbers() {
  for(let i = 1; i <= 10; i++) {
    await delay(1000);
    console.log(i)
  }
}

printNumbers();
```