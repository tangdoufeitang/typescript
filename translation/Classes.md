 # Classes

 TypeScript提供了对ES2015中引入的class关键字的全面支持。

 与其他JavaScript语言特性一样，TypeScript添加了类型注释和其他语法，允许你表达类和其他类型之间的关系。

 ### Class Members

 这是最基本的类——空类
 
 ```ts
class Point {}
 ```

 这个类还不是很有用，所以让我们开始添加一些成员。

 #### Fields

 字段声明在类上创建一个公共可写属性

 ```ts
 class Point {
  x: number;
  y: number;
}

const pt = new Point();
pt.x = 0;
pt.y = 0;
```

与其他位置一样，类型注释是可选的，但如果没有指定，则是隐式的any。

字段也可以有初始化器;这些将在类被实例化时自动运行

```ts
class Point {
  x = 0;
  y = 0;
}

const pt = new Point();
// Prints 0, 0
console.log(`${pt.x}, ${pt.y}`);
```

与const、let和var一样，类属性的初始化式将用于推断其类型

```ts
const pt = new Point();
pt.x = "0";
// Type 'string' is not assignable to type 'number'.
```

#### --strictPropertyInitialization

strictPropertyInitialization设置控制是否需要在构造函数中初始化类字段。  

```ts
class BadGreeter {
  name: string;
// Property 'name' has no initializer and is not definitely assigned in the constructor.
}
```

```ts
Try
class GoodGreeter {
  name: string;

  constructor() {
    this.name = "hello";
  }
}
```

注意，字段需要在构造函数本身中初始化。TypeScript不会分析你从构造函数中调用的方法来检测初始化，因为派生类可能会重写这些方法而无法初始化成员。

如果您想通过构造函数以外的方法来明确初始化字段(例如，可能外部库正在为您填充类的一部分)，那么可以使用确定赋值断言操作符，

```ts
class OKGreeter {
  // Not initialized, but no error
  name!: string;
}
```

#### readonly

字段可以用readonly修饰符作为前缀。这将阻止在构造函数之外对字段进行赋值。

```ts
class Greeter {
  readonly name: string = "world";

  constructor(otherName?: string) {
    if (otherName !== undefined) {
      this.name = otherName;
    }
  }

  err() {
    this.name = "not ok";
// Cannot assign to 'name' because it is a read-only property.
  }
}
const g = new Greeter();
g.name = "also not ok";
// Cannot assign to 'name' because it is a read-only property.
```

#### Constructors

类构造函数与函数非常相似。您可以添加带有类型注释、默认值和重载的参数

```ts
class Point {
  x: number;
  y: number;

  // Normal signature with defaults
  constructor(x = 0, y = 0) {
    this.x = x;
    this.y = y;
  }
}
```

```ts
class Point {
  // Overloads
  constructor(x: number, y: string);
  constructor(s: string);
  constructor(xs: any, y?: any) {
    // TBD
  }
}
```

类构造函数签名和函数签名之间只有一些区别

- 构造函数不能有类型参数——它们属于外部类声明，我们将在后面学习
- 构造函数不能有返回类型注释——类实例类型总是返回的类型

#### Super Calls

就像在JavaScript中，如果你有一个基类，你将需要调用super();在你的构造函数主体中使用任何`this.`成员

```ts
class Base {
  k = 4;
}

class Derived extends Base {
  constructor() {
    // Prints a wrong value in ES5; throws exception in ES6
    console.log(this.k);
// 'super' must be called before accessing 'this' in the constructor of a derived class.
    super();
  }
}
```

在JavaScript中，忘记调用super是一个很容易犯的错误，但是TypeScript会告诉你什么时候需要调用super。

#### Methods

类上的函数属性称为方法。方法可以像函数和构造函数一样使用所有相同的类型注释

```ts
class Point {
  x = 10;
  y = 10;

  scale(n: number): void {
    this.x *= n;
    this.y *= n;
  }
}
```

除了标准的类型注释，TypeScript不会向方法中添加任何新内容。

注意，在方法体中，仍然必须通过这个访问字段和其他方法。方法主体中的非限定名称总是引用封闭作用域中的内容

```ts
let x: number = 0;

class C {
  x: string = "hello";

  m() {
    // This is trying to modify 'x' from line 1, not the class property
    x = "world";
// Type 'string' is not assignable to type 'number'.
  }
}
```

#### Getters / Setters

类也可以有访问器

```ts
 class C {
  _length = 0;
  get length() {
    return this._length;
  }
  set length(value) {
    this._length = value;
  }
}
```

> 请注意，在JavaScript中，具有额外逻辑的字段支持/设置对非常有用。如果您不需要在Get / Set操作期间添加其他逻辑，则曝光公共字段很好。

TypeScript对访问器有一些特殊的推理规则

- 如果get存在但未set，则该属性自动为readonly
- 如果没有指定setter参数的类型，则从getter的返回类型推断它
- getter和setter必须具有相同的成员可见性

从TypeScript 4.3开始，就可以使用不同类型的访问器来获取和设置。

```ts
class Thing {
    _size = 0;

    get size(): number {
        return this._size;
    }

    set size(value: string | number | boolean) {
        let num = Number(value);

        // Don't allow NaN, Infinity, etc

        if (!Number.isFinite(num)) {
            this._size = 0;
            return;
        }

        this._size = num;
    }
}
```

#### Index Signatures

类可以声明索引签名;它们的工作原理与其他对象类型的索引签名相同

```ts
class MyClass {
  [s: string]: boolean | ((s: string) => boolean);

  check(s: string) {
    return this[s] as boolean;
  }
}
```

因为索引签名类型还需要捕获方法的类型，所以要有效地使用这些类型并不容易。一般来说，最好将索引数据存储在其他地方，而不是类实例本身。

### Class Heritage

像其他具有面向对象特性的语言一样，JavaScript中的类可以从基类继承。

#### implements Clauses

您可以使用implements子句来检查类是否满足特定的接口。如果一个类不能正确地实现它，就会发出一个错误

```ts
interface Pingable {
  ping(): void;
}

class Sonar implements Pingable {
  ping() {
    console.log("ping!");
  }
}

class Ball implements Pingable {
Class 'Ball' incorrectly implements interface 'Pingable'.
//   Property 'ping' is missing in type 'Ball' but required in type 'Pingable'. 类'Ball'错误地实现了接口'Pingable'。属性'ping'在类型'Ball'中缺失，但在类型'Pingable'中需要。
  pong() {
    console.log("pong!");
  }
}
```

类也可以实现多个接口，例如`class C implements A, B {`.

#### Cautions

重要的是要理解，implements子句只是检查类是否可以被视为接口类型。它根本不改变类的类型或它的方法。一个常见的错误来源是假定一个implements子句将改变类类型-它不会

```ts
interface Checkable {
  check(name: string): boolean;
}

class NameChecker implements Checkable {
  check(s) {
// Parameter 's' implicitly has an 'any' type.
    // Notice no error here
    return s.toLowercse() === "ok";
                 
// any
  }
}
```

在本例中，我们可能期望s类型会受到check的`name: string`参数的影响。它不是——实现子句不会改变检查类体或推断类类型的方式。

类似地，实现带有可选属性的接口不会创建该属性

```ts
interface A {
  x: number;
  y?: number;
}
class C implements A {
  x = 0;
}
const c = new C();
c.y = 10;
// Property 'y' does not exist on type 'C'.
```

#### extends Clauses

类可以从基类扩展。派生类具有基类的所有属性和方法，还定义了其他成员。

```ts
class Animal {
  move() {
    console.log("Moving along!");
  }
}

class Dog extends Animal {
  woof(times: number) {
    for (let i = 0; i < times; i++) {
      console.log("woof!");
    }
  }
}

const d = new Dog();
// Base class method
d.move();
// Derived class method
d.woof(3);
```

#### Overriding Methods

派生类也可以重写基类字段或属性。你可以用super。访问基类方法的语法。注意，因为JavaScript类是一个简单的查找对象，所以没有超级字段的概念。

TypeScript强制派生类始终是其基类的子类型。

例如，这里有一个覆盖方法的合法方法

```ts
class Base {
  greet() {
    console.log("Hello, world!");
  }
}

class Derived extends Base {
  greet(name?: string) {
    if (name === undefined) {
      super.greet();
    } else {
      console.log(`Hello, ${name.toUpperCase()}`);
    }
  }
}

const d = new Derived();
d.greet();
d.greet("reader");
```

派生类遵循它的基类契约是很重要的。请记住，通过基类引用来引用派生类实例是非常常见的(而且总是合法的!

```ts
// Alias the derived instance through a base class reference
const b: Base = d;
// No problem
b.greet();
```

如果派生没有遵循基地的合同

```ts
class Base {
  greet() {
    console.log("Hello, world!");
  }
}

class Derived extends Base {
  // Make this parameter required
  greet(name: string) {
// Property 'greet' in type 'Derived' is not assignable to the same property in base type 'Base'.
//   Type '(name: string) => void' is not assignable to type '() => void'. 类型“派生”中的属性“greet”不能赋值给基类型“base”中的相同属性。Type '(name: string) => void'不能分配给Type '() => void'。
    console.log(`Hello, ${name.toUpperCase()}`);
  }
}
```

如果我们不顾错误编译这段代码，那么这个示例就会崩溃

```ts
const b: Base = new Derived();
// Crashes because "name" will be undefined
b.greet();
```

#### Initialization Order

在某些情况下，JavaScript类初始化的顺序可能令人惊讶。让我们考虑这段代码

```ts
class Base {
  name = "base";
  constructor() {
    console.log("My name is " + this.name);
  }
}

class Derived extends Base {
  name = "derived";
}

// Prints "base", not "derived"
const d = new Derived();
```

这里发生了什么

JavaScript定义的类初始化顺序是
- Base类字段被初始化
- Base类构造函数运行
- 初始化Derived类字段
- Derived类构造函数运行

这意味着基类的构造函数在自己的构造函数中看到了自己的name值，因为派生类字段的初始化还没有运行。

#### Inheriting Built-in Types 继承内置类型

> 注意:如果你不打算继承内置类型，如Array, Error, Map等，你可以跳过这一节

在ES2015中，返回对象的构造函数隐式地将this的值替换为`super(…)`的任何调用者。对于生成的构造函数代码来说，捕获`super(…)`的任何潜在返回值并将其替换为this是必要的。

因此，子类化Error、Array和其他可能不再按预期工作。这是由于Error、Array等类的构造函数使用了ECMAScript 6 s new。调整目标原型链;但是，没有办法确保new的值。当调用ECMAScript 5中的构造函数时。其他底层编译器默认情况下通常有相同的限制。

对于如下所示的子类

```ts
class MsgError extends Error {
  constructor(m: string) {
    super(m);
  }
  sayHello() {
    return "hello " + this.message;
  }
}
```

你可能会发现：

在构造这些子类返回的对象上，方法可能是未定义的，因此调用sayHello将导致错误。

instanceof将在子类的实例和它们的实例之间中断，因此(new MsgError()) instanceof MsgError将返回false。

作为建议，您可以在任何super(…)调用之后立即手动调整原型。

```ts
class MsgError extends Error {
  constructor(m: string) {
    super(m);

    // Set the prototype explicitly.
    Object.setPrototypeOf(this, MsgError.prototype);
  }

  sayHello() {
    return "hello " + this.message;
  }
}
```

但是，MsgError的任何子类也必须手动设置原型。不支持对象的运行时。setPrototypeOf，你可以使用proto。

不幸的是，这些解决方案在Internet Explorer 10及之前的版本上无法工作。你可以手动将方法从原型复制到实例本身(例如MsgError)。原型到这)，但原型链本身不能修复。

### Member Visibility

你可以使用TypeScript来控制某些方法或属性是否对类外的代码可见。

#### public

类成员的默认可见性是public。任何地方都可以访问公共成员
```ts
class Greeter {
  public greet() {
    console.log("hi!");
  }
}
const g = new Greeter();
g.greet();
```
因为public已经是默认的可见性修饰符 你不需要把它写在类成员上 但可能会选择出于风格/可读性原因。

#### protected

protected受保护的成员只对声明它们的类的子类可见

```ts
class Greeter {
  public greet() {
    console.log("Hello, " + this.getName());
  }
  protected getName() {
    return "hi";
  }
}

class SpecialGreeter extends Greeter {
  public howdy() {
    // OK to access protected member here
    console.log("Howdy, " + this.getName());
  }
}
const g = new SpecialGreeter();
g.greet(); // OK
g.getName();
// Property 'getName' is protected and only accessible within class 'Greeter' and its subclasses. 属性'getName'是受保护的，只能在'Greeter'类及其子类中访问。
```

#### Exposure of protected members

派生类需要遵循它们的基类契约，但是可以选择公开具有更多功能的基类的子类型。这包括公开受保护的成员

```ts
class Base {
  protected m = 10;
}
class Derived extends Base {
  // No modifier, so default is 'public'
  m = 15;
}
const d = new Derived();
console.log(d.m); // OK
```

注意，Derived已经能够自由地读和写m，所以这并不能有效地改变这种情况的安全性。这里要注意的主要事情是，在派生类中，如果不是有意暴露，我们需要小心地重复受保护的修饰符。

#### Cross-hierarchy protected access 跨越层级保护访问

不同的OOP语言对于通过基类引用访问受保护成员是否合法存在分歧

```ts
class Base {
  protected x: number = 1;
}
class Derived1 extends Base {
  protected x: number = 5;
}
class Derived2 extends Base {
  f1(other: Derived2) {
    other.x = 10;
  }
  f2(other: Base) {
    other.x = 10;
// Property 'x' is protected and only accessible through an instance of class 'Derived2'. This is an instance of class 'Base'.属性“x”是受保护的，只能通过类“deriv2”的实例访问。这是'Base'类的一个实例。
  }
}
```

例如，Java认为这是合法的。另一方面，c#和c++认为这段代码是非法的。

TypeScript在这里使用c#和c++，因为访问Derived2中的x应该只在Derived2的子类中是合法的，而Derived1不是其中之一。此外，如果通过deriv2引用访问x是非法的(它当然应该是非法的!)，那么通过基类引用访问x永远不会改善这种情况。

请参见为什么我不能从派生类访问受保护的成员?这解释了更多的c#推理。

#### private

Private类似于protected，但不允许从子类访问成员

```ts
class Base {
  private x = 0;
}
const b = new Base();
// Can't access from outside the class
console.log(b.x);
// Property 'x' is private and only accessible within class 'Base'.
```

```ts
class Derived extends Base {
  showX() {
    // Can't access in subclasses
    console.log(this.x);
// Property 'x' is private and only accessible within class 'Base'.
  }
}
```

因为私有成员对派生类不可见，派生类不能增加它的可见性

```ts
class Base {
  private x = 0;
}
class Derived extends Base {
// Class 'Derived' incorrectly extends base class 'Base'.
//   Property 'x' is private in type 'Base' but not in type 'Derived'.
  x = 1;
}
```

#### Cross-instance private access

不同的OOP语言对于同一个类的不同实例是否可以访问彼此的私有成员存在分歧。虽然像Java、c#、c++、Swift和PHP这样的语言允许这样做，但Ruby不允许。

TypeScript确实允许跨实例的私有访问

```ts
class A {
  private x = 10;

  public sameAs(other: A) {
    // No error
    return other.x === this.x;
  }
}
```

#### Caveats

像TypeScript类型系统的其他方面一样，private和protected只在类型检查时强制执行。这意味着JavaScript运行时构造(如in或简单的属性查找)仍然可以访问私有成员或受保护成员

```ts
class MySafe {
  private secretKey = 12345;
}
```

```ts
// In a JavaScript file...
const s = new MySafe();
// Will print 12345
console.log(s.secretKey);
```

如果您需要保护类中的值不受恶意参与者的影响，您应该使用提供硬运行时隐私的机制，如闭包、弱映射或私有字段。

### Static Members

类可以有静态成员。这些成员不与类的特定实例相关联。可以通过类构造函数对象本身访问它们

```ts
class MyClass {
  static x = 0;
  static printX() {
    console.log(MyClass.x);
  }
}
console.log(MyClass.x);
MyClass.printX();
```

静态成员还可以使用相同的`public`, `protected`和私有可见性`private`修饰符

```ts
class MyClass {
  private static x = 0;
}
console.log(MyClass.x);
Property 'x' is private and only accessible within class 'MyClass'.
```

静态成员也是继承的

```ts
class Base {
  static getGreeting() {
    return "Hello world";
  }
}
class Derived extends Base {
  myGreeting = Derived.getGreeting();
}
```

#### Special Static Names

一般来说重写函数原型中的属性是不安全的 因为类本身是可以用new调用的函数，所以不能使用某些静态名称。函数属性如name、length和call不能定义static成员

```ts
class S {
  static name = "S!";
Static property 'name' conflicts with built-in property 'Function.name' of constructor function 'S'.
}
```

#### Why No Static Classes? 

TypeScript(和JavaScript)没有像c#和Java那样的静态类结构

这些结构之所以存在，是因为这些语言强制所有数据和函数都在一个类中;因为TypeScript中不存在这个限制，所以不需要它们。在JavaScript/TypeScript只有一个实例的类通常表示为普通对象

例如，我们在TypeScript中不需要静态类语法，因为常规对象(甚至顶层函数)也能完成这项工作

```ts
// Unnecessary "static" class
class MyStaticClass {
  static doSomething() {}
}

// Preferred (alternative 1)
function doSomething() {}

// Preferred (alternative 2)
const MyHelperObject = {
  dosomething() {},
};
```

### Generic Classes 泛型类

类很像接口，可以是泛型的。当用new实例化泛型类时，其类型参数的推断方式与函数调用中相同

```ts
class Box<Type> {
  contents: Type;
  constructor(value: Type) {
    this.contents = value;
  }
}

const b = new Box("hello!");
     
// const b: Box<string>
```

类可以像接口一样使用泛型约束和默认值。

#### Type Parameters in Static Members

这段代码不合法，原因可能不明显

```ts
class Box<Type> {
  static defaultValue: Type;
// Static members cannot reference class type parameters. 静态成员不能引用类类型参数。
}
```

记住，类型总是完全擦除的!在运行时，只有一个`Box.defaultValue`属性槽。这意味着设置`Box<string>.defaultvalue`(如果可能的话)也会改变`Box<number>.defaultvalue` -不好。泛型类的静态成员永远不能引用类的类型参数。

### `this` at Runtime in Classes

重要的是要记住，TypeScript不会改变JavaScript的运行时行为，JavaScript在某种程度上以一些特殊的运行时行为而闻名。

JavaScript对this情况的处理确实不寻常

```ts
class MyClass {
  name = "MyClass";
  getName() {
    return this.name;
  }
}
const c = new MyClass();
const obj = {
  name: "obj",
  getName: c.getName,
};

// Prints "obj", not "MyClass"
console.log(obj.getName());
```

长话短说，默认情况下，函数中的this值取决于如何调用函数。在这个例子中，因为函数是通过obj引用调用的，所以this的值是obj而不是类实例。

这不是你想要的!TypeScript提供了一些减少或防止此类错误的方法。

#### Arrow Functions

如果你有一个函数经常会以一种失去它的this上下文的方式被调用，那么使用一个箭头函数属性而不是方法定义是有意义的

```ts
class MyClass {
  name = "MyClass";
  getName = () => {
    return this.name;
  };
}
const c = new MyClass();
const g = c.getName;
// Prints "MyClass" instead of crashing
console.log(g());
```

这有一些权衡
- this值保证在运行时是正确的，即使是在没有使用TypeScript检查的代码中
- 这将使用更多的内存，因为每个类实例都有自己的以这种方式定义的每个函数副本
- 你不能使用`super.getName`在派生类中，因为原型链中没有获取基类方法的条目

#### `this` parameters

在方法或函数定义中，名为this的初始形参在TypeScript中有特殊含义。这些参数在编译期间会被擦除

```ts
// TypeScript input with 'this' parameter
function fn(this: SomeType, x: number) {
  /* ... */
}
// ----------------------------------
// JavaScript output
function fn(x) {
  /* ... */
}
```

TypeScript检查使用this形参调用函数时是否有正确的上下文。我们可以在方法定义中添加this参数，以静态地强制正确调用方法，而不是使用箭头函数

```ts
class MyClass {
  name = "MyClass";
  getName(this: MyClass) {
    return this.name;
  }
}
const c = new MyClass();
// OK
c.getName();

// Error, would crash
const g = c.getName;
console.log(g());
The 'this' context of type 'void' is not assignable to method's 'this' of type 'MyClass'.
```

这种方法采取了与箭头函数方法相反的权衡
- JavaScript调用者仍然可能不正确地使用类方法，而没有实现它
- 每个类定义只分配一个函数，而不是每个类实例分配一个函数
- 基方法定义仍然可以通过super调用。

#### `this` Types

在类中，一个名为this的特殊类型动态引用当前类的类型。让我们看看这是如何有用的

```ts
class Box {
  contents: string = "";
  set(value: string) {
  
// (method) Box.set(value: string): this
    this.contents = value;
    return this;
  }
}
```

这里，TypeScript推断set的返回类型是this，而不是Box。现在让我们创建一个Box的子类

```ts
class ClearableBox extends Box {
  clear() {
    this.contents = "";
  }
}

const a = new ClearableBox();
const b = a.set("hello");
     
// const b: ClearableBox
```

您还可以在参数类型注释中使用它

```ts
class Box {
  content: string = "";
  sameAs(other: this) {
    return other.content === this.content;
  }
}
```

这与编写`other: Box`不同，如果你有一个派生类，它的sameAs方法现在将只接受该派生类的其他实例

```ts
class Box {
  content: string = "";
  sameAs(other: this) {
    return other.content === this.content;
  }
}

class DerivedBox extends Box {
  otherContent: string = "?";
}

const base = new Box();
const derived = new DerivedBox();
derived.sameAs(base);
// Argument of type 'Box' is not assignable to parameter of type 'DerivedBox'.
//   Property 'otherContent' is missing in type 'Box' but required in type 'DerivedBox'.

```

#### `this` -based type guards 基于此类型的保护

你可以在类和接口中方法的返回位置使用`this is Type`当与类型收缩(例如if语句)混合时，目标对象的类型将被缩小为指定的type。

```ts
class FileSystemObject {
  isFile(): this is FileRep {
    return this instanceof FileRep;
  }
  isDirectory(): this is Directory {
    return this instanceof Directory;
  }
  isNetworked(): this is Networked & this {
    return this.networked;
  }
  constructor(public path: string, private networked: boolean) {}
}

class FileRep extends FileSystemObject {
  constructor(path: string, public content: string) {
    super(path, false);
  }
}

class Directory extends FileSystemObject {
  children: FileSystemObject[];
}

interface Networked {
  host: string;
}

const fso: FileSystemObject = new FileRep("foo/bar.txt", "foo");

if (fso.isFile()) {
  fso.content;
  
// const fso: FileRep
} else if (fso.isDirectory()) {
  fso.children;
  
// const fso: Directory
} else if (fso.isNetworked()) {
  fso.host;
  
// const fso: Networked & FileSystemObject
}
```

基于this的类型保护的一个常见用例是允许对特定字段进行延迟验证。例如，当hasValue被验证为true时，这个例子将从盒子中保存的值中删除一个未定义的值

```ts
class Box<T> {
  value?: T;

  hasValue(): this is { value: T } {
    return this.value !== undefined;
  }
}

const box = new Box();
box.value = "Gameboy";

box.value;
     
// (property) Box<unknown>.value?: unknown

if (box.hasValue()) {
  box.value;
       
// (property) value: unknown
}
```

### Parameter Properties

TypeScript提供了特殊的语法来将构造函数参数转换为具有相同名称和值的类属性。这些称为参数属性，通过在构造函数参数前加上一个可见性修饰符public、private、protected或readonly来创建。结果字段获得这些修饰符

```ts
class Params {
  constructor(
    public readonly x: number,
    protected y: number,
    private z: number
  ) {
    // No body necessary
  }
}
const a = new Params(1, 2, 3);
console.log(a.x);
             
// (property) Params.x: number
console.log(a.z);
// Property 'z' is private and only accessible within class 'Params'.
```

### Class Expressions 类表达式

类表达式与类声明非常相似。唯一真正的区别是类表达式不需要名称，尽管我们可以通过它们最终绑定到的任何标识符来引用它们

```ts
const someClass = class<Type> {
  content: Type;
  constructor(value: Type) {
    this.content = value;
  }
};

const m = new someClass("Hello, world");
     
// const m: someClass<string>
```

### `abstract` Classes and Members 抽象类和成员

TypeScript中的类、方法和字段可能是抽象的。

抽象方法或抽象字段是没有提供实现的方法。这些成员必须存在于抽象类中，而抽象类不能直接实例化。

抽象类的角色是作为实现所有抽象成员的子类的基类。当一个类没有任何抽象成员时，就称它为具体类。

让我们来看一个例子

```ts
abstract class Base {
  abstract getName(): string;

  printName() {
    console.log("Hello, " + this.getName());
  }
}

const b = new Base();
Cannot create an instance of an abstract class.
```

我们不能用new实例化Base，因为它是抽象的。相反，我们需要创建一个派生类并实现抽象成员

```ts
class Derived extends Base {
  getName() {
    return "world";
  }
}

const d = new Derived();
d.printName();
```

注意，如果我们忘记实现基类的抽象成员，就会得到一个错误

```ts
class Derived extends Base {
// Non-abstract class 'Derived' does not implement inherited abstract member 'getName' from class 'Base'. 非抽象类'Derived'不实现从类'Base'继承的抽象成员'getName'。
  // forgot to do anything
}
```

#### Abstract Construct Signatures

有时您想要接受某个类构造函数，该函数生成从某个抽象类派生的类的实例。

例如，您可能想要编写以下代码

```ts
function greet(ctor: typeof Base) {
  const instance = new ctor();
// Cannot create an instance of an abstract class. 不能创建抽象类的实例。
  instance.printName();
}
```

TypeScript正确地告诉你，你正在尝试实例化一个抽象类。毕竟，给定了greet的定义，编写这段代码是完全合法的，它最终将构造一个抽象类

```ts
// Bad!
greet(Base);
```

相反，您希望编写一个接受带有构造签名的内容的函数

```ts
function greet(ctor: new () => Base) {
  const instance = new ctor();
  instance.printName();
}
greet(Derived);
greet(Base);
// Argument of type 'typeof Base' is not assignable to parameter of type 'new () => Base'.
//   Cannot assign an abstract constructor type to a non-abstract constructor type. type 'typeof Base'类型的参数不能赋值给type 'new () => Base'类型的参数。不能将抽象构造函数类型指派给非抽象构造函数类型。
```

现在TypeScript正确地告诉你可以调用哪个类的构造函数——派生类可以调用，因为它是具体的，但是基类不能。

### Relationships Between Classes

在大多数情况下，TypeScript中的类在结构上进行比较，与其他类型一样。

例如，这两个类可以互相替换，因为它们是相同的

```ts
class Point1 {
  x = 0;
  y = 0;
}

class Point2 {
  x = 0;
  y = 0;
}

// OK
const p: Point1 = new Point2();
```

类似地，即使没有显式继承，类之间也存在子类型关系

```ts
class Person {
  name: string;
  age: number;
}

class Employee {
  name: string;
  age: number;
  salary: number;
}

// OK
const p: Person = new Employee();
```

这听起来很简单，但有一些情况似乎比其他情况更奇怪。

空类没有成员。在结构类型系统中，没有成员的类型通常是其他任何类型的超类型。因此，如果您编写一个空类(不要!)，任何东西都可以用来代替它

```ts
class Empty {}

function fn(x: Empty) {
  // can't do anything with 'x', so I won't
}

// All OK!
fn(window);
fn({});
fn(fn);
```

END -----------