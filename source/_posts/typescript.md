---
title: TypeScript 学习笔记
date: 2021-06-10 21:41:38
categories:
- TypeScript
- 大前端
tags:
- TypeScript
- 大前端
---
![TypeScript](https://raw.githubusercontent.com/SeaChan0117/SeaChan0117.github.io/master/source/_posts/typescript/1.jpg)
<!--more-->
TypeScript（本文后续统一使用TS），是一门解决JavaScript自有类型系统问题的语音（提高代码可靠程度）。它是JavaScript类型的超集，可以编译出纯净、 简洁的JavaScript代码，并且可以运行在任何浏览器上、Node.js环境中和任何支持ECMAScript 3（或更高版本）的JavaScript引擎中。

# 前置知识
* 强类型与弱类型
* 静态类型与动态类型
* JavaScript自有系统类型问题

# 强类型与弱类型(类型安全)
强类型：在语言层面就限制了函数的实参类型必须与形参类型相同；  
弱类型：语言层面不会显示实参的类型；  

强类型有更强的类型约束，而弱类型中几乎没有约束，强类型语言中不允许任意的隐式类型转换，而弱类型语言则允许任意的数据隐式类型转换。  

JavaScript就是典型的弱类型语言。JavaScript中所有错误都是在代码执行层面，通过逻辑判断手动抛出的。
```javascript
'100' - 50 // 50
Math.floor('foo') // NaN
Math.flor(true)   // 1
```

# 静态类型与动态类型(类型检查)
静态类型：一个变量在声明时它的类型就是明确的，声明过后，它的类型就不可修改；  
动态类型：在代码运行阶段才能明确变量的类型，而且变量的类型随时可以改变；  

可以说动态类型语言中的变量没有类型，变量中存放的值是有类型的，静态类型和动态类型的区别就是，是否允许随时修改变量的类型
```javascript
// JavaScript动态语言示例
let a = 1
a = 'foo'
console.log(a)  // foo
```

以下是一些语言的划分，可以看出强类型不等于静态类型，弱类型不等于动态类型。
![语言类型划分](https://raw.githubusercontent.com/SeaChan0117/SeaChan0117.github.io/master/source/_posts/typescript/2.jpg)

# JavaScript自有系统类型问题
JavaScript是一门标准的弱类型且动态类型的语音，所以JS灵活多变，所以JS缺失类型系统的可靠性。

弱类型问题示例：
```javascript
// js在执行时才检测错误
const obj = {}
obj.foo()    // 报错
setTimeout(() =>{
    obj.foo()    // 报错
}, 1000000)

// 类型不确定导致，代码执行结果不是预期，函数功能发生改变，就需要添加约定去限制入参，然而约定没有保障，不能保证每个开发都会做
function sum(a, b){
    return a + b  // 期望做两个数值相加的结果
} 

sum('100', 100)   // "100100"  结果为字符串拼接

// 数据定义时没有类型要求，导致定义和预期产生偏差，对象索引器的错误用法
let obj = {}
obj[true] = 1
console.log(obj['true'])  // 1   true作为属性时，被自动转为了字符串
```

相较于弱类型语音，强类型有着明显的优势：

* 错误更早暴露（编码阶段就能发现并避免类型异常）；
* 代码更智能，编码更准确（开发工具智能提示能更准确的语法，大大提高编码效率）；
* 重构更牢靠（重构时，能及时暴露出相关代码异常信息，可及时准确的对耦合代码进行修改）；
* 减少不必要的类型判断（在编码阶段，参数类型已经确定，就不用再对类型做约定判断）；


**flow：JavaScript类型检查器工具**  
* [Flow: A Static Type Checker for JavaScript](https://link.zhihu.com/?target=https%3A//flow.org)
* [用Flow提升前端健壮性](https://link.zhihu.com/?target=https%3A//juejin.cn/post/6844903599173173255)
* [Flow（一）-- JavaScript静态类型检查器](https://link.zhihu.com/?target=https%3A//juejin.cn/post/6900912350640275470)
* [Flow（二）-- 简单语法使用](https://link.zhihu.com/?target=https%3A//juejin.cn/post/6900927463296073736)

**TS语言规范与基本运用**  
TS功能强大，生态健全完善，工具支持度高。  
缺点：
* 相较于JavaScript多了很多概念，提高了学习成本；
* 小项目或项目初期，TS会增加成本（要编写很多的类型声明）
## TS 快速上手（nmp为例）
在工程目录（TS为例）下执行 npm init -y 初始化项目管理的 package.json 文件；

执行 npm install typescript --save --dev 安装TS到开发依赖，此时node_modules目录下会多出.bin文件夹，目录下的tsc命令用于后续编译TS代码；  
![图3](https://raw.githubusercontent.com/SeaChan0117/SeaChan0117.github.io/master/source/_posts/typescript/3.jpg)

在TS根目录下新建第一个TS文件，01-getting-started.ts ，执行命令npx tsc 01-getting-started.ts 将TS文件编译为JS文件：  
![图4](https://raw.githubusercontent.com/SeaChan0117/SeaChan0117.github.io/master/source/_posts/typescript/4.jpg)
![图5](https://raw.githubusercontent.com/SeaChan0117/SeaChan0117.github.io/master/source/_posts/typescript/5.jpg)
![图6](https://raw.githubusercontent.com/SeaChan0117/SeaChan0117.github.io/master/source/_posts/typescript/6.jpg)

TS强类型体现：
![图7](https://raw.githubusercontent.com/SeaChan0117/SeaChan0117.github.io/master/source/_posts/typescript/7.jpg)
![图8](https://raw.gitTS文件，也可以编译整个项目或工程，在编译整个项目之前，需要创建一个TS的配置文件，通过命令初始化生成该配置文件 npx tsc --init ，在项目根目录下生成一个 tsconfig.json 的配置文件。  
![图9](https://raw.githubusercontent.com/SeaChan0117/SeaChan0117.github.io/master/source/_posts/typescript/9.jpg)  
tsconfig.json详细配置项详情参考以下链接：  
* [tsconfig.json入门指南](https://link.zhihu.com/?target=https%3A//juejin.cn/post/6844904054770892813)
* [掌握 tsconfig.json](https://link.zhihu.com/?target=https%3A//juejin.cn/post/6844904178234458120)
* [Documentation - What is a tsconfig.json](https://link.zhihu.com/?target=https%3A//www.typescriptlang.org/docs/handbook/tsconfig-json.html)
* [jsconfig.json Refehubusercontent.com/SeaChan0117/SeaChan0117.github.io/master/source/_posts/typescript/8.jpg)
                     
                     ## TS 配置文件
                     tsc命令不单可以编译单个rence](https://link.zhihu.com/?target=https%3A//code.visualstudio.com/docs/languages/jsconfig)
```json
{
  "compilerOptions": {
    /* Visit https://aka.ms/tsconfig.json to read more about this file */

    /* Basic Options */
    // "incremental": true,                         /* Enable incremental compilation */
    "target": "es2015",                                /* Specify ECMAScript target version: 'ES3' (default), 'ES5', 'ES2015', 'ES2016', 'ES2017', 'ES2018', 'ES2019', 'ES2020', or 'ESNEXT'. */
    "module": "commonjs",                           /* Specify module code generation: 'none', 'commonjs', 'amd', 'system', 'umd', 'es2015', 'es2020', or 'ESNext'. */
    // "lib": [],                                   /* Specify library files to be included in the compilation. */
    // "allowJs": true,                             /* Allow javascript files to be compiled. */
    // "checkJs": true,                             /* Report errors in .js files. */
    // "jsx": "preserve",                           /* Specify JSX code generation: 'preserve', 'react-native', 'react', 'react-jsx' or 'react-jsxdev'. */
    // "declaration": true,                         /* Generates corresponding '.d.ts' file. */
    // "declarationMap": true,                      /* Generates a sourcemap for each corresponding '.d.ts' file. */
     "sourceMap": true,                           /* Generates corresponding '.map' file. */
    // "outFile": "./",                             /* Concatenate and emit output to single file. */
     "outDir": "dist",                              /* Redirect output structure to the directory. */
     "rootDir": "src",                             /* Specify the root directory of input files. Use to control the output directory structure with --outDir. */
    // "composite": true,                           /* Enable project compilation */
    // "tsBuildInfoFile": "./",                     /* Specify file to store incremental compilation information */
    // "removeComments": true,                      /* Do not emit comments to output. */
    // "noEmit": true,                              /* Do not emit outputs. */
    // "importHelpers": true,                       /* Import emit helpers from 'tslib'. */
    // "downlevelIteration": true,                  /* Provide full support for iterables in 'for-of', spread, and destructuring when targeting 'ES5' or 'ES3'. */
    // "isolatedModules": true,                     /* Transpile each file as a separate module (similar to 'ts.transpileModule'). */

    /* Strict Type-Checking Options */
    "strict": true,                                 /* Enable all strict type-checking options. */
    // "noImplicitAny": true,                       /* Raise error on expressions and declarations with an implied 'any' type. */
    // "strictNullChecks": true,                    /* Enable strict null checks. */
    // "strictFunctionTypes": true,                 /* Enable strict checking of function types. */
    // "strictBindCallApply": true,                 /* Enable strict 'bind', 'call', and 'apply' methods on functions. */
    // "strictPropertyInitialization": true,        /* Enable strict checking of property initialization in classes. */
    // "noImplicitThis": true,                      /* Raise error on 'this' expressions with an implied 'any' type. */
    // "alwaysStrict": true,                        /* Parse in strict mode and emit "use strict" for each source file. */

    /* Additional Checks */
    // "noUnusedLocals": true,                      /* Report errors on unused locals. */
    // "noUnusedParameters": true,                  /* Report errors on unused parameters. */
    // "noImplicitReturns": true,                   /* Report error when not all code paths in function return a value. */
    // "noFallthroughCasesInSwitch": true,          /* Report errors for fallthrough cases in switch statement. */
    // "noUncheckedIndexedAccess": true,            /* Include 'undefined' in index signature results */
    // "noPropertyAccessFromIndexSignature": true,  /* Require undeclared properties from index signatures to use element accesses. */

    /* Module Resolution Options */
    // "moduleResolution": "node",                  /* Specify module resolution strategy: 'node' (Node.js) or 'classic' (TypeScript pre-1.6). */
    // "baseUrl": "./",                             /* Base directory to resolve non-absolute module names. */
    // "paths": {},                                 /* A series of entries which re-map imports to lookup locations relative to the 'baseUrl'. */
    // "rootDirs": [],                              /* List of root folders whose combined content represents the structure of the project at runtime. */
    // "typeRoots": [],                             /* List of folders to include type definitions from. */
    // "types": [],                                 /* Type declaration files to be included in compilation. */
    // "allowSyntheticDefaultImports": true,        /* Allow default imports from modules with no default export. This does not affect code emit, just typechecking. */
    "esModuleInterop": true,                        /* Enables emit interoperability between CommonJS and ES Modules via creation of namespace objects for all imports. Implies 'allowSyntheticDefaultImports'. */
    // "preserveSymlinks": true,                    /* Do not resolve the real path of symlinks. */
    // "allowUmdGlobalAccess": true,                /* Allow accessing UMD globals from modules. */

    /* Source Map Options */
    // "sourceRoot": "",                            /* Specify the location where debugger should locate TypeScript files instead of source locations. */
    // "mapRoot": "",                               /* Specify the location where debugger should locate map files instead of generated locations. */
    // "inlineSourceMap": true,                     /* Emit a single file with source maps instead of having a separate file. */
    // "inlineSources": true,                       /* Emit the source alongside the sourcemaps within a single file; requires '--inlineSourceMap' or '--sourceMap' to be set. */

    /* Experimental Options */
    // "experimentalDecorators": true,              /* Enables experimental support for ES7 decorators. */
    // "emitDecoratorMetadata": true,               /* Enables experimental support for emitting type metadata for decorators. */

    /* Advanced Options */
    "skipLibCheck": true,                           /* Skip type checking of declaration files. */
    "forceConsistentCasingInFileNames": true        /* Disallow inconsistently-cased references to the same file. */
  }
}
```
配置文件初始化并配置想过配置项后，运行 npx tsc 就可编译整个项目：  
![图10](https://raw.githubusercontent.com/SeaChan0117/SeaChan0117.github.io/master/source/_posts/typescript/10.jpg)  

## TS 原始类型
TypeScript的原始类型包括: boolean、number、string、void、undefined、null、symbol、bigint，在声明变量时都应该加上类型约束。  
![图11](https://raw.githubusercontent.com/SeaChan0117/SeaChan0117.github.io/master/source/_posts/typescript/11.jpg)

## TS 作用域问题

以上 02-primitive.ts 文件中定义了 a 变量，此时在 03-module-scope.ts 中再次声明一个 a 变量，代码提示报错，变量名重复。  
![图12](https://raw.githubusercontent.com/SeaChan0117/SeaChan0117.github.io/master/source/_posts/typescript/12.jpg)  
![图13](https://raw.githubusercontent.com/SeaChan0117/SeaChan0117.github.io/master/source/_posts/typescript/13.jpg)  

## TS Object类型（所有非原始类型，对象、数组、函数等）
```javascript
export {}

// object类型，function, [], {}
// const foo: object = function () {} // []  // {}

// 定义单纯{}字面量对象
const foo: {} = {}
const bar: {a: string, b: number} = {a: '123', b: 123}
```

## TS 数组类型
```javascript
export {}

// Array泛型
const arr: Array<number> = [1, 2, 3]

// [type][] 方式
const arr1: number[] = [1, 2, 3]

// =========================
function sum(...args: number[]) {
    // 入参无强类型约束时，此处需对数组每个值做判断
    args.reduce((pre, cur) => pre + cur, 0)
}
```

## TS 元组类型
**元组类型**：元组可以当做是数组的拓展，它表示已知元素数量和类型的数组。确切地说，是已知数组中每一个位置上的元素的类型。明确元素数量及每个元素类型，元素的类型不必完全相同。
```javascript
export {}

const tuple: [number,string] = [17, 'SeaChan']

// const age = tuple[0]
// const name = tuple[1]

const [age, name] = tuple


// =====================
// Object.entries 就返回元组数据
Object.entries({
    foo: 123,
    bar: '456'
})
```

## TS 枚举类型

export {}
```javascript
/**
 * 枚举： 给一组值定义更好理解的名字；一个枚举中只会存在一组固定的值，不会出现超出范围的可能性；
 */

const enum PostStatus {
    Draft = 0,
    Unpublished = 1,
    Published = 2,
}
const post = {
    title: 'Hello TypeScript',
    content: 'TypeScript is a typed superset of JavaScript',
    status: PostStatus.Draft  // 0 草稿，1 未发布，2 已发布；这几个状态值就可以用枚举来处理
}
```

## TS 函数类型
默认情况下，枚举是基于 0 的，也就是说第一个值是 0，后面的值依次递增。不要担心，当中的每一个值都可以显式指定，只要不出现重复即可，没有被显式指定的值，都会在前一个值的基础上递增，如果是字符串就需要所有显示指定。
```javascript
export {}

// 函数声明类型限制
function f1(a: number, b: number = 10, ...rest: number[]): string {
    return 'string'
}

f1(1)
f1(1, 2)
f1(1, 2, 3, 4)

// 函数表达式类型限制=======> 对f2做类型限制使用箭头函数
const f2: (a: number, b: number, ...rest: number[]) => string = function (a: number, b: number, ...rest: number[]): string {
    return 'string'
}

f2(1, 2)
f1(1, 2, 3, 4)
```

## TS 任意类型（any和unknown）
any 和 unknown 的最大区别是, unknown 是 top type (任何类型都是它的 subtype) , 而 any 即是 top type, 又是 bottom type (它是任何类型的 subtype ) , 这导致 any 基本上就是放弃了任何类型检查。
```javascript
export {}

function stringify(val: any) {
    return JSON.stringify(val)
}

stringify(100)
stringify('100')
stringify(true)

// any 相当于回归JavaScript,动态类型，此时TS对相应的代码不再做类型检查；所以尽量不要使用any
let foo: any = 100
foo = 'string'
foo.fn()
```

## TS 类型断言
类型断言好比其它语言里的类型转换，但是不进行特殊的数据检查和解构, 它没有运行时的影响，只是在编译阶段起作用，TS会假设编码人员已经检查过数据。
```javascript
export {}
// 类型断言
const arr = [1, 2, 3, 4]
const res = arr.find(i => i > 0) // res 被断言为 number | undefined

// const mul = res * res   // 此时res计算会报错，因为ts断言存在undefined情况

// 我们需要告诉ts，res不会是undefined
const num = res as number       // as 断言    推荐使用
const num1 = <number>res        // < [type] > 断言; JSX下会有语法冲突，不能使用这种
const mul = num * num           // 告诉ts，res一定是数字类型时，不再报错
const mul1 = num1 * num1
```

## TS 接口
作为用过Java的猿来说，接口在面向对象的语言中，是一个很重要的概念，它是对一类行为的抽象，具体行为如何，需要具体的类去实现它们。  
TS中的接口可以是针对对象的，约定一个对象中有哪些成员，且这些成员是什么样的；也可以是上边说的行为抽象，下边会讲到。
```javascript
export {}
// 接口
interface Person {
    readonly name: string   // readonly 只读属性
    firsName?: string       // ？ 可选属性
    age: number
}

function showPerson(person: Person) {
    console.log(person.name);
    console.log(person.age);
}

showPerson({
    name: 'SeaChan',
    age: 17
})

interface Cache {
    [key: string]: string       // 动态属性，当前为string类型
}

const cache: Cache = {}
cache.foo = 'foo'
cache.var = 'bar'
```

## TS 类的基本使用
**class**, 描述一类具体事物的抽象特征。
```javascript
export {}
// 类
class Person {
    public name: string  // = 'init name' 必须在此或者构造函数中初始化
    protected readonly age: number  // readonly 只读
    private gender: boolean = true
    constructor(name: string, age: number) {
        this.name = name
        this.age = age
    }

    sayHello(msg: string):void {
        console.log(`Hello, i am ${this.name}, and i am a ${this.gender ? 'boy': 'girl'}, ${msg}`);
    }
}

// 访问修饰符 private私有的，只能在当前内部使用；protected受保护的，能在当前内部和子类里边访问到；public公有的，能在实例对象上访问到
const p1 = new Person('SeaChan', 17)
console.log(p1.name);       // SeaChan
// console.log(p1.age);        // 报错
// console.log(p1.gender);     // 报错

// 子类继承Person后protected可访问到

class Kids extends Person{
    constructor(name: string, age: number) {
        super(name, age);
    }

    sayHi(msg: string):void {
        `Hi, i am ${this.age} years old, ${msg}`
    }
}

let k1 = new Kids('Tom', 12)
k1.sayHello('hahaha')                   // Hi, i am 12 years old, hahaha
```

## TS 接口和类
```javascript
export {}
// 接口和类
// 把一些类之间共通的属性或者特征定位为接口，各个类再去实现响应的接口

class Person implements EatAndRun{
    name: string

    constructor(name: string) {
        this.name = name
    }

    eat(food: string) {
        console.log(food);
    }

    run() {
        console.log(`run with 2 feet`);
    }
}

class Animals implements EatAndRun{
    name: string

    constructor(name: string) {
        this.name = name
    }

    eat(food: string) {
        console.log(food);
    }

    run() {
        console.log(`run with 4 feet`);
    }
}

// Person和Animals中都有eat和run两个方法，他们可抽象为接口，用于约束两个类的行为，在两个类中就得实现接口中的方法
interface EatAndRun {
    eat(food: string): void
    run(): void
}
```

## TS 抽象类
**多态**：父类定义一个方法不去实现，让继承它的子类去实现 每一个子类有不同的表现，使用多态基础是类的继承或者接口实现。
```javascript
export {}
// 抽象类, 只能继承，不能使用 new 去创建对象
// 适用于比较大的概念或者类目，如 Animals

abstract class Animal {
    eat(food: string): void {
        console.log(food);
    }

    abstract run(distance: number): void
}


class Dog extends Animal {
    public name: string
    public age: number

    constructor(name: string, age: number) {
        super();
        this.name = name
        this.age = age
    }

    run(distance: number): void {
        console.log(distance);
    }
}

class Person extends Animal {
    public name: string;
    public age: number;
    private readonly gender: boolean;

    constructor(name: string, age: number, gender: boolean) {
        super();
        this.name = name
        this.age = age
        this.gender = gender
    }

    run(distance: number): void {
        console.log(distance);
    }
}

let d = new Dog('Tom', 2)
d.run(200)          // 200
d.eat('Meat')           // Meat

let p = new Person('SeaChan', 17, true)
p.eat('KFC')            // KFC
p.run(10)             // 10
```

## TS 泛型
在定义接口或函数时，没有定义具体的类型，使用的时候再定义具体类型，能极大程度的复用代码，减少冗余代码。
```javascript
export {}
// 泛型
function createArray<T>(length: number, val: T) {
    return Array<T>(length).fill(val)
}

createArray(3, 100)         // [100, 100, 100]
createArray(3, 'string')    // ['string', 'string', 'string']
```

## TS 类型声明
当使用一些第三个插件或库时，它们不一定是使用TS编写的，就不具有强类型的数据声明，此时可以通过declare进行自定义声明，或者安装相关类型声明模块到开发依赖，如lodash，可以安装@types/lodash。

# 后记
本人于19-20年用过半年的Angular7，但是项目团队中没有系统的TS指导，基本都是在使用一些简单的api，各种CV搞业务，基本没有技术沉淀，只算是接触了一点，现在借着大前端发展的浪头，开始一系列的学习之旅，本文是一些简单的TS学习笔记，更多的需要在实战中去理解和掌握，大前端，冲！！！console.log('加油ヾ(◍°∇°◍)ﾉﾞ')！
