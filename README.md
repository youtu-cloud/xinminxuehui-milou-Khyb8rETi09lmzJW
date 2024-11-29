

> Vue3`组件通信和`Vue2\`的区别：
> 
> 
> * 移出事件总线，使用`mitt`代替
> 
> 
> * `vuex`换成了`pinia`
> * 把`.sync`优化到了`v-model`里面
> * 把`$listeners`所有的东西，合并到`$attrs`
> * `$children`被移除
> 
> 
> [![image-20241128115451741](https://img2023.cnblogs.com/blog/1422712/202411/1422712-20241128115453502-1630228381.png)](https://github.com)


#### 自定义事件(与vue2语法差异)


自定义事件常用于子组件向父组件传递数据


原生事件:特点的click、hover等


自定义事件:事件名是任意名称，事件对象`$event`: 是调用`emit`时所提供的数据，可以是任意类型


1. 在子组件声明自定义事件



```
// 使用defineEmits在子组件声明一个test自定义事件
// 参数是一个数组，可以包含多个自定义事件
const emit = defineEmits(["test"])

```
2. 在父组件绑定自定义事件



```
  <!-- 在父组件的子组件标签里面，绑定test事件，触发getInfo回调 -->
<!-- 回调函数在父组件中 -->
  <UserInfo @test="getInfo"></UserInfo>

```
3. 在子组件触发自定义事件



```
// 在子组件中主动触发自定义事件
// 触发事件的名称，要传递的数据
emit("test",100)

```


#### mitt



> 与消息订阅与发布（`pubsub`）功能类似，可以实现任意组件间通信


1. 安装mitt



```
npm i mitt

```
2. 创建mitt



```
//  src/utils/emitter.ts

// 引入
import mitt  from "mitt";

// 创建emitter,emitter可以绑定、触发事件
const emitter = mitt()

export default emitter



```
3. 绑定事件



```
// 要绑定的事件名，和处理的回调方法
emitter.on("test1",(value)=>{
  console.log("on test1"，value)
})

```
4. 触发事件



```
// 要触发的组件名和传递的参数，在任意地方触发，所有绑定了对应事件的地方都会收到
emitter.emit("test1", 6666)

```
5. 查看全部事件



```
emitter.all

```
6. 解绑指定事件



```
emitter.off("test1")

```
7. 清理全部事件



```
emitter.all.clear()

```
8. 在接收数据的组件中，组件销毁之前解绑事件



```
// 生命周期钩子
onUnmounted(()=>{
  // 解绑事件
  emitter.off('test')
})

```


#### v\-model(与vue2实现区别)


实现父组件、子组件直接互相通信


1. v\-model的原理



```

<input type="text" v-model="user">


<input 
  type="text" 
  :value="user" 
  @input="user =($event.target).value"
>

```
2. 组件标签上的v\-model



```

<UserInfo v-model="user">UserInfo>


<UserInfo :modelValue="user" @update:model-value="user = $event" >UserInfo>






```

父组件和子组件双向通信：



```
<UserInfo v-model="user">UserInfo>

```

在子组件中



```
// 接收v-model的modelValue
defineProps(["modelValue"])

// 声明事件
const emit = defineEmits(['update:model-value'])

```


```
 
<input type="text" :value="modelValue" @input="emit('update:model-value',($event.target).value)">


```
3. 修改modelValue



```

<UserInfo v-model:test="user">UserInfo>


<UserInfo v-model:test="user" v-model:test2="user2">UserInfo>

```


```
// 接收
defineProps(["test","test2"])

// 声明
const emit = defineEmits(['update:test','update:test2'])

```


```
<input type="text" :value="test" @input="emit('update:test',($event.target).value)">

```


#### $attrs



> `$attrs`用于实现当前组件的父组件，向当前组件的子组件通信（祖→孙）
> 
> 
> `$attrs`是一个对象，包含所有父组件传入的标签属性,会自动排除`props`中声明的属性(可以认为声明过的 `props` 被子组件自己消费了)


1. 父组件



```
 
<UserInfo :list="list" :name="user">UserInfo>

```
2. 当前组件


props只接收了name



```
// 当前组件只接受了name
defineProps(["name"])

```

使用attrs将没有接收的数据，向子传递



```
  <Info v-bind="$attrs">Info>

```
3. 子组件



```
// 祖先传递的props，父组件没有接收，通过attrs，在孙组件使用props接受
// 如果在父组件props使用了，则子组件接收不到
defineProps(['list'])

```
4. 可以在祖先中实现一个方法，传递给子孙，子孙中触发，实现子孙向祖先传递数据


#### $refs



> `$refs`用于 ：父向子传递数据
> 
> 
> * 值为对象，包含所有被`ref`属性标识的`DOM`元素或组件实例


1. ref属性



```
  <UserInfo ref="userObj ">UserInfo>

```


```
// 通过该方法可以获取子组件实例对象，进行对应的操作，在子组件要进行defineExpose操作，将数据交给外部
let userObj = ref()


```
2. $refs



```
// 回调函数，触发的地方参数传递$refs
function getAll(v){
  // v是一个对象，包含所有有ref属性的子组件实例、DOM元素
  console.log(v)
}

```


```
 
<button @click="getAll($refs)">button>

```


#### $parent



> `$parent`用于：子向父传递数据
> 
> 
> * 值为对象，当前组件的父组件实例对象


1. 父组件中，使用defineExpose将允许子操作的数据暴露出去
2. 子组件



```
// 回调函数，parent需要传递$parent
function getAll(parent){
  // parent是一个对象，是当前组件的父组件的实例
  console.log(parent)
}

```


```
 
<button @click="getAll($parent)">button>

```


#### provide和inject



> 实现祖孙组件直接通信，不需要通过中间组件


1. 在祖先中通过provide配置向后代组件提供数据



```
import {provide} from 'vue'
// 给子孙提供的key、Value
provide("user",user)
// 也可以value传递一个对象，批量传递数据，也可以传递方法

```
2. 在所有后代中都能通过inject配置声明接收的数据



```
import {inject} from 'vue'
// 接收祖先中传递的user
let user = inject("user")

```


#### 插槽(部分语法区别)


##### 默认插槽


1. 父组件中编写代码



```

  <UserInfo>
    <p>hello worldp>
    
  UserInfo>


```
2. 子组件



```
<template>
  
  <slot>    <p>defaultp>slot>

template>

```


##### 具名插槽



> 使用名字定义、使用插槽，可以插入多段不同名字的插槽代码
> 
> 
> 如果不定义名字，默认插槽的名字默认是default，在组件中使用多个默认擦嘴，因为都是default，会有数量问题


1. 给插槽定义名字



```
  <UserInfo>
    
    <template v-slot:p-message>
      <p>hello worldp>

    template>

  UserInfo>

```


```

 <template #p-message>

```
2. 子组件使用



```
  <slot name="p-message"></slot>

```


##### 作用域插槽



> 数据在组件的自身，数据生成的结构由组件的使用者来决定


1. 子组件



```

<slot :listData='listData'>slot>

```
2. 父组件



```
 

<template v-slot="params">
			
       
       
       <ul>
         <li v-for=" data of params.ListData" :key="data.id">{{data.name}}li>
       ul>

    template>

```


 本博客参考[veee加速器官网](https://youhaochi.com)。转载请注明出处！
