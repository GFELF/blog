---
title: TypeScript使用总结
author: GFELF
date: 2019-06-25 23:50:19
tags: ts 
---
### 最近在项目中比较多的用到了typescript，总结了一下遇到的坑和一些小技巧。

---
<!-- more -->
#### 遇到的坑
1. bable和ts的配置项和模块规范导致的问题
    在项目中把一些历史代码由js改造为ts后，某些单元测试会无法跑通，如下。
    ```javascript
    import Vue from 'vue';
    Vue.directive(....);
    ```
    {% asset_img jest报错.png %}
    原因是ts对CommonJS/AMD/UMD都采取了和ES6 modules一样的处理方式，如
    ```javascript
    // you wrote
    import * as foo from 'foo';
    // treated as equivalent to
    const foo = require("foo");

    // you wrote
    import bar from 'foo';
    // treated as equivalent to
    const bar = require("foo").default;
    ```
    而Vue没有默认导出，所以引发了上面的问题。
    相关github PR: <https://github.com/Microsoft/TypeScript/pull/2460/>
    这个问题在tsconfig.json 中设置 `esModuleInterop: true` 即可兼容。
    但是更改设置后会引发一些潜在的其他问题，因为在项目中使用了d3js并采用了如下引入方式
    ```javascript
    import * as d3 from 'd3';
    d3.event ... // undefined
    d3.select ... // undefined
    ```
    原因是
    > If you use Babel, Webpack, or another ES6-to-ES5 bundler, be aware that the value of d3.event changes during an event! An import of d3.event must be a live binding, so you may need to configure the bundler to import from D3’s ES6 modules rather than from the generated UMD bundle; not all bundlers observe jsnext:main. Also beware of conflicts with the window.event global.
    
    此时需要对d3的模块导入单独处理一些，使用`import {} from 'd3'`或者`require`的形式引入即可。

#### 类型的使用
1. 联合类型的使用
   Dinner中要么有fish要么有meat时
   ```typescript
    // good
    interface Dinner1 {
        fish?: string,
        meat?: string,
    }
    // better
    type Dinner2 = {
        fish: string,
    } | {
        meat: string,
    }
    //i.e.
    let a: Dinner1 = {} // OK
    let b: Dinner2 = {} // Protected
   ```
2. 类型断言的使用
   - 下面的代码不会提示错误，在运行时才会报错
   ```typescript
    let str;
    let foo: number = (str as string).length;
    let foo2: number = (<string>str).length;
   ```
   好处是可以解决掉恼人的错误提示，潜在的影响是如果断言与实际情况不符可能会引发运行时错误，个人建议最好不要用any做断言。
   - 用类型断言手动去除null和undedined
   ```typescript
    let a = document.getElementById('foo');
    a.tagName // 提示可能为null
    a!.tagName // OK
   ```
3. *.d.ts的使用
    在类型定义文件中定义好用到的接口和类型，可以获得舒适的编码体验。
4. ts的结构类型(鸭子类型)系统
    ```typescript
    class Foo {
        func(arg:number): number {
            // do something
        }
        xxx() {}
    }
    class Bar {
        func(arg:number): number { 
            // do something
        }
    }
    let instance: Bar = new Foo(); // OK
    ```
    TypeScript 比较的不是类型定义本身，而是类型定义的形状，即各种约束条件。
    >One of TypeScript’s core principles is that type checking focuses on the shape that values have. This is sometimes called “duck typing” or “structural subtyping”.

#### 类的设计和拆分
1. 面向对象的基本原则
    - 用组合代替继承（has a vs is a）
    - 里氏替换原则
    - 依赖倒转原则
    - 单一职责原则
    .......
    在现在的实现中有些做的不好的地方，比如
    ```typescript
    import {EventEmitter} from 'eventemitter3';
    export default class Foo extends EventEmitter {...}
    ```
    此处不应该使用继承的方式来使用事件触发器，因为Foo类的目的不是扩展事件触发，而且这样实现增加了代码之间的耦合。
    更好的方式是在Foo类中创建Emitter的实例来使用，或者在Foo的构造函数中进行注入。
2. 拆分

#### 小技巧
1. 在 /** */ 里输入 @ 可以看到丰富的注释选择进行清晰的注释
{% asset_img 注释提示.png 注释提示 %}
2. 通过/** */形式的注释可以给接口或类型做标记，编辑器会有更好的提示
```typescript
/** 树形结构节点坐标 */
interface Coordinate {
    readonly x: number,
    readonly y: number
}
```
{% asset_img 注释标记.png 类型标记 %}
