---
description: (响应式编程)
---

# Reactive Programming

[link here](https://zhuanlan.zhihu.com/p/27678951)

### RP: UI events高互动性Webapps，手机apps （交互性）

### 使用异步数据流进行编程

### Everything is a stream

#### _Declarative > Imperative_

Stream --> 对Stream监听并且做出相应

 参考Java8 stream API

1. 发出请求
2. 收到响应
3. 渲染响应

```javascript
var requestStream = Rx.Observable.just('https://api.github.com/users');

var responseStream = requestStream
.flatMap(function(requestUrl) {
    return Rx.Observable.fromPromise(jQuery.getJSON(requestUrl));
});

responseStream.subscribe(function(response) {
// render `response` to the DOM however you wish
}); 
```

```javascript
//merge --> two actions --> one response
var requestOnRefreshStream = refreshClickStream
.map(function() {
    var randomOffset = Math.floor(Math.random()*500);
    return 'https://api.github.com/users?since=' + randomOffset;
});

var startupRequestStream = Rx.Observable.just('https://api.github.com/users');

var requestStream = Rx.Observable.merge(
requestOnRefreshStream, startupRequestStream
); 
```



### Java Stream API 

* **No storage.** A stream is not a data structure that stores elements; instead, it conveys elements from a source such as a data structure, an array, a generator function, or an I/O channel, through a pipeline of computational operations.
* **Functional in nature. **An operation on a stream produces a result, but does not modify its source. 
* **Laziness-seeking.** Many stream operations, such as filtering, mapping, or duplicate removal, can be implemented lazily, exposing opportunities for optimization. 
* **Possibly unbounded. **While collections have a finite size, streams need not. Short-circuiting operations such as `limit(n)` or `findFirst()` can allow computations on infinite streams to complete in finite time.
* **Consumable**. The elements of a stream are only visited once during the life of a stream. Like an [`Iterator`](https://docs.oracle.com/javase/8/docs/api/java/util/Iterator.html), a new stream must be generated to revisit the same elements of the source.











