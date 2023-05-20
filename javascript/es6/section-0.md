## 평가
* 코드가 계산(Evaluation)되어 값을 만드는 것
* 예를 들어 `1+2`는 `3`으로 평가되며, `[1, 2 + 3]`은 `[1, 5]`로 평가된다.

## 일급
* 값으로 담을 수 있다.
  ```js
  const a = 10;
  ```
* 변수에 담을 수 있다.
  ```js
  const add10 => a => a + 10
  ```
* 함수의 인자로 사용될 수 있다.
  ```js
  const r = add10(a);
  ```
* 함수의 결과로 사용될 수 있다.
  ```js
  console.log(r);
  ```

## 일급 함수
* 함수를 값으로 다룰 수 있다.
  ```js
  const add5 = a => a + 5;
  console.log(add5); // 함수를 값으로 사용할 수도 있다.
  console.log(add5(5)); // 함수를 호출하여 사용할 수 있다. 함수는 평가를 한다.

  const f1 = () => () => 1;
  console.log(f1()); // 이 함수의 결과는 함수
  
  const f2 = f1() // 함수를 다른 변수에 담을 수 있다.
  console.log(f2);
  console.log(f2());
  ```
* 함수가 일급이라는 것은, 조합성과 추상화의 도구

## 고차 함수
* 함수를 값으로 다루는 함수이다.
* 고차 함수의 종류
  1. 함수를 인자로 받아서 실행하는 함수
    ```js
    const apply1 = f => f(1); // 함수를 인자로 받고 있음
    const add2 = a => a+2;
    console.log(apply1(add2)); // 인자로 넘겨준 함수를 실행
    console.log(apply1(a => a - 1));

    const times = (f, n) => {
      let i = -1;
      while (++i < n) f(i);
    }

    times(console.log, 3);

    times(a => console.log(a+10), 3);
    ```
  2. 함수를 만들어 반환하는 함수 (클로저를 만들어 반환)
    ```js
    const addMaker = a => b => a + b; // b => a + b 함수를 기억하는 클로저를 만들어 반환
    const add10 = addMaker(10);
    console.log(add10); // 함수를 리턴
    console.log(add10(5)); // 함수를 사용
    ```