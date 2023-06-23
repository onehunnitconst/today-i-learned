# Promise

## 일급
```js
  function add10(a, callback) {
    setTimeout(() => callback(a + 10), 100)
  }

  add10(5, (res) => {
    console.log(res);
  })

  // 콜백 지옥
  add10(5, (res) => {
    add10(res, (res) => {
      add10(res, (res) => {
        add10(res, (res) => {
          console.log(res);
        });
      });
    });
  })

  function add20(a) {
    return new Promise((resolve) => setTimeout(() => resolve(a + 20), 100));
  }

  // 프로미스의 경우 then으로 정리
  add20(5)
    .then(add20)
    .then(add20)
    .then(add20)
    .then(console.log);
```

* 콜백 패턴과 Promise의 가장 중요한 차이는 어떻게 결과를 꺼내느냐가 아닌, 비동기를 **일급**으로 다룬다는 것이다. 즉, 비동기를 값으로 취급

## 값으로써 Promise 활용

```js
const go1 = (a, f) => f(a); // a가 Promise가 아닌 값이 들어와야 실행됨
const add5 = (a) => a + 5;

console.log(go1(10, add5));

// 만약 비동기적으로 시간이 걸리게끔 하려면?
const go1 = (a, f) => a instanceof Promise ? a.then(f) : f(a);
const delay100 = (a) => new Promise((resolve) => setTimeout(() => resolve(a), 100));

console.log(go1(delay100(10), add5));
```

## 합성 관점에서의 Promise
* Promise는 함수 합성을 안전하게 하기 위한 도구로도 해석 가능 (모나드)
* 모나드는 일종의 박스, 박스 안에 값이 없으면 함수를 실행하지 앉도록 함
```js
const g = a => a + 1;
const f = a => a * a;

f(g(1));

Array.of(1).map(g).map(f).forEach(r => console.log(r));

// Promise 활용 (비동기 상황에서의 함수 합성)
Promise.resolve(1).then(g).then(f).then(r => console.log(r));
```

## 클레이슬리 컴포지션 (Kleisli Composition)
* 특정한 규칙을 만들어 합성을 안전하게 만들고, 조금 더 수학적으로 바라볼 수 있게 만드는 함수 합성
```js
  // f . g
  // f(g(x)) = f(g(x))
  // f(g(x)) = g(x)

  var users = [
    { id: 1, name: 'userA' },
    { id: 2, name: 'userB' },
    { id: 3, name: 'userC' },
  ]

  const getUserById = id => find(u => u.id == id, users)

  const f = ({ name }) => name;
  const g = getUserById;

  // 함수 합성
  const fg = id => f(g(id));

  console.log(fg(2)); // userB

  const result = fg(2);

  // 에러 상황으로 결과를 만들지 못함
  users.pop()
  users.pop()

  const result2 = fg(2);

  // 클레이슬리 컴포지션은 이런 상황에서 에러를 방지하고 안전한 합성함수를 만드는 것!

  const getUserById = id => find(u => u.id == id, users) || Promise.reject('Not Found');

  const fg = id => Promise.resolve(id)
    .then(g)
    .then(f)
    .catch(a => a); // Promise가 reject일 때

  fg(2).then(console.log);
```

## go, pipe, reduce에서 비동기 제어
```js
  go(
    1,
    a => a + 10,
    a => Promise.resolve(a + 100), // 중간에 비동기적으로 작동하는 함수가 있다면?
    a => a + 1000, // a가 Promise 객체로 평가됨
    log
  )

  const go1 = (a, f) => a instanceof Promise ? a.then(f): f(a)

  // Reduce를 손봐야 한다
  const reduce = curry((f, acc, iter) => {
    if (!iter) {
      iter = acc[Symbol.iterator]();
      acc = iter.next().value; 
    } else {
      iter = iter[Symbol.iterator]();
    }
    return go1(acc, function recur(acc) {
        let cur;
        while (!(cur = iter.next()).done) {
          const e = cur.value;
          acc = f(acc, e);
          if (acc instanceof Promise) {
            return acc.then(recur);
          }
        }
        return acc;
    })
  };
        
    // let cur;
    // while (!(cur = iter.next()).done) {
    //   const e = cur.value;
    //   acc = f(acc, e);
    //   // acc = acc instanceof Promise ? acc.then(acc => f(acc, a)) : f(acc, a);
    // }
    // return acc;
  });
```
* `acc = acc instanceof Promise ? acc.then(acc => f(acc, a)) : f(acc, a);`
  * 비동기 이후의 동기 함수들을 하나의 콜스택에서 처리하기 어려워짐
  * 불필요한 로드가 생길 수 있음 (성능 저하)

## Promise.then의 중요한 규칙
* then 메소드를 통하여 꺼낸 값은 Promise가 아니어야 한다.
* 중첩된 Promise여도 then 메소드로 한번에 값을 받을 수 있다.
