---
title: '''基于react搭建一个通用的表单管理配置平台（vue同）'''
tags: 'React/Vue'
categories: 'React/Vue'
date: 2020-04-25 22:36:43
---

[原文地址](https://juejin.im/post/5ea1af9f518825737a316cc3?utm_source=gold_browser_extension)

# 前言

最近在做B端的产品，借助该博主的文章学习下通用表单，以下为博文信息

这篇文章是一篇应用性极强的文章，我们通过一个实际的应用场景，去解决某一类的问题，提供一种或者几种解决方案，来探索技术的魅力。接下来笔者主要分析表单定制平台的实现思路和技术方案，来实现一个类似于金数据或者问卷星一样的表单配置平台，大家也可以基于此方案，扩展出功能更加强大的可视化平台。

# 正文

为什么要做一个这样的平台呢？一方面是因为笔者多年来一直服务于B端产品，对于动态表单以及配置化表单有一定的项目积累，并且深知配置化表单的价值所在。举一个很传统的B端表单配置化的例子：传统2B企业在提供saas服务时，为了满足不同企业的定制化需求，往往会给企业客户提供定制化或者自由配置的功能，如下图：

![定制化](https://user-gold-cdn.xitu.io/2020/4/24/171a9a5e9f7d001a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

对于saas系统而言，软件即服务，在提供基础服务的同时，同样要满足用户个性化需求，所以传统的saas软件提供商往往会提供给客户自由配置的空间，这种自由配置的桥梁就是通过表单，举一个简单的例子：

![网站表单](https://user-gold-cdn.xitu.io/2020/4/24/171a9b767e4bb302?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

通过这种方法就可以定制不同风格的企业产品，这里只是举了个比较简单的例子，往往实际项目中会更加复杂，可能会有几十个配置项，当然这种模式是比较传统的配置化方案，也仅仅是saas软件提供的很小的一个服务模块。目前主流的做法是采用可视化方案，而且国内也有非常成熟的方案，但基本的思想是一致的，只不过后者的体验更好，操作难度更低。

本文介绍的表单定制平台,也同样支持表单管理,表单数据分析, 表单数据收集, 表单定制等功能, 笔者将采用比较熟悉的技术栈`react`以及第三方ui库`antd4.0`来开发, 后端采用`node + koa`来设计路由接口.

## 设计思路

![设计思路概况](https://user-gold-cdn.xitu.io/2020/4/24/171aa801e1e7556f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

## 实现效果与分析

### 表单定制管理列表

![表单定制管理列表](https://user-gold-cdn.xitu.io/2020/4/24/171aa823f2fedb04?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

管理列表主要用来查看我们配置的表单模板,分析不同表单模板收集的数据,对表单模板进行编辑删除等操作.

### 表单定制页面

![表单定制页面](https://user-gold-cdn.xitu.io/2020/4/24/171aa83ef454bcbf?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

由上图可知表单定制页面主要用来编辑自定义表单模板,我们可以添加表单标题,表单字段等,目前提供了几种自定义表单控件如下:
* 文本框
* 多行文本框
* 下拉框
* 单选框
* 复选框
* 文件上传控件

基本涵盖了我们所需要的所有表单业务场景.由上图可知我们可以在任意位置插入自定义字段,同时可以编辑修改删除表单字段.如果想象力再大一点,我们可以基于它来实现不仅仅是表单问卷型应用,还可以实现答题,发布内容等场景.(后期可支持富文本控件)

### 草稿管理

![草稿管理](https://user-gold-cdn.xitu.io/2020/4/24/171aa85da2f9c847?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

草稿箱设计的目的是方便使用者在配置表单的过程中不确定是否符合需求或者由于某种临时性举动而无法继续配置,这个时候可以将以配置好的内容存入草稿箱,下次继续编辑,所以笔者专门设计了草稿箱管理列表,一旦用户存在草稿,会在管理页面通知用户并显示草稿的数量.作为一个追求体验的技术人,这一块的设计还是相当有必要的.

### 生成前台表单访问链接

![生成前台表单访问链接](https://user-gold-cdn.xitu.io/2020/4/24/171aa88fb0768a72?imageslim)

当我们配置好表单之后,我们点击保存, 会生成一个前台访问地址,实时访问表单信息,如下图为点击链接之后的页面:

![展示](https://user-gold-cdn.xitu.io/2020/4/24/171ab3d3341b02e4?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

我们也可以根据自己的风格,设计自己的表单录入页面, 具体如何实现这样的过程, 后面我会详细介绍.

### 查看用户已有数据录入

![查看用户已有数据录入1](https://user-gold-cdn.xitu.io/2020/4/24/171aa8e7dcecc4b4?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

![查看用户已有数据录入2](https://user-gold-cdn.xitu.io/2020/4/24/171aa8dce347d3c6?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

我们可以通过点击"查看数据"来访问收集到的表单数据,并通过可视化的工具对数据做分析比较,同时我们也可以在数据列表中删除数据,来控制我们数据展示的纯净.

### 表单数据分析

![表单数据分析](https://user-gold-cdn.xitu.io/2020/4/24/171aa91ab9128d19?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

收集到数据只有,我们会自动集成几个可视化组件来分析表单数据,以上是笔者列出的几个可视化组件,基于`antv G2`来封装.

## 应用场景

以上主要介绍了自定义表单定制平台的一些功能和交互效果, 我们可以利用该平台做很多有意思的事情.因为表单的抽象是数据,我们拿到定制化的表单json数据之后,我们可以有不同的展现形式,比如用户的**问卷调查, 网站平台的投票, 答题页面, 发布动态**等功能,如下图配置:

![配置1](https://user-gold-cdn.xitu.io/2020/4/24/171ab881a1e07227?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
![配置2](https://user-gold-cdn.xitu.io/2020/4/24/171ab88d11dc893b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

以上配置可以实现类似于微信的发布朋友圈的功能, 然后我们可以通过前端的手段根据用户发表的数据渲染成一个朋友圈列表.
如果我们再打开自己的脑洞，我们可以这样配置，配置一个这样的表单，表单包括一个文件上传控件和n个文本输入控件，如下图：

![活动页面配置](https://user-gold-cdn.xitu.io/2020/4/24/171ace33e78b037d?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

将这样的表单配置到H5管理模块，我们**只需要上传三张图，然后填写好对应的配文，然后利用市面上成熟的H5全屏滚动插件，就能轻松的定制各种H5活动页面了**。该方案已被笔者的很多子系统使用，效果还是非常好的。
当然基于该平台甚至能直接配置小型的宣传网站，还有更多想象空间，期待大家去挖掘。

## 代码实现

要想开发这样一个表单定制平台, 核心在于如何实现表单动态配置的机制.这里笔者将其划分为两部分:**基础表单物料和表单编辑生成器**, 如下图所示拆分图:


![表单定制模块](https://user-gold-cdn.xitu.io/2020/4/24/171abbf0cdd52a43?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
### 基础表单物料
基础表单物料主要是为了用户选择自定义表单控件使用，我们常用的表单动态渲染有map循环+条件判断和单层map+对象法，前者如果要渲染一个动态表单，可能实现如下：
```js
{
    list.map((item, i) => {
        return <React.Fragment key={i}>
            {
               item.type === 'input' && <Input />
            }
            {
               item.type === 'radio' && <Radio />
            }
            // ...
        </React.Fragment>
    })
}
```
但是这样做有个明显的缺点就是会产生很多没必要的判断，如果对于复杂表单，性能往往很低，所以笔者采用后者来实现，复杂度可以降到O(n).我们先来做配置模版：

```js
// 基础模版数据
const tpl = [
  {
    label: '文本框',
    placeholder: '请输入内容',
    type: 'text',
    value: '',
    index: uuid(5)
  },
  {
    label: '单选框',
    type: 'radio',
    option: [{label: '男', value: 0}, {label: '女', value: 1}],
    index: uuid(5)
  },
  {
    label: '复选框',
    type: 'checkbox',
    option: [{label: '男', value: 0}, {label: '女', value: 1}],
    index: uuid(5)
  },
  {
    label: '多行文本',
    placeholder: '请输入内容',
    type: 'textarea',
    index: uuid(5)
  },
  {
    label: '选择框',
    placeholder: '请选择',
    type: 'select',
    option: [{label: '中国', value: 0}, {label: '俄罗斯', value: 1}],
    index: uuid(5)
  },
  {
    label: '文件上传',
    type: 'upload',
    index: uuid(5)
  }
]

// 模版渲染组件
const tplMap = {
  text: {
    component: (props) => {
      const { placeholder, label } = props
      return <div className={styles.fieldOption}><span className={styles.fieldLabel}>{label}:</span><Input placeholder={placeholder} /></div>
    }
  },
  textarea: {
    component: (props) => {
      const { placeholder, label } = props
      return <div className={styles.fieldOption}><span className={styles.fieldLabel}>{label}:</span><TextArea placeholder={placeholder} /></div>
    }
  },
  radio: {
    component: (props) => {
      const { option, label } = props
      return <div className={styles.fieldOption}>
              <span className={styles.fieldLabel}>{label}:</span>
              <Radio.Group>
                {
                  option && option.map((item, i) => {
                    return <Radio style={radioStyle} value={item.value} key={item.label}>
                      { item.label }
                    </Radio>
                  })
                }
            </Radio.Group>
        </div>
    }
  },
  checkbox: {
    component: (props) => {
      const { option, label } = props
      return <div className={styles.fieldOption}>
              <span className={styles.fieldLabel}>{label}:</span>
              <Checkbox.Group>
                <Row>
                  {
                    option && option.map(item => {
                      return <Col span={16} key={item.label}>
                              <Checkbox value={item.value} style={{ lineHeight: '32px' }}>
                                { item.label }
                              </Checkbox>
                            </Col>
                    })
                  }
                </Row>
            </Checkbox.Group>
        </div>
    }
  },
  select: {
    component: (props) => {
      const { placeholder, option, label } = props
      return <div className={styles.fieldOption}>
              <span className={styles.fieldLabel}>{label}:</span>
              <Select placeholder={placeholder} style={{width: '100%'}}>
                {
                  option && option.map(item => {
                    return <Option value={item.value} key={item.label}>{item.label}</Option>
                  })
                }
            </Select>
        </div>
    }
  },
  upload: {
    component: (props) => {
      return <div className={styles.fieldOption}>
              <span className={styles.fieldLabel}>{props.label}:</span>
              <Upload
              listType="picture-card"
              className="avatar-uploader"
              showUploadList={false}
              action="https://www.mocky.io/v2/5cc8019d300000980a055e76"
            >
              <div>+</div>
            </Upload>
        </div>
    }
  }
}

export {
  tpl,
  tplMap
}
```

基础物料在下图所示中使用：

![基础物料使用](https://user-gold-cdn.xitu.io/2020/4/25/171ad0ca3515ae67?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

当我们要添加一个表单项时，我们就可以在左边预览操作区看到添加的项，并可以基于表单编辑生成器来编辑表单字段。

### 表单编辑生成器

表单编辑生成器分为2部分， 第一部分是用来生成表单项的容器组件，封装了添加，删除，编辑操作功能，代码如下：

```js
// 表单容器组件
const BaseFormEl = (props) => {
  const {isEdit, onEdit, onDel, onAdd} = props
  const handleEdit = (v) => {
    onEdit && onEdit(v)
  }
  return <div className={styles.formControl}>
    <div className={styles.formItem}>{ props.children }</div>
    <div className={styles.actionBar}>
      <span className={styles.actionItem} onClick={onDel}><MinusCircleOutlined /></span>
      <span className={styles.actionItem} onClick={onAdd}><PlusCircleOutlined /></span>
      <span className={styles.actionItem} onClick={handleEdit}><EditOutlined /></span>
    </div>
  </div>
}
```

第二部分主要用来渲染操作区模版，基于BaseFormEl包装不同类型的表单组件, 这里举一个比较复杂的select来说明，其他表单控件类似：

```js
const formMap = {
  title: {},
  text: {},
  textarea: {},
  radio: {},
  checkbox: {},
  select: {
    component: (props) => {
      const { onDel, onAdd, onEdit, curIndex, index, type, label, placeholder, required, message, option } = props
      return <BaseFormEl 
        onDel={onDel.bind(this, index)}
        onAdd={onAdd.bind(this, index)}
        onEdit={onEdit.bind(this, {index, type, placeholder, label, option, required})}
        isEdit={curIndex === index}
      >
        <Form.Item name={label} label={label} rules={[{ message, required }]}>
          <Select placeholder={placeholder}>
            {
              option && option.map(item => {
                return <Option value={item.value} key={item.label}>{item.label}</Option>
              })
            }
          </Select>
        </Form.Item>
      </BaseFormEl>
    },
    editAttrs: [
      {
        title: '字段名称',
        key: 'label'
      },
      {
        title: '选项',
        key: 'option'
      },
      {
        title: '提示文本',
        key: 'placeholder'
      },
      {
        title: '是否必填',
        key: 'required'
      },
    ]
  },
  upload: {}
}
```

editAttrs主要用来渲染编辑列表，说明哪些表单项可以编辑，这部分代码比较简单，这里直接用图举例：

![在线表单制作平台](https://user-gold-cdn.xitu.io/2020/4/25/171ad19928734c82?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

最后我们来渲染表单生成器组件：

```js
export default (props) => {
  const { 
    formData, 
    handleDelete, 
    handleAdd, 
    handleEdit, 
    curEditRowIdx 
  } = props
  return <Form name="customForm">
            {
              formData && formData.map(item => {
                let CP = formMap[item.type].component
                return <CP {...item} key={item.index}
                  onDel={handleDelete} 
                  onAdd={handleAdd}
                  onEdit={handleEdit}
                  curIndex={curEditRowIdx}
                />
              })
            }
         </Form>
}
```

至此，基本功能模块已经开发完成，我们只需要将这些物料和组件导入到编辑页面，基于业务来操作和请求即可。
