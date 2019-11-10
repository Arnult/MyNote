# 1. object

## 1.1. 语法

对象有两种形式定义：

* 文字语法

```javascript
var myObj = {
  key : value
};
```
* 构造形式

```javascript
var myObj = new Object();
myObj.key= value;
```
## 1.2. 类型

在**javascript**中一共有六种主要类型：

* **string**
* **number**
* **boolean**
* **null**
* **undefine**
* **object**

**内置对象**

* **String**
* **Number**
* **Boolean**
* **Object**
* **Function**
* **Array**
* **Date**
* **RegExp**
* **Error**

在**javascript**中，它们实际上是一些内置函数。这些内置函数可以当作**构造函数**使用。但是能使用文字形式时就不要使用构造形式。

## 1.3. 内容 

属性访问的语法：

```javascript
var myObject = {
   a:2
};
myObject.a; //2
myObject["a"]; //2
```
两种语法主要区别在于 **.** 操作符要求属性名满足标识符的命名规范，而 **[" . . "]** 语法可以接受任意**UTF-8/Unicode**字符串为属性名。

### 1.3.1. 可计算属性名

通过表达式来计算属性名，**ES6**增加了可计算属性名，可在文字形式中使用[ ]包裹一个表达式来当做属性名：

```javascript
var prefix = "foo";

var myObject = {
      [prefix + "a"] : "hello",
      [prefix + "b"]: "world"
};
myObject["fooa"];  //hello
myObject["foob"];  //world
```
### 1.3.2. 复制对象

* **浅复制**

 复制出的新对象中的值会复制旧对象中的**值**，但是引用和旧对象中**引用**的对象是一样的。
* **深复制**

完整的复制对象。

### 1.3.3. 属性描述符

```javascript
var myObject = {
a:2
};
Object.getOwnPropertyDescriptor( myObject, "a" );
// {
//
value: 2,
//
writable: true,
//
enumerable: true,
//
configurable: true
// }
```
普通的对象处理包含数据值，还包含三个特性：**writable**（可写）、**enumrable**（可枚举）和 **configurable**（可配置）。

**writable** 决定是否可以修改属性的值、**configurable** 若为**true**，就可使用 **defineProperty(. .)** 方法来修改属性描述符、**enumerable** 这个描述符控制的是属性是否会出现在对象的属性枚举中,比如说 **for..in** 循环，可枚举”就相当于“可以出现在对象属性的遍历中”。

可以使用**Object.defineProperty(. .)** 来添加一个新属性或者修改一个已有属性：

```javascript
var myObject = {};
Object.defineProperty( myObject, "a", {
value: 2,
writable: true,
configurable: true,
enumerable: true
} );
myObject.a; // 2
```

### 1.3.4. 不变性

1. **对象常量**：通过**writable:false** 和 **configurable:false** 就可以创建一个真正的常量属性。
2. **禁止扩展**：禁止一个对象添加新属性并且保留已有属性, 可以使用 **Object.preventExtensions(..)** 。
3. **密封**：通过 **Object.seal(..)** 可创建一个“密封”的对象，密封之后不仅不能添加新属性,也不能重新配置或者删除任何现有属性，但是可修改属性的值。
4. **冻结**： **Object.freeze(..)**   会创建一个冻结对象, 这个方法实际上会在一个现有对 象上调用 **Object.seal(..)**  并把所有“数据访问”属性标记为 **writable:false** ,这样就无法修改它们的值。

### 1.3.5. Getter和Setter

当你给一个属性定义  **getter** 、**setter** 或者两者都有时,这个属性会被定义为“访问描述符”(和“数据描述符”相对)。对于访问描述符来说,**JavaScript** 会忽略它们的 **value** 和**writable** 特性,取而代之的是关心 **set** 和 **get** (还有 **configurable** 和 **enumerable** )特性。

```javascript
var myObject = {
// 给 a 定义一个 getter
get a() {
return this._a_;
},
// 给 a 定义一个 setter
set a(val) {
this._a_ = val * 2;
}
};
myObject.a = 2;
myObject.a; // 4
```

### 1.3.6. 存在性

当属性访问返回值是**undefined** 时，这个值可能是属性存储着**undefined** ，也可能是该属性不存在。

在不访问属性值的情况下，可以判断对象中是否存在该属性：

```javascript
var myObject = {
a:2
};
("a" in myObject); // true
("b" in myObject); // false
myObject.hasOwnProperty( "a" ); // true
myObject.hasOwnProperty( "b" ); // false
```

**propertyIsEnumerable(..)** 会检查给定的属性名是否直接存在于对象中(而不是在原型链上)并且满足 **enumerable:true** 。

**Object.keys(..)** 会返回一个数组,包含所有可枚举属性,  **Object.getOwnPropertyNames(..)** 会返回一个数组,包含所有属性,无论它们是否可枚举。

```javascript
var myObject = { };
Object.defineProperty(
myObject,
"a",
// 让 a 像普通属性一样可以枚举
{ enumerable: true, value: 2 }
);
Object.defineProperty(
"b",
// 让 b 不可枚举
{ enumerable: false, value: 3 }
);
myObject.propertyIsEnumerable( "a" ); // true
myObject.propertyIsEnumerable( "b" ); // false
Object.keys( myObject ); // ["a"]
Object.getOwnPropertyNames( myObject ); // ["a", "b"]
```

## 1.4. 遍历

**for..in** 遍历的是对象中所有可枚举的属性，需要手动获取属性值。

**for..of** 能遍历数组的值。

```javascript
var myArray = [ 1, 2, 3 ];
for (var v of myArray) {
console.log( v );
}
// 1
// 2
// 3
```

原因是数组对象有内置的迭代器对象@@iterator。

```javascript
var myArray = [ 1, 2, 3 ];
var it = myArray[Symbol.iterator]();
it.next();//{value:1, done:false }
it.next();//{value:2, done:false }
it.next();//{value:3, done:false }
it.next();//{done:true }
```













