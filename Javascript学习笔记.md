# Javascript学习笔记

## 初识Javascript

### Javascript的组成：

ECMAScript（JavaScript语法），DOM（文档对象模型），BOM（浏览器对象模型）

### JavaScript的作用：

表单动态校验，网页特效，服务端开发（Node.js），桌面程序、App、控制硬件、物联网、游戏开发

### JavaScript的书写位置：

#### 行内式：

<input type="button" value="xxx"/>

#### 内嵌式：

<script>alert("xxxxx")</script>

#### 外部式：

<script src="./xxxx.js">

## 变量

### 变量的定义：

变量是一个用来存放数据的空间，可以通过变量名开访问或者修改变量的值

### 变量的使用：

#### 变量声明：

```javascript
var a
```



#### 变量赋值：

```javascript
a = 10
```

#### 变量的初始化：

即声明+赋值

```javascript
var a = 10
```



#### 变量语法扩展：

倘若变量已经初始化：var a = 10；然后再给a赋值：a=11，那么先前给a的值将被后面的值覆盖掉

#### 特殊情况：

若只声明不赋值，控制台输出这个变量结果为未定义（undefined）；

若直接输出一个未初始化的变量，那么控制台输出报错（error）；

只赋值不声明，输出这个值

#### 变量命名规范：

以字母，下划线，$，_组成，不能以数字开头，严格区分大小写，不能是关键字（例如：var var）,最好式驼峰命名

#### 数据类型：

Number，String，Boolean，Undefined，Null

#### Number：

数字类型拥有最大值：

```javascript
Number.MAX_value
```

```javascript
Number.MIN_value
```

以及无限大：Infinity，无限小：-Infinity，非数字类型：NaN

可以用isNaN()判断数据是否是非数字类型

#### String：

字符串型可以是引号内的任意文本，语法为单引号和双引号

字符串的长度：字符串内的字符个数就是字符串的长度

```javascript
var msg = '我真帅'
console.log(msg.length)
```

#### 字符串拼接

口诀：数值相加、字符相连

```javascript
//字符串拼接
alert('hello'+'world');//hello world
//数值字符串相接
alert('100'+'100');//100100
//数值字符串+数值
alert('11'+12);//1112
//字符串拼接加强
var age = 18;
alert('我今年'+age+'岁了')
```

