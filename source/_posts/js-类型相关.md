---
title: js-类型相关
date: 2021-07-04 21:18:56
tags:
---
无论什么语言，最基础的就是类型了，今天把js类型相关的总结下。
# 基本概念
1. 基本数据类型：Undefined、Null、Number、String、Boolean
2. 复杂数据类型：Object
3. 引用类型：Object、Array、Date、RegExp、Function

# 区分方式
## 1. 区分基本数据类型用操作符`typeof`即可 

注意点: 
 

1.`typeof` 返回值有："undefined" "number" "string" "boolean" "object" "function"  

2. `typeof null` 返回"object" 

3. 从技术上角度讲，函数在ECMAscript中是对象，不是一种数据类型。然而，函数也确实有一些特殊的属性，因此通过`typeof`操作符来区分函数和其他的对象是有必要的 （引用红宝书） 

4. 关于`null`和`undefined`的理解：之前我看过一张很形象区分这两者的图片，
![图片](https://pic1.zhimg.com/v2-ae5b7d880c10946840c813b5257ce5a2_r.jpg?source=1940ef5c) 
`null`就是用完的卷纸，只剩下空的纸筒，而`undefined`是什么都没有。 
在实际应用中，如果我们前端需要后端传给我们一个字段如`title`，然而，后端根本没传，此时就是`undefined`，或者后端传了，但是数据库没有，则此时就会得到`null`

## 2. 用`instanceof`操作符来区分引用类型
注意点：`instanceof`操作符存在多个全局作用域（像一个页面包含多个frame）的情况下，就会存在问题。

## 3. 最安全检测类型的方式是`Object.prototype.toString().call()`
> 这里假定原型没有被修改
## 4. 特定数据类型的检查方式
Array.isArray() 
isNaN()

# 区别
基本类型的变量是存放在栈内存（Stack）里的，引用类型是在堆内存中。 

基本类型的变量赋值是直接在内存中增加了一块区域。而引用类型的赋值只是改变下指针，要是引用类型的值改变，则赋值的那个变量也会改变。要是不想这样就需要用深拷贝了