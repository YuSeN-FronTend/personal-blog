---
title: IndexedDB学习
date: 2023-4-14 12:48
categories: javaScript
---
# IndexedDB

IndexedDB是Indexed Database API的简称，是浏览器中存储结构化数据的一种方案。通俗的说IndexedDB就是浏览器提供的本地数据库，它可以被网页脚本创建和操作，IndexedDB允许存储大量数据，提供查找接口，还能建立索引。这些都是LocalStorage所不具备的。就数据库类型而言，IndexedDB不属于关系型数据库，更接近于NoSQL数据库。

## IndexedDB的特点：

- **键值对存储**

  IndexedDB内部采用对象仓库存放数据。所有类型的数据都可以直接存入，包括js对象。对象仓库中数据以键值对的形式保存，每一条数据记录都有对应的主键，主键是独一无二的，不能重复，否则会抛出错误。

- **异步**

  IndexedDB操作时不会锁死浏览器，用户依然可以进行其他操作，这与LocalStorage形成对比，后者的操作是同步的。异步设计是为了**防止大量的数据读写，拖慢网页的表现。**

- **支持事务**

  IndexedDB支持事务(transaction)，这意味着一系列操作步骤之中，只要有一步失败，整个事务就都取消，数据库回滚到事务发生之前的状态，不存在只改写一部分数据的情况。

- **同源限制**

  IndexedDB受到同源限制，每一个数据库对应创建它的域名。网页只能访问自身域名下的数据库，而不能访问跨域的数据库。

- **储存空间大**

  IndexedDB的储存空间比LocalStorage大得多，一般来说不少于250MB，甚至没有上限。

- **支持二进制储存**

  IndexedDB不仅可以储存字符串，还可以储存二进制数据(ArrayBuffer对象和Blob对象)

## 操作流程

下面介绍IndexedDB基本操作流程

从新增数据及之后的操作都可在打开数据库的成功回调中测试，传入对应的db对象

### 打开数据库

使用`indexedDB.open()`方法，接收两个参数，第一个参数是数据库的名字，第二个数据库的版本号。如果指定数据库不存在，则会新建。

```js
let request = window.indexedDB.open(databaseName, version);
```

此方法返回一个IDBRequest对象，这个对象接收三个事件，error、success、upgradeneeded，处理打开数据库的操作结果。

#### error事件

此事件表示打开数据库失败

```js
request.onerror = function (error) {
	console.log('IndexedDB打开失败', error);
}
```

#### success事件

此事件表示成功打开数据库

```js
let db;
request.onsuccess = function (res) {
	console.log('IndexedDB打开成功', res);
	db = res.target.result;
});
```

#### upgradeneeded事件

如果指定的版本号大于数据库的实际版本号，就会发生数据库升级事件，也就是此事件

```js
let db;
request.onupgradeneeded = function (res) {
	console.log('IndexDB升级成功', res);
}
```

### 新建数据库

新建数据库与打开数据库是同一个操作。如果指定的数据库不存在，就会新建。不同之处在于，后续的操作主要在事件的监听函数里面去完成，因为这时版本从无到有，会触发这个事件。

通常新建数据库后，第一件事就是新建对象仓库

```js
request.onupgradeneeded = function(res) {
  console.log('IndexDB升级成功', res);
  db = event.target.result;
  let objectStore = db.createObjectStore('person', { keyPath: 'id' });
}
```

上面代码中，数据库成功新建之后，新增一张叫person的表格，主键是id

更好的写法是判断以下表格是否存在，存在则不重复创建

```js
request.onupgradeneeded = function (event) {
  console.log('IndexDB升级成功', res);
  db = event.target.result;
  let objectStore;
  if (!db.objectStoreNames.contains('person')) {
    objectStore = db.createObjectStore('person', { keyPath: 'id' });
  }
}
```

主键是默认建立索引的属性，如果数据记录中没有适合作为主键的属性，那么可以让indexedDB自动生成主键

```js
let objectStore = db.createObjectStore('person',{ autoIncrement: true });
```

上面代码指定主键为一个递增函数

新建对象仓库后下一步可以新建索引

```js
request.onupgradeneeded = function(event) {
  console.log('IndexDB升级成功', res);
  db = event.target.result;
  let objectStore = db.createObjectStore('person', { keyPath: 'id' });
  objectStore.createIndex('name', 'name', { unique: false });
  objectStore.createIndex('email', 'email', { unique: true });
}
```

### 新增数据

新增数据要通过事务完成

```js
function add(db) {
	let store = db.transaction(['group'], 'readwrite').objectStore('group').add({
		id: new Date().getTime(),
		name: '王二',
		age: 12,
		email: 'XXXX@xxx.com'
});

	store.onsuccess = function (event) {
		console.log('数据添加成功', event);
	};
	store.onerror = function (event) {
		console.log('数据添加失败', event);
	};
}
```

新建时必须指定表格名称和操作模式。新建事务以后，通过`IDBTransaction.objectStore(name)`，拿到IDBObjectStore对象，再通过表格对象的add()方法向表格写入一条记录，写入操作也是异步，通过监听连接对象的success事件和error事件了解是否写入成功

### 读取数据

也是通过事务完成

```js
function read(db) {
	let transaction = db.transaction(['group'])
	let objectStore = transaction.objectStore('group');
	let request = objectStore.get(1);

	request.onerror = (event) => {
		console.log('失败');
	}
	request.onsuccess = (event) => {
		console.log(event)
		if(event.target.result) {
			console.log('Name: ' + request.result.name);
			console.log('Age: ' + request.result.age);
			console.log('Email: ' + request.result.email);
        } else {
			console.log('未获得数据记录');
		}
	}
}
```

`objectStore.get()`可以读取数据，参数是主键的值

### 遍历数据

遍历数据表格所有记录需要使用指针对象IDBCursor

```js
function readAll(db) {
    let objectStore = db.transaction('group').objectStore('group')
    objectStore.openCursor().onsuccess = function (event) {
        let cursor = event.target.result;
        if (cursor) {
            console.log('Id: ' + cursor.key);
            console.log('Name: ' + cursor.value.name);
            console.log('Age: ' + cursor.value.age);
            console.log('Email: ' + cursor.value.email);
            cursor.continue();
        } else {
            console.log('没有更多数据了！');
        }
    }
}
```

上面代码中新建指针对象`openCursor()`方法是一个异步操作，所以要监听success事件

### 更新数据

更新数据要使用`IDBObject.put()`方法

```js
function update(db) {
        var request = db.transaction(['group'], 'readwrite')
            .objectStore('group')
            .put({ id: 1, name: '李四', age: 35, email: 'lisi@example.com' });
        request.onsuccess = function (event) {
            console.log('数据更新成功');
        };
        request.onerror = function (event) {
            console.log('数据更新失败');
        }
    }
```

上面代码中，`put()`方法自动更新可主键为1的记录

### 删除数据

`IDBObjectStore.delete()`方法用于删除记录

```js
function remove(db) {
	let request = db.transaction(['group'], 'readwrite').objectStore('group').delete(1681443704492);
	request.onsuccess = function(event) {
		console.log(`数据删除成功`);
	}
}
```

### 使用索引

索引的意义在于可以搜索任意字段，也就是从任意字段拿到数据记录，如果不建立索引，只能通过搜索主键

假设再新建表格时对name字段进行了索引

```
objectStore.createIndex('indexName', 'name', { unique: false });
```

现在就可以从indexName中找对应的数据了

```js
function indexSearch(db) {
    let transaction =  db.transaction(['group'], 'readonly');
    let store = transaction.objectStore('group');
    let index = store.index('indexName');
    let request = index.get('李四');
    request.onsuccess = (e) => {
        let result = e.target.result;
        console.log(result)
        if(result) {
            // ...
        } else {
            // ...
        }
    }
}
```

