## 常用类型

在本章中，我们将介绍一些JavaScript代码中最常见的值类型，并解释在TypeScript中描述这些类型的相应方法。这不是一个详尽的列表，未来的章节将描述更多命名和使用其他类型的方法。

除了类型注释之外，类型还可以出现在更多的地方。随着我们对类型本身的了解，我们还将了解可以引用这些类型来形成新结构的地方。

我们将从回顾在编写JavaScript或TypeScript代码时可能遇到的最基本和最常见的类型开始。这些稍后将形成更复杂类型的核心构建块。

### 基本类型 string number boolean

JavaScript有三个非常常用的基本类型：string, number, 和 boolean. 每一个在TypeScript中都有一个对应的类型。如您所料，如果您对这些类型的值使用JavaScript typeof操作符，就会看到相同的名称

string表示字符串值，如“Hello, world
number是42之类的数字。JavaScript没有针对整数的特殊运行时值，所以没有与int或float等价的东西——所有东西都是number
boolean是两个值true和false

>类型名称String、Number和Boolean(以大写字母开头)是合法的，但是引用一些特殊的内置类型，这些类型很少出现在您的代码中。类型总是使用字符串、数字或布尔值。

### Arrays

指定数组的类型如`[1,2,3]`，您可以使用语法 `number[]`; 此语法适用于任何类型（例如`string[]`是字符串数组，等等）你也可以把它写成`Array<number> `意思是一样的。当我们讨论泛型时，我们将学习更多关于语法`T<U>`的知识。

> 请注意，`[number]`是不同的东西;请参阅元组类型一节。

#### any

TypeScript还有一个特殊的类型 any，当你不希望某个特定的值导致类型检查错误时，你可以使用它。

当一个值的类型是any时，你可以访问它的任何属性(这将是任何类型的)，像调用一个函数一样调用它，将它赋值给(或从)任何类型的值，或者几乎任何语法上合法的东西

```ts
let obj: any = { x: 0 };
// None of the following lines of code will throw compiler errors.
// Using `any` disables all further type checking, and it is assumed 
// you know the environment better than TypeScript.
obj.foo();
obj();
obj.bar = 100;
obj = "hello";
const n: number = obj;
```

当你不想为了让TypeScript相信某一行代码没问题而写出一个很长的类型时，any类型很有用。

##### noImplicitAny

当你不指定类型时，TypeScript不能从上下文推断出它，编译器通常会默认为any。

但是，您通常希望避免这种情况，因为任何类型都没有进行类型检查。使用编译器标记noImplicitAny将任何隐式的any标记为错误。

#### 变量的类型注释

当使用const、var或let声明变量时，可以选择添加类型注释来显式指定变量的类型

```ts
let myName: string = "Alice";
```

> TypeScript不会在左侧样式声明中使用类型，比如int x = 0;类型注释总是在被键入的东西之后。

但是，在大多数情况下，这是不需要的。只要有可能，TypeScript就会自动推断你代码中的类型。例如，变量的类型是根据其初始化式的类型推断出来的

```ts
// No type annotation needed -- 'myName' inferred as type 'string'
let myName = "Alice";
```

在大多数情况下，你不需要明确地学习推理规则。如果你刚开始，试着少用一些类型注释——你可能会惊讶，你只需要很少的TypeScript就能完全理解发生了什么。

#### Functions

函数是JavaScript中传递数据的主要方法。TypeScript允许你指定函数的输入和输出值的类型。

##### 参数类型注解

在声明函数时，可以在每个参数后添加类型注释，以声明函数接受哪些类型的参数。参数类型注释在参数名称之后

```ts
// Parameter type annotation
function greet(name: string) {
  console.log("Hello, " + name.toUpperCase() + "!!");
}
```

当参数具有类型注释时，将检查该函数的参数

```ts
// Would be a runtime error if executed!
greet(42);
Argument of type 'number' is not assignable to parameter of type 'string'.
```

> 即使你的参数上没有类型注释，TypeScript仍然会检查你传递的参数数量是否正确。

##### 返回类型注释

您还可以添加返回类型注释。 返回类型注释显示参数列表后：

```ts
function getFavoriteNumber(): number {
  return 26;
}
```

就像变量类型注释一样，你通常不需要返回类型注释，因为TypeScript会根据函数的return语句推断函数的返回类型。上面例子中的类型注释没有改变任何东西。一些代码库将显式指定返回类型，以用于文档目的，以防止意外更改，或仅用于个人偏好。

##### 匿名函数

匿名函数与函数声明有一点不同。当一个函数出现在TypeScript可以决定如何调用它的地方时，该函数的形参会自动被赋予类型。

下面是一个示例：

```ts
// No type annotations here, but TypeScript can spot the bug
const names = ["Alice", "Bob", "Eve"];

// Contextual typing for function
names.forEach(function (s) {
  console.log(s.toUppercase());
Property 'toUppercase' does not exist on type 'string'. Did you mean 'toUpperCase'?
});

// Contextual typing also applies to arrow functions
names.forEach((s) => {
  console.log(s.toUppercase());
Property 'toUppercase' does not exist on type 'string'. Did you mean 'toUpperCase'?
})
```

尽管形参s没有类型注释，但TypeScript使用了forEach函数的类型，以及推断出的数组类型来确定s将具有的类型。
这个过程被称为上下文类型，因为函数发生的上下文决定了它应该具有什么类型。与推理规则类似，您不需要显式地了解这是如何发生的，但是理解它确实发生可以帮助您注意到不需要类型注释。稍后，我们将看到更多的例子，说明值所在的上下文如何影响其类型。

#### 对象的类型 Object Types

除了基本类型外，您将遇到的最常见类型是对象类型。这是指任何带有属性的JavaScript值，几乎是所有的!要定义对象类型，只需列出其属性及其类型。
例如，这里有一个函数，它取一个类似点的object

```ts
// The parameter's type annotation is an object type
function printCoord(pt: { x: number; y: number }) {
  console.log("The coordinate's x value is " + pt.x);
  console.log("The coordinate's y value is " + pt.y);
}
printCoord({ x: 3, y: 7 });
```

在这里，我们用具有两个属性(x和y)的类型来注释参数，它们的类型都是number。你可以使用，或者;分隔属性，最后一个分隔符是可选的。
每个属性的类型部分也是可选的。如果您不指定类型，它将被假定为any。

##### 可选属性

对象类型还可以指定其部分或所有属性是可选的。要做到这一点，加一个?在属性名之后

```ts
function printName(obj: { first: string; last?: string }) {
  // ...
}
// Both OK
printName({ first: "Bob" });
printName({ first: "Alice", last: "Alisson" });
```

在JavaScript中，如果您访问一个不存在的属性，您将得到undefined值，而不是运行时错误。因此，当从可选属性读取时，必须在使用之前检查是否为undefined

```ts
function printName(obj: { first: string; last?: string }) {
  // Error - might crash if 'obj.last' wasn't provided!
  console.log(obj.last.toUpperCase());
Object is possibly 'undefined'.
  if (obj.last !== undefined) {
    // OK
    console.log(obj.last.toUpperCase());
  }

  // A safe alternative using modern JavaScript syntax:
  console.log(obj.last?.toUpperCase());
}
```

#### 联合类型 Union Types

TypeScript类型系统允许你使用多种操作符从现有类型中构建新的类型。现在我们知道了如何编写一些类型，是时候开始以有趣的方式组合它们了。

##### 定义一个联合类型

您可能看到的组合类型的第一种方法是联合类型。联合类型是由两个或更多其他类型组成的类型，表示可能是这些类型中的任何一个的值。我们将这些类型统称为联合的成员。

让我们写一个函数，可以操作字符串或数字

```ts
function printId(id: number | string) {
  console.log("Your ID is: " + id);
}
// OK
printId(101);
// OK
printId("202");
// Error
printId({ myID: 22342 });
Argument of type '{ myID: number; }' is not assignable to parameter of type 'string | number'.
  Type '{ myID: number; }' is not assignable to type 'number'.
```

##### 使用联合类型

提供一个匹配联合类型的值是很容易的—只需提供一个匹配任何联合成员的类型。如果你有一个联合类型的值，你如何使用它

TypeScript只允许你使用union做事情，前提是它对union的每个成员都有效。例如，如果你有联合`string | number`，你就不能使用仅在string上可用的方法

```ts
function printId(id: number | string) {
  console.log(id.toUpperCase());
Property 'toUpperCase' does not exist on type 'string | number'.
  Property 'toUpperCase' does not exist on type 'number'.
}
```

解决方案是使用代码缩小联合，就像在没有类型注释的JavaScript中一样。当TypeScript可以根据代码的结构推导出一个更特定的值类型时，就会发生收缩。

例如，TypeScript知道只有字符串值的类型是“string”

```ts
function printId(id: number | string) {
  if (typeof id === "string") {
    // In this branch, id is of type 'string'
    console.log(id.toUpperCase());
  } else {
    // Here, id is of type 'number'
    console.log(id);
  }
}
```

另一个例子是使用像Array.isArray这样的函数

```ts
function welcomePeople(x: string[] | string) {
  if (Array.isArray(x)) {
    // Here: 'x' is 'string[]'
    console.log("Hello, " + x.join(" and "));
  } else {
    // Here: 'x' is 'string'
    console.log("Welcome lone traveler " + x);
  }
}
```

注意，在else分支中，我们不需要做任何特殊的事情——如果x不是`string[]`，那么它一定是string。

有时你会有一个union，所有成员都有共同点。例如，array和string都具slice方法。 如果Union中的每个成员都有一个属性，您可以在不缩小的情况下使用该属性：

```ts
// Return type is inferred as number[] | string
function getFirstThree(x: number[] | string) {
  return x.slice(0, 3);
}
```

> 类型的联合似乎具有这些类型属性的交集，这可能会让人感到困惑。这不是意外—联合这个名字来自类型理论。联合`number|string`是由每种类型的值的联合组成的。注意，给定两个集合，每个集合有对应的事实，只有这些事实的交集适用于集合本身的并集。例如，如果我们有一屋子的高个子戴帽子，另一屋子的西班牙语人士戴帽子，把这些房间合并后，我们对每个人的唯一了解就是他们一定戴着帽子。

#### 类型别名 Type Aliases

通过直接在类型注释中编写对象类型和联合类型，我们一直在使用它们。这很方便，但通常希望多次使用同一类型并使用单个名称引用它。

类型别名就是任何类型的名称。类型别名的语法是

```ts
type Point = {
  x: number;
  y: number;
};

// Exactly the same as the earlier example
function printCoord(pt: Point) {
  console.log("The coordinate's x value is " + pt.x);
  console.log("The coordinate's y value is " + pt.y);
}

printCoord({ x: 100, y: 100 });
```

实际上，您可以使用类型别名来为任何类型提供名称，而不仅仅是对象类型。例如，类型别名可以命名联合类型

`type ID = number | string;`

请注意，别名只是别名-您无法使用类型别名来创建相同类型的不同/不同的“版本”. 当您使用别名时，就像您已经编写了别名类型一样。换句话说，这段代码看起来可能是非法的，但根据TypeScript是可以的，因为这两种类型都是同一类型的别名

```ts
type UserInputSanitizedString = string;

function sanitizeInput(str: string): UserInputSanitizedString {
  return sanitize(str);
}

// Create a sanitized input
let userInput = sanitizeInput(getInput());

// Can still be re-assigned with a string though
userInput = "new input";
```

#### 接口 Interfaces

接口声明是命名对象类型的另一种方式

```ts
interface Point {
  x: number;
  y: number;
}

function printCoord(pt: Point) {
  console.log("The coordinate's x value is " + pt.x);
  console.log("The coordinate's y value is " + pt.y);
}

printCoord({ x: 100, y: 100 });
```

与上面使用类型别名时一样，这个示例的工作原理与使用匿名对象类型一样。TypeScript只关心我们传递给printCoord的值的结构——它只关心它有预期的属性。只关注类型的结构和功能，这就是为什么我们称TypeScript为结构类型的类型系统。

##### 类型别名和接口的区别

类型别名和接口非常相似，在许多情况下可以自由选择。接口的几乎所有特性在类型中都是可用的，关键区别在于`type`不能重新打开以添加新属性，而`interface`总是可扩展的。

- Interface

扩展接口

```ts
interface Animal {
  name: string
}

interface Bear extends Animal {
  honey: boolean
}

const bear = getBear() 
bear.name
bear.honey
```
向现有接口添加新字段

```ts
interface Window {
  title: string
}

interface Window {
  ts: TypeScriptAPI
}

const src = 'const a = "Hello World"';
window.ts.transpileModule(src, {});
```

- type

通过交叉扩展类型

```ts
type Animal = {
  name: string
}

type Bear = Animal & { 
  honey: Boolean 
}

const bear = getBear();
bear.name;
bear.honey;
```

类型创建后不能更改

```ts
type Window = {
  title: string
}

type Window = {
  ts: TypeScriptAPI
}

 // Error: Duplicate identifier 'Window'.
 ```

 您将在后面的章节中学习更多关于这些概念的知识，所以如果您不能马上理解所有这些，也不要担心。
 - 在输入版本4.2之前，唯一的别名名称可能出现在错误消息中，有时会代替等同的匿名类型（可能是也可能是不可取的）。 接口将始终以错误消息命名。
 - 类型别名可能不参与声明合并，但接口可以。
 - 接口只能用于声明对象的形状，不能用于重命名原语。
 - 接口名称将始终以其原始形式出现在错误消息中，但只有在按名称使用时才会出现。

 在大多数情况下，你可以根据个人偏好进行选择，TypeScript会告诉你是否需要其他类型的声明。如果您想使用启发式方法，请使用interface，直到需要使用type中的特性为止。

#### 类型断言 Type Assertions

有时候你会得到TypeScript不知道的值类型的信息。

例如，如果您正在使用文档。getElementById, TypeScript只知道这将返回某种HTMLElement，但你可能知道你的页面将始终拥有一个给定ID的HTMLCanvasElement。

在这种情况下，可以使用类型断言来指定更特定的类型

```ts
const myCanvas = document.getElementById("main_canvas") as HTMLCanvasElement;
```

与类型注释一样，类型断言被编译器删除，不会影响代码的运行时行为。

您也可以使用尖括号语法(除非代码在.tsx文件中)，这是等价的


```ts
const myCanvas = <HTMLCanvasElement>document.getElementById("main_canvas");
```

> 提示:因为类型断言是在编译时删除的，所以不存在与类型断言相关联的运行时检查。如果类型断言错误，将不会生成异常或空值。

TypeScript只允许类型断言转换为更特定或更不特定的类型版本。这条规则防止了不可能的胁迫，比如

```ts
const x = "hello" as number;
Conversion of type 'string' to type 'number' may be a mistake because neither type sufficiently overlaps with the other. If this was intentional, convert the expression to 'unknown' first.
```
有时这条规则可能过于保守，可能不允许更复杂的强制措施。如果发生这种情况，您可以使用两个断言，首先是`any`(或`unknown`，我们将在后面介绍)，然后是所需的类型

```ts
const a = (expr as any) as T;
```

##### 基元类型 Literal Types

除了一般的`string`和`number`类型外，我们还可以在类型位置中引用特定的字符串和数字。

考虑这个问题的一种方法是考虑JavaScript是如何以不同的方式声明变量的。var和let都允许修改变量内部的内容，const则不允许。这反映在TypeScript为字面值创建类型的方式中。

```ts
let changingString = "Hello World";
changingString = "Olá Mundo";
// Because `changingString` can represent any possible string, that
// is how TypeScript describes it in the type system
changingString;
      
let changingString: string

const constantString = "Hello World";
// Because `constantString` can only represent 1 possible string, it
// has a literal type representation
constantString;
      
const constantString: "Hello World"
```

通过自己，文字类型不是很有价值：

```ts
let x: "hello" = "hello";
// OK
x = "hello";
// ...
x = "howdy";
Type '"howdy"' is not assignable to type '"hello"'.
```

只有一个值的变量没有多大用处

但是通过将字面值组合成联合，您可以表达一个更有用的概念——例如，函数只接受一组特定的已知值

```ts
function printText(s: string, alignment: "left" | "right" | "center") {
  // ...
}
printText("Hello, world", "left");
printText("G'day, mate", "centre");
Argument of type '"centre"' is not assignable to parameter of type '"left" | "right" | "center"'.
```

数值文字类型的工作方式相同

```ts
function compare(a: string, b: string): -1 | 0 | 1 {
  return a === b ? 0 : a > b ? 1 : -1;
}
```

当然，您可以将它们与非文字类型结合使用

```ts
interface Options {
  width: number;
}
function configure(x: Options | "auto") {
  // ...
}
configure({ width: 100 });
configure("auto");
configure("automatic");
Argument of type '"automatic"' is not assignable to parameter of type 'Options | "auto"'.
```

还有一种文字类型:布尔文字。只有两种布尔文字类型，正如您可能猜到的那样，它们是`true`和`false`类型。boolean类型本身实际上只是联合`true | false`的别名。

##### 字面推断

当你用对象初始化一个变量时，TypeScript会假定这个对象的属性稍后会改变值。例如，如果你写这样的代码

```ts
const obj = { counter: 0 };
if (someCondition) {
  obj.counter = 1;
}
```

TypeScript并不认为将原来为0的字段赋值为1是错误的。另一种说法是obj。Counter必须具有类型编号，而不是0，因为类型用于确定读写行为。

这同样适用于字符串

```ts
const req = { url: "https://example.com", method: "GET" };
handleRequest(req.url, req.method);
Argument of type 'string' is not assignable to parameter of type '"GET" | "POST"'.
```
在上面的例子中，req.method被推断为字符串，而不是“GET”。因为代码可以在创建req和调用handleRequest之间进行计算，handleRequest可以给req分配一个像“GUESS”这样的新字符串。方法时，TypeScript会认为这段代码有错误。

有两种方法可以解决这个问题。
1. 可以通过在任意位置添加类型断言来更改推断

```ts
// Change 1:
const req = { url: "https://example.com", method: "GET" as "GET" };
// Change 2
handleRequest(req.url, req.method as "GET");
```

change1意味着我打算为req.method始终具有文本类型“GET”，防止在之后将“GUESS”赋值给该字段。change2意味着我知道其他需要的原因。方法的值为“GET”。

2. 您可以使用const将整个对象转换为类型文字

```ts
const req = { url: "https://example.com", method: "GET" } as const;
handleRequest(req.url, req.method);
```

as const后缀的作用类似于const，但对于类型系统，确保所有属性都被赋值为文字类型，而不是更一般的版本，如string或number。

#### null and undefined

JavaScript有两个原始值用来表示缺少或未初始化的值:null和undefined。
TypeScript有两个同名的对应类型。这些类型的行为取决于您是否启用strictNullChecks选项。

strictNullChecks off

通过strictNullChecks，可以正常访问可能为null或undefined的值，null和undefined可以被分配给任何类型的属性。这类似于没有空检查的语言(例如c#、Java)的行为。缺乏对这些值的检查往往是错误的主要来源;我们总是建议人们打开strictNullChecks，如果在他们的代码库中这样做是可行的。

strictNullChecks on

使用strictNullChecks on，当值为空或未定义时，您需要在对该值使用方法或属性之前测试这些值。就像在使用可选属性之前检查undefined一样，我们可以使用收缩来检查可能为空的值

```ts
function doSomething(x: string | null) {
  if (x === null) {
    // do nothing
  } else {
    console.log("Hello, " + x.toUpperCase());
  }
}
```

就像其他类型的断言一样，这不会改变代码的运行时行为，所以只使用!当你知道这个值不能为空或未定义时。

#### 枚举 Enums

枚举是TypeScript添加到JavaScript中的一个特性，它允许描述一个可能是一组命名常量中的一个值。与大多数TypeScript特性不同的是，这不是对JavaScript的类型级别的添加，而是对语言和运行时的添加。正因为如此，你应该知道它的存在，但除非你确定，否则不要使用它。你可以在枚举引用页中[阅读更多关于枚举的信息](https://www.typescriptlang.org/docs/handbook/enums.html)。

#### 不太常见的原语 Less Common Primitives

值得一提的是，JavaScript中其余的原语在类型系统中表示。虽然我们在这里不会深入讨论。

bigint

从ES2020开始，JavaScript中有一个用于非常大的整数的原语，BigInt

```ts
// Creating a bigint via the BigInt function
const oneHundred: bigint = BigInt(100);

// Creating a BigInt via the literal syntax
const anotherHundred: bigint = 100n;
```

您可以在TypeScript 3.2发行说明中了解有关Bigint的[更多信息](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-2.html#bigint)。

symbol

JavaScript中有一个原语，用于通过函数Symbol()创建全局唯一引用。

```ts
const firstName = Symbol("name");
const secondName = Symbol("name");

if (firstName === secondName) {
This condition will always return 'false' since the types 'typeof firstName' and 'typeof secondName' have no overlap.
  // Can't ever happen
}
```
你可以在符号参考页面[了解更多](https://www.typescriptlang.org/docs/handbook/symbols.html)