---
title: 20 进阶：JS里的数据
date: 2019-07-23 23:37:02
tags: 学习笔记
---
### Javascript中的数据类型

> 1. `Number`（数字）
> 2. `String` （字符串）
> 3. `Boolean` （布尔）
> 4. `Symbol` （符号）
> 5. `undefined`
> 6. `null`
> 7. `Object` （对象）

*JS一切皆对象的说法是错的*

#### `Number`

* 十进制

    整数：`1`  
    小数：`1.1`  
    ```javascript
    > 1 + .1
    < 1.1
    ```  
    科学记数法：1.23e2  
    ```javascript
    > 1.23e2
    < 123
    ```  
* 二进制  
    ```javascript
    > 0b11
    < 3
    ```  
* 八进制
    ```javascript
    > 011
    < 9
    ```  
    **电话号码不能用 `number` 类型存储**  
    ```javascript
    //电话号码不应该存储为数字类型
    > var phonenumber = 01012345
    > phonenumber
    < 267493
    //应该使用字符串
    > var phonenumber = "010-12345"
    > phonenumber
    < "010-12345"
    ```  
* 十六进制
    ```javascript
    > 0x11
    < 17
    ```

#### `String`

* 用 `''` 或者 `""` 包裹字符来表示。
  ```javascript
  > '你好'
  > "你好"
  ```  
* 空字符串 `''`、`""` 与空格字符串 `' '`、`" "` 是不同的。  
  ```javascript
  //空字符串长度为0
  > ''
  > ""
  //空格字符串长度为1
  > ' '
  > " "
  ```  
* 转义字符 `\`  
  ```javascript
  > var a = '\''  //单引号
  > var n = '\n'  //回车
  > var t = '\t'  //Tab
  > var b = '\\'  //反斜杠
  //以上长度都是1
  ```  
* 多行字符串  
  ```javascript
  > var s = '12345' +
            '67890'
  //ES6的写法
  > var s = `12345
    67890`
  > s
  < "12345↵67890"
  ```

#### `Boolean`

只有两个值 `true` 和 `false`

* `a && b`

  | b \ a | true  | false |
  | ----- | ----  | ----- |
  | true  | true  | false |
  | false | false | false |

* `a || b`

  | b \ a | true  | false |
  | ----- | ----  | ----- |
  | true  | true  | true  |
  | false | true  | false |

#### `Symbol`

暂时用不上，可以搜 “ 方应杭 + Symbol ” 了解。

#### `null` 和 `undefined`

表示没有值。  
**JS设计的bug，表示什么也没有。**   
1. 一个变量未赋值 - `undefined`（语法）  
    ```javascript
    > var n
    > typeof(n)
    < 'undefined'
    ```
2. 对象object，现在不想赋值 - `null`（惯例）  
   非对象，现在不想赋值 - `undefined`  
   ```javascript
    > var object = null  //空对象
    > var n  //空非对象
   ```

#### `Object`

* 对象就是简单类型（number、string、boolean...）的组合。  
    ```javascript
    > var obj = {
        key0 : value0,
        key1 : value1
      }

    // key 是 String 类型，可以省略引号，但必须遵循变量命名规则
    > var obj = {9a : 'frank'}
    
    // 变量名不可以用数字开头
    > var obj = {'9a' : 'frank'}

    // value 可以是任何类型 number, string, boolean, object, null, undefined
    > var person = {
        name : 'frank',
        age : 18,
        married : true,
        children: {name: 'xxx', age: 1}
      }
* 取值方式 1
    ```javascript 
    // 中括号取值必须加引号
    > person['name']
    < 'frank'
    // 不加引号会被解析为变量
    > var name = 'fuck'
    > person[name]
    < undefined
    ```
* 取值方式 2  
    ```javascript
    > person.name  // name 符合标识符规范的情况下
    < 'frank'  
    ```
* 删除键值  
    ```javascript
    > var person = { name : 'frank' }
    
    // delete 将 key 和 value 都删除
    > delete person['name']
    > person['name']
    < undefined
    > 'name' in person
    < false
    
    // 仅清除 value
    > person.name = undefined
    > 'name' in person
    < true
    ```
* 遍历对象
    ```javascript
    > var person = {name: 'frank', age: 18}
    > for (var key in person) {
        console.log(key, person[key])
      }
      name frank
      age 18
    ```
* `typeof` 方法

    |   |`string`|`number`|`boolean`|`undefined`|`object`|`symbol`|
    |---|---|---|---|---|---|---|
    |`typeof`|'string'|'number'|'boolean'|'undefined'|'object'|'symbol'|
    
    两个BUG
    ```javascript
    //
    > var n = null
    > typeof n
    < 'object'

    //
    > function f() {}
    > typeof f
    < 'function'
    ```