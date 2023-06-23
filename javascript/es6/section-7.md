## 결과를 만드는 reduce, take
* map, filter는 지연성을 가질 수 있음
* reduce, take는 연산의 시작점을 알리는 함수
  
## queryStr
```js
const queryStr => obj => go(
  obj,
  Object.entries,
  map([k, v] => `${k}=${v}`),
  reduce((a, b) => `${a}&${b}`),
);

console.log(queryStr({ limit: 10, offset: 10, type: 'notice' }))
```

## Array.prototype.join보다 다형성이 높은 join
* Reduce 계열의 함수
* 구현
```js
const join = (sep, iter) => curry(reduce((a, b) => `${a}${sep}${b}`, iter));
```
* queryStr에 적용
```js
const queryStr = pipe(
  Object.entries,
  L.map([k, v] => `${k}=${v}`),
  join('&')
);
```

## take와 find
* Map 계열의 함수 (take 활용)
```js
const users = [
  {age: 32},
  {age: 31},
  {age: 37},
  {age: 28},
  {age: 25},
  {age: 32},
  {age: 31},
  {age: 37}
];

const find = (f, iter) => go(
  iter,
  filter(f),
  take(1),
  ([a]) => a,
)
```

## L.map과 L.filter로 map과 filter 만들기
* L.map으로 map 만들기
```js
const map = curry(pipe(
  L.map(f),
  take(Infinity) // 앞에서 만들어지는 map이 모두 평가되도록 함
));
```
* L.filter로 filter 만들기
```js
const filter = curry(pipe(
  L.filter(f),
  take(Infinity),
));
```

## L.flatten과 flatten
* 배열 내부에 Iterable한 객체가 있으면 플랫하게 펼쳐주는 기능
```js
const isIterable = (a) => a && a[Symbol.iterator];

L.flatten = function *(iter) {
  for (const a of iter) {
    if (isIterable(a)) {
      for (const b of a) yield b; // 또는 yield *a
    } else {
      yield a;
    }
  }
}

var it = [[1, 2], 3, 4, [5, [6, 7]], 8];

console.log([...L.flatten(it)]);
```


## L.deepFlat
* 배열 내부에 깊은 배열까지 모두 펼쳐준다.
```js
L.deepFlat = function *f(iter) {
  for (const a of iter) {
    if (isIterable(a)) yield *f(a);
    else yield a;
  }
};

var it = [[1, 2], 3, 4, [5, [6, 7]], 8];

console.log([...L.deepFlat(it)]);
```

## L.flatMap, flatMap
* flatten이 적용되는 map
* L.flatMap
```js
L.flatMap = curry(pipe(
  L.map,
  L.flatten,
));
```
* flatMap
```js
flatMap = curry(pipe(
  L.map,
  flatten
));
```

## 2차원 배열 다루기
```js
const arr = [
  [1, 2],
  [3, 4, 5],
  [6, 7, 8],
  [9, 10],
]
go(
  arr,
  flatten, // 내부 배열을 펼쳐줌
  filter(a => a % 2) // 홀수만 남기도록 필터링
  take(3),
  console.log,
)
```