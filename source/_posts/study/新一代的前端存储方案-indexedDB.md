---
title: 新一代的前端存储方案--indexedDB
date: 2018-05-27 17:33:41
tags: "indexedDB"
---
# 前端存储

![indexedDB](https://user-gold-cdn.xitu.io/2018/5/27/163a05ff03a923bd?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

我们都知道在前端开发当中,有时会因为某些需求,要将一些数据存储在前端本地当中.比如说:为了优化性能,将一些常用的数据存在本地,这样以后需要的时候直接从本地拿,不需要再向后端进行请求.还有就是为了防止CSRF攻击,后端给前端一个token,前端就需要将这个token存在本地.之后每次请求都需要带上这个token.等等不一而足.

而这些需求就不油避免的造就一个前端的发展方向--**前端存储**

在前端的'上古时代'里,我们前端想要存储数据,只有一种方式,那就是`Cookie`.但是`Cookie`虽然可以做前端存储方案,但是却也有着很多局限性.首先它的存储空间大小只有`4K`,其次它的存储有效时间有限制,然后存在`Cookie`中的数据,在你每次进行请求的时候都会将它带上.使得每次的请求数据都会无意义的增大.

最后,也是最重要的一点.`Cookie`设计之初就不是就是让我们前端存数据用的.它只是为了让网站验证用户身份用的.至于`Cookie`的本地存储功能只是它的一个手段而已.关于这点你们可以看下我的另外一篇文章---[在HTML5的时代,重新认识Cookie](https://juejin.im/post/59708bbe518825103c098332)

  综上所述,使用Cookie作为前端存储有这许多缺点,所以经过前端社区的不断努力,在HTML5中有了真正的前端存储方案Web Storage.它分为两种,一种是`永久存储的localStorage`,一种是`会话期间存储的sessionStorage`.对比Cookie,`Web Storage`的**优势**很明显:

> * 存储空间更大,有5M大小
* 在浏览器发送请求是不会带上`web Storage`里的数据
* 更加友好的API
* 可以做永久存储(localStorage).

<!-- more -->
这一切看起来很完美,但是随着前端的不断发展,web Storage也有了一些不太合适的地方:

> * 随着web应用程序的不断发展,5M的存储大小对于一些大型的web应用程序来说有些不够
* web Storage**只能存储string类型**的数据.对于Object类型的数据只能先用`JSON.stringify()`转换一下在存储.

基于上述原因,前端社区又提出了浏览器数据库存储这个概念.而`Web SQL Database`和`indexedDB`(索引数据库)是对这个概念的实现.其中*Web SQL Database在目前来说基本已经被放弃*.所以目前主流的浏览器数据库的实现就是indexedDB(索引数据库).也就是我们要介绍的 新一代的前端存储方案--**indexedDB**

## 什么是indexedDB

### indexedDB的介绍

> IndexedDB 是一种使用浏览器存储大量数据的方法.它创造的数据可以被查询，并且可以离线使用. IndexedDB对于那些需要存储大量数据，或者是需要离线使用的程序是非常有效的解决方法. --- MDN

### indexedDB的概念

> 使用IndexedDB，你可以存储或者获取数据，使用一个key索引的。 你可以在事务(transaction)中完成对数据的修改。和大多数web存储解决方案相同，indexedDB也遵从同源协议(same-origin policy). 所以你只能访问同域中存储的数据，而不能访问其他域的。
API包含异步(asynchronous) API 和同步(synchronous)API两种。  异步API适合大多数情况, 同步API必须同 WebWorkers一同使用. 目前，没有主流浏览器支持同步API。 即使同步API被支持了，你也会在大多数的情况使用异步API。
IndexedDB 是 WebSQL 数据库的取代品, W3C组织在2010年11月18日废弃了webSql.  IndexedDB 和WebSQL的不同点在于WebSQL 是关系型数据库（复杂）IndexedDB 是key-value型数据库（简单好使）.

上面是[MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/IndexedDB_API/Basic_Concepts_Behind_IndexedDB)上对于IndexedDB的介绍.其简单而言,indexedDB就是一个基于事务操作的key-value型数前端数据库.其API大多是异步的

## indexedDB的使用

### 创建一个indexedDB数据库

```js
const request = indexedDB.open('myDatabase',1);
request.addEventListener('success', e => {
   console.log("连接数据库成功");
});

request.addEventListener('error', e => {
    console.log("连接数据库失败");
});
```

在上面代码中我们使用`indexedDB.open()`创建一个`indexedDB`数据库`.open()`方法接受可以接受两个参数.

第一个是数据库名,第二个是数据库的版本号.
同时返回一个`IDBOpenDBRequest对象`用于操作数据库.其中对于open()的第一个参数数据库名,open()会先去查找本地是否已有这个数据库,如果有则直接将这个数据库返回,如果没有,则先创建这个数据库,再返回.对于第二个参数版本号,则是一个可选参数,如果不传,默认为1.但如果传入就必须是一个整数.

在通过对`indexedDB.open()`方法拿到一个数据库对象`IDBOpenDBRequest`我们可以通过监听这个对象的success事件和error事件来执行相应的操作.

### 创建一个对象仓库

再有了一个数据库之后,我们获取就想要去存储数据了,但是单只有数据库还不够,我们还需要有`对象仓库(object store)`.`对象仓库(object store)`是indexedDB数据库的基础,其类似于MySQL中表的概念.

要创建一个对象仓库必须在`upgradeneeded`事件中,而`upgradeneeded`事件只会在版本号更新的时候触发.这是因为*indexedDB API中不允许数据库中的数据仓库在同一版本中发生变化*

```js
const request = indexedDB.open('myDatabase',2);

request.addEventListener('upgradeneeded',e => {
    const db = e.target.result;
    const store = db.createObjectStore('User',{keyPath: 'userId',autoIncrement: false});
    console.log('创建对象仓库成功');
})
```

在上述代码中我们监听`upgradeneeded`事件,并在这个事件触发时使用`createObjectStore()`方法创建了一个对象仓库.

`createObjectStore()`方法接受两个参数,第一个是对象仓库的名字,在同一数据库中,**仓库名不能重复**.第二个是可选参数.用于指定数据的主键,以及是否自增主键.

### 创建事务

  OK现在我们有了数据库和对象仓库了,我们是否就可以存储数据了了.很抱歉,还是不行.我们还差最后一样东西----事务.

#### 什么是事务

> 一个数据库事务通常包含了一个序列的对数据库的读/写操作。它的存在包含有以下两个目的
* 为数据库操作序列提供了一个从失败中恢复到正常状态的方法，同时提供了数据库即使在异常状态下仍能保持一致性的方法。
* 当多个应用程序在并发访问数据库时，可以在这些应用程序之间提供一个隔离方法，以防止彼此的操作互相干扰。

> 并非任意的对数据库的操作序列都是数据库事务。数据库事务拥有以下四个特性，习惯上被称之为ACID特性。
* 原子性（Atomicity）：事务作为一个整体被执行，包含在其中的对数据库的操作要么全部被执行，要么都不执行
* 一致性（Consistency）：事务应确保数据库的状态从一个一致状态转变为另一个一致状态。一致状态的含义是数据库中的数据应满足完整性约束
* 隔离性（Isolation）：多个事务并发执行时，一个事务的执行不应影响其他事务的执行
* 持久性（Durability）：已被提交的事务对数据库的修改应该永久保存在数据库中

上面是维基百科上对数据库事务的解释.简单来说事务就是用来保证数据库操作要么全部成功,要么全部失败的一个限制.比如,在修改多条数据时,前面几条已经成功了.,在中间的某一条是失败了.那么在这时,如果是基于事务的数据库操作,那么这时数据库就应该重置前面数据的修改,放弃后面的数据修改.直接返回错误,一条数据也不修改.

```js
const request = indexedDB.open('myDatabase', 3);

request.addEventListener('success', e => {
    const db = e.target.result;

    const tx = db.transaction('Users','readwrite');
});
```

上述代码中我们使用`transaction()`来创建一个事务.

`transaction()`接受两个参数,第一个是你要操作的对象仓库名称,第二个是你创建的事务模式.传入 `readonly`时只能对对象仓库进行读操作,无法写操作.可以传入`readwrite`进行读写操作.

### 操作数据

> 好了现在有了数据库,对象仓库,事务之后我们终于可以存储数据了.
* add() : 增加数据。接收一个参数，为需要保存到对象仓库中的对象。
* put() : 增加或修改数据。接收一个参数，为需要保存到对象仓库中的对象。
* get() : 获取数据。接收一个参数，为需要获取数据的主键值。
* delete() : 删除数据。接收一个参数，为需要获取数据的主键值。

add 和 put 的作用类似，区别*在于 put 保存数据时，如果该数据的主键在数据库中已经有相同主键的时候，则会修改数据库中对应主键的对象，而使用 add 保存数据，如果该主键已经存在，则保存失败*。

#### 添加数据

```js
const request = indexedDB.open('myDatabase', 3);

request.addEventListener('success', e => {
    const db = e.target.result;

    const tx = db.transaction('Users','readwrite');

    const store = tx.objectStore('Users');

    // 保存数据
    const reqAdd = store.add({'userId': 1, 'userName': '李白', 'age': 24});

    reqAdd.addEventListener('success', e => {
      console.log('保存成功')
    })

    // 获取数据
    const reqGet = store.get(1);

    reqGet.addEventListener('success', e => {
      console.log(this.result.userName);    // 李白
    })

    //删除数据
    const reqDelete = store.delete(1);

    reqDelete.addEventListener('success', e => {
      console.log('删除数据成功');    // 李白
    })

});

```

### 使用游标

  在上面当中我们使用`get()`方法传入一个主键来获取数据,但是这样只能够获取到一条数据.如果我们想要获取多条数据了怎么办.我们可以**使用游标,来获取一个区间内的数据**.

要使用游标,我们需要使用对象仓库上的`openCursor()`方法创建币打开.`openCursor()`方法接受两个参数.

> openCursor(range?: IDBKeyRange | number | string | Date | IDBArrayKey, direction?: IDBCursorDirection): IDBRequest;

  第一个是范围,范围可以是一个`IDBKeyRange`对象.用以下方式创建.

```js
// boundRange 表示主键值从1到10(包含1和10)的集合。
// 如果第三个参数为true，则表示不包含最小键值1，如果第四参数为true，则表示不包含最大键值10，默认都为false
var boundRange = IDBKeyRange.bound(1, 10, false, false);

// onlyRange 表示由一个主键值的集合。only() 参数则为主键值，整数类型。
var onlyRange = IDBKeyRange.only(1);

// lowerRaneg 表示大于等于1的主键值的集合。
// 第二个参数可选，为true则表示不包含最小主键1，false则包含，默认为false
var lowerRange = IDBKeyRange.lowerBound(1, false);

// upperRange 表示小于等于10的主键值的集合。
// 第二个参数可选，为true则表示不包含最大主键10，false则包含，默认为false
var upperRange = IDBKeyRange.upperBound(10, false);

```

  第二个参数是方向.主要有一下几种

```js
next : 游标中的数据按主键值升序排列，主键值相等的数据都被读取
nextunique : 游标中的数据按主键值升序排列，主键值相等只读取第一条数据
prev : 游标中的数据按主键值降序排列，主键值相等的数据都被读取
prevunique : 游标中的数据按主键值降序排列，主键值相等只读取第一条数据
```

  下面让我们来看一个完整的例子

```js
const request = indexedDB.open('myDatabase', 4);

request.addEventListener('success', e => {
    const db = e.target.result;

    const tx = db.transaction('Users','readwrite');

    const store = tx.objectStore('Users');

    const range = IDBKeyRange.bound(1,10);

    const req = store.openCursor(range, 'next');

    req.addEventListener('success', e => {
      const cursor = this.result;
        if(cursor){
            console.log(cursor.value.userName);
            cursor.continue();
        }else{
            console.log('检索结束');
        }
    })
});

```

  在上面的代码中如果检索到符合条件的数据时,我们可以:

> 使用`cursor.value`拿到数据.
使用`cursor.updata()`更新数据.
使用`cursor.delete()`删除数据.
使用`cursor.continue()`读取下一条数据.

### 索引

  在上面代码中我们获取数据都是用的主键.但是,在很多情况下我们并不知道我们需要数据的主键是什么,我们知道一个大概的条件.比如说年龄大于20岁的用户.这个时候我们就需要用到索引.以便有条件的查找.

#### 创建索引

  我们使用对象仓库的`createIndex()`方法来创建一个索引.

> createIndex(name: string, keyPath: string | string[], optionalParameters?: IDBIndexParameters): IDBIndex;

createIndex()方法接收三个参数:

* 第一个参数name是索引名,不能重复.
* 第二个参数keyPath是你要在存储对象上的那个属性上建立索引,可以是一个单个的key值,也可以是一个包含key值集合的数组.
* 第三个参数optionalParameters是一个可选的对象参数{unique, multiEntry}

1.`unique`: 用来指定索引值是否可以重复,为true代表不能相同,为false时代表可以相同
2.`multiEntry`: 当第二个参数keyPath为一个数组时.如果multiEntry是true,则会以数组中的每个元素建立一条索引.如果是false,则以整个数组为keyPath值,添加一条索引.

```js
const request = indexedDB.open('myDatabase', 5);

request.addEventListener('upgradeneeded', e => {
    const db = e.target.result;
    const  store = db.createObjectStore('Users', {keyPath: 'userId', autoIncrement: false});
    const idx = store.createIndex('ageIndex','age',{unique: false})
});
```

  这样我们就创建了一条索引.

#### 使用索引

  这在创建了一条索引之后我们就可以来使用它了.我们使用对象仓库上的`index`方法,通过传入一个索引名.来拿到一个索引对象.

```js
const index = store.index('ageIndex');
```

  然后我们就可以使用这个索引了.比如说我们要拿到年龄在20岁以上的数据,升序排列.

```js
const request = indexedDB.open('myDatabase', 4);

request.addEventListener('success', e => {
    const db = e.target.result;

    const tx = db.transaction('Users','readwrite');

    const store = tx.objectStore('Users');

    const index = store.index('ageIndex');

    const req = index.openCursor(IDBKeyRange.lowerBound(20), 'next');//20岁以上的数据,升序排列

    req.addEventListener('success', e => {
      const cursor = e.target.result;
        if(cursor){
            console.log(cursor.value.age);
            cursor.continue();
        }else{
            console.log('检索结束');
        }
    })
});
```

## indexedDB的兼容性

![indexedDB兼容性](https://user-gold-cdn.xitu.io/2018/5/27/163a05e409de5bd7?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

[原文地址](https://juejin.im/post/5b09a641f265da0dcd0b674f?utm_source=gold_browser_extension)