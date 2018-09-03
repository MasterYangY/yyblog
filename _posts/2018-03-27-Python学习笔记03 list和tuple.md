---
layout: post
title: 'Python学习笔记03 list和tuple'
date: 2018-03-27
author: YY
cover: 'https://upload-images.jianshu.io/upload_images/11362503-74678216336f3a72.jpg'
tags:  Python
---
# 特别注意 #
`list`定义用的是`[]`,`tuple`定义用的是`()`,从定义用的括号区分

## 1、list ##
`list`为有序的元素集合，从0开始索引
```python
>>> classmates = ['Michael', 'Bob', 'Tracy']
>>> classmates
['Michael', 'Bob', 'Tracy']
>>> classmates[0]
'Michael'
```
使用`len()`函数可以获得list
```python
>>> len(classmates)
3
```
可以从最后一个开始索引，最后一个是`-1`
```python
>>> classmates[-1]
'Tracy'
>>> classmates[-3]
'Michael'
```
使用`.append()`追加元素到`list`末尾
```python
>>> classmates.append('Adam')
>>> classmates
['Michael', 'Bob', 'Tracy', 'Adam']
```
使用`.`insert()`把元素插入到指定位置
```python
>>> classmates.insert(1, 'Jack')
>>> classmates
['Michael', 'Jack', 'Bob', 'Tracy', 'Adam']
```
使用`pop()`删除,不加参数即为删除最后一个,也可以删除指定索引位置的元素
```python
>>> classmates.pop()#删除最后一个元素
'Adam'
>>> classmates
['Michael', 'Jack', 'Bob', 'Tracy']
>>> classmates.pop(1)#删除索引为1的元素
'Jack'
>>> classmates
['Michael', 'Bob', 'Tracy']
```
替换`list`的元素可以对指定索引位置赋值
```python
>>> classmates[1] = 'Sarah'
>>> classmates
['Michael', 'Sarah', 'Tracy']
```

特别注明,`list`里的元素可以是不同的数据类型
```python
>>> L = ['Apple', 123, True]
```
`list`里也可以有另一个`list`
```python
>>> s = ['python', 'java', ['asp', 'php'], 'scheme']#['asp', 'php']也是一个list
>>> len(s)
4
>>> p = ['asp', 'php']
>>> s = ['python', 'java', p, 'scheme']
```
要拿到`php`可以写`p[1]`,也可以写`s[2][2]`,此时`s`可以看做一个二维数组
`list`也可以为空,长度为0
```python
>>> L = []
>>> len(L)
0
```

## 2、tuple ##
`tuple`也是有序的元素集合,但是一旦初始化后就无法修改,不能增减也不能改变,可以利用`tuple`不可变的原理,用`tuple`代替`list`,使代码更安全
```python
>>> classmates = ('Michael', 'Bob', 'Tracy')
```
`tuple`也可以为空
```python
>>> t = ()
>>> t
()
```
但是要特别注意的是,如果`tuple`只有一个元素的时候,必须在元素后加`,`才能生成`tuple`,因为`()`既有运算小括号的意思,也有`tuple`的意思,而Python中默认按预算的小括号进行计算
```python
>>> t = (1,)
>>> t
(1,)#此处生成的是一个tuple
```
```python
>>> t = (1)
>>> t
(1)#是一个运算使t=1
```
可以用`tuple`里嵌套`list`的方法使`tuple`可变
```python
>>> t = ('a', 'b', ['A', 'B'])
>>> t[2][0] = 'X'
>>> t[2][1] = 'Y'
>>> t
('a', 'b', ['X', 'Y'])
```
`[]`中的两个元素为`list`的元素，所以可变，并不影响`tuple`的不可变性