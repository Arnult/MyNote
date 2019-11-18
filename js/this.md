# 1. this全面解析
## 1.1. 调用位置

  * **调用栈**：为达到当前执行位置所调用的所用函数。
   
   * **调用位置**：在当前正在执行的函数的**前一个**调用函数中。

```javascript
function bar(){
   //当前作用栈baz->bar
   //作用位置在bar中
 console.log('bar');
}
function baz(){
   //当前调用栈是：baz
   //当前调用位置是全局作用域
 console.log('baz');
 bar();
}
 baz();
```

## 1.2. 绑定规则

### 1.2.1. 默认绑定

 **非严格模式**：**this**绑定到全局作用域。

```javascript
function foo(){
    console.log(this.a);
}
var a = 2;
    foo(); // 2
```

**严格模式（strict mode）**：**this**的绑定规则完全取决于调用位置，但是只有运行在**非strict mode**下时，默认绑定才能绑定到全局对象。
```javacript
function foo(){
  "use strict";
  console.log(this.a);
}
var a = 2;
    foo(); // TypeError:this is undefined
```

使用**严格模式（strict mode）**，全局对象将无法使用默认绑定，**this**会绑定到**undefined**。

### 1.2.2. 隐式绑定

调用位置在上下文对象中，或是说被某个对象拥有或包含。
  
```javascript
function foo(){
  console.log(this.a);
}
var obj = {
  a: 2,
  foo: foo
};
obj.foo(); // 2
```
**隐式丢失**：被隐式绑定的函数会丢失绑定对象，会应用**默认绑定**，使**this**绑定到全局对象或者是**undefined**上。

```javascript
function foo(){
  console.log(this.a);
}
var obj = {
  a:2,
  foo:foo
};
var bar = obj.foo; // 函数别名
var a = "global"; // 全局对象的属性
    bar(); // global
```
**bar**是**obj.foo**的一个引用，但是实际上，它引用的是**foo**函数本身，因此此时的 **bar()** 其实是一个不带任何修饰的函数调用，使用了默认绑定。

因为**obj.foo**赋值后的对象直接引用**foo**，所以**obj.foo**作为参数传入**回调函数**也将**隐式丢失**。

### 1.2.3. 显式绑定

显示绑定，想在某个对象上强制调用函数，使用到 **call( . . )** 和 **apply( . . )** 方法。

这两个方法的第一个参数是一个对象，它们会把这个对象绑定到**this**，接着在调用函数时指定这个**this**。

``` javascript
function foo(){
  console.log(this.a);
}
var obj = {
   a:2
};
foo.call(obj); // 2
```
1. **硬绑定**

```javascript
function foo(){
  console.log(this.a)l
}
var obj = {
    a:2
};
var bar = function(){
   foo.call(obj);
}
bar(); // 2
```
创建了函数 **bar()** ，并在它的内部手动调用 **foo.call(obj)** ，强制把**foo**的**this**绑定到了**obj**。

由于**硬绑定**是一种非常常用的模式，**ES5**提供了内置方法**Function.prototype.bind** 

```javascript
function foo(something){
  console.log(this.a,something);
  return this.a+something;
}
var obj = {
  a:2
};
var bar = foo.bind(obj);
var b = bar(3);
console.log(b); // 5
```

2. **API调用的“上下文”**

许多第三方函数、内置函数都会提供一个可选的参数，通常成为“上下文”（context），确保你的回调函数使用指定的**this**。

```javascript
function foo(el){
  console.log(el,this.id);
}
var obj = {
  id:"test"
}
[1,2,3].foreach(foo,obj);
// 1 test 2 test 3 test
```

### 1.2.4. new绑定

**JavaScript**中**new**的机制实际上和面向类的语言完全不同，在**JavaScript**中，构造函数只是一些使用**new**操作符是被调用的函数，这些函数不属于某一个类，也不会实例化一个类，只是普通函数。

使用**new**来调用函数：

1. 创建-个全新的对象。
2. 这个新对象会被执行**原型**连接。
3. 这个新对象会绑定到函数调用的**this**。
4. 如果函数没有返回其他对象，那么**new**表达式中的函数调用会自动返回这个新对象

```javascript 
function foo(a){
  this.a = a;
}
var bar = new foo(2); //bar为新对象
console.log(bar.a); //2
```
使用**new**来调用 **foo()** 时，创建一个新对象并把它绑定到 **foo( . . )**调用中的**this**上。

## 1.3. 优先级

**new**绑定>显式绑定>隐式绑定>默认绑定

**new**绑定会使用新创建的 **this** 替换硬绑定的 **this**。

```javascript
function foo(something){
   this.a = something;
}
var obj1 = {};
var bar = foo.bind(obj1);
    bar(2);
    console.log(obj1.a); // 2
var baz = new bar(3);
    console.log(obj1.a); // 2 硬绑定到obj1的this
    console.log(baz.a); // 3 绑定到baz的this
```
**new** 中使用了硬绑定函数，使得 **new** 进行初始化时就可传入其余的参数。

**bind( . . )** 的功能之一就是可以把除了第一个参数(第一个参数用于绑定 this )之外的其他参数都传给下层的函数。（部分应用，”柯里化“的一种）。

```javascript
function foo(p1,p2) {
  this.val = p1 + p2;
}
// 之所以使用 null 是因为在本例中我们并不关心硬绑定的 this 是什么
// 反正使用 new 时 this 会被修改
var bar = foo.bind( null, "p1" );//把p1参数也传给了bar
var baz = new bar( "p2" );
  baz.val; // p1p2
```
## 1.4. 绑定例外

### 1.4.1. 忽略this

如果把 **null**  或 **undefine** 作为 **this** 的绑定对象传入**call** 、**apply** 或者**bind**，这些值在调用时会被忽略，会应用默认绑定规则。

传入**null** 可"展开"一个数组，或应用在**柯里化**

```javascript
function foo(a,b) {
console.log( "a:" + a + ", b:" + b );
}
// 把数组“展开”成参数
foo.apply( null, [2, 3] ); // a:2, b:3
// 使用 bind(..) 进行柯里化
var bar = foo.bind( null,2);
bar(3); // a:2, b:3
```

若传入**this**或**undefine**某个应用了**this**的函数，导致该函数的**this**绑定到**全局**或是**undefine**会出现严重后果。

**更安全的this**

利用**Object.create(null)**可创造出一个空对象”**DMZ**“。

### 1.4.2. 间接引用

间接引用下，调用这个函数将会应用默认绑定规则。

### 1.4.3. this词法

箭头函数，箭头函数不使用**this**的四种规则，而是根据外层作用域来决定**this**。

```javascript
function foo() {
// 返回一个箭头函数
return (a) => {
//this 继承自 foo()
console.log( this.a );
};
}
var obj1 = {
a:2
};
var obj2 = {
a:3};
var bar = foo.call( obj1 );
bar.call( obj2 ); // 2, 不是 3 !
```


