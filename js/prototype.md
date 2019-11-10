# 1. prototype

## 1.1. [[Prototype]]

**JavaScript** 中的对象有一个特殊的 **[[Prototype]]** 内置属性,其实就是对于**其他对象的引用**。

当引用对象的属性时会触发 **[[Get]]** 操作，对于默认的 **[[Get]]** 操作，第一步是检查对象本身是否有该属性，若有即使用，若不在就需要使用对象的 **[[prototype]]** 链。

### 1.1.1. object.prototype

所有普通的  **[[Prototype]]** 链最终都会指向内置的 **Object.prototype** 。

### 1.1.2. 属性设置和屏蔽

```javascript
myObject.foo="bar";
```
* 如果**myObject**对象中包含foo，则为修改已有属性值。
* 如果**foo**即出现在**myObject**中也出现在**myObject**的 **[[prototype]]** 链上层，会发生屏蔽。**myObject**中的**foo**会屏蔽 **[[prototype]]** 上层的**foo**，因为**myObject.foo**总是会选择原型链中**最底层**的**foo**属性。
   1. 如果 **[[prototype]]** 链上层存在**foo**且没标记为只读 **( writable:true )**，会直接在**myObject**中添加一个命名为**foo**的新属性，为**屏蔽属性**。
   2. 如果 **[[prototype]]** 链上层存在**foo**且标记为只读 **( writable:false )** ，那么无法修改已有属性或是创建**屏蔽属性** 。

## 1.2. “类”

**JavaScript** 和面向类的语言不同,它并没有类来作为对象的抽象模式或者说蓝图。**JavaScript** 中只有对象。

### 1.2.1. “类” 函数

所有的函数默认都会拥有一个名为**prototype**的公有并且不可枚举的属性。

```javascript
function Foo() {
// ...
}
var a = new Foo();
Object.getPrototypeOf( a ) === Foo.prototype; // true
```

**new Foo()** 会生成一个新对象(我们称之为 **a** ),这个新对象的内部链接 **[[Prototype]]** 关联的是 **Foo.prototype** 对象。

我们并没有初始化一个类,实际上我们并没有从“类”中复制任何行为到一个对象中,只是让两个对象互相关联。

**new Foo()** 这个函数调用实际上并没有直接创建关联,这个关联只是一个意外的副作用。

### 1.2.2. “构造函数”

函数本身并不是构造函数,然而,当你在普通的函数调用前面加上 **new** 关键字之后,就会把这个函数调用变成一个“构造函数调用”。实际上, **new** 会劫持所有普通函数并**用构造对象的形式来调用**它。

```javascript
function NothingSpecial() {
console.log( "Don't mind me!" );
}
var a = new NothingSpecial();
// "Don't mind me!"
a; // {}
```
换句话说,在 **JavaScript** 中对于“构造函数”最准确的解释是,所有带 **new** 的函数调用。函数不是构造函数,但是当且仅当使用 **new** 时,函数调用会变成“构造函数调用”。

## 1.3. （原型）继承

```javascript
Bar.prototype = Object.create( Foo.prototype );
```

调用**Object.create(..)** 会凭空创建一个“新”对象并把新对象内部的 **[[Prototype]]** 关联到你指定的对象。

```javascript
// 基本上满足你的需求,但是可能会产生一些副作用 :(
Bar.prototype = new Foo();
```

**Bar.prototype = new Foo()** 的确会创建一个关联到 **Bar.prototype** 的新对象。但是它使用了 **Foo(..)** 的**“构造函数调用”**, 会运行一遍**Foo()** ，可能造成不必要的麻烦。

使用**Object.create(..)**的缺点是需要创建一个新对象然后把旧对象抛弃掉，不能直接修改已有的对象。**ES6** 添加了辅助函数 **Object.setPrototypeOf(..)** 可以用来修改关联。

```javascript
// ES6 之前需要抛弃默认的 Bar.prototype
Bar.ptototype = Object.create( Foo.prototype );
// ES6 开始可以直接修改现有的 Bar.prototype
Object.setPrototypeOf( Bar.prototype, Foo.prototype );
```
**检查“类”关系**

检查一个实例(**JavaScript** 中的对象)的继承祖先(**JavaScript** 中的委托关联)通常被称为**内省**(或者**反射**)。

```javascript
function Foo() {
// ...
}
Foo.prototype.blah = ...;
var a = new Foo();
```

判断对象( a )和函数(带 .prototype 引用的 Foo )之间的关系。

```javascript
a instanceof Foo; // true
```

使用内置的 **.bind(..)** 函数来生成的硬绑定函数并无 **.prototype** 。

还可以用 **isPrototypeOf()** :

```javascript
Foo.prototype.isPrototypeOf( a ); // true
```

## 1.4. 对象关联

```javascript
var foo = {
something: function() {
console.log( "Tell me something good..." );
}
};
var bar = Object.create( foo );
bar.something(); // Tell me something good...
```

**Object.create(..)** 会创建一个新对象 **( bar )** 并把它关联到我们指定的对象 **( foo )** 。
