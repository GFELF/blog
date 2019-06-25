---
title: TypeScript使用总结
date: 2019-06-25 23:50:19
tags: ts 
---
### 最近在项目中比较多的用到了typescript，总结了一下需要注意的点和小技巧。

---
<!-- more -->
#### 遇到的坑
#### 类型的使用
  1. 联合类型的使用
   Dinner中要么有fish要么有meat时
   ```typescript
    // good
    interface Dinner {
        fish?: string,
        meat?: string,
    }
    // better
    type Dinner = {
        fish: string,
    } | {
        meat: string,
    }
   ```
#### 类的设计和拆分
#### 小技巧
  1. 在 /** */ 里输入 @ 可以看到丰富的注释选择进行清晰的注释
  2. 
