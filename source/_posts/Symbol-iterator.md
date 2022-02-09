---
title: 使用for-of遍历对象
date: 2022-02-09 10:32:46
tags: javascript
---

如何遍历对象, 一般来说会想到 `for-in`

```javascript
let obj = {
  a: "Jane",
  b: "Kevin",
};
for (let k in obj) {
  console.log(k, obj[k]);
}
// 输出结果
// a Jane
// b Kevin
```

但是`for-in`会遍历对象的原型链, 一些继承属性就被遍历出来了, 如果只想遍历对象自身的属性这时候就得加判断了

```javascript
let obj = {
  a: "Jane",
  b: "Kevin",
};
let newObj = Object.create(obj);
newObj.c = "Duke";
newObj.e = "James";
for (let k in newObj) {
  console.log(k, newObj[k]);
}
// 输出结果
//  c Duke
//  e James
//  a Jane
//  b Kevin

for (let k in newObj)
  if (newObj.hasOwnProperty(k)) {
    {
      console.log(k, newObj[k]);
    }
  }
// 输出结果
// c Duke
// e James
```

接下来我们尝试一些其他的方式

在 `ES6`中提供了其他 `for-of`, 可以很方便的遍历 `Array`、`Map`、`Set`、`String`、某些类数组如`arguments`等数据类型, 但是却无法遍历普通的`object`对象.

查阅资料后得知, 原来可以被 `for-of` 遍历的数据类型提供了 `iterator` 接口. 那如果我们在对象上自己实现一个 `iterator` 接口, 结果会如何呢?

```javascript
newObj[Symbol.iterator] = function () {
  let index = 0;
  let self = this;
  let keys = Object.keys(self);
  return {
    next() {
      if (index < keys.length) {
        return {
          value: { key: keys[index], value: self[keys[index++]] },
          done: false,
        };
      } else {
        return {
          value: undefined,
          done: true,
        };
      }
    },
  };
};
for (let { key, value } of newObj) {
  console.log(key, value);
}
// 输出结果
// c Duke
// e James
```

现在我们通过实现了 `iterator` 接口, 使得对象可以被 `for-of` 遍历, 那么背后的原理是什么呢? 其实, `Symbol.iterator` 函数返回了一个 `next` 函数, 每次迭代都会调用 `next` 函数, 通过返回值中的 `value` 属性拿到遍历的结果,  `done` 属性来判断是否需要继续遍历.

接下来我们通过手动调用来验证一下

```javascript
let iterator = newObj[Symbol.iterator]();
let result = iterator.next();
while (!result.done) {
  let { key, value } = result.value;
  console.log(key, value);
  result = iterator.next();
}
// 输出结果
// c Duke
// e James
```


