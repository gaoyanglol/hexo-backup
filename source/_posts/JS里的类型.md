---
title: 21 进阶：JS里的类型
date: 2019-08-01 23:30:15
tags: 学习笔记
---
### 数据类型转换

#### 转换为字符串
* `toString` 方法
  ```javascript
  // number 类型
  var n = 1
  n.toString()
  '1'

  // boolean 类型
  var n = true
  n.toString()
  'true'

  // null 报错
  var n = null
  n.toString()
  Uncaught TypeError: Cannot read property 'toString' of null
  
  // undefined 报错
  var n = undefined
  n.toString()
  Uncaught TypeError: Cannot read property 'toString' of undefined

  // object 类型
  var obj = {}
  obj.toString()
  "[object Object]"
  ```
* 快捷方法
  ```javascript
  1 + ''
  '1'

  true + ''
  'true'

  null + ''
  'null'

  undefined + ''
  'undefined'

  var obj = {}
  obj + ''
  '[object Object]'
  ```
  原理：JS中加号如果发现左右任意一边有字符串，会尝试将另外一边也变成字符串。
  ```javascript
  1 + '1'
  '11'
  // 等价于
  (1).toString() + '1'
  '11'
  ```
* `String()` 全局函数
  ```javascript
  window.String(1)
  '1'

  window.String({})
  '[object Object]'

  window.String(true)
  'true'

  window.String(null)
  'null'

  window.String(undefined)
  'undefined'
  ```

#### 转换为布尔值
* `Boolean()` 函数
  ```javascript                                  
  Boolean(1)
  true

  Boolean(0)
  false

  Boolean('123')
  true

  Boolean('  ')
  true

  Boolean({})
  true

  Boolean({name:'frank'})
  true
  ```
* 快捷方法
  ```javascript
  !! 1
  true

  !! 0
  false

  !! ''
  false

  !! '   '
  true

  !! {name:'frank'}
  true
  ```
* 5个 falsy 值
  ```javascript
  !! 0
  false

  !! NaN
  false

  !! ''
  false

  !! null
  false

  !! undefined
  false
  ```

#### 转换为数字

* `Number()` 函数
  ```javascript
  Number('1')
  1
  ```
* `parseInt()`
  ```javascript
  parseInt('1', 10)
  1

  parseInt('011')  // 默认转为十进制
  11

  parseInt('011', 8)
  9

  parseInt('1s')
  1

  parseInt('s')
  NaN
  ```
* `parseFloat()`
  ```javascript
  parseFloat('1.23')
  1.23
  ```
* 快捷方法
  ```javascript
  '1' - 0
  1
  
  '1.23' - 0
  1.23

  + '-1'
  -1

  - '-1'
  1
  ```
#### 基本数据类型和复杂数据类型的区别

1. 在内存中的存储方式不同。
  
   * 基本类型存在栈内存 (stack) 中；  
   * 复杂类型存在堆内存 (heap) 中，在栈中存储引用地址。
   ```javascript
                    //      stack        heap
   var a = 1        // a |64位浮点数 |
   var b = 2        // b |64位浮点数 |  { 100
   var o = {        // o |64位地址100|   --------------
     name: 'frank', // c |  1       |    name:'frank' 
     age: 18        //   |          |    age: 18      
   }                //   |          |    gender:'male'}
   var c = true     //   |          |   
   ```
2. 访问方式不同。

   * 基本类型直接访问栈内存  
   * 复杂类型先访问对象在栈内存中的地址，再按地址访问堆内存中对象。

3. 复制机制不同

   * 基本类型：a = b 是将b中原始值的副本赋值给a，a和b相互独立，互不影响
   * 复杂类型：a = b 是将b中存储的对象的引用地址赋值给a，a和b指向同一个对象，其中一个做了改变，另一个也会受影响。
   ```javascript
   var b = {
     age : 10
   }
   var a = b
   a.age = 20
   b.age
   20
   ```
4. 例题
   ```javascript
   // 第一题          //     stack       heap
   var a = {}        // a | ADDR33 |  { 33
   a.self = a        //   |        |   -------------
   a.self.self.name  //   |        |   self: ADDR33 
   'a'               //   |        |   name:'a'     }
                     //   |        |   

                     
   
   // 第二题         //      stack       heap
   var a = {n:1}    //  a |ADDR34  |  { 34
   var b = a        //  b |ADDR34  |   -------------
   a.x = a = {n:2}  //    |        |   n : 1        
                    //    |        |   x : ADDR54   }
   alert(a.x)       //    |        |   
   undefined        //  a |ADDR54  |  { 54
   alert(b.x)       //    |        |   -------------
   [object Object]  //    |        |   n : 2        }
                    //    |        |
   ```
   
#### 垃圾回收

如果一个对象没有被引用，它就是垃圾，将被回收。
* 举例1
  ```javascript
                      //     stack       heap
  var a = {           // a |ADDR60 |  { 30 (垃圾)
    name : 'a'        // b |ADDR60 |   ----------
  }                   //   |       |   name: 'a' }
  var b = {           //   |       |
    name : 'b'        //   |       |  { 60
  }                   //   |       |   ----------
  a = b               //   |       |   name: 'b' }
  ```
* 举例2
  ```javascript
  var fn = function() {}
  document.body.onclick = fn
  fn = null

  //            stack                heap
  //       fn |null    |                    { 110 (不是垃圾)
  // document |ADDR222 |                     ---------
  //          |        |                     function }
  //          |        | 
  //          |        |                       ↑
  //          |        |
  //          |        | { 222              { 333    
  //          |        |  -------------  →  ----------------
  //          |        |  body:ADDR333 }    onclick:ADDR110 }
  ```
#### 浅拷贝 vs 深拷贝
* 深拷贝
  ```javascript
  // b 变不影响 a
                //     stack            heap  
  var a = 1     // a | 1     |
  var b = a     // b | 1     |
                //   |       |
  b = 2         // b | 2     |
  a
  1
  // 所有的基本类型，赋值就是深拷贝
  ```
* 浅拷贝
  ```javascript
  // 复杂类型（对象）的赋值是浅拷贝
                  //     stack           heap
  var a = {       // a |ADDR44 |     { 44
    name : 'a'    // b |ADDR44 |      -----------
  }               //   |       |      name : 'b' }
  var b = a
  b.name = 'b'
  a.name
  'b'
  ```