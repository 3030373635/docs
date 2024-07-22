---
title: vuex
---



# 一、四个核心部分

## 1.1 state

## 1.2 actions

## 1.3 mutations

## 1.4 getters

## 1.5 Modules

## 二、使用

### 2.1 引入

```html
import Vue from "Vue"
import Vuex from 'vuex'
```

### 2.2 挂载

```html
Vue.use(Vuex)
```

### 2.3 逻辑

#### 2.3.1 main.js中

```html
const store = new Vuex.Store({
    state: {
        num: 2
    },
    mutations: {
        set_num1(state,val){
            state.num += val
        },
        set_num2(state,val){
           state.num += val

        }
    },
    actions: {
        set_num1(context,val){
            console.log(context)
            context.commit("set_num1",val)
        },
        set_num2(context,val){
            setTimeout(()=>{
                context.commit("set_num2",val)
            },1000)

        }

    }
})
```

#### 2.3.2 组件中

```html
methods:{
            add_num1(){
               this.$store.dispatch("set_num1",1)
            },
            add_num2(){
               this.$store.dispatch("set_num2",5)
            }
        }
```

### 2.4 挂载

```html
new Vue({
    el: '#app',
    store,
    router,
    components: {App},
    template: '<App/>'
})