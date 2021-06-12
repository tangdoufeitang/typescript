## Object Types

在JavaScript中，分组和传递数据的基本方式是通过对象。在TypeScript中，我们通过`object types`来表示它们。

正如我们所见，他们可以是匿名的

```ts
function greet(person: { name: string; age: number }) {
  return "Hello " + person.name;
}
```

或者可以使用接口来命名它们

```ts
interface Person {
  name: string;
  age: number;
}

function greet(person: Person) {
  return "Hello " + person.name;
}
```

或者类型别名。

```ts
type Person = {
  name: string;
  age: number;
};

function greet(person: Person) {
  return "Hello " + person.name;
}
```

在上面的三个例子中，我们已经编写了包含属name(必须是string)和age(必须是number)的函数。

#### Property Modifiers 属性修饰词

对象类型中的每个属性都可以指定以下几点:类型、属性是否可选以及属性是否可以写入。

##### Optional Properties

很多时候，我们会发现自己在处理可能具有属性集的对象。在这种情况下，我们可以通过在属性名的末尾添加问号(?)来将这些属性标记为可选的。

```ts
interface PaintOptions {
  shape: Shape;
  xPos?: number;
  yPos?: number;
}

function paintShape(opts: PaintOptions) {
  // ...
}

const shape = getShape();
paintShape({ shape });
paintShape({ shape, xPos: 100 });
paintShape({ shape, yPos: 100 });
paintShape({ shape, xPos: 100, yPos: 100 });
```

在本例中，xPos和yPos都被认为是可选的。我们可以选择提供它们中的任何一个，这样上面对paintShape的每次调用都是有效的。所有可选性实际上是说，如果属性设置了，它最好有一个特定的类型。

```ts
interface PaintOptions {
  shape: Shape;
  xPos?: number;
  yPos?: number;
}

function paintShape(opts: PaintOptions) {
  // ...
}

const shape = getShape();
paintShape({ shape });
paintShape({ shape, xPos: 100 });
```

我们也可以从这些属性中读取——但是当我们在strictNullChecks下读取时，TypeScript会告诉我们它们可能是未定义的。

```ts
function paintShape(opts: PaintOptions) {
  let xPos = opts.xPos;
                   
(property) PaintOptions.xPos?: number | undefined
  let yPos = opts.yPos;
                   
(property) PaintOptions.yPos?: number | undefined
  // ...
}
```

在JavaScript中，即使属性从未被设置，我们仍然可以访问它-它将给我们的值未定义。我们可以专门处理undefined。

```ts
function paintShape(opts: PaintOptions) {
  let xPos = opts.xPos;
                   
(property) PaintOptions.xPos?: number | undefined
  let yPos = opts.yPos;
                   
(property) PaintOptions.yPos?: number | undefined
  // ...
}
```

在JavaScript中，即使属性从未被设置，我们仍然可以访问它-它将给我们的值未定义。我们可以专门处理undefined。

```ts
function paintShape(opts: PaintOptions) {
  let xPos = opts.xPos === undefined ? 0 : opts.xPos;
       
let xPos: number
  let yPos = opts.yPos === undefined ? 0 : opts.yPos;
       
let yPos: number
  // ...
}
```

请注意，这种为未指定值设置默认值的模式非常常见，以至于JavaScript有语法支持它。

```ts
function paintShape({ shape, xPos = 0, yPos = 0 }: PaintOptions) {
  console.log("x coordinate at", xPos);
                                  
var xPos: number
  console.log("y coordinate at", yPos);
                                  
var yPos: number
  // ...
}
```

这里我们为paintShape参数使用了解构模式，并为xPos和yPos提供了默认值。现在xPos和yPos都明确地出现在paintShape的主体中，但是对于任何paintShape的调用者来说都是可选的

> 请注意，目前还没有办法在解构模式中放置类型注释。这是因为下面的语法在JavaScript中已经有了不同的含义。

```ts
function draw({ shape: Shape, xPos: number = 100 /*...*/ }) {
  render(shape);
Cannot find name 'shape'. Did you mean 'Shape'?
  render(xPos);
Cannot find name 'xPos'.
}
```

在对象解构模式中，shape: shape表示获取属性shape并将其作为一个名为shape的变量局部重新定义。同样，xPos: number创建一个名为number的变量，其值基于参数xPos。

##### readonly Properties

TypeScript也可以将属性标记为只读。虽然它不会在运行时改变任何行为，但在类型检查期间不能写入标记为readonly的属性。

```ts
interface SomeType {
  readonly prop: string;
}

function doSomething(obj: SomeType) {
  // We can read from 'obj.prop'.
  console.log(`prop has the value '${obj.prop}'.`);

  // But we can't re-assign it.
  obj.prop = "hello";
Cannot assign to 'prop' because it is a read-only property.
}
```

使用readonly修饰符并不一定意味着一个值是完全不可变的——或者换句话说，它的内部内容不能被改变。它只是意味着属性本身不能被重写。

```ts
interface Home {
  readonly resident: { name: string; age: number };
}

function visitForBirthday(home: Home) {
  // We can read and update properties from 'home.resident'.
  console.log(`Happy birthday ${home.resident.name}!`);
  home.resident.age++;
}

function evict(home: Home) {
  // But we can't write to the 'resident' property itself on a 'Home'.
  home.resident = {
Cannot assign to 'resident' because it is a read-only property.
    name: "Victor the Evictor",
    age: 42,
  };
}
```

管理只读所暗示的期望是很重要的。在开发过程中，在TypeScript中指示一个对象应该如何使用是很有用的。当检查两种类型是否兼容时，TypeScript不会考虑这两种类型的属性是否为只读，所以只读属性也可以通过别名改变。

```ts
interface Person {
  name: string;
  age: number;
}

interface ReadonlyPerson {
  readonly name: string;
  readonly age: number;
}

let writablePerson: Person = {
  name: "Person McPersonface",
  age: 42,
};

// works
let readonlyPerson: ReadonlyPerson = writablePerson;

console.log(readonlyPerson.age); // prints '42'
writablePerson.age++;
console.log(readonlyPerson.age); // prints '43'
```

##### Index Signatures

有时您不知道类型属性的所有名称，但您知道值的形状。

在这些情况下，您可以使用索引签名来描述可能值的类型

```ts
interface StringArray {
  [index: number]: string;
}

const myArray: StringArray = getStringArray();
const secondItem = myArray[1];
          
const secondItem: string
```
上面，我们有一个StringArray接口，它有一个索引签名。这个索引签名表明，当一个StringArray用数字进行索引时，它将返回一个字符串。

索引签名属性类型必须是字符串或数字。

可以支持这两种类型的索引器，但从数字索引器返回的类型必须是从字符串索引器返回的类型的子类型。这是因为当索引与一个“数字”，JavaScript将实际转换为一个“字符串”之前，索引到一个对象。这意味着用“100”(一个数字)索引和用“100”(一个字符串)索引是一样的，所以两者需要一致。

```ts
interface Animal {
  name: string;
}

interface Dog extends Animal {
  breed: string;
}

// Error: indexing with a numeric string might get you a completely separate type of Animal!
interface NotOkay {
  [x: number]: Animal;
// Numeric index type 'Animal' is not assignable to string index type 'Dog'.
  [x: string]: Dog;
}
```

虽然字符串索引签名是描述字典模式的一种强大方式，但它们还强制要求所有属性与其返回类型匹配。这是因为字符串索引声明了`obj.Property`也可用`obj[" Property "]`。在下面的例子中，name的类型与字符串索引的类型不匹配，类型检查器会给出一个错误

```ts
interface NumberDictionary {
  [index: string]: number;

  length: number; // ok
  name: string;
Property 'name' of type 'string' is not assignable to string index type 'number'.
}
```

但是，如果索引签名是属性类型的联合，则可以接受不同类型的属性

```ts
interface NumberOrStringDictionary {
  [index: string]: number | string;
  length: number; // ok, length is a number
  name: string; // ok, name is a string
}
```

最后，可以将索引签名设置为只读，以防止对其索引进行赋值

```ts
interface ReadonlyStringArray {
  readonly [index: number]: string;
}

let myArray: ReadonlyStringArray = getReadOnlyStringArray();
myArray[2] = "Mallory";
// Index signature in type 'ReadonlyStringArray' only permits reading.
```

你不能设置myArray[2]，因为索引签名是只读的。

#### Extending Types

有可能是其他类型的更具体版本的类型是很常见的。例如，我们可能有一个BasicAddress类型，它描述了在美国发送信件和包裹所需的字段

```ts
interface BasicAddress {
  name?: string;
  street: string;
  city: string;
  country: string;
  postalCode: string;
}
```

在某些情况下，这就足够了，但如果一个地址上的建筑有多个单元，地址通常有一个与之相关的单元号。然后我们可以描述AddressWithUnit。

```ts
interface AddressWithUnit {
  name?: string;
  unit: string;
  street: string;
  city: string;
  country: string;
  postalCode: string;
}
```

这就完成了这项工作，但缺点是，当更改纯粹是附加的时候，我们必须重复BasicAddress中的所有其他字段。相反，我们可以扩展原始的BasicAddress类型，只添加对AddressWithUnit唯一的新字段。

```ts
interface BasicAddress {
  name?: string;
  street: string;
  city: string;
  country: string;
  postalCode: string;
}

interface AddressWithUnit extends BasicAddress {
  unit: string;
}
```

接口上的extends关键字允许我们有效地从其他命名类型复制成员，并添加我们想要的任何新成员。这有助于减少我们必须编写的类型声明样板文件的数量，并告知意图相同属性的几个不同声明可能是相关的。例如，AddressWithUnit不需要重复street属性，因为street来源于BasicAddress，所以读取器知道这两种类型在某种程度上是相关的。

接口还可以从多种类型进行扩展。


```ts
interface Colorful {
  color: string;
}

interface Circle {
  radius: number;
}

interface ColorfulCircle extends Colorful, Circle {}

const cc: ColorfulCircle = {
  color: "red",
  radius: 42,
};
```

#### Intersection Types

接口允许我们通过扩展其他类型来构建新的类型。TypeScript提供了另一种构造称为交集类型，主要用于组合现有的对象类型。

交集类型是使用&运算符定义的。

```ts
interface Colorful {
  color: string;
}
interface Circle {
  radius: number;
}

type ColorfulCircle = Colorful & Circle;
```

在这里，我们已经相交了彩色和圆圈，以产生一个拥有丰富多彩和圈子的所有成员的新类型。

```ts
function draw(circle: Colorful & Circle) {
  console.log(`Color was ${circle.color}`);
  console.log(`Radius was ${circle.radius}`);
}

// okay
draw({ color: "blue", radius: 42 });

// oops
draw({ color: "red", raidus: 42 });
// Argument of type '{ color: string; raidus: number; }' is not assignable to parameter of type 'Colorful & Circle'.
//   Object literal may only specify known properties, but 'raidus' does not exist in type 'Colorful & Circle'. Did you mean to write 'radius'?
```

#### Interfaces vs. Intersections

我们刚刚讨论了两种组合相似但实际上有细微差别的类型的方法。通过接口，我们可以使用extends子句从其他类型中进行扩展，并且我们能够对交集做类似的事情，并使用类型别名来命名结果。两者之间的主要区别在于如何处理冲突，而这种区别通常是您在接口和交集类型的类型别名之间选择其中之一的主要原因。

```ts
interface Box {
  contents: any;
}
```

现在，contents属性的类型是any，这是可行的，但是可能会导致意外。

我们可以使用unknown，但这意味着在我们已经知道内容类型的情况下，我们需要做预防性检查，或者使用容易出错的类型断言。

```ts
interface Box {
  contents: unknown;
}

let x: Box = {
  contents: "hello world",
};

// we could check 'x.contents'
if (typeof x.contents === "string") {
  console.log(x.contents.toLowerCase());
}

// or we could use a type assertion
console.log((x.contents as string).toLowerCase());
```

一种类型安全的方法是为每种类型的内容构建不同的Box类型。

```ts
interface NumberBox {
  contents: number;
}

interface StringBox {
  contents: string;
}

interface BooleanBox {
  contents: boolean;
}
```

但这意味着我们必须创建不同的函数，或函数的重载来操作这些类型。

```ts
function setContents(box: StringBox, newContents: string): void;
function setContents(box: NumberBox, newContents: number): void;
function setContents(box: BooleanBox, newContents: boolean): void;
function setContents(box: { contents: any }, newContents: any) {
  box.contents = newContents;
}
```

这是一大堆样板文件。此外，我们以后可能需要引入新的类型和重载。这是令人沮丧的，因为我们的框类型和重载实际上都是相同的。

相反，我们可以创建一个泛型Box类型来声明类型参数

```ts
interface Box<Type> {
  contents: Type;
}
```

你可以把它读成A Box of Type，它的内容的类型是Type。稍后，当我们引用Box时，我们必须给出一个类型参数来代替type。

```ts
let box: Box<string>;
```

可以将Box看作是一个真实类型的模板，其中type是一个占位符，它将被其他类型取代。当TypeScript看到`Box<string>`时，它会将`Box<Type>`中的每个Type实例替换为string，并最终使用类似`{contents: string}`的东西。换句话说，`Box<string>`和我们之前的StringBox工作方式相同。

```ts
interface Box<Type> {
  contents: Type;
}
interface StringBox {
  contents: string;
}

let boxA: Box<string> = { contents: "hello" };
boxA.contents;
        
(property) Box<string>.contents: string

let boxB: StringBox = { contents: "world" };
boxB.contents;
        
(property) StringBox.contents: string
```

Box是可重用的，因为Type可以被任何东西替代。这意味着当我们需要一个用于新类型的box时，我们根本不需要声明一个新的box类型(尽管如果我们想这样做，我们当然可以这样做)。

```ts
interface Box<Type> {
  contents: Type;
}

interface Apple {
  // ....
}

// Same as '{ contents: Apple }'.
type AppleBox = Box<Apple>;
```

这也意味着我们可以通过使用泛型函数来完全避免重载。

```ts
function setContents<Type>(box: Box<Type>, newContents: Type) {
  box.contents = newContents;
}
```

值得注意的是，类型别名也可以是泛型的。我们可以定义新的`Box<Type>`接口

```ts
interface Box<Type> {
  contents: Type;
}
```

通过使用类型别名

```ts
type Box<Type> = {
  contents: Type;
};
```

因为类型别名与接口不同，不仅可以描述对象类型，我们还可以使用它们来编写其他类型的泛型帮助器类型。

```ts
type OrNull<Type> = Type | null;

type OneOrMany<Type> = Type | Type[];

type OneOrManyOrNull<Type> = OrNull<OneOrMany<Type>>;
           
type OneOrManyOrNull<Type> = OneOrMany<Type> | null

type OneOrManyOrNullStrings = OneOrManyOrNull<string>;
               
type OneOrManyOrNullStrings = OneOrMany<string> | null
```

稍后我们将回到别名的类型。

##### The Array Type

泛型对象类型通常是某种容器类型，独立于它们所包含的元素类型。数据结构以这种方式工作是非常理想的，这样它们就可以跨不同的数据类型重用。

事实证明，我们一直在使用一个类型，就像在整个手册:阵列类型。每当我们写出`number[]`或`string[]`这样的类型时，这实际上只是`Array<number>`和`Array<string>`的简写。

```ts
function doSomething(value: Array<string>) {
  // ...
}

let myArray: string[] = ["hello", "world"];

// either of these work!
doSomething(myArray);
doSomething(new Array("hello", "world"));
```
与上面的Box类型非常相似，Array本身也是一种泛型类型。

```ts
interface Array<Type> {
  /**
   * Gets or sets the length of the array.
   */
  length: number;

  /**
   * Removes the last element from an array and returns it.
   */
  pop(): Type | undefined;

  /**
   * Appends new elements to an array, and returns the new length of the array.
   */
  push(...items: Type[]): number;

  // ...
}
```

现代JavaScript还提供了其他泛型数据结构，比如`Map<K, V>`， `Set<T>`， `Promise<T>`。所有这些实际上意味着，由于Map、Set和Promise的行为方式，它们可以与任何类型集一起工作。

##### The ReadonlyArray Type

ReadonlyArray是一种特殊类型，用于描述不应更改的数组。

```ts
function doStuff(values: ReadonlyArray<string>) {
  // We can read from 'values'...
  const copy = values.slice();
  console.log(`The first value is ${values[0]}`);

  // ...but we can't mutate 'values'.
  values.push("hello!");
Property 'push' does not exist on type 'readonly string[]'.
}
```

很像属性的只读修饰符，它主要是一个我们可以用于意图的工具。当我们看到返回ReadonlyArrays的函数时 它告诉我们，我们根本不打算改变内容 当我们看到一个使用ReadonlyArray的函数时 它告诉我们，我们可以将任何数组传递给这个函数，而不必担心它会改变它的内容。

与Array不同，我们没有可以使用的ReadonlyArray构造函数。

```ts
new ReadonlyArray("red", "green", "blue");
'ReadonlyArray' only refers to a type, but is being used as a value here.
```

相反，我们可以将常规数组赋给ReadonlyArrays。

```ts
const roArray: ReadonlyArray<string> = ["red", "green", "blue"];
```

就像TypeScript为`Array<Type>`提供了`Type[]`的简写语法一样，它也为`ReadonlyArray<Type>`提供了`readonlytype[]`的简写语法。

```ts
function doStuff(values: readonly string[]) {
  // We can read from 'values'...
  const copy = values.slice();
  console.log(`The first value is ${values[0]}`);

  // ...but we can't mutate 'values'.
  values.push("hello!");
Property 'push' does not exist on type 'readonly string[]'.
}
```

最后要注意的一点是，与readonly属性修饰符不同的是，在常规数组和ReadonlyArrays之间的可分配性不是双向的。

```ts
let x: readonly string[] = [];
let y: string[] = [];

x = y;
y = x;
The type 'readonly string[]' is 'readonly' and cannot be assigned to the mutable type 'string[]'
```

##### Tuple Types

元组类型是另一种数组类型，它确切地知道它包含多少个元素，以及它在特定位置上确切地包含哪些类型

```ts
type StringNumberPair = [string, number];
```

这里，StringNumberPair是字符串和数字的元组类型。和ReadonlyArray一样，它在运行时没有表示，但对TypeScript很重要。对于类型系统，StringNumberPair描述了数组，其0索引包含一个字符串，其1索引包含一个数字。

```ts
function doSomething(pair: [string, number]) {
  const a = pair[0];
       
const a: string
  const b = pair[1];
       
const b: number
  // ...
}

doSomething(["hello", 42]);
```

如果我们尝试索引超过元素的数量，我们将得到一个错误。

```ts
function doSomething(pair: [string, number]) {
  // ...

  const c = pair[2];
Tuple type '[string, number]' of length '2' has no element at index '2'.
}
```

我们也可以使用JavaScript的数组解构来解构元组。

```ts
function doSomething(stringHash: [string, number]) {
  const [inputString, hash] = stringHash;

  console.log(inputString);
                  
const inputString: string

  console.log(hash);
               
const hash: number
}
```

> 元组类型在高度基于约定的api中非常有用，其中每个元素的含义都很明显。当我们解构变量时，这给了我们灵活的命名方式。在上面的例子中，我们可以将元素0和1命名为我们想要的任何名称。然而，由于并非每个用户都对显而易见的事情持有相同的观点，因此可能值得重新考虑是否使用具有描述性属性名称的对象更适合您的API。

除了这些长度检查，像这样的简单元组类型等价于类型是数组的版本，声明特定索引的属性，并声明长度与数字文字类型。

```ts
interface StringNumberPair {
  // specialized properties
  length: 2;
  0: string;
  1: number;

  // Other 'Array<string | number>' members...
  slice(start?: number, end?: number): Array<string | number>;
}
```
您可能感兴趣的另一件事是，元组可以通过写一个问号(?在元素的类型之后)。可选元组元素只能出现在末尾，也会影响长度的类型。

```ts
type Either2dOr3d = [number, number, number?];

function setCoordinate(coord: Either2dOr3d) {
  const [x, y, z] = coord;
              
const z: number | undefined

  console.log(`Provided coordinates had ${coord.length} dimensions`);
                                                  
(property) length: 2 | 3
}
```

元组还可以有rest元素，必须是array/tuple type

```ts
type StringNumberBooleans = [string, number, ...boolean[]];
type StringBooleansNumber = [string, ...boolean[], number];
type BooleansStringNumber = [...boolean[], string, number];
```

- stringnumberboolean描述了一个元组，它的前两个元素分别是字符串和数字，但后面可能有任意数量的布尔值。
- stringboolean number描述了一个元组，该元组的第一个元素是字符串，然后是任意数量的布尔值，并以数字结尾。
- BooleansStringNumber描述了一个元组，它的元素以任意数量的布尔值开始，以字符串和数字结束。

具有rest元素的元组没有设置长度——它只有一组位于不同位置的已知元素。

```ts
const a: StringNumberBooleans = ["hello", 1];
const b: StringNumberBooleans = ["beautiful", 2, true];
const c: StringNumberBooleans = ["world", 3, true, false, true, false, true];
```

为什么可选元素和rest元素可能有用?它允许TypeScript将元组与参数列表相对应。元组类型可以在rest参数和实参中使用，以便如下所示

```ts
function readButtonInput(...args: [string, number, ...boolean[]]) {
  const [name, version, ...input] = args;
  // ...
}
```

基本上相当于

```ts
function readButtonInput(name: string, version: number, ...input: boolean[]) {
  // ...
}
```

当您想要使用一个带有rest参数的可变数量的参数，并且需要最少数量的元素，但又不想引入中间变量时，这是很方便的。

##### readonly Tuple Types

关于元组类型的最后一个注意事项——元组类型具有只读变量，可以通过在其前面添加一个只读修饰符来指定——就像数组速记语法一样。

```ts
function doSomething(pair: readonly [string, number]) {
  // ...
}
```

正如你所料，在TypeScript中不允许写入只读元组的任何属性。

```ts
function doSomething(pair: readonly [string, number]) {
  pair[0] = "hello!";
Cannot assign to '0' because it is a read-only property.
}
```

在大多数代码中，元组倾向于创建并不进行修改，因此在可能的情况下将类型注释为只读元组是一个很好的默认做法。考虑到带有const断言的数组字面值将被推断为只读元组类型，这一点也很重要。

```ts
let point = [3, 4] as const;

function distanceFromOrigin([x, y]: [number, number]) {
  return Math.sqrt(x ** 2 + y ** 2);
}

distanceFromOrigin(point);
Argument of type 'readonly [3, 4]' is not assignable to parameter of type '[number, number]'.
  The type 'readonly [3, 4]' is 'readonly' and cannot be assigned to the mutable type '[number, number]'.
```

在这里，distanceFromOrigin从不修改它的元素，但期望一个可变元组。因为point s的类型被推断为readonly[3,4]，它不会与[number, number]兼容，因为该类型不能保证point s的元素不会被改变。

end