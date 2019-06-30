---
layout: post
title: "Vue.js: basic concepts"
description: ""
categories: [vue]
tags: []
redirect_from:
  - /2019/03/09/
---


普通的vue instance物件的data attribute可以是一個物件，但是vue component的data一定要是function，因為這樣它才會建立專屬於這個vue component的this object，讓這個local vue component有自己的local state。

----------------------------------------------------------------

研究vue router的時候讀的前端路由概念相關文章：

[前端路由简介以及vue-router实现原理](https://zhuanlan.zhihu.com/p/37730038)

[跟著 Vue 闖盪前端世界 - 08 網站路由 vue-router](https://dotblogs.com.tw/wasichris/2017/03/06/235449)

[官方文件](https://router.vuejs.org/zh/guide/#javascript)

[Vue.js 技术揭秘](https://ustbhuangyi.github.io/vue-analysis/vue-router/)

前端router和後端的概念有點不同，不是用來redirect到新的頁面，而是在一個SPA裡面切換視圖，一個大視圖裡面可以包含很多小視圖（一個或多個元件），有些SPA甚至會有複數個大視圖，在前端router概念出現之前，切換內容卻不換頁是使用hash（https://root_path/#/xxxx）。

-------------------------------------------------------------------------------------

什麼是Vuex?

A state management pattern and library.

沒有使用Vuex的時候，各個local component 各自擁有自己的local data，但是Vuex可以用來中央管理global state。

Vuex三要素：
- The Store: creates the global state object
- Mutations: change the state
- Actions: commit mutations

------------------------------------------------------------------------------------

[Javascript import](https://stackoverflow.com/questions/36795819/when-should-i-use-curly-braces-for-es6-import)

如果A.js這樣寫：`export default 42`

那就是default import，在其他檔案裡怎麼命名都可以：
in B.js:
~~~javascript
import A from './A.js'   //  => A == 42
import a from '.A.js'    // => a == 42
~~~

如果是named import，A.js: `export const A = 42`
那就一定要指定這個名稱才行：
in B.js:
~~~javascript
import { A } from './A.js'  // => A == 42
~~~

-------------------------------------------------------------------------------------

`$store.dispatch` triggers the action
