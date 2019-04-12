# Vuex-study
Vuex系统学习笔记
---
# vuex学习笔记
---
## 1、vuex简介
每一个Vuex应用的核心就是store（仓库）。store就是一个容器，它存储这应用中的大部分状态
## 2、Vuex与全局对象的差别
1. Vuex的状态存储是响应式的，当Vue组件从store中读取状态的是你，如果store中的状态发生改变，那么相应的组件也会更新
2. 最好不要直接改变store中的状态，改变store中的状态的唯一的途径就是显式提交（commit）mutation。
---
## 3、state
### 3.1、state概念
vuex使用单一状态树，用一个对象就包含了全部的应用层级状态，它作为了唯一的数据源，每一个应用只能包含一个store实例，单一状态树与模块化开发并不冲突。
### 在vue组件中获取vuex状态
由于vuex的状态存储是响应式的，从store实例中读取状态的最简单的额方法就是在计算属性中返回某一个状态。
```js
const Counter = {
    template:`<div>{{cont}}</div>`,
    computed:{
        count(){
            return store.state.count
        }
    }
}
```
每当==store.state.count==变化的时候，就会重新求取计算属性，并且触发DOM更新。但是这种状态在模块化的系统构建系统中，在每一个需要使用state的组件中需要频繁的导入，并且在测试组件是需要模拟状态。
Vuex通过store选项，提供了一种机制将状态从根组件注入到每一个子组件中。
```js
import Vue from 'vue'
import Vuex from 'vuex'
Vue.use(Vuex)
const app = new Vue({
    el:'#app',
    store,
    component:{
        Counter
    },
    template:`
        <div id="app">
            <counter></counter>
        </div>
    `
})
```
通过在根实例中注册==store==选项，该==store==实例会注入到根组件下的所以子组件中，且子组件能够通过==this.$store==访问。更新Counter
```js
count Counter = {
    template:`<div>{{ count }}</div>`,
    computed:{
        count(){
            return this.$store.state.count
        }
    }
}
```
### 3.2、mapState辅助函数
当一个组件需要很多状态的时候，将这些状态都声明为计算属性会做编辑很多重复的代码，为了解决这个问题，可以使用mapState辅助函数来生成计算属性。
```js
//单独构建的版本中辅助函数为Vuex.mapState
import {mapState} from 'vuex'
export default{
    //...
    computed:mapState({
        //箭头函数可以使代码更简练
        count:state => state.count,
        //传字符串参数count 等于 'state => state.count'
        countAlias:'count',
        //但是如果需要使用this来获取局部状态，必须使用常规的函数
        countPlusLocalState(state){
            return state.count + this.localCount
        }
        
    }
}
```
当映射的计算属性的名称与state的子节点名称相同是，也可以给mapState传一个字符串数组。
```js
computed:mapState([
    'count'
])
```
### 3.3、mapState使用过程中结合对象的展开运算符使用
mapState函数返回的是一个对象，当组件中的计算属性中有其他的属性时，将会很繁琐，可以结==合对象运算符==大大简化代码写法：
```js
computed:{
    localComputed(){},
    //使用展开运算符将此对象混入到外部对象中
    ...mapState({
      //...  
    })
    /*
    ...mapState([
        'count'
    ])
    */
}
```
### 3.4、组件仍然要保留有局部的状态
使用Vuex并不需要将所有的状态都放到Vuex里面，如果全部放入的话代码量会很大，如果有些状态严格属于单个组件，最好还是在子组件中单独设置状态。

---

## 2、Getter
有时候在store中的state中派生出一些状态，例如对列表进行过滤计数：
```js
computed:{
    doneTodosCount(){
        return this.$store.state.todes.filter(tode =>tode.done).length
    }
    /*
    doneTodosCount(){
        return this.$store.state.todes.filter((val,index,arr)=>{
            return val.done
        })
    }
    */
}
```