## map
### for...of를 활용할 때
```js
const products = [
  { name: '반팔티', price: 15000 },
  { name: '긴팔티', price: 20000 },
  { name: '핸드폰케이스', price: 15000 },
  { name: '후드티', price: 30000 },
  { name: '바지', price: 25000 },
];

let names = [];
for (const p of products) {
  names.push(p.name);
}
console.log(names);
```

### 커스텀 map을 활용할 때
```js
const map = (f, iter) => {
  let res = [];
  for (const e of iter) {
    names.push(f(p)); // 어떤 값을 수집할지는 f 함수에게 위임함.
  }
  console.log(res);
}

// 사용
console.log(map(p => p.name, products));
```

* 함수형 프로그래밍은 함수에 보조 함수를 매개변수로 전달하여 사용할 수 있음
* 함수를 값으로 다루는 고차함수의 일종

### 이터러블 프로토콜을 따른 map의 다형성
```js
console.log([1, 2, 3].map(a => a + 1));

console.log(document.querySelectorAll('*').map(el => el.nodeName)); // map 함수가 구현이 안되어있음

// 앞에서 만든 코드는 사용 가능
// 이터러블/이터레이터 프로토콜을 사용하고 있기 때문!
console.log(map(el => el.nodeName, document.querySelectorAll('*')));
```

## filter
### for...of
```js
let under20000 = []
for (const p of products) {
  if (p.price < 20000) under20000.push(p);
}
console.log(...under20000);
```

### filter 함수로 추출 리팩토링
```js
const filter = (f, iter) => {
  let res = [];
  for (const e of iter) {
    if (f(e)) res.push(e);
  }
  return res;
}

filter(p => p.price < 20000, products);
```

## reduce
### for...of로 구현할 때
```js
const nums = [1, 2, 3, 4, 5];

let total = 0;

for (const n of nums) {
  total = total + n;
}

log(total);
```

### reduce 함수로 추출 리팩토링
* 기본 구현
```js
const reduce = (f, acc, iter) => {
  if (!iter) {
    iter = acc[Symbol.iterator]();
    acc = iter.next().value; // 3번째 인자가 없을 경우 acc로 받은 배열 값의 첫 번째 원소를 acc값으로 둠
  }
  for (const a of iter) {
    acc = f(acc, a);
  }
  return acc;
};

const add = (a, b) => a + b;

console.log(reduce(add, 0, [1, 2, 3, 4, 5]));

// 동작 방식
console.log((add(add(add(add(0, 1), 2), 3), 4), 5));
```
* 복합적인 객체 역시 사용가능
```js
console.log(reduce((total_price, product) => total_price + product.price, 0, products));
```

## 중첩 사용
* 2만원 이하의 상품들만 추출
```js
console.log(map(p => p.price), filter(p => price < 20000, products));
```

