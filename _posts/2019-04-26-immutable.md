---
layout: mypost
title: Immutable—不可变数据
categories: [React]
---

## 一、什么是Immutable

Facebook 工程师 Lee Byron 花费 3 年时间打造，与 React 同期出现，但没有被默认放到 React 工具集里。它内部实现了一套完整的 Persistent Data Structure（持久化数据结构），还有很多易用的数据类型。像 `Collection`、`List`、`Map`、`Set`、`Record`、`Seq`。有非常全面的`map`、`filter`、`groupBy`、`reduce`、`find`函数式操作方法。同时 API 也尽量与 Object 或 Array 类似。

其中有 3 种最重要的数据结构说明一下：

- Map：键值对集合，对应于 Object，ES6 也有专门的 Map 对象
- List：有序可重复的列表，对应于 Array
- Set：无序且不可重复的列表

###  1.1 持久化数据结构

Immutable.js提供了7种不可变的数据类型: `List`、`Map` `Stack` `OrderedMap` `Set` `OrderedSet`、`Record`。对Immutable对象的操作均会返回新的对象，例如:

```
var obj = {count: 1};
var map = Immutable.fromJS(obj);
var map2 = map.set('count', 2);

console.log(map.get('count')); // 1
console.log(map2.get('count')); // 2
```

关于Persistent data structure 请查看 [wikipedia](https://en.wikipedia.org/wiki/Persistent_data_structure)

### 1.2 结构共享

当我们对一个Immutable对象进行操作的时候，ImmutableJS基于哈希映射树(hash map tries)和vector map tries，只clone该节点以及它的祖先节点，其他保持不变，这样可以共享相同的部分，大大提高性能。

```
var obj = {
  count: 1,
  list: [1, 2, 3, 4, 5]
}
var map1 = Immutable.fromJS(obj);
var map2 = map1.set('count', 2);

console.log(map1.list === map2.list); // true
```

下图可解释结构共享的过程:

![Immutable Change](http://zhenhua-lee.github.io/img/immutable/change.gif)

### 1.3 延迟操作

ImmutableJS借鉴了Clojure、Scala、Haskell这些函数式编程语言，引入了一个特殊结构`Seq(全称Sequence)`, 其他Immutable对象(例如`List`、`Map`)可以通过`toSeq`进行转换。

`Seq`具有两个特征: 数据不可变(Immutable)、计算延迟性(Lazy)。在下面的demo中，直接操作1到无穷的数，会超出内存限制，抛出异常，但是仅仅读取其中两个值就不存在问题，因为没有对map的结果进行暂存，只是根据需要进行计算。

```
Immutable.Range(1, Infinity)
.map(n => -n)
// Error: Cannot perform this action with an infinite size.

Immutable.Range(1, Infinity)
.map(n => -n)
.take(2)
.reduce((r, n) => r + n, 0); 
// -3
```

### 1.4 强大API机制

ImmutableJS提供了大量的方法，有些方法沿用原生js的类似，降低学习成本，有些方法提供了便捷操作，例如`setIn`、`UpdateIn`可以进行深度操作。

```
var obj = {
  a: {
    b: {
      list: [1, 2, 3]
    }
  }
};
var map = Immutable.fromJS(obj);
var map2 = map.updateIn(['a', 'b', 'list'], (list) => {
  return list.push(4);
});

console.log(map2.getIn(['a', 'b', 'list']))
// List [ 1, 2, 3, 4 ]
```

## 二、为什么使用Immutable

### 2.1 js中引用的副作用

js中存在两种数据结构: 基础类型(string、number、boolean、null、undefined)、引用类型(object)。js中的对象非常灵活、多变，这给我们的开发带来了不少好处，但是也引起了非常多的问题。

业务场景1:

```javascript
const obj = {
	count: 1
};
const clone = obj;
clone.count = 2;

console.log(clone.count) // 2
console.log(obj.count) // 2
```

业务场景2:

```javascript
const obj = {
	count: 1
};

unKnownFunction(obj);
console.log(obj.count) // 不知道结果是多少? 
```

### 2.2 深度拷贝的性能问题

针对引用的副作用，有人会提出可以进行深度拷贝(`deep clone`), 请看下面深度拷贝的代码:

```javascript
function isObject(obj) {
  return typeof obj === 'object';
}

function isArray(arr) {
  return Array.isArray(arr);
}
function deepClone(obj) {
  if (!isObject(obj))  return obj;
  var cloneObj = isArray(obj) ? [] : {};
  
  for(var key in obj) {
    if (obj.hasOwnProperty(key)) {
      var value = obj[key];
      var copy = value;
      
      if (isObject(value)) {
        cloneObj[key] = deepClone(value);
      } else {
        cloneObj[key] = value;
      }
    }
  }
  return cloneObj;
}

var obj = {
  age: 5,
  list: [1, 2, 3]
};

var obj2 = deepClone(obj)
console.log(obj.list === obj2.list) // false
```

假如仅仅只是对`obj.age`进行操作，使用深度拷贝同样需要拷贝`list`字段，而两个对象的`list`值是相同的，对`list`的拷贝明显是多余，因此深度拷贝存在性能缺陷的问题。

```javascript
var obj = {
  age: 5,
  list: [1, 2, 3]
};
var obj2 = deepClone(obj)
obj2.age = 6;
```

### 2.3 js本身的无力

```javascript
const shitObj = {
    first: {
        second: [
        	{
            	third: [
            		{
            			hello: 1
            		},
            		{
            			world: 2
            		}
            	]
        	}
        ]
    }
};
//更新shitObj.first.second[0].third[0].hello的值

// 原生方法
shitObj.first.second[0].third[0].hello = 2;

// Immutable
let immutableMap = Immutable.Map(shitObj);
immutableMap = immutableMap.setIn(['first', 'second', 0, 'third', 0, 'hello'], 2);

// 获取hello的值呢？
// 当third属性不存在时，获取hello的值时呢
```

## 三、如何使用Immutable

### 3.1 安装

```shell
npm install -D immutable
```

### 3.2 使用

```javascript
import Immutable, { Map, List } from 'immutable';

// 直接创建Immutable对象
const $$map1 = Map({a: 1});
const $$map2 = map.set('a', 2);
console.log('map1.a:', $$map1.get('a'));
console.log('map2.a:', $$map2.get('a'));

// 将已有对象转化为Immutable对象
const obj = {a: 1};
const &&map = Map(obj);

const arr = [1, 2, 3, 4, 5];
const $$list = List(arr);

const deepObj = {
    a: [
        {
            b: 1,
        },
        {
            c: 1,
        },
        {
            d: 1,
        },
    ]
};
const &&deepMap = Immutable.fromJS(deepObj);

// 将Immutable对象转化为原生对象
&&map.toObject();
&&list.toArray();
&&deepMap.toJS();
```

### 3.3 与Vue结合？

Vue的反应性系统依赖于观察对象和数组的突变。底层使用了Object.defineProperty为每个属性创建getter和setter。

同样VueX默认是可变的，如果在 store上使用了 immutable 结构，将不利于监听数据变化。

故可以在store 的数据使用普通的数据，在需要这些数据的地方通过 immutable 提供的 `fromJS` 转换,在需要普通数据的地方再通过 immutable 的 `toJS` 转换成普通数据。

可在以下位置使用Immutable：

1. props、data
2. events
3. utils

## 四、总结

1. Immutable虽好，但不实用于Vue，同时网络上使用Immutable的大部分是React应用。
2. 使用Immutable有学习成本，Immutable提供了大量的API。在应用中会存在两种对象，Immutable对象和Object，在开发时难免需要思维上的转换。
3. Immutable源文件过大，源码共5K+行，压缩后16kb。