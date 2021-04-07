# JavaScript 总结



## 异步

### Promise

- 手动实现一个Promise.all

  ```javascript
  Promise.myall = function (args) {
      let len = args.length, count = 0
      let result = []
      return new Promise((resolve, reject) => {
          args.forEach((item, index) => {
              item.then((res) => {
                 result[index] = res
                  count++
                  if (count === len) {
                      resolve(result)
                  }
              })
          })
      })
  }
  
  // test
  let p1 = new Promise((resolve, reject) => {
      setTimeout(() => {
          resolve(1)
      }, 1000)
  })
  let p2 = new Promise((resolve, reject) => {
      setTimeout(() => {
          resolve(2)
      }, 500)
  })
  
  Promise.myall([p1, p2]).then((res)=> {
      console.log(res)
  })
  ```



- Promise.all 并发限制

  ```javascript
  let urls = ['a', 'b', 'c', 'd', 'e', 'f']
  
  function asyncLimit (arr, limit) {
      let lim = Math.min(arr.length, limit)
      let exe = []
      
      for (let i = 0; i< lim; i++) {
          let p = enqueue()
          exe.push(p)
      }
      
      Promise.all(exe).then(()=>{
          console.log('全部发送完成')
      })
      
      function enqueue () {
          return new Promise((resolve, reject) => {
              let url = urls.pop()
              setTimeout((res)=>{
                  console.log(res, new Date())
                  resolve()
              }, parseInt(Math.random()*5000), url)
          }).then(()=>{
              if (urls.length !== 0) {
                  return enqueue()
              }
          })
      } 
  }
  
  asyncLimit(urls, 2)
  ```

  

## 垃圾回收算法

无法判定一些内存是不是“不在需要”

- 引用
  - 一个对象有访问另一个对象的权限，叫一个对象引用另一个对象
  - 对象包括
    - Javascript对象
    - 函数作用域（全局词法作用域）
- 引用计数垃圾收集
  - “对象是否不再需要” -> "对象有没有其他对象引用到"
  - 循环引用问题
  - IE6，7使用引用技术对DOM对象垃圾回收
- 标记-清除算法
  - 设定一个根对象(Javascript是全局对象)
  - 垃圾回收器会从根节点找所有可以引用的对象



## 代理与反射

### 代理

#### 创建代理

```javascript
const target = {
    id: 'target'
}

const handler = {}
const proxy = new Proxy(target, handler)
```

#### 捕获器

- 在处理程序对象中定义的“基本操作拦截器”

- 在代理对象上调用这些操作时，代理可以在这些操作传播到目标对象之前先调用捕获器函数，从而拦截修改相应行为

  ```javascript
  const handler = {
      get () {
          return 'handle'
      }
  }
  ```

- 反射api，**全局Reflect对象**的同名方法可以重建原始行为

  ```javascript
  const handler = {
      get () {
          return Reflect.get(...arguments)
      }
  }
  
  //简洁
  const handler = {
      get: Reflect.get
  }
  
  //否则
  const handler = {
      get (target, property, receiver) {
          return target[property]
      }
  }
  ```

- 捕获器不变式

  - 如果目标对象有一个不可配置且不可写的数据属性，捕获器返回与该属性不同的值时会报错

- 可撤销代理

  - Proxy构造函数有`revocable()`方法，返回撤销函数`revoke()`

    ```javascript
    const {proxy, revoke} = Proxy.revocable(target, handler)
    
    // ...
    
    revoke()
    
    // 在使用代理会抛出TypeError
    ```

- 反射api

  - 反射api不限于捕获处理程序

  - 大多数反射api在Object类型上有对应的方法

  - 状态标记

    - 很多反射方法会返回“状态标记的布尔值”

      ```javascript
      const o = {}
      if(Reflect.defineProperty(o, 'foo', {value: 'bar'})) {
          console.log('success')
      } else {
          console.log('failure')
      }
      ```

    - 下面方法会返回**状态标记**

      - Reflect.defineProperty()
      - Reflect.preventExtensions()
      - Reflect.setPrototypeOf()
      - Reflect.set()
      - Reflect.deleteProperty()

    - 代替操作符

      - Reflect.get(); 代替对象访问操作符
      - Reflect.set(); 代替=赋值
      - Reflect.has(); 代替in或with()
      - Reflect.deleteProperty(); 代替delete操作符
      - Reflect.construct(); 代替new操作符

  - 代理另一个代理

  - 代理的问题

    - 代理的this

      ```javascript
      const target = {
          thisVal () {
              return this === proxy
          }
      }
      
      const proxy = new Proxy(target, {})
      
      target.thisVal() // false
      proxy.thisVal()  // true
      ```

    - 内部槽位

      - 有些内置类型会依赖代理无法控制的机制
      - Date类型方法依赖内部槽位[[NumberDate]]



#### 代理捕获器操作（13种）

- get()
- set()
- has()
- defineProperty()
- getOwnPropertyDescriptor()
- deleteProperty()
- ownKeys()
- getPrototypeOf()
- setPrototypeOf()
- isExtensible()
- preventExtensibles()
- apply()
- construct()



#### 代理模式（使用代理有用的编程模式）

- 跟踪属性访问
- 隐藏属性
- 属性验证
- 函数与构造函数属性验证
- 数据绑定与可观察对象



### vue3.0通过Proxy劫持对象



## 对象Object

### 创建对象

#### 工厂模式

```javascript
function create () {
    let o = {}
    o.name = '123'
    o.age = 22
    o.sayName = function () {
        console.log(this.name)
    }
    return o
}
```

#### 构造函数

都是实例属性

```javascript
function Person (name, age, job) {
    this.name = name;
    this.age = age;
    this.job = job;
    this.sayName = function () {
        console.log(this.name)
    }
}
```

#### 原型模式

会共享属性

```javascript
function Person () {}

Person.prototype.name = '123'
Person.prototype.age = 123
Person.prototype.sayName = function () {
    console.log(this.name)
}
```



### 继承

#### 原型链继承

```javascript
function SuperType () {
    this.property = true;
}

SuperType.prototype.getSuperValue = function () {
    return this.property
}

function SubType () {
    this.subproperty = false;
}

SubType.prototype = new SuperType();

SubType.prototype.getSubValue = function () {
    return this.subproperty
}
```

- 问题
  - 父类的实例属性会变为子类的原型属性
  - 子类实例化不能给父类传参

#### 盗用构造函数

```javascript
function SuperType () {
    this.colors = ["red", "blue"]
}

function SubType () {
    SuperType.call(this)
}
```



#### 组合继承

```javascript
function SuperType(name) {
    this.name = name
    this.colors = ['red', 'blue']
}

SuperType.prototype.sayName = function () {
    console.log(this.name)
}

function SubType (name, age) {
    SuperType.call(this, name)
    this.age = age
}

SubType.prototype = new SuperType()

SubType.prototype.sayAge = function () {
    console.log(this.age)
}
```



#### 原型式继承

```javascript
function object(o) {
    function F() {}
    F.prototype = o
    return new F()
}
```

- 有一个对象，想在它的基础上创建新对象
- `Object.create()`将原型式继承规范化



#### 寄生式继承

```javascript
function createAnother (original) {
    let clone = object(original)
    clone.sayHi = function () {
        console.log("hi")
    }
    return clone
}
```



#### 寄生式组合继承

```javascript
function inheritPrototype(subType, superType) {
    let prototype = object(superType.prototype)
    prototype.constructor = subType
    subType.prototype = prototype
}

function SuperType(name) {
    this.name = name
    this.colors = ['red', 'blue']
}

SuperType.prototype.sayName = function () {
    console.log(this.name)
}

function SubType (name, age) {
    SuperType.call(this, name)
    this.age = age
}

inheritPrototype(subType, superType)

SubType.prototype.sayAge = function () {
    console.log(this.age)
}
```



### js获取对象

- Object.keys()
  - 获取对象实例自身可枚举属性键
- Object.getOwnPropertyNames()
  - 获取对象实例自身全部属性键
- for...in
  - 输出对象实例自身以及原型链上可枚举属性键



## 判断数据类型 

- **typeof**

| 输入值    | 输出值 (String) |
| :-------- | :-------------- |
| undefined | undefined       |
| Function  | Function        |
| **null**  | **Object**      |
| Boolean   | Boolean         |
| String    | String          |
| Number    | Number          |
| Symbol    | Symbol          |
| Object    | Object          |

- 



## 数据类型

### Set

- 成员不能重复
- 只要键值，没有键名
- 可以遍历，`add`,`delete`,`has`

### weakSet

- 成员都是对象
- 成员都是弱引用，不会阻止垃圾回收
- 不可迭代，`add`,`delete`,`has`

### Map

- 键值对的集合
- 可以迭代

### weakMap

- 只接受对象作为键名(null除外)，不接受其他类型值作为键名
- 键名所指向的对象，不计入垃圾回收机制
- 不可迭代
- 方法：`get`,`set`,`has`,`delete`



## 高级方法

### 柯里化

- 柯里化是一种将使用多个参数的一个函数转化为一系列使用一个参数的函数的技术

  ```javascript
  function add(a, b) {
      return a + b;
  }
  
  add(1, 2)
  
  var addCurry = curry(add)
  addCurry(1)(2)
  ```

- 用途
  - 参数复用，降低通用性，提高适用性

      ```javascript
      function ajax(type, url, data) {
          var xhr = new XMLHttpRequest()
          xhr.open(type, url, true)
          xhr.send()
      }

      ajax('POST', 'www.test.com', "name=kevin")

      var ajaxCurry = curry(ajax);

      // 以 POST 类型请求数据
      var post = ajaxCurry('POST');
      post('www.test.com', "name=kevin");

      // 以 POST 类型请求来自于 www.test.com 的数据
      var postFromTest = post('www.test.com');
      postFromTest("name=kevin");
      ```
      
  - 传给回调函数，如map
  
      ```javascript
      var person = [{name: 'kevin'}, {name: 'daisy'}]
      
      // 传统
      var name = person.map(function (item) {
          return item.name;
      })
      
      //curry
      var prop = curry(function (key, obj) {
          return obj[key]
      });
      
      var name = person.map(prop('name'))
      ```
  
- 实现

  - 第一版

    ```javascript
    var curry = function (fn) {
        var args = [].slice.call(arguments, 1)
        return function () {
            var newArgs = args.concat([].slice.call(arguments))
            return fn.apply(this, newArgs)
        }
    }
    
    function add(a, b) {
        return a + b;
    }
    
    var addCurry = curry(add, 1, 2);
    addCurry() // 3
    //或者
    var addCurry = curry(add, 1);
    addCurry(2) // 3
    //或者
    var addCurry = curry(add);
    addCurry(1, 2) // 3
    ```

    

