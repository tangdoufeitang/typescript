 # Modules

 JavaScript有很长一段处理模块化代码的不同方法。TypeScript在2012年就已经出现了，已经实现了对这些格式的支持，但是随着时间的推移，社区和JavaScript规范已经融合在一种叫做ES Modules(或ES6 Modules)的格式上。您可能会知道它是`import/export`语法

 ES模块在2015年被添加到JavaScript规范中，并在2020年获得了大多数浏览器和JavaScript运行时的广泛支持。

 对于重点，本手册将涵盖ES模块和其流行的前游标CommonJS模块。exports = syntax，你可以在Modules下的reference部分找到关于[其他模块模式](https://www.typescriptlang.org/docs/handbook/modules.html)的信息。

 ### How JavaScript Modules are Defined

 在TypeScript中，就像在ECMAScript 2015中一样，任何包含顶层导入或导出的文件都被认为是一个模块。

 相反，没有任何顶级导入或导出声明的文件会被视为脚本，其内容在全局作用域中可用(因此也适用于模块)。

 模块是在它们自己的作用域中执行的，而不是在全局作用域中。这意味着在模块中声明的变量、函数、类等在模块外部是不可见的，除非它们被使用一种导出表单显式导出。相反，要使用从不同模块导出的变量、函数、类、接口等，必须使用其中一个导入表单导入。

 ### Non-modules

 在我们开始之前，了解TypeScript对模块的定义是很重要的。JavaScript规范声明，任何没有导出或顶层等待的JavaScript文件都应该被视为脚本，而不是模块。

 脚本文件中声明变量和类型是在全球范围内共享,和它年代认为你要么使用——输出文件的编译器选项多个输入文件加入到一个输出文件,或使用多个脚本< >标记在你的HTML加载这些文件(以正确的顺序)。

 如果您有一个当前没有任何导入或导出的文件，但您希望被视为一个模块，请添加这一行

 ```ts
 export {};

 ```

 这会将文件更改为一个不导出任何内容的模块。无论你的模块目标是什么，这种语法都可以工作。

 ### Modules in TypeScript

 在用TypeScript编写基于模块的代码时，主要有三件事需要考虑
 - 语法:我想使用什么语法来导入和导出东西
 - 模块解析:模块名(或路径)和磁盘上的文件之间的关系是什么
 - 模块输出目标:我的JavaScript模块应该是什么样的
  
 #### ES Module Syntax

 文件可以通过export default声明一个主导出

 ```ts
 // @filename: hello.ts
    export default function helloWorld() {
    console.log("Hello, world!");
    }
 ```
 然后通过导入
 ```ts
 import hello from "./hello.js";
 hello();
 ```
 除了默认导出之外，通过省略default，您还可以有多个变量和函数的导出

 ```ts
 // @filename: maths.ts
export var pi = 3.14;
export let squareTwo = 1.41;
export const phi = 1.61;

export class RandomNumberGenerator {}

export function absolute(num: number) {
  if (num < 0) return num * -1;
  return num;
}
```

这些可以通过导入语法在另一个文件中使用

```ts
import { pi, phi, absolute } from "./maths.js";

console.log(pi);
const absPhi = absolute(phi);
        
// const absPhi: number
```

#### Additional Import Syntax

导入可以使用`import {old as new }`这样的格式进行重命名

```ts
import { pi as π } from "./maths.js";

console.log(π);
```

您可以将上述语法混合并匹配到单个导入中

```ts
// @filename: maths.ts
export const pi = 3.14;
export default class RandomNumberGenerator {}

// @filename: app.ts
import RNGen, { pi as π } from "./maths.js";

RNGen;
 
// (alias) class RNGen
// import RNGen

console.log(π);
           
// (alias) const π: 3.14
// import π
```

您可以使用`* as name`，将所有导出的对象放入单个名称空间中

```ts
// @filename: app.ts
import * as math from "./maths.js";

console.log(math.pi);
const positivePhi = math.absolute(math.phi);
          
// const positivePhi: number
```

您可以导入一个文件，而不将任何变量包含到当前模块中 如`import "./file"`

```ts
// @filename: app.ts
import "./maths.js";

console.log("3.14");
```

在本例中，导入不执行任何操作。不过，所有的代码都是数学的。对t进行了评估，它可能引发影响其他物体的副作用。

#### TypeScript Specific ES Module Syntax

类型可以使用与JavaScript值相同的语法导出和导入

```ts
// @filename: animal.ts
export type Cat = { breed: string; yearOfBirth: number };

export interface Dog {
  breeds: string[];
  yearOfBirth: number;
}

// @filename: app.ts
import { Cat, Dog } from "./animal.js";
type Animals = Cat | Dog;
```

TypeScript使用import type扩展了import语法，它是一个只能导入类型的import。

```ts
// @filename: animal.ts
export type Cat = { breed: string; yearOfBirth: number };
// 'createCatName' cannot be used as a value because it was imported using 'import type'.
export type Dog = { breeds: string[]; yearOfBirth: number };
export const createCatName = () => "fluffy";

// @filename: valid.ts
import type { Cat, Dog } from "./animal.js";
export type Animals = Cat | Dog;

// @filename: app.ts
import type { createCatName } from "./animal.js";
const name = createCatName();
```

这种语法允许像Babel、swc或esbuild这样的非typescript编译器知道哪些导入可以安全地删除。

#### ES Module Syntax with CommonJS Behavior

TypeScript有ES Module语法，直接与CommonJS和AMD的需求相关。在大多数情况下，使用ES Module的导入与这些环境的要求是一样的，但是这种语法确保你的TypeScript文件与CommonJS输出是1对1匹配的

```ts
import fs = require("fs");
const code = fs.readFileSync("hello.ts", "utf8");
```

您可以在[模块引用页面](https://www.typescriptlang.org/docs/handbook/modules.html#export--and-import--require)中了解更多关于此语法的信息。

### CommonJS Syntax

CommonJS是大多数npm模块的交付格式。即使你正在使用上面的ES Modules语法编写代码，对CommonJS语法的工作原理有一个简单的了解也会帮助你更容易地进行调试。

#### Exporting

标识符是通过在一个名为全局模块上设置exports属性导出的。

```ts
function absolute(num: number) {
  if (num < 0) return num * -1;
  return num;
}

module.exports = {
  pi: 3.14,
  squareTwo: 1.41,
  phi: 1.61,
  absolute,
};
```

然后可以通过require语句导入这些文件

```ts
const maths = require("maths");
maths.pi;
```

或者，您可以使用JavaScript中的解构特性进行一点简化

```ts
const { squareTwo } = require("maths");
squareTwo;
```

#### CommonJS and ES Modules interop

CommonJS和ES Module的特性是不匹配的，因为ES Module只支持将默认导出作为对象，而不支持将默认导出作为函数。TypeScript有一个编译器标志来减少esModuleInterop两组不同约束之间的摩擦。

### TypeScript’s Module Resolution Options TypeScript模块解析选项

模块解析是从import或require语句中获取一个字符串，并确定该字符串指向哪个文件的过程。

TypeScript包括两种解析策略:Classic和Node。经典的，当编译器标志模块不是commonjs时，默认会包含向后兼容性。Node策略复制Node.js在CommonJS模式下的工作方式，附加.ts和.d.ts检查。

在TypeScript中有很多TSConfig标志会影响模块策略:moduleResolution, baseUrl, paths, rootDirs。

关于这些策略如何工作的完整细节，您可以参考Module Resolution。

### TypeScript’s Module Output Options

有两个选项会影响生成的JavaScript输出
- [目标](https://www.typescriptlang.org/tsconfig#target)，它决定了哪些JS特性被降级(转换为运行在更老的JavaScript运行时)，哪些完好无损
- [模块](https://www.typescriptlang.org/tsconfig#module)，它决定了模块之间使用什么代码进行交互

你使用哪个目标是由你想要运行TypeScript代码的JavaScript运行时的可用特性决定的。这可能是:你支持的最老的web浏览器，你希望运行的Node.js的最低版本，或者可能来自你运行时的唯一约束——比如Electron。

所有模块之间的通信都是通过模块加载器进行的，编译器标记模块决定使用哪个模块。在运行时，模块加载器负责在执行模块之前定位和执行模块的所有依赖项。

例如，下面是一个使用ES Modules语法的TypeScript文件，其中展示了模块的一些不同选项

```ts
import { valueOfPi } from "./constants.js";

export const twoPi = valueOfPi * 2;
```

#### ES2020

```ts
import { valueOfPi } from "./constants.js";
export const twoPi = valueOfPi * 2;
```

#### CommonJS

```ts
"use strict";
Object.defineProperty(exports, "__esModule", { value: true });
exports.twoPi = void 0;
const constants_js_1 = require("./constants.js");
exports.twoPi = constants_js_1.valueOfPi * 2;
```

#### UMD

```ts
(function (factory) {
    if (typeof module === "object" && typeof module.exports === "object") {
        var v = factory(require, exports);
        if (v !== undefined) module.exports = v;
    }
    else if (typeof define === "function" && define.amd) {
        define(["require", "exports", "./constants.js"], factory);
    }
})(function (require, exports) {
    "use strict";
    Object.defineProperty(exports, "__esModule", { value: true });
    exports.twoPi = void 0;
    const constants_js_1 = require("./constants.js");
    exports.twoPi = constants_js_1.valueOfPi * 2;
});
```

> 注意，ES2020实际上与原始的index.ts相同。

您可以在TSConfig Reference for模块中看到所有可用的选项以及它们所发出的JavaScript代码。

### TypeScript namespaces

TypeScript有自己的模块格式，叫做命名空间，它早于ES Modules标准。这种语法具有许多用于创建复杂定义文件的有用特性，并且在DefinitelyTyped中仍然有积极的应用。命名空间中的大部分特性都存在于ES模块中，我们建议您使用它来与JavaScript的方向保持一致。您可以在名称空间引用页面中了解有关名称空间的更多信息。

END
    