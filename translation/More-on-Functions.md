# More on Functions

函数是任何应用程序的基本构建块，无论它们是本地函数、从其他模块导入的函数，还是类上的方法。它们也是数据，就像其他数据一样，TypeScript有很多方法来描述如何调用函数。让我们学习如何编写描述函数的类型。

### 函数类型表达式 Function Type Expressions

描述函数最简单的方法是使用函数类型表达式。这些类型在语法上类似于箭头函数

```ts
function greeter(fn: (a: string) => void) {
  fn("Hello, World");
}

function printToConsole(s: string) {
  console.log(s);
}

greeter(printToConsole);
```

语法`(a: string) => void`表示一个函数只有一个参数，名为`a`，类型为`string`，没有返回值。就像函数声明一样，如果一个形参类型没有指定，它就隐式地为`any`。

> 注意，参数名称是必需的。函数`type (string) => void`表示函数的形参为`any`类型的

当然，我们可以使用类型别名来命名函数类型

```ts
type GreetFunction = (a: string) => void;
function greeter(fn: GreetFunction) {
  // ...
}
```

#### Call Signatures 调用签名

在JavaScript中，函数除了可调用外，还可以具有属性。但是，函数类型表达式语法不允许声明属性。如果我们想用属性描述一些可调用的东西，我们可以在对象类型中编写调用签名

```ts
type DescribableFunction = {
  description: string;
  (someArg: number): boolean;
};
function doSomething(fn: DescribableFunction) {
  console.log(fn.description + " returned " + fn(6));
}
```

注意，该语法与函数类型表达式- 使用略有不同:在形参列表和返回类型之间，而不是=>。

### Construct Signatures 构造签名

也可以用new操作符调用JavaScript函数。TypeScript将其称为构造函数，因为它们通常会创建一个新对象。通过在调用签名前添加new关键字，可以编写构造签名

```ts
type SomeConstructor = {
  new (s: string): SomeObject;
};
function fn(ctor: SomeConstructor) {
  return new ctor("hello");
}
```

一些对象，比如JavaScript的Date对象，可以使用new或不使用new调用。可以任意组合调用和构造同一类型的签名

```ts
interface CallOrConstruct {
  new (s: string): Date;
  (n?: number): number;
}
```

#### Generic Functions

通常编写的函数中，输入的类型与输出的类型相关，或者两个输入的类型以某种方式相关。让我们考虑一个返回数组第一个元素的函数

```ts
function firstElement(arr: any[]) {
  return arr[0];
}
```

这个函数完成了它的工作，但不幸的是返回类型是any。如果函数返回数组元素的类型会更好。

在TypeScript中，当我们想要描述两个值之间的通信时，就会使用泛型。我们通过在函数签名中声明类型参数来做到这一点

```ts
function firstElement<Type>(arr: Type[]): Type {
  return arr[0];
}
```

通过向这个函数添加类型参数type并在两个地方使用它，我们在函数的输入(数组)和输出(返回值)之间创建了一个链接。当我们叫它的时候，一个更具体的类型出现了

```ts
// s is of type 'string'
const s = firstElement(["a", "b", "c"]);
// n is of type 'number'
const n = firstElement([1, 2, 3]);
```

##### Inference

注意，我们不必在这个示例中指定Type。类型是由TypeScript自动选择的。

我们也可以使用多个类型参数。例如，map的独立版本如下所示

```ts
function map<Input, Output>(arr: Input[], func: (arg: Input) => Output): Output[] {
  return arr.map(func);
}

// Parameter 'n' is of type 'string'
// 'parsed' is of type 'number[]'
const parsed = map(["1", "2", "3"], (n) => parseInt(n));
```

注意，在这个例子中，TypeScript可以根据函数表达式的返回值(number)推断输入类型参数的类型(从给定的字符串数组)，以及输出类型参数的类型。


##### Constraints

我们已经写了一些泛型函数，可以处理任何类型的值。有时我们希望将两个值关联起来，但只能对值的某个子集进行操作。在这种情况下，我们可以使用约束来限制类型参数可以接受的类型类型。

让我们编写一个返回两个值较长的函数。 为此，我们需要一个数字的length属性。 通过编写extends子句，我们将类型参数限制为此类型：

```ts
function longest<Type extends { length: number }>(a: Type, b: Type) {
  if (a.length >= b.length) {
    return a;
  } else {
    return b;
  }
}

// longerArray is of type 'number[]'
const longerArray = longest([1, 2], [1, 2, 3]);
// longerString is of type 'string'
const longerString = longest("alice", "bob");
// Error! Numbers don't have a 'length' property
const notOK = longest(10, 100);
//Argument of type 'number' is not assignable to parameter of type '{ length: number; }'.
```

在这个例子中有一些有趣的事情需要注意。我们允许TypeScript推断longest的返回类型。返回类型推断也适用于泛型函数。

因为我们将`Type`限制为`{length: number}`，所以我们可以访问a和b参数的`.length`属性。如果没有类型约束，我们将无法访问这些属性，因为这些值可能是没有长度属性的其他类型。

longerArray和longerString的类型是根据参数推断出来的。请记住，泛型都是关于将两个或多个相同类型的值关联起来

最后，正如我们所希望的那样，对`longest(10,100)`的调用被拒绝，因为数字类型没有`.length`属性。

##### 使用约束值

当使用通用约束时，这里有一个常见的错误

```ts
function minimumLength<Type extends { length: number }>(
  obj: Type,
  minimum: number
): Type {
  if (obj.length >= minimum) {
    return obj;
  } else {
    return { length: minimum };
// Type '{ length: number; }' is not assignable to type 'Type'.
//   '{ length: number; }' is assignable to the constraint of type 'Type', but 'Type' could be instantiated with a different subtype of constraint '{ length: number; }'.
  }
}
```

这个函数可能看起来是OK - Type被限制为`{length: number}`，并且函数返回Type或匹配该约束的值。问题是，该函数承诺返回与传入的相同类型的对象，而不仅仅是一些匹配约束的对象。如果这些代码是合法的，那么您可以编写绝对不能工作的代码

```ts
// 'arr' gets value { length: 6 }
const arr = minimumLength([1, 2, 3], 6);
// and crashes here because arrays have
// a 'slice' method, but not the returned object!
console.log(arr.slice(0));
```

##### 指定类型参数

TypeScript通常可以推断出泛型调用中想要的类型参数，但也不总是这样。例如，假设你写了一个函数来合并两个数组

```ts
function combine<Type>(arr1: Type[], arr2: Type[]): Type[] {
  return arr1.concat(arr2);
}
```

通常，如果使用不匹配的数组调用该函数将会出错

```ts
const arr = combine([1, 2, 3], ["hello"]);
Type 'string' is not assignable to type 'number'.
```

但是，如果您打算这样做，您可以手动指定Type

```ts
const arr = combine<string | number>([1, 2, 3], ["hello"]);
```

##### Guidelines for Writing Good Generic Functions

编写泛型函数很有趣，而且很容易被类型参数冲晕。有太多类型参数或在不需要的地方使用约束会使推断不那么成功，使函数的调用者感到沮丧。

###### Push Type Parameters Down

下面有两种编写类似函数的方法

```ts
function firstElement1<Type>(arr: Type[]) {
  return arr[0];
}

function firstElement2<Type extends any[]>(arr: Type) {
  return arr[0];
}

// a: number (good)
const a = firstElement1([1, 2, 3]);
// b: any (bad)
const b = firstElement2([1, 2, 3]);
```
乍一看，它们可能是相同的，但firstElement1是编写这个函数的更好方法。它的推断返回类型是type，但是firstElement2的推断返回类型是any，因为TypeScript必须使用约束类型来解析arr[0]表达式，而不是在调用期间等待解析元素。

> 规则:如果可能，使用类型参数本身而不是约束它

###### 类型参数应该出现两次

有时我们忘记了一个函数可能不需要是泛型的

```ts
function greet<Str extends string>(s: Str) {
  console.log("Hello, " + s);
}

greet("world");
```

我们也可以简单地写出一个更简单的版本

```ts
function greet(s: string) {
  console.log("Hello, " + s);
}
```

记住，类型参数用于关联多个值的类型。如果类型参数只在函数签名中使用一次，那么它与任何东西都没有关联。

##### optional parameters

JavaScript中的函数通常有不同数量的参数。例如，number的toFixed方法接受一个可选的数字计数

```ts
function f(n: number) {
  console.log(n.toFixed()); // 0 arguments
  console.log(n.toFixed(3)); // 1 argument
}
```

我们可以在TypeScript中通过将参数标记为可选来建模

```ts
function f(x?: number) {
  // ...
}
f(); // OK
f(10); // OK
```

虽然参数被指定为类型编号，但x参数实际上的类型`number|undefined`的，因为JavaScript中未指定的参数的值是undefined的。

您还可以提供一个参数默认值

```ts
function f(x = 10) {
  // ...
}
```

现在在f的主体中，x将有类型number，因为任何undefined的参数都将被替换为10.请注意，当参数是可选时，呼叫者可以始终通过undefined，因为这只是模拟“缺少”参数：

```ts
declare function f(x?: number): void;
// cut
// All OK
f();
f(10);
f(undefined);
```

##### Optional Parameters in Callbacks

一旦了解了可选参数和函数类型表达式，就会在编写调用回调的函数时，非常容易进行以下错误：

```ts
function myForEach(arr: any[], callback: (arg: any, index?: number) => void) {
  for (let i = 0; i < arr.length; i++) {
    callback(arr[i], i);
  }
}
```

人们写index时通常打算做什么?作为一个可选参数，它们希望这两个调用都是合法的

```ts
myForEach([1, 2, 3], (a) => console.log(a));
myForEach([1, 2, 3], (a, i) => console.log(a, i));
```

这实际上意味着调用callback时可能只有一个参数。换句话说，函数定义说实现可能是这样的

```ts
function myForEach(arr: any[], callback: (arg: any, index?: number) => void) {
  for (let i = 0; i < arr.length; i++) {
    // I don't feel like providing the index today
    callback(arr[i]);
  }
}
```

反过来，TypeScript会强制执行这一含义，并发出不太可能的错误

```ts
myForEach([1, 2, 3], (a, i) => {
  console.log(i.toFixed());
// Object is possibly 'undefined'.
});
```

在JavaScript中，如果调用一个参数多于参数的函数，那么额外的参数就会被忽略。TypeScript也有同样的行为。具有较少参数(相同类型)的函数总是可以取代具有更多参数的函数。

当为回调函数编写函数类型时，永远不要编写可选形参，除非你想在不传递实参的情况下调用函数

#### Function Overloads

可以在各种参数计数和类型中调用一些JavaScript函数。 例如，您可以编写一个函数来生成带时间戳（一个参数）或一个月/日/年规范（三个参数）的日期。

在TypeScript中，我们可以通过编写重载签名来指定一个可以以不同方式调用的函数。为此，编写一些函数签名(通常是两个或更多)，然后是函数体

```ts
function makeDate(timestamp: number): Date;
function makeDate(m: number, d: number, y: number): Date;
function makeDate(mOrTimestamp: number, d?: number, y?: number): Date {
  if (d !== undefined && y !== undefined) {
    return new Date(y, mOrTimestamp, d);
  } else {
    return new Date(mOrTimestamp);
  }
}
const d1 = makeDate(12345678);
const d2 = makeDate(5, 5, 5);
const d3 = makeDate(1, 3);
// No overload expects 2 arguments, but overloads do exist that expect either 1 or 3 arguments.
```

在这个例子中，我们编写了两个重载:一个接受一个参数，另一个接受三个参数。前两个签名称为重载签名。

然后，我们编写了一个带有兼容签名的函数实现。函数有一个实现签名，但是不能直接调用这个签名。即使我们编写的函数在必需的参数之后有两个可选参数，也不能用两个参数调用它

##### Overload Signatures and the Implementation Signature

这是一个常见的混淆来源。人们经常会编写这样的代码，却不明白为什么会有错误

```ts
function fn(x: string): void;
function fn() {
  // ...
}
// Expected to be able to call with zero arguments
fn();
// Expected 1 arguments, but got 0.
```

同样，用于编写函数体的签名不能从外部看到。

> 实现的签名从外部是不可见的。在编写重载函数时，应该始终在函数实现的上方有两个或多个签名。

实现签名还必须与重载签名兼容。例如，这些函数有错误，因为实现签名没有以正确的方式匹配重载

```ts
function fn(x: boolean): void;
// Argument type isn't right
function fn(x: string): void;
// This overload signature is not compatible with its implementation signature.
function fn(x: boolean) {}
```

```ts
function fn(x: string): string;
// Return type isn't right
function fn(x: number): boolean;
This overload signature is not compatible with its implementation signature.
function fn(x: string | number) {
  return "oops";
}
```

##### Writing Good Overloads

与泛型一样，在使用函数重载时也需要遵循一些指导原则。遵循这些原则将使您的函数更易于调用、理解和实现。

让我们考虑一个返回字符串或数组长度的函数

```ts
function len(s: string): number;
function len(arr: any[]): number;
function len(x: any) {
  return x.length;
}
```

这个函数很好;我们可以用字符串或数组调用它。但是，我们不能用字符串或数组的值来调用它，因为TypeScript只能将函数调用解析为一次重载

```ts
len(""); // OK
len([0]); // OK
len(Math.random() > 0.5 ? "hello" : [0]);
// No overload matches this call.
//   Overload 1 of 2, '(s: string): number', gave the following error.
//     Argument of type 'number[] | "hello"' is not assignable to parameter of type 'string'.
//       Type 'number[]' is not assignable to type 'string'.
//   Overload 2 of 2, '(arr: any[]): number', gave the following error.
//     Argument of type 'number[] | "hello"' is not assignable to parameter of type 'any[]'.
//       Type 'string' is not assignable to type 'any[]'.
```

因为两个重载具有相同的实参计数和相同的返回类型，所以我们可以编写一个非重载的函数版本

```ts
function len(x: any[] | string) {
  return x.length;
}
```

这样好多了!调用者可以用任何一种值调用它，而且作为额外的好处，我们不需要找出正确的实现签名。

> 尽可能使用联合类型的形参而不是重载形参

##### Declaring this in a Function

TypeScript会通过代码流分析推断出函数中的this应该是什么，如下所示

```ts
const user = {
  id: 123,

  admin: false,
  becomeAdmin: function () {
    this.admin = true;
  },
};
```

TypeScript理解函数用户。becomeAdmin有一个对应的this，这是外部对象user。这在很多情况下都足够了，但在很多情况下你需要更多地控制这个表示什么对象。JavaScript规范规定，你不能有一个名为this的形参，所以TypeScript使用这个语法空间让你在函数体中声明this的类型

```ts
interface DB {
  filterUsers(filter: (this: User) => boolean): User[];
}

const db = getDB();
const admins = db.filterUsers(function (this: User) {
  return this.admin;
});
```

这种模式在回调风格的api中很常见，在回调风格的api中，另一个对象通常控制调用函数的时间。注意，您需要使用函数而不是箭头函数来获得这种行为

```ts
interface DB {
  filterUsers(filter: (this: User) => boolean): User[];
}

const db = getDB();
const admins = db.filterUsers(() => this.admin);
// The containing arrow function captures the global value of 'this'.
// Element implicitly has an 'any' type because type 'typeof globalThis' has no index signature.
```

#### Other Types to Know About

在处理函数类型时，还需要识别一些经常出现的其他类型。与所有类型一样，您可以在任何地方使用它们，但它们在函数上下文中尤其相关。

`void`

Void表示不返回值的函数的返回值。当一个函数没有任何return语句，或者没有从这些return语句中返回任何显式值时，它就是推断类型

```ts
// The inferred return type is void
function noop() {
  return;
}
```

在JavaScript中，一个不返回任何值的函数将隐式地返回undefined。然而，void和undefined在TypeScript中并不是一回事。在这一章的末尾有更多的细节。

> void并不等同于undefined

object

特殊类型对象指的是任何非原语的值(字符串、数字、布尔值、符号、null或undefined)。这不同于空对象类型{}，也不同于全局类型object。很可能你永远不会使用Object。

> object不是Object。总是使用object

注意，在JavaScript中，函数值是对象:它们有属性，有Object.prototype在它们的原型链中，是Object的实例，你可以调用Object.keys，等等。因此，函数类型在TypeScript中被认为是对象。

unknown

未知类型表示任何值。这与任何类型类似，但更安全，因为使用未知值是不合法的

```ts
function f1(a: any) {
  a.b(); // OK
}
function f2(a: unknown) {
  a.b();
// Object is of type 'unknown'. 
}
```

这在描述函数类型时很有用，因为您可以描述接受任何值而在函数体中没有任何值的函数。

相反，你可以描述一个返回未知类型值的函数

```ts
function safeParse(s: string): unknown {
  return JSON.parse(s);
}

// Need to be careful with 'obj'!
const obj = safeParse(someRandomString);
```

never

有些函数从不返回值

```ts
function fail(msg: string): never {
  throw new Error(msg);
}
```

never类型表示永远不会被观察到的值。在返回类型中，这意味着函数抛出异常或终止程序的执行。

当TypeScript确定union中没有剩余时，never也不会出现。

```ts
function fn(x: string | number) {
  if (typeof x === "string") {
    // do something
  } else if (typeof x === "number") {
    // do something else
  } else {
    x; // has type 'never'!
  }
}
```

Function

全局类型Function描述诸如bind、call、apply等JavaScript中所有函数值上的属性。它还有一个特殊的属性，Function类型的值总是可以被调用;这些调用返回任何

```ts
function doSomething(f: Function) {
  f(1, 2, 3);
}
```

这是一个非类型化函数调用，通常最好避免，因为任何返回类型都不安全。

如果您需要接受任意函数，但不打算调用它，`type () => void`通常更安全。

#### Rest Parameters and Arguments

##### Rest Parameters

除了使用可选形参或重载使函数能够接受各种固定的实参计数外，我们还可以使用rest形参定义接受无界数目实参的函数。

rest参数出现在所有其他参数之后，并使用…语法

```ts
function multiply(n: number, ...m: number[]) {
  return m.map((x) => n * x);
}
// 'a' gets value [10, 20, 30, 40]
const a = multiply(10, 1, 2, 3, 4);
```

在TypeScript中，这些参数上的类型注释隐式地是`any[]`而不是any，并且给出的任何类型注释必须是`Array<T>`或`T[]`的形式，或者是元组类型(我们将在后面学习)。

##### Rest Arguments

相反，我们可以使用spread语法从数组中提供数量可变的参数。例如，数组的push方法接受任意数量的参数

```ts
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
arr1.push(...arr2);
```

注意，通常情况下，TypeScript并不假定数组是不可变的。这会导致一些令人惊讶的行为

```ts

// Inferred type is number[] -- "an array with zero or more numbers",
// not specifically two numbers
const args = [8, 5];
const angle = Math.atan2(...args);
// Expected 2 arguments, but got 0 or more.
```

这种情况的最佳解决方案取决于您的代码，但通常const上下文是最简单的解决方案

```ts
// Inferred as 2-length tuple
const args = [8, 5] as const;
// OK
const angle = Math.atan2(...args);
```

使用rest参数可能需要在针对旧运行时时打开downlevelIteration。

#### Parameter Destructuring

可以使用形参解构方便地将作为实参提供的对象解包到函数体中的一个或多个局部变量中。在JavaScript中，它是这样的

```ts
function sum({ a, b, c }) {
  console.log(a + b + c);
}
sum({ a: 10, b: 3, c: 9 });
```

对象的类型注释位于解构语法之后

```ts
function sum({ a, b, c }: { a: number; b: number; c: number }) {
  console.log(a + b + c);
}
```

这看起来有点冗长，但是您也可以在这里使用命名类型

```ts
// Same as prior example
type ABC = { a: number; b: number; c: number };
function sum({ a, b, c }: ABC) {
  console.log(a + b + c);
}
```

#### Assignability of Functions

Return type void

函数的void返回类型可能产生一些不寻常的，但预期的beh

返回类型为void的上下文类型并不强制函数不返回某些内容。另一种说法是，这是一个带有void返回类型的上下文函数类型`(type vf = () => void)`，当实现时，可以返回任何其他值，但它将被忽略。

因此，`type () => void`的以下实现是有效的

```ts
type voidFunc = () => void;

const f1: voidFunc = () => {
  return true;
};

const f2: voidFunc = () => true;

const f3: voidFunc = function () {
  return true;
};
```

当其中一个函数的返回值被赋值给另一个变量时，它将保持void类型

```ts
const v1 = f1();

const v2 = f2();

const v3 = f3();
```

此行为的存在使以下代码有效，即使`Array.prototype.push`返回一个数字，而`Array.prototype.forEach`方法期望一个返回类型为void的函数。

```ts
const src = [1, 2, 3];
const dst = [0];

src.forEach((el) => dst.push(el));
```

还有一种特殊情况需要注意，当文字函数定义的返回类型为void时，该函数不能返回任何东西。

```ts
function f2(): void {
  // @ts-expect-error
  return true;
}

const f3 = function (): void {
  // @ts-expect-error
  return true;
};
```

有关void的更多信息，请参考这些其他文档条

[v1 handbook](https://www.typescriptlang.org/docs/handbook/basic-types.html#void)
[v2 handbook](https://www.typescriptlang.org/docs/handbook/2/functions.html#void)
[FAQ - “Why are functions returning non-void assignable to function returning void?”](https://github.com/Microsoft/TypeScript/wiki/FAQ#why-are-functions-returning-non-void-assignable-to-function-returning-void)

end