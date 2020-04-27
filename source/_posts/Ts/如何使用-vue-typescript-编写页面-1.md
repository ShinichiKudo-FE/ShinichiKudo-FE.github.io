---
title: 如何使用 vue + typescript 编写页面(1)
date: 2019-07-15 17:10:33
tags: "Ts"
categories: "Js"
---

[原文地址](https://juejin.im/post/5c662a7de51d4562e71c4277#comment)
# 起步

使用 vue-cli 创建项目

`vue create demo` ，demo就是创建项目的名称
提示选择预设，选择 `Manually select features` 回车确认
选择`typescript` `vuex`  `router` `babel` `css pre-processors`，不使用`linter`，不选单元测试有需要另说
回车后按照需要选择合适的选项
选择完毕后回车等待资源准备

## 编码开始

熟悉几个vue的装饰器 `vue-property-decorator`

以下的装饰器的功能和原js编写的功能相同/相似，可以参照官方文档类比解读。

```js
import { Vue, Component, Inject, Provide, Prop, Model, Watch, Emit, Mixins } from 'vue-property-decorator'
```

### 1. Vue 实际上就是 Vue 本身,继承vue相关属性和函数

```js
class MyComponent extends Vue { }
```

### 2. @Component 声明成一个vue的组件实例，如果不使用，则不能得到一个vue组件

第一种方式，不需要定义额外内容:

```js
@Component
class MyComponent extends Vue { }
```

第二种方式，定义相关内容:

```js
@Component({
    /* 这里和js版本编写的 vue 组件内容相同，
     * 凡是不能在ts里面完成的都可以在这里完成 
     * 最终会被合并到一个实例中。
     * 在这里定义的内容，不会被语法器获取到，因此必须要同步在class中声明
     */
    data(){
        return { myname:"",age:18 }
    }
})
class MyComponent extends Vue {
    private myname:string;
    mounted(){
        this.myname;    
        this.age;// 语法器报错，当前类找不到age属性
    }
}
```

### 3.@Provide 向任意层级的子组件提供可访问的属性，默认为当前属性的名称，可以指定名称

```js
@Component
class ParentComponent extends Vue { 
    @Provide() private info!:string;
    @Provide("next") private infoNext!:string;
}
```

### 4. @Inject 获取父级由Provide提供的属性，默认为当前属性的名称，可以指定名称，多个父级提供相同名称属性时，获取最近父级的名称属性

```js
@Component
class MyComponent extends Vue { 
    @Inject() private info!:string;
    @Inject("next") private infoNext!:string;
}
```

### 5. @Prop 由标签属性注入，获取对应标签属性上值,可配置具体prop内容，参照js版本props内容

```js
@Component
class MyComponent extends Vue { 
   @Prop() age!:number;
   @Prop({default:1}) sex!:number;
}
```

```html
<template>
    <MyComponent :age="16" />
</template>

<script lang="ts">  
import MyComponent from './MyComponent.vue';

@Component({
    components:{ MyComponent }
})
class PComponent extends Vue { }
</script>
```
<!-- more -->
### 6. @Model 是v-model的装饰器，当自定义组件想使用v-model时，可以使用这种方式，配合emit可以双向传递属性值

```html
<template>
    <input type="checkbox" :checked="checked" @change="changed"/>
</template>

<script lang="ts">
@Component
class MyComponent extends Vue { 
   @Model("change") checked!:number;
   changed(event:any){ /* 这里是偷懒写的any，在实际项目中需要避免 */
       this.$emit("change",event.target.value)
   }
}
</script>
```

```html
<template>
    <MyComponent :age="16" v-model="mycheck" />
</template>

<script lang="ts"> 
import MyComponent from './MyComponent.vue';

@Component({
    components:{ MyComponent }
})
class PComponent extends Vue {
    private mycheck:boolean = false;
}
</script>
```

### 7. @Watch 观察某个属性更新
```js
@Component
class MyComponent extends Vue { 
   @Prop() age!:number;
   @Watch("age")
   ageChange(newVal:number,oldVal:number){
       /*age属性更新时，处理相关内容*/
   }
}
```

### 8. @Emit this.$emit 的装饰器,如果没有指定名称，默认使用函数名称。有返回值时，使用返回值，没有则使用

```js
@Component
class MyComponent extends Vue { 
   private myname = "";
   
   @Emit()
   ageChangeA(){ /* 仅发送 this.$emit('ageChangeA') */   }
   
   @Emit()
   ageChangeB(age:number){ /* 发送 age  this.$emit('ageChangeB',age) */   }
   
   @Emit()
   ageChangeC(age:number){  return 1 /* this.$emit('ageChangeC',1) 发送return 结果*/ }
}

```

### 9. Mixins

```js
// MyMixin.ts
@Component
export default class MyMixin extends Vue { 
   /* 如果使用private 修饰，则两个相同的 私有属性混入时，会产生冲突 */
   protected myname = "张三";
   created(){  /* 混入对象有自己的 生命周期函数*/ }
   getMyName(){ console.log("张三混入") }
}
```

```js
// OtherMixin.ts
@Component
export default class OtherMixin extends Vue { 
   /* 如果使用private 修饰，则两个相同的 私有属性混入时，会产生冲突 */
   protected myname = "李四";
   created(){  /* 混入对象有自己的 生命周期函数*/ }
   getMyName(){ console.log("李四混入") }
}
```

```js
@Component
class MyComponent extends Mixins(MyMixin,OtherMixin) { 
   private myname = ""; /* 混入对象已经定义，这里产生属性冲突 */
   mounted(){
       this.getMyName() // 李四混入
   }
}
```

### 对于computed，使用 get 替代

```js
@Component
class MyComponent extends Vue { 
   private myname = ""; /* 混入对象已经定义，这里产生属性冲突 */
   get upperName(){
       return "A" + this.myname
   }
}
```

**装饰器可以参照 [vue-property-decorator](https://link.juejin.im/?target=http%3A%2F%2Fnpm.taobao.org%2Fpackage%2Fvue-property-decorator)**

* 没有filters,没有指令相关装饰器，有需要可以在@Component里面补充，或者可以直接定义函数调用计算返回值 在class里定义的属性即data属性，需要赋值初始值。

## typescript 混用js

当我们引用了一个js编写的模块时会报错，这时候，如果不在引入的index文件里面添加.d.ts描述文件，那么这个模块就没法在语义上一致通过.

### 通用兼容性解决方案

如果这个模块是由npm下载的，并且有@types版本，可以直接使用`npm install @types/xx`下载

可惜,大部分都是没有编写声明文件的.但是又需要使用到这个模块时,应该怎么做.

**自定义描述文件**

在项目的【根目录】下定义【模块相同的名称】的描述文件A.d.ts,在描述文件内编写模块声明描述

```js
// A.d.ts
declare module '*';
```

这样属于一劳永逸的描述,但是无法描述到具体模块内容,因此这种方式仅仅引入的组件没有任何操作时,比如引入的是第三方开发的vue组件,可以用这种方式偷懒.

**详细描述文件**

```js
// A.d.ts
declare module  "A" {
    // 添加具体的描述内容
};
```

可以从这里查询到[@type](http://microsoft.github.io/TypeSearch/)包查询，已经被编写的文件声明可以直接下载