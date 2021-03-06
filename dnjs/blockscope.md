### 词法作用域/函数作用域/块级作用域
#### 词法作用域
> 简单来说，词法作用域就是定义在词法阶段的作用域，也就是由我们在写代码时将变量和块作用域写在哪里决定的，因此当词法分析器处理代码时会保持作用域不变。 <br>

作用域查找会在找到第一个匹配的标识符时停止。在多层嵌套作用域中可以定义同名的标识符——“遮蔽效应”（内部的标识符“遮蔽”了外部的标识符）。抛开遮蔽效应，作用域的查找始终从运行时所处的最内部作用域开始，逐级向外或者说是向上进行，直到遇见第一个匹配标识符为止。 <br>
> 全局变量会自动全局对象的属性，因此可以不直接通过全局对象的词法名称，而是间接通过对全局对象属性的引用来访问全局对象。这样可以访问那些被同名变量所遮蔽的全局变量。 <br>
但是非全局变量，如果被遮蔽了，就无法访问了。 <br>

无论函数在哪里被调用，无论函数如何被调用，它的词法作用域都只由函数被声明时所处的位置决定。 <br>
词法作用域查找只会查找一级标识符，比如a、b、c，如果代码中引用了foo.bar.baz，词法作用域查找只会试图查找foo标识符，找到这个变量后，对象属性访问规则会分别接管对bar和baz属性的访问。 <br>
> 编译的词法分析阶段基本能够知道全部标识符在哪里，以及是如何声明的，从而能够预测在执行过程中如何对它们进行查找。 <br>

###### 欺骗词法作用域（修改词法作用域）——在运行期修改书写期的词法作用域
###### eval
javascript中的eval()函数可以接收一个字符串作为参数，并将其中的内容视为好像在书写时就存在于程序中的这个位置。也就是说，可以在我们写的代码中用程序生成代码并运行，就好像代码是写在那个位置一样。 <br>
```javascript
function(str,a){
  eval(str);       // eval()调用"var b = 3;"，这段代码会被当作就在那里一样来处理
  // 由于eval()调用的代码声明了一个新的变量b，因此它对已经存在的foo()的词法作用域进行了修改（也就是说，这段代码实际上在foo()内部创建了一个变量b，并且遮蔽了外部（全局变量）作用域中的同名变量）
  console.log(a,b);
}
var b = 2;
foo("var b = 3;",1);        // 1,3
```
> 实际上，可以非常容易地根据程序逻辑动态地将字符串拼接在一起后再传进代码里面。eval()通常被用来执行动态创建的代码。 <br>


默认情况下，如果eval()中所执行的代码包含有一个或多个声明（无论是变量还是函数），都会对eval()所处的词法作用域进行修改。 <br>
在严格模式中，eval()在运行时有其自己的词法作用域，也就是说其中的声明无法修改所在的作用域。
> setTimeout()、setInterval()、new Function()也可以像eval()一样，参数可以是字符串，字符串的内容可以被解析为一段动态生成的函数代码，但是这种在程序中动态生成代码的使用场景，会非常消耗性能。 <br>


###### with
with通常被当作重复引用同一个对象中的多个属性的快捷方式，可以不需要重复引用对象本身。 <br>
with会将对象及其属性放进一个作用域并同时分配标识符。
```javascript
function foo(obj){
  // with可以将一个没有或有多个属性的对象处理为一个完全隔离的词法作用域，因此这个对象的属性也会被处理为定义在这个作用域中的词法标识符
  with(obj){
    a = 2;   // LHS引用，将2赋值给a
  }
}
var o1 = {
  a:3;
};
var o2 = {
  b:3;
};
foo(o1);         // 将o1传递给foo()时，a=2赋值操作找到了o1.a并将2赋值给它
console.log(o1.a);     // 2
foo(o2);        // 将o2传递给foo()时，o2并没有a属性，因此不会创建这个属性
console.log(o2.a);     // undefined
console.log(a);        // 2——a被泄漏到全局作用域上了
```
> eval()函数如果接受了含有一个或多个声明的代码，就会修改其所处的词法作用域。但是with声明实际上是根据我们传递给它的对象凭空创建了一个全新的词法作用域。 <br>

###### 性能
javascript引擎会在编译阶段进行数项性能优化，其中有些优化依赖于能够根据代码的词法进行静态分析，并预先确定所有变量和函数的定义位置，才能在执行过程中快速找到标识符。 <br>
但是如果引擎在代码中发现了eval()/with，它只能简单地假设关于标识符位置的判断都是无效的，因为无法再词法分析阶段明确指定eval()会接收到什么代码，这些代码如何对作用域进行修改，也无法知道传递给with用来创建新词法作用域的对象内容到底是什么。 <br>
如果出现了eval()/with，很可能所有的优化都是无意义的。



#### 函数作用域
函数作用域是指：属于这个函数的全部变量都可以在整个函数的范围内使用及复用（事实上，在嵌套的作用域中也可以使用）。
###### 定义一个函数
```javascript
// 函数声明——常用来定义全局作用域下的函数（全局函数）
function maxinum(a,b){
 if(a>b) return a;
 else return b;
}
maxinum(5,6);   // 6

// 函数表达式——常用来定义一个作为对象方法的函数
var obj = new Object();
obj.maxinum = function(a,b){
 if(a>b) return a;
 else return b;
};
obj.maxinum(5,6);   // 6

// Function构造函数——没有很好的可读性，只在特定情况下使用
var maxinum = new Function("a","b","if(a>b) return a;else return b;");
maxinum(5,6);     // 6
```
###### 隐藏内部实现
从所写的代码中挑选一个任意的片段，然后用函数声明来对其进行包装，实际上就是把这些代码给“隐藏”起来。 <br>
实际上就是在这个代码片段的周围创建一个作用域气泡，也就是说这段代码中的任何声明（变量或函数）都将绑定在这个新创建的包装函数的作用域中，而不是先前所在的作用域中。所以说，可以把变量和函数包裹在一个函数的作用域中，然后用这个作用域来“隐藏”它们。 <br>
> 最小授权或最小暴露原则——在软件设计中，应该最小限度地暴露必要的内容，将其他内容都“隐藏”起来，比如某个模块或对象的API设计 <br>


```javascript
function dosomething(a){
 b = a + dosomethingElse(a*2);
 console.log(b*3);
}
function dosomethingElse(a){
 return a - 1;
}
var b;
dosomething(2);          // 15
// 上面的代码给予外部作用域对b和doSomethingElse()的访问权限不仅没有必要，而且还有可能被有意或无意地以非预期的方式使用，从而超出了doSomething()的适用条件
// 改进
function dosomething(a){
 function dosomethingElse(a){
  return a - 1;
 }
 var b;
 b = a + dosomethingElse(a*2);
 console.log(b*3);
}
dosomething(2);          //15
```
###### 规避冲突
“隐藏”作用域中的变量和函数可以避免同名标识符之间的冲突，两个同名标识符可能用途不一样，无意间会造成冲突，并且可能发生变量的值被覆盖。
```javascript
function foo(){
 function bar(a){
  i = 3;    // 修改了for循环所属作用域中的i
  console.log(a+i);
 }
 for(var i = 0;i<10;i++){
  bar(i*2);   // 出现无限循环
 }
}
foo();
```
解决方法：
> 全局命名空间——在全局作用域中声明一个名字足够特别的变量，通常是一个对象。这个对象被用作库的命名空间，所有需要暴露给外界的功能都会变成这个对象（命名空间）的属性，而不是将自己的标识符暴露在顶级的词法作用域中。 <br>
> 模块管理——从众多模块管理器中挑选一个来使用，使用这些工具，任何库都无需将标识符加入到全局作用域中，而是通过依赖管理器的机制将库的标识符显式地导入到另一个特定的作用域中。 <br>
模块管理器的原理是：利用作用域的规则强制所有的标识符都不能注入到共享作用域中，而是保持在私有、无冲突的作用域中，这样就可以有效地规避所有冲突。 <br>


###### 函数作用域
在任意代码片段外部添加包装函数，可以将内部的变量和函数定义“隐藏”起来，外部作用域无法访问包装函数内部的任何东西。
```javascript
var a = 2;
function foo(){
 var a = 3;
 console.log(a);   // 3
}
foo();
console.log(a);    // 2
```
但是这样会增加额外的问题：首先，必须要声明一个具名函数foo()，这样foo()本身就“污染”了所在作用域；其次，必须显示地通过函数名(foo())调用这个函数才能运行其中的代码。 <br>
改进：
```javascript
var a = 2;
(function foo(){
 var a = 3;
 cosole.log(a);   // 3
})();
console.log(a);   // 2
```
(function foo(){...})()会被看成函数表达式而不是一个标准的函数声明来处理。 <br>
> 第一个片段的foo被绑定在所在作用域中，可以直接通过foo()来调用；第二个片段中foo被绑定在函数表达式自身的函数中而不是所在作用域中。 <br>
> 也就是说，(function foo(){...})()作为函数表达式意味着foo只能在...所代表的位置中能被访问，外部作用域则不行。
###### foo变量名被隐藏在自身的函数中而不是在所在作用域中。

###### 匿名函数
匿名函数表达式，比较典型的就是回调函数：
```javascript
setTimeout(function(){
 console.log("xxxx");
},1000);
```
匿名函数表达式书写起来简洁快捷，但是存在几个缺点：
* 匿名函数在栈追踪中不会显示出有意义的函数名，使得调试很困难
* 如果没有函数名，当函数需要引用自身时只能用arguments.callee引用（已过期）
* 匿名函数省略了对于代码可读性/可理解性很重要的函数名 <br>
使用行内函数表达式能够解决这个问题,始终给函数表达式命名是一个最佳实践：
```javascript
setTimeout(function timeoutHandler(){
 console.log("XXX");
},1000);
```
###### 立即执行函数表达式
IIFE：立即执行函数表达式：
```javascript
var a = 2;
(function foo(){
 var a = 3;
 cosole.log(a);   // 3
})();
console.log(a);   // 2
```
函数被包含在一对()括号内部，因此成为了一个表达式，通过在末尾添加另一个()可以立即执行这个函数。 <br>
IFEE有一个非常普遍的进阶用法——把它们当作函数调用并传递参数进去：
```javascript
var a = 2;
(function IFEE(global){   // 可以从外部作用域传递任何我们需要的东西
 var a = 3; 
 console.log(a);   // 3
 console.log(global.a);  // 2
})(window);
console.log(a);  // 2
```
IFEE还有一个用途就是倒置代码的运行顺序：将需要运行的函数放在第二位，在IFEE执行之后当作参数传递进去：
```javascript
var a = 2;
// 函数表达式def定义在片段的第二部分，然后当作参数（这个参数也叫def）被传递进IFEE函数定义的第一部分
// 最后参数def（也就是传递进去的函数）被调用，并将window传入当作global参数的值
(function IFEE(def){
 def(window);
})(function def(global){
 var a = 3;
 console.log(a);    // 3
 console.log(global.a);  // 2
});
```

#### 块级作用域
> 变量的声明应该距离使用的地方越近越好，并最大限度地本地化。 <br>

块作用域是指变量和函数不仅可以属于所处的作用域，也可以属于某个代码块（通常是指{...}内部）。 <br>
块级作用域是一个队最小授权原则进行扩展的工具，将代码从在函数中隐藏信息扩展为在块中隐藏信息。 <br>
```javascript
for(var i = 0; i<10;i++){        // 在for循环的头部直接定义变量i是为了i只在for循环内部的上下文中使用，但是由于i是var声明的，所以i会被绑定在外部作用域（函数或全局）中
 console.log(i);
}
```
更多关于块级作用域的解说，可以点击 [这里](https://github.com/HecateDK/LearningNotes/blob/master/ECMAScript6.md#let%E5%92%8Cconst)


#### 提升
包括变量和函数在内的所有声明都会在任何代码被执行前首先被处理——变量和函数声明从它们的代码中出现的位置被“移动”到了最上面，这过程叫做“提升”。
```javascript
a = 2;
var a;
console.log(a);  // 2
//之所以会输出2，是因为var a = 2;在js中被认为是两个声明：var a;（在编译阶段进行）和a= 2;（赋值声明会被留在原地等待执行），所以上面代码片段在js中进行的形式如下：
var a;
a = 2;
console.log(a);    // 2


console.log(a);
var a = 2;       // undefined
// 上面的代码片段在js中的进行形式如下：
var a;
console.log(a);   // undefined
a = 2;


foo();
function foo(){
 console.log(a);   // undefined
 var a = 2;
}
// 上面的代码片段在js中的进行形式如下：
function foo(){
 var a;
 console.log(a);      // undefined
 a = 2;
}
foo();       // 可以看出函数声明会被提升，但是函数表达式却不会被提升


foo();    // 不是ReferenceError，而是TypeError
var foo = function bar(){
 // ...
}   // 因为变量标识符foo会被提升并分配给所在作用域，因此foo()不会抛出ReferenceError。但是foo此时并没有赋值（如果它是函数声明而不是函数表达式，就会被赋值），foo由于对undefined的值进行了操作，所以会抛出TypeError


foo();
bar();
var foo = function bar(){
 // ....
}
// 上面的代码片段在js中的进行形式如下：
var foo;
foo();   // TypeError
bar();   // ReferenceError
foo = function bar(){
 // ...
}
```

###### 函数优先
函数声明和变量都会被提升，但需要注意的是，函数会首先被提升，然后才是变量。
```javascript
foo();   // 1
var foo;
function foo(){
 console.log(1);
}
foo = function(){
 console.log(2);
}

foo();        // 3（重复的var声明会被忽略，但是出现在后面的函数声明还是可以覆盖前面的）
function foo(){
 console.log(1);
}
var foo = function(){
 console.log(2);
}
function foo(){
 console.log(3);
}
```


















