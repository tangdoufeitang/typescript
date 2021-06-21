 ## Creating Types from Types

 TypeScript的类型系统非常强大，因为它允许用其他类型来表示类型。

 这种思想最简单的形式是泛型，实际上我们有各种各样的类型操作符。也可以用我们已有的值来表示类型。

 通过组合各种类型操作符，我们可以以一种简洁、可维护的方式表达复杂的操作和值。在本节中，我们将介绍用现有类型或值表示新类型的方法。

 - [Generics](https://www.typescriptlang.org/docs/handbook/2/generics.html) - 带参数的类型
 - [Keyof Type Operator]() - 使用Keyof运算符创建新类型
 - [Typeof Type Operator ]() - 使用TypeOf运算符创建新类型
 - [Indexed Access Types]() - 使用Type['a']语法访问类型的子集
 - [Conditional Types ]() - 类似于类型系统中的if语句的类型
 - [Mapped Types]() - 通过映射现有类型中的每个属性来创建类型
 - [Template Literal Types ]() - 映射类型，通过模板字面值字符串改变属性

 ## Generics 泛型

 软件工程的一个主要部分是构建组件，这些组件不仅具有定义良好和一致的api，而且还可以重用。能够处理当前数据和未来数据的组件将为构建大型软件系统提供最灵活的功能。

 在C＃和Java这样的语言中，工具箱中的一个主要工具中的一个用于创建可重用组件的主要工具是泛型，即，能够创建可以使用多种类型而不是单个类型的组件而不是单个类型。 这允许用户使用这些组件并使用自己的类型。

 ### Hello World of Generics

 首先，让我们来做泛型的hello world:恒等函数。恒等函数将返回传入的任何东西。您可以以类似于echo命令的方式来考虑这一点。

 如果没有泛型，我们将不得不给恒等函数一个特定的类型

 ```ts
 function identity(arg: number): number {
  return arg;
}
```

或者，我们可以用any类型来描述恒等函数

```ts
function identity(arg: any): any {
  return arg;
}
```

虽然使用any肯定是通用的，因为它将导致函数接受任何和所有类型的arg类型，但当函数返回时，我们实际上丢失了有关该类型的信息。如果我们传入一个数字，我们所拥有的唯一信息就是可以返回任何类型。

相反，我们需要一种捕获参数类型的方式，这样我们也可以使用它来表示返回的内容。这里，我们将使用类型变量，这是一种特殊类型的变量，它作用于类型而不是值。

```ts
function identity<Type>(arg: Type): Type {
  return arg;
}
``` 

我们现在为恒等函数添加了类型变量type。这种类型允许我们捕获用户提供的类型(例如数字)，以便我们以后使用该信息。这里，我们再次使用Type作为返回类型。通过检查，我们现在可以看到对参数和返回类型使用了相同的类型。这允许我们在函数的一端传输类型信息，在另一端输出。

我们说恒等函数的这个版本是泛型的，因为它适用于一系列类型。与使用any不同，它与第一个使用数字作为参数和返回类型的标识函数一样精确(例如，它不会丢失任何信息)。

一旦我们写出了一般恒等函数，我们可以用两种方法之一来调用它。第一种方法是将所有参数(包括类型参数)传递给函数

```ts
let output = identity<string>("myString");
``` 

在这里，我们显式地将Type设置为string作为函数调用的参数之一，使用参数周围的<>而不是()来表示。

第二种方式或许也是最常见的。这里我们使用类型参数推断，也就是说，我们希望编译器根据传入的参数类型自动为我们设置type的值

```ts
let output = identity("myString");
``` 

注意，我们不必显式地将类型传递到尖括号(<>)中;编译器只是查看值“myString”，并将Type设置为它的类型。虽然类型参数推断是保持代码更短、更易读的有用工具，但是当编译器无法推断类型时(在更复杂的示例中可能会发生这种情况)，您可能需要显式地传入类型参数，就像我们在前面的示例中所做的那样。

### Working with Generic Type Variables 使用泛型类型变量

当您开始使用泛型时，您会注意到，当您创建像identity这样的泛型函数时，编译器将强制您在函数体中正确地使用任何泛型参数。也就是说，您实际上将这些参数视为它们可以是任何类型或所有类型。

我们用之前的恒等函数

```ts
function identity<Type>(arg: Type): Type {
  return arg;
}
``` 

如果我们还想在每次调用时记录控制台参数arg的长度，该怎么办?我们可能会这样写

```ts
function loggingIdentity<Type>(arg: Type): Type {
  console.log(arg.length);
//Property 'length' does not exist on type 'Type'.
  return arg;
}
``` 

当这样做时，编译器会给我们一个错误，说我们正在使用arg的`.length`成员，但我们没有说过arg有这个成员。记住，我们前面说过，这些类型变量代表任何和所有类型，所以使用这个函数的人可以传入一个没有`.length`成员的`number`。

假设我们实际上打算让这个函数处理Type数组，而不是直接处理Type数组。因为我们使用数组，所以.length成员应该是可用的。我们可以像创建其他类型的数组一样描述这一点

```ts
function loggingIdentity<Type>(arg: Type[]): Type[] {
  console.log(arg.length);
  return arg;
}
``` 

您可以将loggingIdentity的类型读取为泛型函数loggingIdentity接受类型参数type和参数arg(一个类型数组)，并返回一个类型数组。如果传入一个数字数组，则返回一个数字数组，因为Type将绑定到number。这允许我们使用泛型类型变量type作为我们所处理的类型的一部分，而不是整个类型，这给了我们更大的灵活性。

我们也可以这样编写示例示例

```ts
function loggingIdentity<Type>(arg: Array<Type>): Array<Type> {
  console.log(arg.length); // Array has a .length, so no more error
  return arg;
}
``` 

您可能已经从其他语言中熟悉了这种类型风格。在下一节中，我们将介绍如何创建自己的泛型类型，如`Array<Type>`。

### Generic Types

在前几节中，我们创建了适用于一系列类型的通用标识函数。在本节中，我们将探讨函数本身的类型以及如何创建泛型接口。

泛型函数的类型就像非泛型函数的类型一样，类型参数列在前面，类似于函数声明

```ts
function identity<Type>(arg: Type): Type {
  return arg;
}

let myIdentity: <Type>(arg: Type) => Type = identity;
``` 

只要类型变量的数量和类型变量的使用方式一致，我们也可以对类型中的泛型类型参数使用不同的名称。

```ts
function identity<Type>(arg: Type): Type {
  return arg;
}

let myIdentity: <Input>(arg: Input) => Input = identity;
```

我们还可以将泛型类型编写为对象字面量类型的调用签名

```ts
function identity<Type>(arg: Type): Type {
  return arg;
}

let myIdentity: { <Type>(arg: Type): Type } = identity;
```

这就引导我们编写了第一个泛型接口。让我们把前面例子中的对象文字移到一个接口中

```ts
interface GenericIdentityFn {
  <Type>(arg: Type): Type;
}

function identity<Type>(arg: Type): Type {
  return arg;
}

let myIdentity: GenericIdentityFn = identity;
```

在类似的示例中，我们可能希望将泛型参数移动为整个接口的参数。这让我们看到泛型的类型(例如`Dictionary<string>`而不是`Dictionary`)。这使得类型参数对接口的所有其他成员可见。

```ts
interface GenericIdentityFn<Type> {
  (arg: Type): Type;
}

function identity<Type>(arg: Type): Type {
  return arg;
}

let myIdentity: GenericIdentityFn<number> = identity;
```

注意，我们的示例略有不同。我们现在有一个非泛型函数签名，它是泛型类型的一部分，而不是描述泛型函数。当我们使用GenericIdentityFn时，我们现在还需要指定相应的类型参数(这里是:number)，从而有效地锁定底层调用签名将使用的内容。理解何时将类型参数直接放在调用签名上，以及何时将其放在接口本身上，将有助于描述类型的哪些方面是泛型的。

除了泛型接口之外，我们还可以创建泛型类。注意，不可能创建通用泛型和名称空间。

### Generic Classes

泛型类具有与泛型接口相似的形状。泛型类在类名后面有一个尖括号(<>)中的泛型类型参数列表。


```ts
class GenericNumber<NumType> {
  zeroValue: NumType;
  add: (x: NumType, y: NumType) => NumType;
}

let myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = function (x, y) {
  return x + y;
};

```

这是GenericNumber类的字面用法，但是您可能已经注意到，没有什么限制它只使用number类型。我们可以使用string或者更复杂的对象。

```ts
let stringNumeric = new GenericNumber<string>();
stringNumeric.zeroValue = "";
stringNumeric.add = function (x, y) {
  return x + y;
};

console.log(stringNumeric.add(stringNumeric.zeroValue, "test"));
```

与interface一样，将type参数放在类本身上可以确保类的所有属性都使用相同的类型。

正如我们在关于类的一节中所述，类的类型有两个方面:静态方面和实例方面。泛型类仅在它们的实例端而不是静态端上是泛型的，因此在处理类时，静态成员不能使用类的类型参数。

### Generic Constraints

如果您还记得前面的示例，您有时可能想要编写一个泛型函数，它作用于一组类型，您对这组类型将具有的功能有一些了解。在我们的logingidentity示例中，我们希望能够访问arg的.length属性，但是编译器不能证明每种类型都有.length属性，因此它警告我们不能做出这种假设。

```ts
function loggingIdentity<Type>(arg: Type): Type {
  console.log(arg.length);
//Property 'length' does not exist on type 'Type'.
  return arg;
}
```

我们不想使用任何和所有类型，而是希望约束此函数，使其能够使用同样具有.length属性的任何和所有类型。只要类型有这个成员，我们就允许它，但它必须至少有这个成员。为此，我们必须将我们的需求作为Type可以是什么的约束列出。

为此，我们将创建一个描述约束的接口。这里，我们将创建一个具有单个.length属性的接口，然后使用该接口和extends关键字来表示我们的约束

```ts
interface Lengthwise {
  length: number;
}

function loggingIdentity<Type extends Lengthwise>(arg: Type): Type {
  console.log(arg.length); // Now we know it has a .length property, so no more error
  return arg;
}
```

因为泛型函数现在受到约束，它将不再适用于任何和所有类型

```ts
loggingIdentity(3);
// Argument of type 'number' is not assignable to parameter of type 'Lengthwise'.
```

相反，我们需要传入类型具有所有必需属性的值

```ts
loggingIdentity({ length: 10, value: 3 });
```

### Using Type Parameters in Generic Constraints 在泛型约束中使用类型参数

可以声明受另一个类型参数约束的类型参数。例如，这里我们希望从给定名称的对象中获取属性。我们希望确保我们不会意外地获取obj上不存在的属性，因此我们将在这两种类型之间设置一个约束

```ts
function getProperty<Type, Key extends keyof Type>(obj: Type, key: Key) {
  return obj[key];
}

let x = { a: 1, b: 2, c: 3, d: 4 };

getProperty(x, "a");
getProperty(x, "m");
//Argument of type '"m"' is not assignable to parameter of type '"a" | "b" | "c" | "d"'.
```

### Using Class Types in Generics 在泛型中使用类类型

当在TypeScript中使用泛型创建工厂时，需要通过类的构造函数引用类类型。例如

```ts
function create<Type>(c: { new (): Type }): Type {
  return new c();
}
```

更高级的示例使用prototype属性来推断和约束构造函数和类类型的实例端之间的关系。

```ts
class BeeKeeper {
  hasMask: boolean = true;
}

class ZooKeeper {
  nametag: string = "Mikle";
}

class Animal {
  numLegs: number = 4;
}

class Bee extends Animal {
  keeper: BeeKeeper = new BeeKeeper();
}

class Lion extends Animal {
  keeper: ZooKeeper = new ZooKeeper();
}

function createInstance<A extends Animal>(c: new () => A): A {
  return new c();
}

createInstance(Lion).keeper.nametag;
createInstance(Bee).keeper.hasMask;
```

此模式用于支持[mixins](https://www.typescriptlang.org/docs/handbook/mixins.html)设计模式。

## Keyof Type Operator

###  The keyof type operator

keyof操作符接受对象类型并产生其键的字符串或数字字面值联合

```ts
type Point = { x: number; y: number };
type P = keyof Point;
```

如果该类型具有string或number索引签名，则keyof将返回这些类型

```ts
type Arrayish = { [n: number]: unknown };
type A = keyof Arrayish;

type Mapish = { [k: string]: boolean };
type M = keyof Mapish;
```

注意，在这个例子中，M是`string|number`，这是因为JavaScript对象键总是强制为一个字符串，所以`obj[0]`总是与`obj["0"]`相同。

Keyof类型在与映射类型结合使用时特别有用，我们将在后面进一步了解映射类型。

## Typeof Type Operator

### Typeof Type Operator

JavaScript已经有了可以在表达式上下文中使用的类型操作符

```ts
// Prints "string"
console.log(typeof "Hello world");
```

TypeScript添加了typeof操作符，你可以在类型上下文中使用它来引用变量或属性的类型

```ts
let s = "hello";
let n: typeof s;
```

这对于基本类型不是很有用，但是结合其他类型操作符，您可以使用typeof方便地表示许多模式。例如，让我们从预定义的类型`ReturnType<T>`开始。它接受一个函数类型并产生它的返回类型

```ts
type Predicate = (x: unknown) => boolean;
type K = ReturnType<Predicate>;
```

如果我们试图在函数名上使用ReturnType，就会看到一个有指导意义的错误

```ts
function f() {
  return { x: 10, y: 3 };
}
type P = ReturnType<f>;
//'f' refers to a value, but is being used as a type here. Did you mean 'typeof f'?
```

记住，值和类型不是一回事。要引用值f的类型，我们使用typeof

```ts
function f() {
  return { x: 10, y: 3 };
}
type P = ReturnType<typeof f>;
```

#### Limitations

TypeScript有意限制了你可以使用typeof的表达式类型。

具体来说，只有在标识符(即变量名)或其属性上使用typeof才合法。这有助于避免编写您认为正在执行但实际上没有执行的代码这种令人困惑的陷阱

```ts
// Meant to use = ReturnType<typeof msgbox>
let shouldContinue: typeof msgbox("Are you sure you want to continue?");
//',' expected.
```

## Indexed Access Types 索引访问类型

我们可以使用索引访问类型来查找另一种类型上的特定属性

```ts
type Person = { age: number; name: string; alive: boolean };
type Age = Person["age"];
```

索引类型本身就是一种类型，因此我们可以完全使用union、keyof或其他类型

```ts
type I1 = Person["age" | "name"];
     
//type I1 = string | number

type I2 = Person[keyof Person];
     
//type I2 = string | number | boolean

type AliveOrName = "alive" | "name";
type I3 = Person[AliveOrName];
//type I3 = string | boolean
```

如果试图索引一个不存在的属性，您甚至会看到一个错误

```ts
type I1 = Person["alve"];
//Property 'alve' does not exist on type 'Person'
```

使用任意类型进行索引的另一个例子是使用number获取数组元素的类型。我们可以将它与typeof结合，方便地捕获数组字面量的元素类型

```ts
const MyArray = [
  { name: "Alice", age: 15 },
  { name: "Bob", age: 23 },
  { name: "Eve", age: 38 },
];

type Person = typeof MyArray[number];
       
// type Person = {
//     name: string;
//     age: number;
// }
type Age = typeof MyArray[number]["age"];
     
// type Age = number
// Or
type Age2 = Person["age"];
// type Age2 = number
```

您只能在索引时使用类型，这意味着您不能使用const来进行变量引用

```ts
const key = "age";
type Age = Person[key];
// Type 'any' cannot be used as an index type.
// 'key' refers to a value, but is being used as a type here. Did you mean 'typeof key'?
```

但是，您可以使用类型别名来进行类似的重构

```ts
type key = "age";
type Age = Person[key];
```

## Conditional Types 条件类型

在大多数有用的程序的核心，我们必须根据输入作出决定。JavaScript程序也没有什么不同，但是由于值可以很容易地进行内省，所以这些决策也是基于输入的类型。条件类型有助于描述输入和输出类型之间的关系。

```ts
interface Animal {
  live(): void;
}
interface Dog extends Animal {
  woof(): void;
}

type Example1 = Dog extends Animal ? number : string;
        
// type Example1 = number

type Example2 = RegExp extends Animal ? number : string;
        
// type Example2 = string
```

条件类型的形式有点像条件表达式(`condition ? trueExpression : falseExpression`)

```ts
SomeType extends OtherType ? TrueType : FalseType;
```

当extends函数左边的类型可分配给右边的类型时，您将获得第一个分支(真正的分支)中的类型;否则，您将在后一个分支(假分支)中获得类型。

从上面的例子来看，条件类型可能不会马上有用——我们可以告诉自己Dog是扩展了Animal，选择了数字还是字符串!但是条件类型的力量来自于将它们与泛型一起使用。

例如，让我们使用下面的createLabel函数

```ts
interface IdLabel {
  id: number /* some fields */;
}
interface NameLabel {
  name: string /* other fields */;
}

function createLabel(id: number): IdLabel;
function createLabel(name: string): NameLabel;
function createLabel(nameOrId: string | number): IdLabel | NameLabel;
function createLabel(nameOrId: string | number): IdLabel | NameLabel {
  throw "unimplemented";
}
```

createLabel的这些重载描述了一个JavaScript函数，该函数根据其输入类型进行选择。注意以下几点
1. 如果一个库必须在其API中反复做出相同的选择，这就变得很麻烦。
2. 我们必须创建三个重载:当我们确定类型时，一个用于每种情况(一个用于字符串，一个用于数字)，另一个用于最一般的情况(使用string|number)。对于createLabel可以处理的每一种新类型，重载的数量都会呈指数增长。

相反，我们可以将该逻辑编码为条件类型

```ts
type NameOrId<T extends number | string> = T extends number
  ? IdLabel
  : NameLabel;
```

然后，我们可以使用该条件类型将重载简化为一个没有重载的单一函数。

```ts
function createLabel<T extends number | string>(idOrName: T): NameOrId<T> {
  throw "unimplemented";
}

let a = createLabel("typescript");
   
// let a: NameLabel

let b = createLabel(2.8);
   
// let b: IdLabel

let c = createLabel(Math.random() ? "hello" : 42);
// let c: NameLabel | IdLabel
```

#### Conditional Type Constraints 条件类型约束

条件类型中的检查通常会为我们提供一些新的信息。就像使用类型保护收缩可以给我们提供更具体的类型一样，条件类型的真正分支将通过我们所检查的类型进一步约束泛型。

例如，让我们需要：

```ts
type MessageOf<T> = T["message"];
// Type '"message"' cannot be used to index type 'T'.   
```

在这个例子中，TypeScript会出错是因为T不知道它有一个名为message的属性。我们可以约束T, TypeScript就不会再抱怨了

```ts
type MessageOf<T extends { message: unknown }> = T["message"];

interface Email {
  message: string;
}

interface Dog {
  bark(): void;
}

type EmailMessageContents = MessageOf<Email>;
              
// type EmailMessageContents = string
```

但是，如果我们想要MessageOf接受任何类型，如果消息属性不可用，则默认为never ?我们可以通过将约束移出并引入条件类型来实现这一点

```ts
type MessageOf<T> = T extends { message: unknown } ? T["message"] : never;

interface Email {
  message: string;
}

interface Dog {
  bark(): void;
}

type EmailMessageContents = MessageOf<Email>;
              
// type EmailMessageContents = string

type DogMessageContents = MessageOf<Dog>;
             
// type DogMessageContents = never
```

在真正的分支中，TypeScript知道T将有一个message属性。

作为另一个例子，我们还可以编写一个名为Flatten的类型，它将数组类型扁平化为它们的元素类型，而不去管它们

```ts
type Flatten<T> = T extends any[] ? T[number] : T;

// Extracts out the element type.
type Str = Flatten<string[]>;
     
// type Str = string

// Leaves the type alone.
type Num = Flatten<number>;
     
// type Num = number
```

当Flatten被给定一个数组类型时，它使用带number的索引访问来获取string[]的元素类型。否则，它只返回给定的类型。

#### 条件类型中的推断

我们只是发现自己使用条件类型来应用约束，然后提取类型。这是一种非常常见的操作，条件类型使其变得更容易。

条件类型为我们提供了一种方法，可以使用infer关键字从真实分支中比较的类型中推断。例如，我们可以推断Flatten中的元素类型，而不是使用索引访问类型手动提取它

```ts
type Flatten<Type> = Type extends Array<infer Item> ? Item : Type;
```

这里，我们使用infer关键字以声明的方式引入一个名为Item的新泛型类型变量，而不是指定如何在真正的分支中检索T的元素类型。这使我们不必去思考如何挖掘和探索我们感兴趣的类型的结构。

我们可以使用infer关键字编写一些有用的助手类型别名。例如，对于简单的情况，我们可以从函数类型中提取返回类型

```ts
type GetReturnType<Type> = Type extends (...args: never[]) => infer Return
  ? Return
  : never;

type Num = GetReturnType<() => number>;
     
// type Num = number

type Str = GetReturnType<(x: string) => string>;
     
// type Str = string

type Bools = GetReturnType<(a: boolean, b: boolean) => boolean[]>;
      
// type Bools = boolean[]
```

当从具有多个调用签名的类型(例如重载函数的类型)进行推断时，将从最后一个签名(这可能是最宽松的捕获所有情况)进行推断。不能基于参数类型列表执行重载解析。

```ts
declare function stringOrNum(x: string): number;
declare function stringOrNum(x: number): string;
declare function stringOrNum(x: string | number): string | number;

type T1 = ReturnType<typeof stringOrNum>;
     
// type T1 = string | number
```

### Distributive Conditional Types 分配条件类型

当条件类型作用于泛型类型时，当给定联合类型时，条件类型就变成了分布式的。举个例子

```ts
 type ToArray<Type> = Type extends any ? Type[] : never;
```

如果我们将联合类型插入到ToArray中，那么条件类型将应用于该联合的每个成员。

```ts
type ToArray<Type> = Type extends any ? Type[] : never;

type StrArrOrNumArr = ToArray<string | number>;
           
// type StrArrOrNumArr = string[] | number[]
```

这里发生的是StrOrNumArray分布在

```ts
string | number;
```

并将联合的每个成员类型映射到有效的

```ts
ToArray<string> | ToArray<number>;
```

给我们留下来

```ts
string[] | number[];
```

通常，分布式是期望的行为。为了避免这种行为，可以用方括号包围extends关键字的每一边。

```ts
type ToArrayNonDist<Type> = [Type] extends [any] ? Type[] : never;

// 'StrOrNumArr' is no longer a union.
type StrOrNumArr = ToArrayNonDist<string | number>;
         
// type StrOrNumArr = (string | number)[]
```

## Mapped Types 映射类型

当你不想重复的时候，有时候一种类型需要基于另一种类型。

映射类型建立在索引签名的语法之上，索引签名用于声明尚未提前声明的属性类型

```ts
type OnlyBoolsAndHorses = {
  [key: string]: boolean | Horse;
};

const conforms: OnlyBoolsAndHorses = {
  del: true,
  rodney: false,
};
```

映射类型是一种泛型类型，它使用PropertyKeys联合(通常通过keyof创建)来遍历键以创建类型

```ts
type OptionsFlags<Type> = {
  [Property in keyof Type]: boolean;
};
```

在本例中，OptionsFlags将从type类型中获取所有属性，并将其值更改为布尔值。

```ts
type FeatureFlags = {
  darkMode: () => void;
  newUserProfile: () => void;
};

type FeatureOptions = OptionsFlags<FeatureFlags>;
           
// type FeatureOptions = {
//     darkMode: boolean;
//     newUserProfile: boolean;
// }
```

#### Mapping Modifiers 映射修饰符

在映射期间可以应用两个额外的修饰符:readonly和?它们分别影响可变性和可选性。

你可以用-或+作为前缀来删除或添加这些修饰语。如果不添加前缀，则假定为+。

```ts
// Removes 'readonly' attributes from a type's properties
type CreateMutable<Type> = {
  -readonly [Property in keyof Type]: Type[Property];
};

type LockedAccount = {
  readonly id: string;
  readonly name: string;
};

type UnlockedAccount = CreateMutable<LockedAccount>;
           
// type UnlockedAccount = {
//     id: string;
//     name: string;
// }
```

```ts
// Removes 'optional' attributes from a type's properties
type Concrete<Type> = {
  [Property in keyof Type]-?: Type[Property];
};

type MaybeUser = {
  id: string;
  name?: string;
  age?: number;
};

type User = Concrete<MaybeUser>;
      
// type User = {
//     id: string;
//     name: string;
//     age: number;
// }
```

### Key Remapping via `as` 通过as键重新映射

在TypeScript 4.1及以后的版本中，你可以用as子句来重新映射映射类型中的键

```ts
type MappedTypeWithNewProperties<Type> = {
    [Properties in keyof Type as NewKeyType]: Type[Properties]
}
```

您可以利用[模板文字类型](https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html)等特性从以前的属性名称创建新的属性名称

```ts
type Getters<Type> = {
    [Property in keyof Type as `get${Capitalize<string & Property>}`]: () => Type[Property]
};

interface Person {
    name: string;
    age: number;
    location: string;
}

type LazyPerson = Getters<Person>;
         
// type LazyPerson = {
//     getName: () => string;
//     getAge: () => number;
//     getLocation: () => string;
// }
```

您可以通过never通过条件类型生成密钥来过滤密钥

```ts
// Remove the 'kind' property
type RemoveKindField<Type> = {
    [Property in keyof Type as Exclude<Property, "kind">]: Type[Property]
};

interface Circle {
    kind: "circle";
    radius: number;
}

type KindlessCircle = RemoveKindField<Circle>;
           
// type KindlessCircle = {
//     radius: number;
// }
```

#### Further Exploration

映射类型与此类型操作部分中的其他特性很好地配合，例如，这里是[使用条件类型的映射类型](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html)，该条件类型根据对象是否将属性pii设置为字面量true返回true或false

```ts
type ExtractPII<Type> = {
  [Property in keyof Type]: Type[Property] extends { pii: true } ? true : false;
};

type DBFields = {
  id: { format: "incrementing" };
  name: { type: string; pii: true };
};

type ObjectsNeedingGDPRDeletion = ExtractPII<DBFields>;
                 
type ObjectsNeedingGDPRDeletion = {
    id: false;
    name: true;
}
```

## Template Literal Types 模板文字类型

模板文字类型建立在[字符串文字类型](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#literal-types)之上，并能够通过联合扩展成许多字符串。

它们与JavaScript中的模板字面值字符串具有相同的语法，但用于类型位置。当与具体文字类型一起使用时，模板文字通过连接内容产生一个新的字符串文字类型。

```ts
type World = "world";

type Greeting = `hello ${World}`;
        
// type Greeting = "hello world"
```

当在插入位置使用联合时，该类型是每个联合成员可以表示的所有可能字符串字面值的集合

```ts
type EmailLocaleIDs = "welcome_email" | "email_heading";
type FooterLocaleIDs = "footer_title" | "footer_sendoff";

type AllLocaleIDs = `${EmailLocaleIDs | FooterLocaleIDs}_id`;
          
// type AllLocaleIDs = "welcome_email_id" | "email_heading_id" | "footer_title_id" | "footer_sendoff_id"
```

我们通常建议人们对大型字符串联合使用提前生成，但这在较小的情况下很有用。

#### String Unions in Types

模板字面值的强大之处在于基于类型内的现有字符串定义新字符串。

例如，JavaScript中的一个常见模式是基于对象当前拥有的字段扩展对象。我们将为函数提供类型定义，它增加了对on函数的支持，让你知道什么时候值发生了变化

```ts
const person = makeWatchedObject({
  firstName: "Saoirse",
  lastName: "Ronan",
  age: 26,
});

person.on("firstNameChanged", (newValue) => {
  console.log(`firstName was changed to ${newValue}!`);
});
```

注意，在监听事件“firstNameChanged”，而不仅仅是“firstName”时，模板字面量提供了一种在类型系统中处理这种字符串操作的方法

```ts
type PropEventSource<Type> = {
    on(eventName: `${string & keyof Type}Changed`, callback: (newValue: any) => void): void;
};

/// Create a "watched object" with an 'on' method
/// so that you can watch for changes to properties.
declare function makeWatchedObject<Type>(obj: Type): Type & PropEventSource<Type>;

```

有了它，我们就可以构建一些在给定错误属性时出错的东西

```ts
const person = makeWatchedObject({
  firstName: "Saoirse",
  lastName: "Ronan",
  age: 26
});

person.on("firstNameChanged", () => {});

// It's typo-resistent
person.on("firstName", () => {});
// Argument of type '"firstName"' is not assignable to parameter of type '"firstNameChanged" | "lastNameChanged" | "ageChanged"'.

person.on("frstNameChanged", () => {});
// Argument of type '"frstNameChanged"' is not assignable to parameter of type '"firstNameChanged" | "lastNameChanged" | "ageChanged"'.
```

#### Inference with Template Literals

请注意，最后一个示例没有重用原始值的类型。回调函数使用any。模板文字类型可以从替换位置推断。

我们可以使最后一个示例成为泛型，从eventName字符串的部分推断出关联的属性。

```ts
type PropEventSource<Type> = {
    on<Key extends string & keyof Type>
        (eventName: `${Key}Changed`, callback: (newValue: Type[Key]) => void ): void;
};

declare function makeWatchedObject<Type>(obj: Type): Type & PropEventSource<Type>;

const person = makeWatchedObject({
  firstName: "Saoirse",
  lastName: "Ronan",
  age: 26
});

person.on("firstNameChanged", newName => {
                                
// (parameter) newName: string
    console.log(`new name is ${newName.toUpperCase()}`);
});

person.on("ageChanged", newAge => {
                          
// (parameter) newAge: number
    if (newAge < 0) {
        console.warn("warning! negative age");
    }
})
```

这里我们把它变成了一个通用的方法。

当用户调用字符串“firstNameChanged”时，TypeScript会尝试为Key推断出正确的类型。要做到这一点，它将Key与“Changed”之前的内容进行匹配，并推断出字符串“firstName”。一旦TypeScript发现了这一点，on方法就可以获取原始对象的firstName类型，在本例中是字符串。类似地，当使用"ageChanged"调用时，TypeScript会找到age属性的类型，即number。

### Intrinsic String Manipulation Types

为了帮助字符串操作，TypeScript包含了一组可用于字符串操作的类型。这些类型是编译器为提高性能而内置的，在.d。TypeScript中包含的ts文件。

`Uppercase<StringType>`

将字符串中的每个字符转换为大写版本。

例如

```ts
type Greeting = "Hello, world"
type ShoutyGreeting = Uppercase<Greeting>
           
// type ShoutyGreeting = "HELLO, WORLD"

type ASCIICacheKey<Str extends string> = `ID-${Uppercase<Str>}`
type MainID = ASCIICacheKey<"my_app">
       
// type MainID = "ID-MY_APP"
```

`Lowercase<StringType>`

将字符串中的每个字符转换为等价的小写字符。

```ts
type Greeting = "Hello, world"
type QuietGreeting = Lowercase<Greeting>
          
type QuietGreeting = "hello, world"

type ASCIICacheKey<Str extends string> = `id-${Lowercase<Str>}`
type MainID = ASCIICacheKey<"MY_APP">
       
type MainID = "id-my_app"
```


`Capitalize<StringType>`

将字符串中的第一个字符转换为大写等价字符。

```ts
type LowercaseGreeting = "hello, world";
type Greeting = Capitalize<LowercaseGreeting>;
        
// type Greeting = "Hello, world"
```
`Uncapitalize<StringType>`

将字符串中的第一个字符转换为等价的小写字符。

```ts
type UppercaseGreeting = "HELLO WORLD";
type UncomfortableGreeting = Uncapitalize<UppercaseGreeting>;
              
// type UncomfortableGreeting = "hELLO WORLD"
```

end