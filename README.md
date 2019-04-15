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

## 4、Getter
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
但是如果多个组件中需要使用到这个属性的话，就需要不断的复制这个函数，或者抽取到一个公用的文件中去不应用，这样就不是很理想。

Vuex提供了在store中定义“getter”，就像计算属性一样，getter的返回值会根据它的依赖被缓存起来，而且自后当它依赖的值发生改变的时候才会被重新计算。

Getter接受state作为第一个参数
```js
const store = new Vuex.stroe({
  state:{
    todes:[
      {id:1,text:'...',done:true},
      {id:2,text:'...',done:true},
      {id:3,text:'...',done:false},
      {id:4,text:'...',done:false},
    ]
  },
  getters:{
    doneTodes:state=>{
      return state.todes.filter(tode => tode.done)
    }
  }
})
```
### 4.1、getter通过属性访问
getter会暴露为store.getters对象，你可以一苏醒的形式访问这些值
```js
store.getters.doneTodes //返回的是符合filter的数组
```
getter也可以接受其他getter作为第二个参数
```js
getters:{
  //...
  doneTodesCount:(state,getters)=>{
    return getters.doneTodes.length
  }
}

store.getters.doneTodesCount  // -> 1

//我们可以很泳衣的在任何组件中使用
computed:{
  doneTodesCunt(){
    return this.$store.getters.doneTodesCount
  }
}
```
注意，getter 在通过属性访问时是作为 Vue 的响应式系统的一部分缓存其中的。

### 4.2、getter通过方法访问
你也可以通过躺getter返回一个函数，来实现给getter传参。在你对store里的数组进行查询时非常有用。
```js
getters:{
  getterTodoById:(state)=>(id)=>{
    return state.todes.find(todo=>todo.id === id)
  }
}

store.getters.getYodoById(2)  // -> {id:2,text:'...',done:false}
```
注意 getter在通过方法访问的时候，每次都会进行调用，而不会缓存结果

### 4.3、mapGetters 辅助函数
mapGetter 辅助韩式仅仅是通过store中的getter映射到局部计算属性中
```js
import {mapGetters} from 'vuex'

export default{
  //...
  computed:{
    //使用对象展开运算符将getter混入 computed对象中
    ...mapGetters([
      'doneTodesCount',
      'anotherGetter'
    ]),
    //如果想将getter属性另外取一个名字，使用对象形式
    ...mapGetters({
      doneCount:'doneTodesCount'
    })

  }
}
```

## 5、Mutation
更改Vuex的store中的状态的唯一的方法就是提交mutation。Vuex中的mutatuin非常类似于时间：每个mutation都有一个字符串的时间类型和一个回调函数，这个回调函数就是我能实际进行状态改进的地方，并且他会接受state作为第一个参数。
```js
const store = new Vuex.Store({
  state:{
    count:1
  },
  mutations:{
    increment(state){
      state.count++
    }
  }
})
```
但是不能直接调用mutatuin handler，需要以相应的type调用store.commit方法：
```js
store.commit('increment')
```
### 5.1、提交载荷
你可以向store.commit 传入额外的参数，即moutation的载荷：
```js
//...
moutations:{
  increment(state,n){
    state.count +=n
  }
}

store.commit('increment',10)
```
在大多数情况下，载荷应该是一个对象，这样可以包含多个字段并且记录的 mutation 会更易读：
```js
// ...
mutations: {
  increment (state, payload) {
    state.count += payload.amount
  }
}

store.commit('increment', {
  amount: 10
})

```

### 5.2、对象方式提交载荷     
提交mution的另外一种方式就是直接使用type属性的对象
```js
store.commit({
  type:'increment',
  amount:10
})
```      
### 5.3、Mutation需遵守vue的相应规则
Vuex的store中的状态是响应式的，那么当我们变更状态时，监视状态的 Vue 组件也会自动更新。这也意味着 Vuex 中的 mutation 也需要与使用 Vue 一样遵守一些注意事项：
1. 最好提交在你的store中初始化好所以需要的属性。
2. 当需要在对象上添加新属性是应该使用：
  *使用Vue.set(obj,'newProp',123)
  *以新对象替换老对象，例如
    ```js
    state.obj = {
      ...state.obj,newProp:123
    }
    ```
### 5.4、Mutation必须是同步函数
Mutation必须是同步函数这是一条很重要的原则
```js
mutations:{
  someMutation(state){
    api.callAsycMethod(()=>{
      state.coint++
    })
  }
}
```
现在想象，我们正在 debug 一个 app 并且观察 devtool 中的 mutation 日志。每一条 mutation 被记录，devtools 都需要捕捉到前一状态和后一状态的快照。然而，在上面的例子中 mutation 中的异步函数中的回调让这不可能完成：因为当 mutation 触发的时候，回调函数还没有被调用，devtools 不知道什么时候回调函数实际上被调用——实质上任何在回调函数中进行的状态的改变都是不可追踪的。

### 5.5在组件中提交Mutation
你可以在组件中使用this.$store.commit('***')提交mution，或者是使用mapMutations辅助韩式将组件中的methods映射为store.commit调用（需要在根节点中注入store）。
```js
import {mapMutations} from 'vuex'

export default{
  //...
  methods:{
    ...mapMutations([
      'increment',//将this.increment()映射为this.$store.commit('increment')
      //mapMutations也支持载荷
      'incrementBy'//将this.incrementBy(amount)映射为this.$store.commit('incrementBy',amount)
    ]),
    ...mapMutations({
      add:'increment'//将this.add()映射为this.$store.commit('increment')
    })
  }
}
```

## 6、Action
Action类似于mution,不同在于：
* Action提交的是mutation,而不是借鉴变更状态。
* Action可以包含任意异步操作。
让我们来注册一个简单的action:
```js
const store = new Vuex.Store({
  state:{
    cont:0
  },
  mutations:{
    increment(state){
      state.count++
    }
  },
  actions:{
    increment(context){
      context.commit('increment')
    }
  }
})
```
Action函数接受一个与store实例具有相同方法和属性的context对象，因此你可以调用context.commit提交一个mutation,或者通过context.state和context.getters来获取state和getters。

五一放假我回家
