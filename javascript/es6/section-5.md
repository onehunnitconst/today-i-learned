## range
* 0부터 n까지의 배열 생성
```js
const add = (a, b) => a + b;

const range = length => {
  let index = -1;
  let res = [];
  while (++index < length) {
    res.push(index);
  }
  return res;
};

log(range(5)); // [0, 1, 2, 3, 4]

log(range(2)); // [0, 1]

var list = range(4); // [0, 1, 2, 3]

log(reduce(add, list)); // 6
```

## 느긋한(Lazy) range
```js
const L = {};
L.range = function *(length) {
  let index = -1;
  while (++index < length) {
    yield index;
  }
};

var list = L.range(4); // [0, 1, 2, 3]

log(reduce(add, list)); // 6
```

## take
```js
const take = (length, iter) => {
  let res = [];
  for (const a of iter) {
    res.push(a);
    if (res.length == length) { // length만큼 값을 push하면 바로 리턴
      return res;
    }
  }
  return res;
}
```

## 이터러블 중심 프로그래밍에서의 지연 평가
* 제때 계산법
* 느긋한 계산법
* 제너레이터/이터레이터 프로토콜을 기반으로 구현

## L.map
* 지연성을 가진 map
* 제너레이터/이터레이터 프로토콜 기반
```js
const L = {}

L.map = function *(f, iter) {
  for (const e of iter) {
    yield f(e);
  }
}

var it = L.map(a => a + 10, [1, 2, 3]);
```

## L.filter
* 지연성을 가진 filter
* 제너레이터/이터레이터 프로토콜 기반
```js
const L = {}

L.filter = function *(f, iter) {
  for (const e of iter) {
    if (f(e)) {
      yield e;
    }
  }
}

var it = L.filter(a => a % 2, [1, 2, 3, 4]); // 1, 3
```

## range, map, filter, take, reduce 중첩 사용
```js
    const go = (...args) => reduce((a, f) => f(a), args)
    const pipe = (f, ...fs) => (...as) => go(f(...as), ...fs);
    const curry = f =>
      (a, ..._) => _.length ? f(a, ..._) : (..._) => f(a, ..._);

    const range = length => {
      let index = -1;
      let res = [];
      while (++index < length) {
        res.push(index);
      }
      return res;
    };

    const take = curry((length, iter) => {
      let res = [];
      iter = iter[Symbol.iterator]();
      let cur;
      while (!(cur = iter.next()).done) {
        const e = cur.value;
        if (res.length == length) { // length만큼 값을 push하면 바로 리턴
          return res;
        }
      }
      return res;
    })

    const map = curry((f, iter) => {
      let res = [];
      iter = iter[Symbol.iterator]();
      let cur;
      while (!(cur = iter.next()).done) {
        const e = cur.value;
        res.push(f(e));
      }
      return res;
    });

    const filter = curry((f, iter) => {
      let res = [];
      iter = iter[Symbol.iterator]();
      let cur;
      while (!(cur = iter.next()).done) {
        const e = cur.value;
        if (f(e)) res.push(e);
      }
      return res;
    });

    const reduce = curry((f, acc, iter) => {
      if (!iter) {
        iter = acc[Symbol.iterator]();
        acc = iter.next().value; // 3번째 인자가 없을 경우 acc로 받은 배열 값의 첫 번째 원소를 acc값으로 둠
      } else {
        iter = iter[Symbol.iterator]();
      }
      
      let cur;
      while (!(cur = iter.next()).done) {
        const e = cur.value;
        acc = f(acc, e);
      }
      return acc;
    });

    go(
      range(10), 
      map(n => n + 10),
      filter(n => n % 2),
      take(2),
      reduce((a, b) => a + b),
      console.log,
    );
```

## L.range, L.map, L.filter, take 중첩 사용