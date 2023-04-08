---
title: "为什么forEach不支持async/await"
description: "想同时对Array中的每个元素执行异步函数，使用forEach+async/await为什么错误？应该如何编写？"
pubData: "April 08 2023"
heroImage: "/ts.png"
---

## forEach、Promise.all和for...of

在下面的代码中forEach、Promise.all和for...of会表现出不同的结果

```ts
function sleep(ms: number) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

async function processItem(item: number) {
  // Simulate a long-running asynchronous operation
  await sleep(Math.random() * 100);
  return item * 2;
}

async function testForEach(array: number[]) {
  const result: number[] = [];
  array.forEach(async item => {
    const processed = await processItem(item);
    result.push(processed);
  });
  console.log("forEach:", result);
}

async function testPromiseAll(array: number[]) {
  const promises = array.map(processItem);
  const result = await Promise.all(promises);
  console.log("Promise.all:", result);
}

async function testForOf(array: number[]) {
  const result: number[] = [];
  for (const item of array) {
    const processed = await processItem(item);
    result.push(processed);
  }
  console.log("for...of:", result);
}

const array = Array.from({ length: 1000000 }, () => Math.floor(Math.random() * 1000));

testForEach(array);
testPromiseAll(array);
testForOf(array);
```

![compare](/foreach.png)

结果：使用foreach并没有等待异步操作完成；使用Promise.all同时开启了多个异步操作并等待这些异步操作完成；使用for...of则是一个一个的顺序执行异步操作，因此花费了更长的时间。


## 为什么forEach不支持async/await

通过以上例子我们可以看到，forEach只支持同步，不支持异步。

这是因为forEach是没有return返回值的，forEach里面的回调函数因为加了async，所以默认会返回一个promise，但是因为forEach的实现并没有返回值，所以导致返回的这个promise对象不能被处理。

而map也是跟forEach是一样的，在map中使用async/await，同样也是不能等待异步操作的完成。

而Promise.all的返回不是void也不是promise\<void\>，而是promise\<T\>，所以可以等待异步操作完成。

```ts
// 下面代码会输出和forEach一样的结果
async function testMap(array: number[]) {
const result: number[] = [];
array.map(async item => {
    const processed = await processItem(item);
    result.push(processed);
});
console.log("map:", result);
}

```

## 总结

+ **forEach、map**: forEach或map方法遍历数组，并为每个元素启动一个异步操作，但是不会等待异步操作的完成。
+ **Promise.all** 方法将所有的异步操作并行执行，并等待所有的操作完成。这种方法可以最大限度地利用 CPU 和 IO 资源，因此可以更快地处理数据。
+ **for...of** 循环遍历数组，并在每个元素上执行异步操作。由于 for...of 循环是一个同步方法，它会等待每个异步操作完成，然后才会继续执行下一个操作。这种方法可能会比 Promise.all 方法慢，但是它可以避免一次性处理过多的数据，从而提高系统的稳定性。