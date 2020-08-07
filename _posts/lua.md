---
title: Lua
date: 2020-08-05 10:02:13
---
##### Lua 简单介绍
##### Lua 语法
##### Lua 数据结构



##### Lua好玩的
```lua
-- 配合各种Escape Sequence
string.rep('hahaha\t', 10)
```
##### 相关介绍

- lua 的 metatable 和 js的 prototype
```javascript
js:
var person = {
    name: "xkx",
    age: "25",
    info: function() {
        console.log("name:" + this.name, "age:" + this.age)
    }
}
属性值:var name = person.name
返回函数的定义:var info = person.info
定义函数的结果返回:var info_function_result = person.info()

构造函数：
function Person(name, height) {
    this.name = name;
    this.height = height;
}
通过构造函数生成对象实例时，会将对象实例的原型指向构造函数的prototype属性。每一个构造函数都有一个prototype属性，这个属性就是对象实例的原型对象
var p = new Person("xkx","24");
p.__proto__  == Person.prototype  true
p.constructor == Person.prototype.constructor true


JS中万物皆对象。方法（Function）是对象，方法的原型(Function.prototype)是对象。因此，它们都会具有对象共有的特点。即：对象具有属性__proto__，可称为隐式原型，一个对象的隐式原型指向构造该对象的构造函数的原型，这也保证了实例能够访问在构造函数原型中定义的属性和方法
方法这个特殊的对象，除了和其他对象一样有上述_proto_属性之外，还有自己特有的属性——原型属性（prototype），这个属性是一个指针，指向一个对象，这个对象的用途就是包含所有实例共享的属性和方法（我们把这个对象叫做原型对象）。原型对象也有一个属性，叫做constructor，这个属性包含了一个指针，指回原函数
```


- There are two kinds of expressions in Lua 

lvalue − Expressions that refer to a memory location is called "lvalue" expression. An lvalue may appear as either the left-hand or right-hand side of an assignment.

rvalue − The term rvalue refers to a data value that is stored at some address in memory. An rvalue is an expression that cannot have a value assigned to it, which means an rvalue may appear on the right-hand side, but not on the left-hand side of an assignment.





##### 参考
- https://www.tutorialspoint.com/lua/index.htm
- https://www.lua.org/manual/5.3/manual.html

