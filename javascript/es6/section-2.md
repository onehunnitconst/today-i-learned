## 제너레이터/이터레이터
* 제너레이터: 이터레이터이자 이터러블을 생성하는 함수
```js
function *gen() {
  yield 1;
  yield 2;
  yield 3;
}

let iter = gen();
log(iter[Symbol.iterator]())
```

## 제너레이터 예시

```js
function *odds(l) { // l번 진행 가능한 이터레이터
  for (let i = 0; i < l; i++) {
    if (i % 2) yield i;
  }
}

let iter = odds(10); // 10번 진행
```

```js
function *infinity(i = 0) {
  while (true) yield i++;
}

let iter = infinity(); // 무한히 진행. while(true)가 사용되었지만 next()를 할 때만 실행되므로 프로그램에 문제가 되진 않는다.
```

```js
function *infinity(i = 0) {
  while (true) yield i++;
}

function *odds(l) { // l번 진행 가능한 이터레이터
  for (const a of infinity(1)) {
    if (a % 2) yield a;
    if (a == l) return;
  }
}

let iter = odds(10); // 10번 진행

for (const a of iter) console.log(a); // 1, 3, 5, 7, 9
```

```js
function *infinity(i = 0) {
  while (true) yield i++;
}

function *limit(l, iter) {
  for (const a of infinity(1)) {
    yield a;
    if (a == l) return;
  }
}

let iter = limit(10); // 10번 진행

for (const a of iter) console.log(a); // 1, 2, 3, 4, 5, 6, 7, 8, 9, 10

console.log(...odds(5)); // 전개 연산자
console.log([...odds(5)]); // 전개 연산자

const [head, ...tail] = odds(5); // 구조 분해
```