## Narrowing

想象一下，我们有一个名为padleft的函数。

```ts
function padLeft(padding: number | string, input: string): string {
  throw new Error("Not implemented yet!");
}
```

如果padding是一个数字，它会将其视为我们想要添加到输入的空格的数量。如果padding是一个字符串，它应该在input前面加上padding。让我们尝试实现当padLeft被传递一个数字填充时的逻辑。

```ts
function padLeft(padding: number | string, input: string) {
  return new Array(padding + 1).join(" ") + input;
Operator '+' cannot be applied to types 'string | number' and 'number'.
}
```

我们在填充+ 1上得到一个错误。TypeScript警告我们，在`number|string`中添加一个数字可能不会得到我们想要的结果，这是正确的。换句话说，我们没有明确地首先检查填充是否是一个数字，也没有处理它是一个字符串的情况，所以让我们这样做。

```ts
function padLeft(padding: number | string, input: string) {
  if (typeof padding === "number") {
    return new Array(padding + 1).join(" ") + input;
  }
  return padding + input;
}
```

如果这看起来像无趣的JavaScript代码，这就是问题的关键。除了我们放置的注释之外，这个TypeScript代码看起来像JavaScript。这个想法是TypeScript的类型系统的目的是使它尽可能容易地编写典型的JavaScript代码，而不是向后弯曲以获得类型安全。

虽然看起来不太像，但实际上这里有很多内幕。就像TypeScript使用静态类型分析运行时值一样,它将类型分析覆盖在JavaScript的运行时控制流结构上，如`if/else`三元运算符、循环、真实性检查等都可能影响这些类型

在if检查中，TypeScript看到`typeof padding === "number"`，并将其理解为一种称为类型保护的特殊代码形式。TypeScript遵循我们的程序可以采用的可能的执行路径来分析给定位置上最特定类型的值。它关注这些特殊的检查(称为类型保护)和赋值，将类型细化为比声明的更特定的类型的过程称为收缩。在许多编辑器中，我们可以在这些类型变化时观察它们，在我们的示例中也将这样做。

```ts
function padLeft(padding: number | string, input: string) {
  if (typeof padding === "number") {
    return new Array(padding + 1).join(" ") + input;
                       
(parameter) padding: number
  }
  return padding + input;
           
(parameter) padding: string
}
```

有几个不同的构造类型术语理解缩小

### `typeof` type guards

正如我们所看到的，JavaScript支持类型运算符，它可以提供关于运行时值类型的非常基本的信息。TypeScript期望它返回一组特定的字符串

- "string"
- "number"
- "bigint"
- "boolean"
- "symbol"
- "undefined"
- "object"
- "function"

就像我们在padLeft中看到的那样，这个操作符经常出现在许多JavaScript库中，TypeScript可以理解它来缩小不同分支中的类型。在TypeScript中，对typeof返回的值进行检查是一种类型保护。因为TypeScript对typeof如何对不同的值进行操作进行了编码，所以它知道它在JavaScript中的一些怪癖。例如，请注意，在上面的列表中，typeof不返回字符串null。看看下面的例子

```ts
function printAll(strs: string | string[] | null) {
  if (typeof strs === "object") {
    for (const s of strs) {
Object is possibly 'null'.
      console.log(s);
    }
  } else if (typeof strs === "string") {
    console.log(strs);
  } else {
    // do nothing
  }
}
```

在printAll函数中,我们试着检查STRS是否是一个对象，看看它是否是一个数组类型.但在JavaScript中，typeof null实际上是“object”!这是历史上的不幸事件之一。

有足够经验的用户可能不会感到惊讶，但并不是每个人都在JavaScript中遇到过这种情况 TypeScript让我们知道strs只被缩小到`string[] | null`，而不是`string[]`

#### Truthiness narrowing

你可能在字典里找不到“真实”这个词，但你会在JavaScript中听到它。

在JavaScript中，我们可以在条件语句、&&s、||s、if语句和布尔否定(!)等中使用任何表达式。例如，if语句并不期望它们的条件总是有布尔类型。

```ts
function getUsersOnlineMessage(numUsersOnline: number) {
  if (numUsersOnline) {
    return `There are ${numUsersOnline} online now!`;
  }
  return "Nobody's here. :(";
}
```

在JavaScript中，if之类的构造首先将它们的条件强制为布尔值，以使它们有意义,然后根据结果是真还是假来选择分支。价值观就像
0 NaN ''0n null undefined
所有值强制为false，其他值强制为true。您总是可以通过布尔函数运行值，或使用更短的双布尔否定来将值强制为布尔值。(后者的优势在于，TypeScript会推断出一个狭义的布尔类型true，而第一个则推断为布尔类型。)

利用这种行为非常流行，特别是在防止null或undefined等值时。作为一个例子，让我们尝试在printAll函数中使用它。

```ts
function printAll(strs: string | string[] | null) {
  if (strs && typeof strs === "object") {
    for (const s of strs) {
      console.log(s);
    }
  } else if (typeof strs === "string") {
    console.log(strs);
  }
}
```

你会注意到，我们通过检查strs是否为真来消除上述错误。这至少可以防止我们在运行代码时出现可怕的错误

请记住，对原语进行真实性检查常常容易出错。例如，考虑编写printAll的另一种尝试

```ts
function printAll(strs: string | string[] | null) {
  // !!!!!!!!!!!!!!!!
  //  DON'T DO THIS!
  //   KEEP READING
  // !!!!!!!!!!!!!!!!
  if (strs) {
    if (typeof strs === "object") {
      for (const s of strs) {
        console.log(s);
      }
    } else if (typeof strs === "string") {
      console.log(strs);
    }
  }
}
```

我们将整个函数体封装在一个true check中，但这有一个微妙的缺点:我们可能不再正确地处理空字符串情况。

TypeScript在这里一点也不伤害我们，但是如果你对JavaScript不太熟悉的话，这种行为值得注意。TypeScript通常可以帮助你在早期发现bug，但如果你选择对一个值什么都不做，那么在没有过度规定的情况下，它能做的也就这么多了。如果需要，可以确保使用linter处理此类情况。

最后一个关于缩小真理的词是布尔否定!过滤出否定分支。

```ts
function multiplyAll(
  values: number[] | undefined,
  factor: number
): number[] | undefined {
  if (!values) {
    return values;
  } else {
    return values.map((x) => x * factor);
  }
}
```

#### Equality narrowing

TypeScript还使用switch语句和===、!==、==和!=等相等性检查来缩小类型。例如

```ts
function example(x: string | number, y: string | boolean) {
  if (x === y) {
    // We can now call any 'string' method on 'x' or 'y'.
    x.toUpperCase();
          
(method) String.toUpperCase(): string
    y.toLowerCase();
          
(method) String.toLowerCase(): string
  } else {
    console.log(x);
               
(parameter) x: string | number
    console.log(y);
               
(parameter) y: string | boolean
  }
}
```

当我们在上面的例子中检查x和y是否相等时，TypeScript知道它们的类型也必须相等。由于string是x和y可以使用的唯一常见类型，TypeScript知道x和y必须是第一个分支中的字符串。

检查特定的文字值(与变量相反)也可以工作。在我们关于真实性缩小的部分中，我们写了一个容易出错的printAll函数，因为它不小心没有正确处理空字符串。相反，我们可以做一个特定的检查来阻止空值，TypeScript仍然正确地从strs类型中删除空值。

```ts
function printAll(strs: string | string[] | null) {
  if (strs !== null) {
    if (typeof strs === "object") {
      for (const s of strs) {
                       
(parameter) strs: string[]
        console.log(s);
      }
    } else if (typeof strs === "string") {
      console.log(strs);
                   
(parameter) strs: string
    }
  }
}
```

JavaScript的==和!=更松散的相等性检查也被正确地缩小了。如果你不熟悉，检查是否something == null实际上不仅检查它是否是null值——它还检查它是否有潜在的未定义。这同样适用于== undefined:它检查一个值是null还是undefined。

```ts
interface Container {
  value: number | null | undefined;
}

function multiplyValue(container: Container, factor: number) {
  // Remove both 'null' and 'undefined' from the type.
  if (container.value != null) {
    console.log(container.value);
                           
(property) Container.value: number

    // Now we can safely multiply 'container.value'.
    container.value *= factor;
  }
}
```

##### The in operator narrowing

Javascript有一个操作符用于确定对象是否具有具有名称的属性:in操作符。TypeScript将此作为一种缩小潜在类型的方法。

例如，使用代码:value in x，其中"value"是一个字符串字面值，而x是一个联合类型。真正的分支将具有可选或必需属性值的x类型缩小，而假分支将具有可选或缺少属性值的类型缩小。

```ts
type Fish = { swim: () => void };
type Bird = { fly: () => void };

function move(animal: Fish | Bird) {
  if ("swim" in animal) {
    return animal.swim();
  }

  return animal.fly();
}
```

为了重新迭代，可选属性将存在于两边以缩小范围，例如，一个人可以游泳和飞行(使用正确的设备)，因此应该显示在检查的两边

```ts
type Fish = { swim: () => void };
type Bird = { fly: () => void };
type Human = {  swim?: () => void, fly?: () => void };

function move(animal: Fish | Bird | Human) {
  if ("swim" in animal) { 
    animal
      
(parameter) animal: Fish | Human
  } else {
    animal
      
(parameter) animal: Bird | Human
  }
}
```

##### instanceof narrowing

JavaScript有一个操作符用于检查一个值是否为另一个值的实例。更具体地说，在JavaScript中，`x instanceof Foo`检查x的原型链是否包含`Foo.prototype`。虽然我们在这里不会深入讨论，当我们进入类时，您将看到更多这方面的内容，但是对于大多数可以用new构造的值，它们仍然是有用的。你可能已经猜到了，instanceof也是一种类型保护，TypeScript会在instanceofs保护的分支中进行缩窄。

```ts
function logValue(x: Date | string) {
  if (x instanceof Date) {
    console.log(x.toUTCString());
               
(parameter) x: Date
  } else {
    console.log(x.toUpperCase());
               
(parameter) x: string
  }
}
```

##### Assignments

正如我们之前提到的，当我们赋值给任何变量时，TypeScript会查看赋值的右侧，并适当地缩小左侧。

```ts
let x = Math.random() < 0.5 ? 10 : "hello world!";
   
//let x: string | number
x = 1;

console.log(x);
           
//let x: number
x = "goodbye!";

console.log(x);
           
//let x: string
```

注意，这些分配都是有效的。即使观察x的类型改为数量后我们的第一个任务,我们仍然能够分配一个字符串x。这是因为声明类型的x - x开始类型是`string|number`,和可转让性总是反对声明的类型检查。

如果我们给x赋一个布尔值，我们会看到一个错误，因为那不是声明的类型的一部分。

```ts
let x = Math.random() < 0.5 ? 10 : "hello world!";
   
// let x: string | number
x = 1;

console.log(x);
           
// let x: number
x = true;
Type 'boolean' is not assignable to type 'string | number'.

console.log(x);
           
// let x: string | number
```

#### Control flow analysis

到目前为止，我们已经学习了一些关于TypeScript如何在特定分支中缩小范围的基本例子。但是，除了从每个变量中遍历并在if、while、条件语句等中寻找类型保护之外，还有更多的工作要做。例如


```ts
function padLeft(padding: number | string, input: string) {
  if (typeof padding === "number") {
    return new Array(padding + 1).join(" ") + input;
  }
  return padding + input;
}
```

Padleft从其第一个IF块中返回。TypeScript能够分析这段代码，并发现在填充为数字的情况下，主体的其余部分(return padding + input;)是不可访问的。因此，它能够将number从填充类型中移除(从`string| number`缩小到`string`)，用于函数的其余部分。

这种基于可达性的代码分析称为控制流分析，TypeScript在遇到类型保护和赋值时使用这种流分析来缩小类型。当分析一个变量时，控制流可以分离并一次又一次地重新合并，并且可以观察到该变量在每个点具有不同的类型。

```ts
function example() {
  let x: string | number | boolean;

  x = Math.random() < 0.5;

  console.log(x);
             
let x: boolean

  if (Math.random() < 0.5) {
    x = "hello";
    console.log(x);
               
let x: string
  } else {
    x = 100;
    console.log(x);
               
let x: number
  }

  return x;
        
let x: string | number
}
```

#### 使用类型谓词 Using type predicates

到目前为止，我们已经使用现有的JavaScript构造来处理缩窄，但是有时您希望对整个代码中类型的变化进行更直接的控制。

要定义用户定义的类型保护，只需定义其返回类型为类型谓词的函数

```ts
function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined;
}
```

pet is Fish是本例中的类型谓词。谓词采用parameterName为Type的形式，其中parameterName必须是当前函数签名中的参数名称。

当isFish调用某个变量时，如果原始类型兼容，TypeScript就会将该变量缩小到该特定类型。

```ts
// Both calls to 'swim' and 'fly' are now okay.
let pet = getSmallPet();

if (isFish(pet)) {
  pet.swim();
} else {
  pet.fly();
}
```

请注意，TypeScript不仅知道宠物是if分支中的Fish;它也知道在else分支中，你没有Fish，所以你必须有Bird。

你可以使用类型守卫isFish来过滤一个Fish数组| Bird并获得一个Fish数组

```ts
const zoo: (Fish | Bird)[] = [getSmallPet(), getSmallPet(), getSmallPet()];
const underWater1: Fish[] = zoo.filter(isFish);
// or, equivalently
const underWater2: Fish[] = zoo.filter(isFish) as Fish[];

// The predicate may need repeating for more complex examples
const underWater3: Fish[] = zoo.filter((pet): pet is Fish => {
  if (pet.name === "sharkey") return false;
  return isFish(pet);
});
```

此外，类可以使用这是Type来缩小其类型。

#### Discriminated union

到目前为止，我们看到的大多数示例都集中在使用简单类型(如字符串、布尔值和数字)收缩单个变量。虽然这很常见，但在JavaScript中，大多数时候我们将处理稍微复杂一些的结构。

出于某种动机，让我们假设我们正在尝试编码圆形和正方形等形状。圆表示半径，正方形表示边长。我们将使用kind字段来判断要处理的形状。这是定义形状的第一次尝试。

```ts
interface Shape {
  kind: "circle" | "square";
  radius?: number;
  sideLength?: number;
}
```

注意，我们使用了字符串文字类型的联合:“circle”和“square”来告诉我们应该分别将形状视为圆形还是正方形。通过使用“circle”|“square”而不是string，我们可以避免拼写错误。

```ts
function handleShape(shape: Shape) {
  // oops!
  if (shape.kind === "rect") {
This condition will always return 'false' since the types '"circle" | "square"' and '"rect"' have no overlap.
    // ...
  }
}
```

我们可以编写一个getArea函数，根据它处理的是圆形还是正方形，应用正确的逻辑。我们先来处理圆。

```ts
function getArea(shape: Shape) {
  return Math.PI * shape.radius ** 2;
Object is possibly 'undefined'.
}
```

在strictNullChecks下，它会给我们一个错误——这是适当的，因为radius可能没有定义。但是如果我们对类性质进行适当的检查

```ts
function getArea(shape: Shape) {
  if (shape.kind === "circle") {
    return Math.PI * shape.radius ** 2;
Object is possibly 'undefined'.
  }
}
```

嗯，TypeScript仍然不知道在这里做什么。我们已经达到了一个点，我们比类型检查器更了解我们的值。我们可以尝试使用非空断言(a !在shape.radius之后)表示半径是绝对存在的。

```ts
function getArea(shape: Shape) {
  if (shape.kind === "circle") {
    return Math.PI * shape.radius! ** 2;
  }
}
```

但这感觉并不理想。我们必须用那些非空断言(!)向类型检查器喊话，以使它相信那个形状。定义了Radius，但是如果我们开始移动代码，这些断言很容易出错。此外，在strictNullChecks之外，我们可以意外地访问这些字段中的任何一个(因为可选属性在读取它们时被假定始终存在)。我们绝对可以做得更好。

这种形状编码的问题是，类型检查器没有任何方法来知道半径或边长是否基于kind属性。我们需要将我们所知道的信息传达给类型检查器。考虑到这一点，让我们再来定义Shape。

```ts
interface Circle {
  kind: "circle";
  radius: number;
}

interface Square {
  kind: "square";
  sideLength: number;
}

type Shape = Circle | Square;
```

这里，我们正确地将Shape划分为两种类型，它们的kind属性值不同，但是radius和sidebength被声明为各自类型中的必需属性。

让我们看看当我们试图访问一个形状的半径时会发生什么。

```ts
function getArea(shape: Shape) {
  return Math.PI * shape.radius ** 2;
// Property 'radius' does not exist on type 'Shape'.
//   Property 'radius' does not exist on type 'Square'.
}
```

就像我们对Shape的第一个定义一样，这仍然是一个错误。当radius是可选的时候，我们会得到一个错误(仅在strictNullChecks中)，因为TypeScript无法判断该属性是否存在。现在Shape是一个union, TypeScript告诉我们Shape可能是一个Square，而Squares没有定义半径!两种解释都是正确的，但是只有Shape的新编码仍然会导致strictNullChecks之外的错误。

但如果我们试着再次检查kind属性呢

```ts
function getArea(shape: Shape) {
  if (shape.kind === "circle") {
    return Math.PI * shape.radius ** 2;
                      
(parameter) shape: Circle
  }
}
```

这样就消除了错误!当联合中的每个类型都包含一个与文字类型相同的属性时，TypeScript就会认为这是一个有区别的联合，并可以缩小联合的成员范围。

在这种情况下，kind是公共属性(这被认为是形状的一个判别属性)。检查kind属性是否为“circle”，可以删除Shape中所有没有kind属性的类型“circle”。缩小到圆形的形状。

同样的检查也适用于switch语句。现在，我们可以尝试编写完整的getArea而没有任何麻烦!非空的断言。

```ts
function getArea(shape: Shape) {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
                        
// (parameter) shape: Circle
    case "square":
      return shape.sideLength ** 2;
              
// (parameter) shape: Square
  }
}
```

这里最重要的是《Shape》的编码。与TypeScript沟通正确的信息——Circle和Square实际上是两种独立的类型，具有特定的类型字段——是至关重要的。这样做可以让我们写出类型安全的TypeScript代码，看起来和我们之前写的JavaScript没有什么不同。从那里，类型系统能够做正确的事情，并找出我们的switch语句的每个分支中的类型。

> 顺便说一下，试着使用上面的例子并删除一些返回的关键字。您将看到，当不小心在switch语句中漏掉不同的子句时，类型检查可以帮助避免错误。

识别联合比讨论方和圆更有用。它们很适合在JavaScript中表示任何类型的消息传递方案 像当通过网络发送消息时 或者在状态管理框架中编码突变。

#### The never type

当缩小时，您可以将联合的选项减少到一个点，即您已经删除了所有的可能性，没有任何剩余。在这种情况下，TypeScript会使用never类型来表示一个不应该存在的状态。

### 穷尽性检查 Exhaustiveness checking

never类型可分配给每一种类型;但是，没有类型可以赋值给never(除了never本身)。这意味着您可以使用收缩，并依赖于从不打开来在switch语句中执行穷举检查。

例如，向我们的getArea函数添加一个默认值，该函数试图将形状赋值为never，当所有可能的情况都没有被处理时，将引发异常。

```ts
type Shape = Circle | Square;

function getArea(shape: Shape) {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.sideLength ** 2;
    default:
      const _exhaustiveCheck: never = shape;
      return _exhaustiveCheck;
  }
}
```

向Shape联合中添加新成员会导致TypeScript错误

```ts
interface Triangle {
  kind: "triangle";
  sideLength: number;
}

type Shape = Circle | Square | Triangle;

function getArea(shape: Shape) {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.sideLength ** 2;
    default:
      const _exhaustiveCheck: never = shape;
Type 'Triangle' is not assignable to type 'never'.
      return _exhaustiveCheck;
  }
}
```

end

