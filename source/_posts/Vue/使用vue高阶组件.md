---
title: 使用vue高阶组件
date: 2019-11-25 10:48:54
tags: "Vue"
categories: "Vue"
---

# 前文

`高阶组件(HOC)`是一种架构模式，在 React 中非常常见，但也可以在 Vue 中使用。它可以被描述为一种**在组件之间共享公共功能而不需要重复代码**的方法。HOC 的目的是增强组件的功能。它允许在项目中实现可重用性和可维护性。

- 只要你向一个方法传入组件，然后返回一个新的组件，这就是一个 HOC。

> 高阶组件在以下方面非常有用: 1.操作属性。 2.操作数据和数据抽象。 3.代码重用

## 预备知识

在我们开始教程之前，需要了解以下几点：

使用 Vue 框架的经验。
知道如何使用 vue-cli 设置应用程序。
JavaScript 和 Vue 的基本知识
Node (8)
npm (5.2.0)

## Vue 中的高阶组件模式

虽然高阶组件通常与 React 相关联，但是为 Vue 组件创建高阶组件是很有可能的。在 Vue 中创建高阶组件的模式如下所示。

```js
// hoccomponent.js

import Vue from "vue"
import ComponentExample from '@/components/ComponentExample.vue'
const HoComponent = (component) => {
    return Vue.component('withSubscription', {
        render(createElement) {
            return createElement(component)
        }
    }
}
const HoComponentEnhanced = HoComponent(ComponentExample);

```

如上面的代码块所示，`HoComponent` 函数接受一个组件作为参数，并创建一个新组件来渲染传进来的组件。

## 一个简单的 HOC 示例

在本教程中，我们将介绍一个使用高阶组件的示例。在介绍高阶组件之前，我们将了解在没有高阶组件的情况下，当前的代码库是如何工作的，然后了解如何进行抽象。

[https://codesandbox.io/embed/llvq04nx4l](https://codesandbox.io/embed/llvq04nx4l)

正如上面的 CodeSandbox 所示，该应用程序会显示一个纸业公司及其净资产的列表，以及《办公室》(美国)中的人物及其获奖情况。

我们获得应用程序所需的所有数据来源只有一个，那就是`mockData.js`文件。

<!-- more -->

```js
// src/components/mockData.js
const staff = [
  {
    name: "Michael Scott",
    id: 0,
    awards: 2
  },
  {
    name: "Toby Flenderson",
    id: 1,
    awards: 0
  },
  {
    name: "Dwight K. Schrute",
    id: 2,
    awards: 10
  },
  {
    name: "Jim Halpert",
    id: 3,
    awards: 1
  },
  {
    name: "Andy Bernard",
    id: 4,
    awards: 0
  },
  {
    name: "Phyllis Vance",
    id: 5,
    awards: 0
  },
  {
    name: "Stanley Hudson",
    id: 6,
    awards: 0
  }
];
const paperCompanies = [
  {
    id: 0,
    name: "Staples",
    net: 10000000
  },
  {
    id: 1,
    name: "Dundler Mufflin",
    net: 5000000
  },
  {
    id: 2,
    name: "Michael Scott Paper Company",
    net: 300000
  },
  {
    id: 3,
    name: "Prince Family Paper",
    net: 30000
  }
];

export default {
  getStaff() {
    return staff;
  },
  getCompanies() {
    return paperCompanies;
  },
  increaseAward(id) {
    staff[id].awards++;
  },
  decreaseAward(id) {
    staff[id].awards--;
  },
  setNetWorth(id) {
    paperCompanies[id].net = Math.random() * (5000000 - 50000) + 50000;
  }
};
```

在上面的代码片段中，有几个 const 变量保存了公司和员工列表的信息。我们也导出了一些函数，

> 实现以下功能：
> 返回员工列表
> 返回公司列表
> 增加和减少对特定员工的奖励，最后
> 最后，设定公司的净值

接下来，我们看看 `Staff.vue` 和 `Companies.vue` 组件。

```html
// src/components/Staff.vue
<template>
  <main>
    <h3>Staff List</h3>
    <div v-for="(staff, i) in staffList" :key="i">
      {{ staff.name }}: {{ staff.awards }} Salesman of the year Award 🎉
      <button @click="increaseAwards(staff.id);">+</button>
      <button @click="decreaseAwards(staff.id);">-</button>
    </div>
  </main>
</template>
<script>
  import mockData from "./mockData.js";
  export default {
    data() {
      return {
        staffList: mockData.getStaff()
      };
    },
    methods: {
      increaseAwards(id) {
        mockData.increaseAward(id);
        this.staffList = mockData.getStaff();
      },
      decreaseAwards(id) {
        mockData.decreaseAward(id);
        this.staffList = mockData.getStaff();
      }
    }
  };
</script>
```

在上面的代码块中，数据实例变量staffList被赋值为函数mockData.getStaff()返回的内容。我们也有increaseAwards和decreaseAwards函数，分别调用mockData.increaseAward 和 mockData.decreaseAward。传递给这些函数的id 是从渲染的模板中获得的。

```html
    // src/components/Companies.vue
    <template>
      <main>
        <h3>Paper Companies</h3>
        <div v-for="(companies, i) in companies" :key="i">
          {{ companies.name }} - ${{ companies.net
          }}<button @click="setWorth(companies.id);">Set Company Value</button>
        </div>
      </main>
    </template>

    <script>
      import mockData from "./mockData.js";
        export default {
        data() {
          return {
            companies: mockData.getCompanies()
          };
        },
        methods: {
          setWorth(id) {
            mockData.setNetWorth(id);
            this.companies = mockData.getCompanies();
          }
        }
      };
    </script>

```

在上面的代码块中，数据实例变量companies 被赋值为函数mockData.getCompanies()的返回内容。我们还有setWorth函数，它通过将公司的 id 传递给mockData.setNetWorth来设置一个随机值作为净值。传递给函数的id是从渲染的模板中获得的。


现在我们已经看到了这两个组件是如何工作的，我们可以
> 找出它们之间的共同点，并将其抽象如下：

从数据源获取数据 (mockData.js)
onClick 函数

我们来看看如何将上面的操作放到高阶组件中，以避免代码重复并确保可重用性。


HoComponent 函数接受两个参数，一个组件和fetchData。fetchData方法用于确定要在表示组件中显示什么。这意味着无论在哪里使用高阶组件，作为fetchData 传递的函数都将被用来从mockData 中获取数据。

然后将数据实例returnedData 设置为fetchData的内容，然后作为props 传递给在高阶组件中创建的新组件。

让我们看看新创建的高阶组件如何在应用程序中使用。我们需要编辑Staff.vue 和Companies.vue。


```html
    // src/components/Staff.vue
    <template>
      <main>
        <h3>Staff List</h3>
        <div v-for="(staff, i) in returnedData" :key="i">
          {{ staff.name }}: {{ staff.awards }} Salesman of the year Award 🎉
          <button @click="increaseAwards(staff.id);">+</button>
          <button @click="decreaseAwards(staff.id);">-</button>
        </div>
      </main>
    </template>
    <script>
      export default {
        props: ["returnedData"]
      };
    </script>

```
```html
    // src/components/Companies.vue
    <template>
      <main>
        <h3>Paper Companies</h3>
        <div v-for="(companies, i) in returnedData" :key="i">
          {{ companies.name }} - ${{ companies.net
          }}<button @click="setWorth(companies.id);">Set Company Value</button>
        </div>
      </main>
    </template>
    <script>
      export default {
        props: ["returnedData"]
      };
    </script>

```
正如你在上面的代码块中看到的，对于这两个组件，我们去掉了函数和数据实例变量，显示内容所需的所有数据现在都将从这些props中获得。对于删掉的函数，我们将很快会讲到。


在 App.vue 组件中，用以下代码编辑script 标签中的现有内容：

```html
    // src/App.vue
    <script>
      // import the Companies component
      import Companies from "./components/Companies";
      // import the Staff component
      import Staff from "./components/Staff";
      // import the higher order component
      import HoComponent from "./components/HoComponent.js";

      // Create a const variable which contains the Companies component wrapped in the higher order component
      const CompaniesComponent = HoComponent(Companies, mockData => mockData.getCompanies()
      );

      // Create a const variable which contains the Staff component wrapped in the higher order component
      const StaffComponent = HoComponent(Staff, mockData => mockData.getStaff());

      export default {
        name: "App",
        components: {
          CompaniesComponent,
          StaffComponent
        }
      };
    </script>

```

在上面的代码块中，HoComponent用于包装 Staff 和 Companies 组件。

每个组件作为HoComponent 的第一个参数传入，第二个参数是一个函数，它返回另一个函数，指定应该从mockData获取什么数据。这是我们之前创建的高阶组件(HoComponent.js)中的fetchData 函数。

如果你现在刷新应用程序，你应该仍然可以看到来自mockData 文件的数据像往常一样呈现。唯一的区别是，这些按钮无法工作，因为它们还没有绑定到任何函数。让我们解决这个问题。

```html
    // src/App.vue
    <template>
      <div id="app">
        <CompaniesComponent @click="onEventHappen" />
        <StaffComponent @click="onEventHappen" />
      </div>
    </template>

    <script>
      // import the Companies component
      import Companies from "./components/Companies";
      // import the Staff component
      import Staff from "./components/Staff";
      // import the higher order component
      import HoComponent from "./components/HoComponent.js";
      // import the source data from mockData only to be used for event handlers
      import sourceData from "./components/mockData.js";
      // Create a const variable which contains the Companies component wrapped in the higher order component
      const CompaniesComponent = HoComponent(Companies, mockData =>
      mockData.getCompanies()
      );
      // Create a const variable which contains the Staff component wrapped in the higher order component
      const StaffComponent = HoComponent(Staff, mockData => mockData.getStaff());

      export default {
        name: "App",
        components: {
          CompaniesComponent,
          StaffComponent
          },
        methods: {
          onEventHappen(value) {
            // set the variable setFunction to the name of the function that was passed iin the value emitted from child component i.e. if value.name is 'increaseAward', setFunction is set to increaseAward()
            let setFunction = sourceData[value.name];
            // call the corresponding function with the id passed as an argument.
            setFunction(value.id);
          }
        }
      };
    </script>

    <style>
    #app {
      font-family: "Avenir", Helvetica, Arial, sans-serif;
      -webkit-font-smoothing: antialiased;
      -moz-osx-font-smoothing: grayscale;
      text-align: center;
      color: #2c3e50;
      margin-top: 60px;
    }
    </style>
```

在上面的代码块中，我们在App.vue组件中添加了一个事件监听器。

vue的组件。每当Staff.vue 或Companies.vue 组件被点击时，onEventHappen 方法将会被调用。

在onEventHappen方法中，我们将变量setFunction 设置为从子组件发出的值中传递的函数名，也就是说，如果value.name是'increaseAward'，那么setFunction设置为increaseAward()。setFunction 将以id作为参数执行。

最后，为了将事件监听器传递给封装在高阶组件中的组件，我们需要在 HoComponent.js文件中添加下面这行代码。

```js
    props: {
    returnedData: this.returnedData
    },
    on: { ...this.$listeners }

```

## vue-hoc
或者，您可以使用[vue-hoc](https://github.com/jackmellis/vue-hoc)库来帮助创建高阶组件。vue-hoc帮助你轻松地创建高阶组件，你要做的就是传递基本组件、应用于HOC的一系列组件选项和在渲染阶段传递给组件的数据属性。
vue-hoc 可用如下命令安装：
    `npm install --save vue-hoc`

复制代码vue-hoc插件有一些例子可以让你开始创建更高阶的组件，你可以[查看这里](https://github.com/jackmellis/vue-hoc/blob/master/packages/vue-hoc/README.md).