# 코드 스니펫

## 자료구조

#### Graph

```js
function Graph(start, end) {
  if (!new.target) {
    return new Graph(start, end);
  }
  range(start, end).forEach(i => (this[i] = []));
}
Graph.prototype.appendEdge = function (u, v) {
  this[u].push(v);
};
```

## 행렬, 2차원 배열

#### Symmetry

```js
const reverse = arr => [...arr].reverse();

const reverseUD = arr => reverse(arr);
const reverseLR = arr => arr.map(reverse);
```

#### Transpose

```js
const transpose = matrix =>
  matrix[0].map((col, i) => matrix.map(row => row[i]));
```

#### Rotate

```js
const rotateCW = arr => reverseLR(transpose(arr));
const rotateCCW = arr => reverseUD(transpose(arr));
```

## 함수 조작

#### memoize

```js
const memoize = (fn, resolver = (...args) => args.join('-')) => {
  const cache = {};
  return (...args) => {
    const key = resolver(...args);
    if (key in cache) {
      return cache[key];
    }
    return (cache[key] = fn(...args));
  };
};
```
