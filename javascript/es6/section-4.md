## go
* 즉시 함수나 인자를 전달해서 값을 평가

```js
const go = (...args) => reduce((a, f) => f(a), args)


// 특정 함수로 축약 -> reduce

go(
  0,
  a => a + 1,
  a => a + 10,
  a => a + 100,
  console.log
); // 111
```

## pipe
* 합성된 함수를 리턴해줌
```js
const go = (...args) => reduce((a, f) => f(a), args)
const pipe = (f, ...fs) => (...as) => go(f(...as), ...fs);

const f = pipe(
  a => a + 1,
  a => a + 10,
  a => a + 100,
);

console.log(f(0));
```

## go를 사용하여 읽기 좋은 코드로 만들기
```js
go(
  products,
  products => filter(p => p.price < 20000, products),
  products => map(p => price, products),
  prices =>   reduce(add, prices),
  console.log
);
```

## curry
* 받아둔 함수를 내가 원하는 시점에 평가
```js
const curry = f =>
  (a, ..._) =>_.length ? f(a, ..._) : (..._) => f(a, ..._);

const mult = curry((a, b) => a * b);
console.log(mult(1)(2));

const mult2 = mult(3);

console.log(mult2(10)); //30
```
1. 먼저 함수를 리턴
2. 인자가 두 개 이상이라면 함수를 실행
3. 그렇지 않으면 함수를 리턴

* go+curry
  * map, filter, reduce에 모두 curry를 적용하면 아래와 같이 표현할 수 있음
  ```js
  go(
    products,
    products => filter(p => p.price < 20000)(products),
    products => map(p => price)(products),
    prices =>   reduce(add)(prices),
    console.log
  );
  ```
  ```js
  go(
    products,
    filter(p => p.price < 20000), //함수를 리턴하고, 함수의 인자로 product가 들어감.
    map(p => price),
    reduce(add),
    console.log
  );
  ```

## 함수 조합으로 함수 만들기
```js
const totalPrice = pipe(
  map(p => price),
  reduce(add)
);

const baseTotalPrice = predicate => pipe(
  filter(predicate),
  totalPrice,
)

go(
  products,
  baseTotalPrice(p => p.price < 20000)
  console.log,
)
```

## 총 수량, 총 가격
```js
const products = [
  { name: '반팔티', price: 15000, quantity: 1 },
  { name: '긴팔티', price: 20000, quantity: 2 },
  { name: '핸드폰케이스', price: 15000, quantity: 3 },
  { name: '후드티', price: 30000, quantity: 4 },
  { name: '바지', price: 25000, quantity: 5 },
];

const add = (a, b) => a + b;

const totalQuantity = pipe(
  map(p => p.quantity),
  reduce((a, b) => a + b),
);

const totalPrice = pipe(
  map(p => p.price * p.quantity),
  reduce((a, b) => a + b),
);
```

```js
const add = (a, b) => a + b;

const sum = (f, iter) => go(
  iter,
  map(f),
  reduce(add),
);

const totalQuantity = products => sum(p => p.quantity, products);

const totalPrice = products => sum(p => p.price * p.quantity, products);
```

```js
const add = (a, b) => a + b;

const sum = curry((f, iter) => go(
  iter,
  map(f),
  reduce(add),
));

const totalQuantity = products => sum(p => p.quantity);

const totalPrice = products => sum(p => p.price * p.quantity);
```