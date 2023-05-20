## ES5와 ES6 이후의 리스트 순회
* ES5: `for i++`
* ES6 이후: `for of`

```js
const list = [1, 2, 3];

// ES5
for (var i = 0; i < list.length; i++) {
  console.log(list[i]);
}

// ES6
for (const a of list) {
  console.log(a);
}
```

## Array, Set, Map
* Array
  ```js
  const arr = [1, 2, 3];

  let iter1 = arr[Symbol.iterator]();
  iter1.next();
  iter1.next();

  for (const a of arr) {
    console.log(a); // 3
  }
  ```
* Set
  ```js
  const set = new Set([1, 2, 3]);
  for (const a of set) {
    console.log(a);
  }
  ```
* Map
  ```js
  const map = new Map([['a', 1], ['b', 2], ['c', 3]]);
  for (const a of map) {
    console.log(a);
  }
  ```

## 이터러블/이터레이터 프로토콜
* 이터러블: 이터레이터를 리턴하는 `[Symbol.iterator]()`를 가진 값
* 이터레이터: `{ value, done }` 객체를 리턴하는 `next()`를 가진 값
  * 순회가 진행 중이면 `{ value: (값), done: false }`
  * 순회가 끝났으면 `{ value: undefined, done: true }`
* 이터러블/이터레이터 프로토콜: 이터러블을 `for...of`, 전개 연산자 등과 함께 동작하도록 한 규약

## 사용자 정의 이터러블
```js
  const iterable = {
    [Symbol.iterator]() {
      let i = 3;
      return {
        next () {
          return i == 0 ? { done: true }: { value: i--, done: false }
        }
      }
    }
  };

  let iterator = iterable[Symbol.iterator](); 

  for (const a of iterable) {
    console.log(a);
  } // 순회가 가능해짐
```

* 잘 구현한 이터레이터는 이터레이터를 진행하다 순회를 할 수도 있고, 이터레이터를 for문으로 모든 값을 순회할 수 있도록 되어있음

## 전개 연산자
* 전개 연산자도 이터러블/이터레이터 프로토콜을 따른다.
* 이터러블/이터레이터 프로토콜을 null로 두면 사용할 수 없다. (이터레이터가 아니게 되기 때문)
```js
const a = [1, 2];
console.log([...a, ...[3, 4]]);
```