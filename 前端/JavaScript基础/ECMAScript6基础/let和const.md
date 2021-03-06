## let和const

### let
ES6新增了let命令，用来声明变量。它的用法类似于var，但是所声明的变量，只在let命令所在的代码块内有效。

for循环的计数器，就很合适使用let命令。

``` javascript
for (let i = 0; i < 10; i++) {}

console.log(i);
//ReferenceError: i is not defined
```


**let不存在变量提升**，let命令改变了语法行为，它所声明的变量一定要在声明后使用，否则报错。

#### for循环和let

注意一下一下代码
``` javascript
var a = [];
for (let i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i);
  };
}
a[6](); // 6
```

上面代码中，变量i是let声明的，当前的i只在本轮循环有效，所以每一次循环的i其实都是一个新的变量，所以最后输出的是6。

这里可能会令人困惑，但确实每一次的循环，i都是一次全新的值，js引擎内部会记录每次循环i的值，而每次i都是一个全新的变量并且由上次的循环值加1。

还有一个隐藏注意点是，**for循环里，循环语句部分是一个父作用域，而循环体内部是一个单独的子作用域。**

#### 暂时性死区
在代码块内，使用let命令声明变量之前，该变量都是不可用的。这在语法上，称为“暂时性死区”（temporal dead zone，简称 TDZ）。

``` javascript
if (true) {
  // TDZ开始
  tmp = 'abc'; // ReferenceError
  console.log(tmp); // ReferenceError

  let tmp; // TDZ结束
  console.log(tmp); // undefined

  tmp = 123;
  console.log(tmp); // 123
}
```

### const命令
const声明一个只读的常量。一旦声明，常量的值就不能改变。

``` javascript
const PI = 3.1415;
PI // 3.1415

PI = 3;
// TypeError: Assignment to constant variable.
```

const声明的变量不得改变值，这意味着，const一旦声明变量，就必须立即初始化，不能留到以后赋值。

**对于const来说，只声明不赋值，就会报错。**

还要注意以下几点
 - const的作用域与let命令相同：只在声明所在的块级作用域内有效。
 - const命令声明的常量也是不提升，同样存在暂时性死区，只能在声明的位置后面使用。
 - const声明的常量，也与let一样不可重复声明。

#### const的本质
const实际上保证的，并不是变量的值不得改动，而是变量指向的那个内存地址不得改动。对于简单类型的数据（数值、字符串、布尔值），值就保存在变量指向的那个内存地址，因此等同于常量。但对于复合类型的数据（主要是对象和数组），变量指向的内存地址，保存的只是一个指针，const只能保证这个指针是固定的，至于它指向的数据结构是不是可变的，就完全不能控制了。因此，将一个对象声明为常量必须非常小心。


### let和const从顶层对象脱钩
顶层对象的属性与全局变量挂钩，被认为是JavaScript语言最大的设计败笔之一。这样的设计带来了几个很大的问题，首先是没法在编译时就报出变量未声明的错误，只有运行时才能知道（因为全局变量可能是顶层对象的属性创造的，而属性的创造是动态的）；其次，程序员很容易不知不觉地就创建了全局变量（比如打字出错）；最后，顶层对象的属性是到处可以读写的，这非常不利于模块化编程。另一方面，window对象有实体含义，指的是浏览器的窗口对象，顶层对象是一个有实体含义的对象，也是不合适的。

ES6为了改变这一点，一方面规定，为了保持兼容性，var命令和function命令声明的全局变量，依旧是顶层对象的属性；另一方面规定，**let命令、const命令、class命令声明的全局变量，不属于顶层对象的属性。也就是说，从ES6开始，全局变量将逐步与顶层对象的属性脱钩**。

``` javascript
var a = 1;
// 如果在Node的REPL环境，可以写成global.a
// 或者采用通用方法，写成this.a
window.a // 1

let b = 1;
window.b // undefined
```

上面代码中，全局变量a由var命令声明，所以它是顶层对象的属性；全局变量b由let命令声明，所以它不是顶层对象的属性，返回undefined。

### 什么时候用let和const
ES6 里新增了两种声明变量的方式，let 和 const，加上原来的 var，一共就有三种方式来声明变量了。那到底该用哪个呢？**关于“尽可能不用 var” 这一点，大家应该没有什么意见分歧**（其实还是有少数人不这么想的），关于“是用 let 还是用 const”，社区里主要有两种不同的观点：

**1)默认全用 let**，只在符合一些写代码的人的主观判断条件的时候用 const，下面举个这样的“主观判断条件”的例子（实际代码中用到 const 的几率大概会是 0.1%）：

- 你能 100% 确定该变量永远不会在其它的代码行里被重新赋值，但是该变量的初始值有可能在未来会被调整（有点配置项的意思），且
- 该变量的初始值是个数字字面量（或者多个数字组成的数学运算表达式）、字符串字面量、布尔字面量、数组字面量、对象字面量、正则字面量、symbol "字面量"（打引号是因为 symbol 其实没有字面量），且
- 该变量是个全局变量，或者是模块内的全局变量

2)默认全用 const，只有该变量需要被重新赋值才用 let （实际代码中用到 const 的几率大概会是 95%），和上面举的三个主观判断条件对比一下差异：

- 不能 100% 确定该变量永远不会被重新赋值（可能只是现在是没被重新赋值，说不定未来会），或者
- 该变量的初始值不是字面量，比如说是个函数调用表达式，或者
- 该变量是个局部变量

以上是社区观点，我更偏向于，默认用const，当你一个变量初始化以后不再重新赋值的时候，用const，一个变量会进行重新赋值和改变的时候用let。