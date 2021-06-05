## 基础知识

欢迎来到手册的第一页，如果这是你第一次接触TypeScript，你可以从“入门”指南开始
JavaScript中的每个值都有一组行为，可以通过运行不同的操作观察到。这听起来很抽象，但是作为一个简单的例子，考虑我们可能在一个名为message的变量上运行的一些操作。
```ts
// Accessing the property 'toLowerCase'
// on 'message' and then calling it
message.toLowerCase();
// Calling 'message'
message();
```
如果我们将其分解，第一行可运行代码将访问一个名为toLowerCase的属性，然后调用它。第二个尝试直接调用message。
但是假设我们不知道message的值——这是很常见的——我们不能可靠地说我们将从运行这些代码中得到什么结果。每个操作的行为完全取决于我们首先拥有的值。

1.message可调用吗？
2.它是否有一个名为tolowercase的属性？
3.如果它，会是tolowercase，可调用吗？
4.如果这两个值都是可调用的，它们会返回什么？

当我们编写JavaScript时，这些问题的答案通常都在我们的脑海中，我们希望我们能够正确地处理所有细节。
让我们假设message是按照以下方式定义的。

```ts
const message = "Hello World!";
```

正如您可能猜到的那样，如果我们尝试运行message.toLowerCase()，我们将只获得相同的小写字符串。
那么第二行代码呢?如果您熟悉JavaScript，您就会知道此操作会出现异常

```ts
TypeError: message is not a function
```
如果我们能避免这样的错误就好了

当我们运行代码时，JavaScript运行时选择做什么的方式是通过计算值的类型——它有什么样的行为和功能。这是TypeError所暗示的一部分-它说字符串“Hello World!”不能作为一个函数被调用。

对于一些值，如字符串和数字，我们可以在运行时使用typeof操作符标识它们的类型。但是对于其他东西，比如函数，没有相应的运行时机制来标识它们的类型。例如，考虑这个函数

```ts
function fn(x) {
  return x.flip();
}
```

我们可以通过读取代码观察到，这个函数只有在给定一个具有可调用flip属性的对象时才会工作，但是JavaScript没有以一种我们可以在代码运行时检查的方式显示这个信息。在纯JavaScript中，告诉fn对特定值做了什么的唯一方法是调用它，看看会发生什么。这种行为使得在代码运行之前很难预测它将做什么，这意味着在编写代码时更难知道它将做什么。

从这个角度看，类型是描述哪些值可以传递给fn，哪些会崩溃的概念。JavaScript只真正提供动态类型——运行代码看看会发生什么。

另一种方法是使用静态类型系统在代码运行之前对代码进行预测。

### 静态类型检查

回想一下我们之前试图将string作为函数调用时,遇到的TypeError。大多数人不喜欢在运行他们的代码时得到任何类型的错误——这些被认为是错误!

当我们编写新代码时，我们会尽量避免引入新的bug。如果我们只添加一些代码，保存文件，重新运行代码，并立即看到错误，我们可能能够快速隔离问题;但情况并非总是如此。我们可能没有对该特性进行足够彻底的测试，因此我们可能永远不会遇到可能抛出的潜在错误.或者，如果我们足够幸运地看到了错误，我们可能会以进行大规模重构而告终，并添加许多不同的代码，而我们不得不挖掘这些代码。

例如，规范说，试图调用不可调用的东西应该抛出一个错误。也许这听起来像是显而易见的行为，但是您可以想象访问对象上不存在的属性也应该抛出错误。相反，JavaScript给了我们不同的行为并返回undefined

```ts
const user = {
  name: "Daniel",
  age: 26,
};
user.location; // returns undefined
```
最终，静态类型系统必须调用应该在其系统中标记为错误的代码，即使是有效的JavaScript不会立即抛出错误。在TypeScript中，下面的代码会产生一个关于未定义位置的错误

```ts
const user = {
  name: "Daniel",
  age: 26,
};

user.location;
//Property 'location' does not exist on type '{ name: string; age: number; }'.
```

虽然有时这意味着在您所能表达的内容上要有所取舍，但其目的是捕获我们程序中的合法bug。TypeScript捕获了许多合理的bug。

例如:输入错误

```ts
const announcement = "Hello World!";

// How quickly can you spot the typos?
announcement.toLocaleLowercase();
announcement.toLocalLowerCase();

// We probably meant to write this...
announcement.toLocaleLowerCase();
```

未经证明的功能，

```ts
function flipCoin() {
  // Meant to be Math.random()
  return Math.random < 0.5;
Operator '<' cannot be applied to types '() => number' and 'number'.
}
```

或者基本逻辑错误。

```typescript
const value = Math.random() < 0.5 ? "a" : "b";
if (value !== "a") {
  // ...
} else if (value === "b") {
This condition will always return 'false' since the types '"a"' and '"b"' have no overlap.
  // Oops, unreachable
}
```

类型工具

当我们在代码中犯错误时，TypeScript可以捕获错误。这很好，但TypeScript也可以在一开始就防止我们犯这些错误。类型检查器具有一些信息来检查诸如我们是否正在访问变量和其他属性上的正确属性之类的事情。一旦有了该信息，它还可以开始建议您可能需要使用哪些属性。这意味着TypeScript也可以用来编辑代码，核心类型检查器可以在你在编辑器中输入时提供错误消息和代码完成。这就是人们常说的

TypeScript非常重视工具的使用，而不仅仅是在你输入时完成和出错。支持TypeScript的编辑器可以提供快速修复来自动修复错误，可以通过重构来轻松地重新组织代码，还可以提供有用的导航功能来跳转到变量的定义，或者找到对给定变量的所有引用。所有这些都是建立在类型检查器之上的，而且是完全跨平台的，所以你喜欢的编辑器很可能支持TypeScript。
