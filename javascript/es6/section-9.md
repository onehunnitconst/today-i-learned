## 지연 평가 + Promise - L.map, map, take
* 지연 평가 + Promise로 비동기를 잘 제어할 수 있도록 할 수 있음
* 동기에서는 문제 없음
```js
go(
  [1, 2, 3],
  L.map(a => a + 10),
  take(2),
  console.log
);
```
* 비동기에서는? 정상적인 연산이 안됨
```js
go(
  [Promise.resolve(1), Promise.resolve(2), Promise.resolve(3)],
  L.map(a => a + 10),
  take(2),
  console.log
);
```
* 이를 동작하게 만들기 위해서는 수정이 필요함
  * L.map
    ```js
    const go1 = (a, f) => a instanceof Promise ? a.then(f) : f(a);

    L.map = curry(function* (f, iter) {
      for (const a of iter) {
        // yield f(a);
        yield go1(a, f);
      }
    });
    
    const take = curry((l, iter) => {
      let res = [];
      iter = iter[Symbol.iterator]();
      return function recur() {
        let cur;
        while (!(cur = iter.next()).done) {
          const a = cur.value;
          if (a instanceof Promise) {
            return a.then(a => {
              res.push(a);
              if (res.length == l) {
                return res;
              } else {
                return recur();
              }
            });
          }
          res.push(a);
          if (res.length == l) {
            return res;
          }
        }
        return res;
      } ()
    });
    ```
    - map: 이전에 만들었던 go1 함수를 활용
    - take: recur 호출을 통한 Promise 해결
  

## Kleisli Composition - L.filter, filter, nop, take
```js
go(
  [1, 2, 3],
  L.map(a => Promise.resolve(a * a))
  L.filter(a => a % 2),
  take(2),
  console.log
);
```
* 역시 비동기 처리를 위한 수정 필요
  * L.filter
    ```js
    const nop = Symbol('nop');

    L.filter = function *(f, iter) {
      for (const a of iter) {
        const b = go1(a, f);
        if (b instanceof Promise) {
          yield b.then(b => b ? a : Promise.reject(nop));
        } else {
          yield a;
        }
      }
    };

    const take = curry((l, iter) => {
      let res = [];
      iter = iter[Symbol.iterator]();
      return function recur() {
        let cur;
        while (!(cur = iter.next()).done) {
          const a = cur.value;
          if (a instanceof Promise) {
            return a.then(a => {
              res.push(a);
              if (res.length == l) {
                return res;
              } else {
                return recur();
              }
            }).catch(e => {
              if (e == nop) {
                return recur();
              } else {
                return Promise.reject(e);
              }
            });
          }
          res.push(a);
          if (res.length == l) {
            return res;
          }
        }
        return res;
      } ()
    });
    ```

## reduce에서 nop 지원
```js
const add = (a, b) => a + b;
const nop = Symbol('nop');

go(
  [1, 2, 3, 4],
  L.map(a => Promise.resolve(a * a))
  L.filter(a => Promise.resolve(a % 2)),
  reduce(add),
  console.log
);
```

* nop을 지원하도록 reduce 바꾸기
```js
  const go1 = (a, f) => a instanceof Promise ? a.then(f): f(a)

  const reduceF = (acc, a, f) => a instanceof Promise ? a.then(a => f(acc, a), e => e == nop ? acc : Promise.reject(e)) : f(acc, a);

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
          acc = reduceF(acc, cur.value, f)
          if (acc instanceof Promise) {
            return acc.then(recur);
          }
        }
        return acc;
    })
  };
```

## 지연 평가 + Promise의 효율성

## 지연된 함수열을 병렬적으로 평가하기 - C.reduce, C.take (concurrency)
* 많은 자원을 빨리 꺼내야 할 때 필요하다.
```js
const C = {}

function noop() {}
const catchNoop = arr => (arr.forEach(a => a instanceof Promise ? a.catch(noop) : a), arr)

C.reduce = curry((f, acc, iter) => {
  const iter2 = catchNoop(iter ? [...iter] : [...acc]);

  return iter ?
    reduce(f, acc, iter2) :
    reduce(f, iter2);
})

C.take = curry((l, iter) => take(l, catchNoop(iter)))

const delay500 = a => new Promise(resolve =>
  setTimeout(() => resolve(a), 500)
);

go(
  [1, 2, 3, 4, 5],
  L.map(a => delay500(a * a))
  L.filter(a => a % 2),
  C.reduce(add),
  console.log // 5개의 값이 동시에 평가
);
```

## 즉시 병렬적으로 평가하기 - C.map, C.filter
```js
C.takeAll = C.take(Infinity);
C.map = curry(pipe(L.map, C.takeAll));
C.filter = curry(pipe(L.filter, C.takeAll));
```

## 즉시, 지연, Promise, 병렬적 조합하기

* 기본
```js
go(
  [1, 2, 3, 4, 5, 6, 7, 8],
  map(a => a * a),
  filter(a => a % 2),
  map(a => a * a),
  take(2),
  console.log
);
```

* 500ms 지연 (엄격하게 평가)
```js
const delay500 = a => new Promise(resolve =>
  setTimeout(() => resolve(a), 500)
);

go(
  [1, 2, 3, 4, 5, 6, 7, 8],
  map(a => delay500(a * a)),
  filter(a => delay500(a % 2)),
  map(a => delay500(a * a)),
  take(2),
  console.log
);
```

* 지연 평가로 평가를 최소화하기
```js
const delay500 = a => new Promise(resolve =>
  setTimeout(() => resolve(a), 500)
);

go(
  [1, 2, 3, 4, 5, 6, 7, 8],
  L.map(a => delay500(a * a)),
  L.filter(a => delay500(a % 2)),
  L.map(a => delay500(a * a)),
  take(2),
  console.log
);
```

* 특정 부분 전까지 즉시 평가하기
```js
const delay500 = a => new Promise(resolve =>
  setTimeout(() => resolve(a), 500)
);

go(
  [1, 2, 3, 4, 5, 6, 7, 8],
  L.map(a => delay500(a * a)),
  C.filter(a => delay500(a % 2)), // 이 부분까지 병렬 평가
  L.map(a => delay500(a * a)),
  take(2),
  console.log
);
```

## Node.js에서 SQL 병렬 평가로 얻은 효율