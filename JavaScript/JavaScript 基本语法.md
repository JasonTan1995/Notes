### JavaScript 基本语法

#### 1. 语句

JavaScript 程序的执行单位为行（line），也就是一行一行地执行。一般情况下，每一行就是一个语句。

##### 赋值与声明

```javascript
var a = 1 + 3;
```

这条语句先用var命令,声明了变量a,然后将1+3的运算结果赋值给变量a.

```javascript
var a = 1 + 3; var b = 'abc'
```

**语句以分号结尾,一个分号就表示一个语句结束**,多个语句可以写在一行内.

#### 2. 变量

变量是对“值”的具名引用。变量就是为“值”起名，然后引用这个名字，就等同于引用这个值。变量的名字就是变量名。

```javascript
var a = 1;		
```

- JavaScript 的变量名区分大小写,A和a是两个不同的变量.

- 变量的声明和赋值,是分开的两个步骤,上面的代码将他们合在一起,实际的步骤是下面这样.

  ```javascript
  var  a;
  a = 1;
  ```

- 如果只是声明变量而没有赋值,则该变量的值是undefined.undefined是一个特殊的值,表示无定义(未定义).

- 建议使用var命令声明变量.如果没有声明变量就直接使用,JavaScript会报错.

- 同一条var命令中可以声明多个变量.

  ```javascript
  var a,b;
  ```

- JavaScript是一种动态类型语言.变量的类型没有限制,变量可以随时更改类型.

  ```javascript
  var a = 1;
  a = 'hello';	
  ```

- 如果使用var再次声明一个已经存在的变量,是无效的.

  ``` javascript
  var x = 1;
  var x;
  ```

- 但是,如果第二次声明的时候还进行了赋值,则会覆盖掉前面的值.

       ```javascript
var x = 1;
var x = 2;

console.log(x); //x = 2;
       ```

##### 变量提升

JavaScript 引擎的工作方式是先解析,获取所有被声明的变量.然后再一行一行地运行.这造成的结果,就是所有的变量的声明语句,都会被提升到代码的头部.这就叫做变量提升(hoisting).

```javascript
console.log(a);
var a = 1;
```

上面代码首先使用console.log方法,在控制台显示变量a的值.这时变量a还没有声明和赋值,所以这是一种错误的做法.但是实际上不会报错.因为存在变量提升.真正运行的是下面的代码.

```javascript
var a;
console.log(a);
a = 1;	
```



#### 3. 标识符

标识符命名规则如下.

1. 第一个字符,可以是任意Unicode字母(包括英文字母和其他语言的字母),以及美元符号($)和下划线(_).
2. 第二个字符及后面的字符,除了Unicode字母,美元符号和下划线,还可以用数字0-9.

下面这些是不合法的标识符.

```javascript
1a  // 第一个字符不能是数字
23  // 同上
***  // 标识符不能包含星号
a+b  // 标识符不能包含加号
-d  // 标识符不能包含减号或连词线
```

JavaScript中的保留字,不能用作标识符

```javascript
arguments、break、case、catch、class、const、continue、debugger、default、delete、do、else、enum、eval、export、extends、false、finally、for、function、if、implements、import、in、instanceof、interface、let、new、null、package、private、protected、public、return、static、super、switch、this、throw、true、try、typeof、var、void、while、with、yield
```



#### 4. 注释

源码中被JavaScript引擎忽略的部分就叫做注释,它的作用是对代码进行解释.Javascript 提供两种注释的写法：一种是单行注释，用`//`起头；另一种是多行注释，放在`/*`和`*/`之间。

```javascript
// 这是单行注释

/*
 这是
 多行
 注释
*/
```

此外,JavaScript可以兼容HTML代码的注释,所以<!--和-->也被视为合法的单行注释.

```
x = 1; <!-- x =2;-->
x = 3;
```



#### 5.区块

JavaScript 使用大括号,将多个相关的语句组合在一起,称为"区块"(block).

不过对于var 命令来说,JavaScript的区块不构成单独的作用于(scope).

```
{
    var a = 1;
}
console.log(a); //1
```

上面代码在区块内部，使用`var`命令声明并赋值了变量`a`，然后在区块外部，变量`a`依然有效，区块对于`var`命令不构成单独的作用域，与不使用区块的情况没有任何区别.

#### 6. 条件语句

##### 6.1 if 结构

`if`结构先判断一个表达式的布尔值，然后根据布尔值的真伪，执行不同的语句。所谓布尔值，指的是 JavaScript 的两个特殊值，`true`表示真，`false`表示`伪`。

```javascript
if (布尔值)
  语句;

// 或者
if (布尔值) 语句;

if(m === 3){
    m+= 1;
}
```

建议if 语句中使用大括号,因为这样方便插入语句.

注意,if 后面的表达式之中,要小心处理表达式(=),严格相等运算符(===)和相等运算符(==).

尤其是赋值表达式不具有比较作用.

```javascript
var x = 1;
var y = 2;
if(x = y){
    console.log(x);
}	
```

上面代码的原意是，当`x`等于`y`的时候，才执行相关语句。但是，不小心将严格相等运算符写成赋值表达式，结果变成了将`y`赋值给变量`x`，再判断变量`x`的值（等于2）的布尔值（结果为`true`）。

##### 6.2 if...else 结构

if 代码块后面,还可以跟一个else 代码块,表示不满足条件时要执行的代码.

```java 
if (m === 3){
    //满足条件时,执行的语句
}else {
    //不满足条件时执行的语句
}
```

#### 6.3 switch 结构

如果使用多个if...else 连在一起使用的时候,可以转为使用更方便的switch结构.

```javascript
switch (fruit) {
  case "banana":
    // ...
    break;
  case "apple":
    // ...
    break;
  default:
    // ...
}
```

1. 每个case代码块内部的break语句都不能少.否则会接下去执行下一个`case`代码块，而不是跳出`switch`结构。
2. 需要注意的是，`switch`语句后面的表达式，与`case`语句后面的表示式比较运行结果时，采用的是严格相等运算符（`===`），而不是相等运算符（`==`），这意味着比较时不会发生类型转换。

#### 7. 标签(label)

JavaScript 语言允许，语句的前面有标签（label），相当于定位符，用于跳转到程序的任意位置，标签的格式如下。

```
label:
   语句
```

标签可以试任意的标识符,但不能是保留字,语句部分可以是任意语句.

标签通常与break语句和continue语句配合使用,跳出特定的循环.

```javascript
top:
  for (var i = 0; i < 3; i++){
    for (var j = 0; j < 3; j++){
      if (i === 1 && j === 1) break top;
      console.log('i=' + i + ', j=' + j);
    }
  }
// i=0, j=0
// i=0, j=1
// i=0, j=2
// i=1, j=0
```

上面代码为一个双重循环区块，`break`命令后面加上了`top`标签（注意，`top`不用加引号），满足条件时，直接跳出双层循环。如果`break`语句后面不使用标签，则只能跳出内层循环，进入下一次的外层循环。

标签也可以用于跳出代码块.

```javascript
foo: {
  console.log(1);
  break foo;
  console.log('本行不会输出');
}
console.log(2);
// 1
// 2
```

上面代码执行到break foo,就会跳出区块.

continue语句也可以与标签配合使用.

```javascript
top:
  for (var i = 0; i < 3; i++){
    for (var j = 0; j < 3; j++){
      if (i === 1 && j === 1) continue top;
      console.log('i=' + i + ', j=' + j);
    }
  }
// i
```

上面代码中，`continue`命令后面有一个标签名，满足条件时，会跳过当前循环，直接进入下一轮外层循环。如果`continue`语句后面不使用标签，则只能进入下一轮的内层循环。





