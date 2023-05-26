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