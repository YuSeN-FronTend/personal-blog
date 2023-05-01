---
title: vue3学习
date: 2023-4-29 9:20
categories: vue
---
# vue3

## 基础

### 基础应用创建

脚手架安装过后，再main.js中需做如下配置。每个vue的应用都是通过createApp函数创建的，如果是我们最常用的根文件组件，可以从外部直接导入。最后使用mount('根组件类名')挂载上去

```js
import { createApp } from "vue";
import App from "./App.vue";
const app = createApp(App);
app.mount("#app");
```

### 模板语法

模板语法和vue2类似，不再重复学习

### 响应式基础

#### `<script setup>`

由于在vue3中，大部分的配置项方法生命周期都在setup中去书写，并且书写完毕需要大量return，所以这样写可以大幅度的简化代码

#### DOM更新时机

在DOM渲染完成后再执行里面的逻辑，就是nextTick()，和vue2中的$nextTick类似，所以不多赘述

#### 深层响应式

vue3的响应式默认都是深层次的，也就是不论有多少层，他都是响应式的，即便如下这种错综复杂的结构也可以

```vue
<template>
  <div>
    <button @click="countAdd">{{ obj.b.d.f.g.h }}</button>
  </div>
</template>

<script setup>
      import { getCurrentInstance, onMounted, reactive, ref } from 'vue';
      let obj = reactive({
        a: 1,
        b: {
          c: 2,
          d:{
            e:3,
            f:{
              g:{
                h:29
              }
            }
          }
        }
      })
      function countAdd() {
        obj.b.d.f.g.h++;
      }
</script>
```

#### 响应式代理vs原始对象

值得注意的式reactive返回的是一个原始对象的Proxy，它和原始对象是不相等的。并且为了保证访问代理的一致性，在一个Proxy对象上再调用reactive则返回它自己，如下所示

```js
	  let obj = {
        count:123
      }
      let obj1 = reactive(obj)
      console.log(obj === obj1); // false
      console.log(obj1 === reactive(obj)); // true
      console.log(reactive(obj1) === obj1); // true
```

并且嵌套对象无论多少层，每一层都是原始对象的代理

#### `reactive()`局限性

它只对数组，对象和Map，Set这样的集合类型有效，对于string，number和boolean这样的原始类型无效。

vue3的响应式系统是通过属性方位进行追踪的，所以我们必须保持对该响应式对象的相同引用，不然就会导致原始引用的响应式连接丢失，如下所示

```vue
<template>
  <div>
    <button @click="countAdd">{{ n }}</button>
  </div>
</template>
<script setup>
	let obj = reactive({
        count: 1
      })
	let n = obj.count;
	function countAdd() {
        n++
      }
</script>
```

#### 用`ref()`定义相应式变量

vue3提供ref()方法来允许我们创建可以使用任何值类型的响应式ref。但是有一点需要特别注意，用ref包装的响应式，调取时需要加上.value，否则调取不到，如下所示

```js
	  let count = ref(0);
      console.log(count); // RefImpl对象，里面有value
      console.log(count.value); // 0
```

当ref包装的值为对象类型值时会用reactive自动转换它的.value，并且ref被传递给函数或是从一般对象上被解构时，不会丢失响应性

#### ref在模板中的解包

ref在模板中使用时不用加.value，会自动完成解包，如下所示

```vue
<template>
  <div>
    <button @click="countAdd">{{ count }}</button>
  </div>
</template>
<script setup>
	let count = ref(0);
	function countAdd() {
        count.value++
      }
</script>
```

### 计算属性

在模板语法中，可以写一些简单的表达式，并且是非常方便的。但是如果出现太多逻辑，会使得代码非常臃肿，所以vue给我们提供了更好的解决方案——计算属性，请看下述示例

```js
<template>
    <div>
        <span :style="{color: textColor}">你好啊</span>
        <button @click="handleArr">改变颜色</button>
    </div>
</template>

<script setup>
import { computed, reactive } from 'vue';

    let arr = reactive(['red', 'blue', 'green']);

    function handleArr() {
        if(arr.length){
            arr.pop();
        }
    }

    let textColor = computed(() => {
        return arr.length ? 'green': 'red';
    })
</script>
```

创建一个数组，再创建一个方法使方法每次触发都删除数组最后一个元素。将此方法绑定到一个按钮上。在创建一个计算属性，在arr数组长度为0时改变textColor的值，并且把textColor的值当作span中文字的color。当点击三次后，span中的字颜色变了，说明计算属性可以监听值的变化。

#### 计算属性缓存vs方法

计算属性是会被缓存的，如果计算属性所监听的值没有发生变化，则无论重新渲染多少次页面，里面的值都不会被重新计算，而是使用先前存储好的值。这样其实可以节省掉很多性能，并且计算属性只会基于响应式依赖缓存，如下代码

```js
let time = computed(() => Date.now())
```

由于`Date.now()`不是响应式的，所以不会更改

#### 最佳实践

计算属性默认只是只读的，可以改写成可以被修改，但是没必要。因为计算属性只是基于原始响应式的一个快照，直接修改快照是没有任何意义。所以计算属性应该永远不被更改，想要改变就应该更新他所依赖的原装态以触发新的计算。

### 类与样式绑定

#### 绑定类

- 使用v-bind:class="{ active: isActive }"，意思是active类存在取决于isActive的值是否为真
- 使用v-bind:class="[类名, 类名]"，可以绑定多个类名，还可以在数组内使用三元表达式
- 还可以使用上述方法调用子组件时给子组件绑定类名也是可以实现的

#### 绑定内联样式

- 使用v-bind:style="{css属性}"，可以直接添加样式，也可以在js部分定义好对象直接调用
- v-bind:style同样可以绑定数组，添加包含多个样式的对象

由于这一部分和vue2中基本没有区别，不做太多赘述

### 条件渲染

- v-if
- v-else-if
- v-else
- v-show

和vue2中基本类似，不做赘述

### 列表渲染

v-for，注意要绑定key值，因为key是元素的唯一标识，在每次重新渲染页面是，diff算法可以检测到这些节点并且保存没更改的节点，可以加快页面的更新速度。

同样的和vue2基本类似，不做赘述，但需要注意的是，v-if和v-for不能一起用，因为优先级很不明显。

### 事件处理

事件处理器的值可以是：

- 内联事件处理器，事件被触发的时候执行内联的js语句
- 方法事件处理器，一个指向组件上定义的方法的属性名或是路径

简单说就是前者直接执行逻辑，后者调用js中的方法

#### 修饰符

这里不每个列举了，因为和vue2类似

### 表单输入绑定

v-model详解，和vue2类似，不赘述

### 生命周期

和vue2很像，但也也有一点点区别：

- setup是最先执行的，可以理解为原来的beforeCreate和created但实际上还在他们之前。
- beforeDestory变为beforeUnmount，destoryed变为unmounted，

### 侦听器

#### watch

他接受两个参数，第一个是需要监听的数据源，第二个是需要执行的回调，以下是基本示例

```vue
<template>
    <div>
        {{ count }}
        <button @click="count++">count+1</button>
    </div>
</template>

<script setup>
    import { ref, watch } from 'vue';
    let count = ref(0);
    watch(count, (newVal, oldVal) => {
        console.log(newVal, oldVal);
    })
</script>
```

由此可见他可以直接监听ref，此外他还可以监听getter或者多数据源组成的数组

- getter

  ```vue
  <template>
      <div>
          {{ sum1 }}
          <button @click="count++">count+1</button>
          <button @click="num+=2">num+2</button>
      </div>
  </template>
  
  <script setup>
      import { ref, watch } from 'vue';
      let count = ref(0);
      let num = ref(0);
      let sum1 = ref(0)
      watch(() => count.value + num.value, (sum) => {
          sum1.value = sum
      })
  </script>
  ```

- 多数据源数组

  ```vue
  <template>
      <div>
          <button @click="count++">count+1</button>
          <button @click="num+=2">num+2</button>
      </div>
  </template>
  
  <script setup>
      import { ref, watch } from 'vue';
      let count = ref(0);
      let num = ref(0);
      let sum = ref(0)
      watch([count, () => num.value + sum.value], (num1) => {
          console.log(num1);
      })
  </script>
  ```

  传入的数组，返回的num1也同样是数组

- 还有就是监听对象中属性时不能够直接监听，需要写成getter形式，如下所示

  ```
  watch(() => obj.count, (newVal) => {
          console.log(newVal);
      })
  ```

##### 深层侦听器

在watch的第三个参数的位置写入{deep: true}，可以用来监听对象中所有数据的变化，但是需要注意的是，如果被监听的对象特别大时，就要考虑是否值得深度监听并且需要考虑性能的消耗

##### 即时回调侦听器

就是在第三个参数位置传入{immediate: true}，默认watch在数值第一次声明时不做监听，而加入此属性则会使回调立即执行

#### watchEffect

侦听器的回调和源监听属性完全相同的响应式状态是很常见的，如下代码

```vue
<template>
    <div>
        {{ count }}
        <button @click="count++">count+1</button>
    </div>
</template>

<script setup>
    import { ref, watch } from 'vue';
    let count = ref(0);
    watch(count,() => {
        console.log(count.value);
    })
</script>
```

此时我们可以改写成

```vue
<template>
    <div>
        {{ count }}
        <button @click="count++">count+1</button>
    </div>
</template>

<script setup>
    import { ref, watchEffect } from 'vue';
    let count = ref(0);
    watchEffect(() => {
        console.log(count.value);
    })
</script>
```

在watchEffect中回调会立即执行，不需要指定{immediate: true}，对于只有一个依赖项的场景，watchEffect好处比较少，但是如果有多个依赖项，使用watchEffect可以消除手动维护依赖列表的负担，如果需要侦听一个嵌套数据结构的几个属性，watchEffect比深度监听更加有效，因为它只会跟踪回调中被使用到的属性而不是递归跟踪所有的属性

#### `watch` vs `watchEffect`

主要区别是追踪响应式依赖的方式

- watch只追踪明确侦听的数据源，他不会追踪任何在回调中访问的东西，只在数据源发生改变时才会触发回调，因此我们可以更精确的把控回调函数触发时机
- watchEffect则会在副作用发生期间追踪依赖，自动追踪所有能访问到的响应式属性，这会使代码更方便也更简洁，但是响应式依赖关系不是很明确

#### 回调触发时机

默认情况下用户创建侦听器回调，会在vue组件更新之前被调用。这意味着在侦听器回调中访问的DOM使被vue更新之前的状态，如果想访问vue更新之后的DOM，需要指明flush: 'post'选项，如下所示

```js
watch(source, callback, {
  flush: 'post'
})

watchEffect(callback, {
  flush: 'post'
})
```

后者还有一个更方便的别名

```js
import { watchPostEffect } from 'vue'

watchPostEffect(() => {
  /* 在 Vue 更新后执行 */
})
```

#### 停止侦听器

如果是同步语句创建侦听器，它会在宿主组件卸载时自动停止。

但如果是异步创建，需要手动停止侦听器来防止内存泄露

```js
watchEffect(() => {})
```

以上代码就可以停止侦听器了

### 模板引用

在模板中给一个标签添加ref属性，在js部分生命一个变量，必须与ref属性的名字相同即可获取到对应元素

```vue
<template>
    <div>
        <input type="text" ref="input">
    </div>
</template>

<script setup>
import { onMounted, ref } from 'vue';
    const input = ref(null);
    onMounted(() => {
        input.value.focus()
    })
</script>
```

#### v-for的ref使用

```vue
<template>
    <div>
        <ul>
            <li v-for="item in list" ref="index">{{ item }}</li>
        </ul>
    </div>
</template>

<script setup>
import { onMounted, ref } from 'vue';
    const list = ref(['小红', '小鞠', '小张'])
    const index = ref([]);
    onMounted(() => {
        console.log(index.value);
    })
</script>
```

index数组中即可获取到遍历出来的li元素

#### 组件上的ref使用

组件也可以用于ref获取，并且可以获取到子组件中所有的属性和方法，但大多数情况还是需要使用标准的props和emit接口来实现父子组件交互。

还有一点需要注意，使用了`<script setup>`的组件是默认私有的，父组件无法从中获取属性和方法，除非子组件在其中通过`defineExpose`显示暴露。

## 组件

### 注册组件

#### 全局注册

使用vue示例的`app.component()`，在main.js中引入需要作为组件的vue文件，如下所示

```js
import app from './app.vue';
import componentApp from './componentApp.vue';
app.component('componentApp', componentApp);
```

app.component()还支持链式调用，如下代码

```js
app
  .component('ComponentA', ComponentA)
  .component('ComponentB', ComponentB)
  .component('ComponentC', ComponentC)
```

全局注册好的组件可以此项目的任何位置使用

#### 局部注册

全局注册虽然很方便，但有几个问题：

- 全局注册之后，没有被使用的组件没有办法在打包之后自动移除，如果全局注册了一个组件，即使它没有被实际引用，他依然会出现在打包后的js文件中
- 全局注册在大型项目中使项目的依赖关系变得不那么明确，在父组件使用子组件时不太容易定位子组件的实现。和大多数的全局变量一样，会影响应用长期的可维护性。

由此可见，局部注册的组件需要它的父组件显式导入，并且只能在该父组件中使用，这样做的好处是会使组件之间的依赖更加明确，也在打包时更加友好。

局部注册的方法和vue2中大相径庭，不再赘述

#### 组件名格式

`<MyComponent />`或者`<my-component></my-component>`

### props

#### props声明

- 父组件代码

  ```vue
  <template>
      <div>
          <Demo :name="name"/>
      </div>
  </template>
  
  <script setup>
      import Demo from '../components/demo.vue'
      const name = '小红'
  </script>
  ```

- 子组件代码

  ```vue
  <template>
      <div>
          {{ props.name }}
      </div>
  </template>
  
  <script setup>
      const props = defineProps(['name'])
  </script>
  ```

在调用组件时传入一个属性，在子组件用defineProps()获取

还可以传入对象如下所示

```js
const props = defineProps({
        name: String,
        age: Number
    })
```

属性后面定义的类型，如果传入的类型不符合，则会抛出警告，但还是会渲染在页面上

#### Prop名字格式

使用camelCase或者kebab-case格式

#### 静态 vs 动态 props

如果要传入变量，需要在属性前加一个`:`，为动态属性，如果直接传入一个固定值为静态属性

#### 使一个对象绑定多个prop

可以直接用`v-bind`绑定如下所示

```vue
<template>
    <div>
        <Demo v-bind="obj"/>
    </div>
</template>

<script setup>
    import Demo from '../components/demo.vue'
    let obj = {
        name: '小红',
        age: 18
    }
</script>
```

以上代码等同于

```vue
<template>
    <div>
        <Demo :name="obj.name" :age="obj.age"/>
    </div>
</template>

<script setup>
    import Demo from '../components/demo.vue'
    let obj = {
        name: '小红',
        age: 18
    }
</script>
```

#### 单项数据流

props遵循单项数据流，也就是子组件无法更改父组件传过来的props属性。如果遇到场景需要修改传过来的props，可以使用以下方法

- 可以将获取到的props属性重新赋值即可，但后续props更新就与counter无关了

  ```js
  const props = defineProps(['initialCounter'])
  const counter = ref(props.initialCounter)
  ```

- 将props的值定义为一个计算属性，这样props更新，计算属性会随之更新

  ```js
  const props = defineProps(['size'])
  const normalizedSize = computed(() => props.size.trim().toLowerCase())
  ```

一般情况下，要避免这种修改，但如果真的有此需求，子组件应该抛出一个事件通知父组件进行修改props

#### props校验

也就是上文提到过的，接受对象形式参数，如果类型不符合，则会抛出警告，内容和vue2中的校验基本一致，不多赘述

#### Boolean类型的转换

用以下代码作为示例

```html
<!-- 等同于传入 :disabled="true" -->
<Demo disabled/>
<!-- 等同于传入 :disabled="false" -->
<Demo/>
```

在组件可以如此获取

```javascript
defineProps({
  disabled: Boolean
})
```

一个prop被声明为多种类型时，Boolean类型的特殊转换规则都可以被应用

```js
defineProps({
  disabled: [Boolean, Number]
})
```

### 组件事件

直接在模板中使用$emit即可实现，本方法传入两个参数，第一个为事件名，第二个为传入的参数。见如下代码：

```html
<button @click="$emit('handle', Math.random())">handle</button>
```

子组件如此书写，参数为一个随机数

```html
<Demo @handle="(i) => count+=i"/>
<span>{{ count }}</span>
```

父组件用@加自定义事件即可获取到子组件穿过来的参数，每次点击按钮，count的值就会与传过来的参数相加。并且自定义指令也可以加`.once`修饰符，方法执行一次后不会再执行

#### 声明触发的事件

由于用$emit方法不能在组件`<script setup>`部分中使用，但`defineEmits()`会返回一个相同作用的函数供我们使用：

子组件中：

```vue
<template>
    <div>
        <button @click="btnClick">handle</button>
    </div>
</template>

<script setup>
    const emit = defineEmits(['handle'])
    function btnClick() {
        emit('handle', Math.random())
    }
</script>
```

这样等同于上面效果，但是就可以再js部分操纵了

### 组件v-model

v-model可以实现双向绑定，但是v-model的直接使用，等同于如下代码

```html
<input
  :value="searchText"
  @input="searchText = $event.target.value"
/>
```

所以如果对组件使用v-model我们则需要再子祖先写入如下代码

```vue
<template>
    <div>
        <input type="text" :value="props.modelValue" @input="$emit('update:modelValue', $event.target.value)">
    </div>
</template>

<script setup>
    let props = defineProps(['modelValue']);
    let emit = defineEmits(['update:modelValue'])
</script>
```

用来实现以下两点：

- 将内部原生input元素的value绑定到modelValue属性上
- 当原生的input事件触发时，触发了一个携带新值的`update:modelValue`自定义事件

这样即可以实现双向绑定，在组件上直接v-model绑定即可

#### v-model参数

默认情况下，v-model在组件都是使用modelValue作为prop，并以update:modelValue来作为对应事件，我们也可以更改，如下

```html
<MyComponent v-model:title="bookTitle"/>
```

#### 多个v-model绑定

方法见上

#### 处理v-model修饰符

除了一些内置的修饰符，我们还可以自定义修饰符，见如下代码

- 子组件

  ```vue
  <template>
      <div>
          <input type="text" :value="props.modelValue" @input="handle">
      </div>
  </template>
  
  <script setup>
      let props = defineProps({
          modelValue: String,
          modelModifiers: {default: () => {} }
      });
      let emit = defineEmits(['update:modelValue'])
      function handle(e) {
          let value = e.target.value;
          if(props.modelModifiers.capitalize) {
              value = value.charAt(0).toUpperCase() + value.slice(1);
          }
          emit('update:modelValue', value)
      }
  </script>
  ```

- 父组件

  ```vue
  <template>
      <div>
          <Demo v-model.capitalize="math"/>
      </div>
  </template>
  
  <script setup>
      import Demo from '../components/demo.vue';
      import { ref } from 'vue'
      let math = ref('abc');
  </script>
  ```

这样就可以实现一个将输入框内字符串首字母大写的修饰符

### Attributes继承

再组件上定义类名，id名或者点击事件等等，都会给子组件的根元素添加上，如果子组件的根元素还是个组件，就会继续透传直到html标签根元素

#### 禁用attributes继承

如果不想要一个组件自动继承attribute，则需要设置`inheritAttrs: false`，如果使用了`<script setup>`，则需要一个额外的`<script>`块来添加这个选项

```html
<script>
    export default {
        inheritAttrs: false
    }
</script>
```

这样组件则不会再从父盒子中继承属性

#### $attrs

在模板中使用此属性就可以获取除props，emits之外的一切属性，但有几点注意事项如下

- 透传在js中保留了大小写，像foo-bar这样的attribute需要通过`$attrs['foo-bar']`
- 像@click这样一个事件在此对象下被暴露为一个函数$attrs.onClick

如果想要把透传过来的属性传给组件内的非根元素，则可以先设置禁用，再用`v-bind="$attrs"`

#### 在js区域访问透传

使用useAttrs()API来访问一个组件的所有透传

### 插槽

父组件在引入组件时，在组件标签中间书写内容，在子组件用`<slot></slot>`可以展示出写入的相应数据

#### 插槽作用域

如果用模板语法(`{{}}`)来传入动态值，父组件只能访问父组件作用域，子组件只能访问子组件作用域

#### 默认内容

如果我们在子组件标签中写入如下代码，父组件在引入子组件并未写入内容时，则会用Submit充当默认内容，但是一旦再写入内容，插槽中的内容就会被写入的内容所替代

```
<button type="submit">
  <slot>
    Submit
  </slot>
</button>
```

#### 具名插槽

可以将插槽传到指定位置，v-slot:header等同于#header

- 父组件

  ```html
  <template>
      <div>
          <Demo>
              <template #header>
                  <h1>Here might be a page title</h1>
              </template>
  
              <!-- 隐式的默认插槽 -->
              <p>A paragraph for the main content.</p>
              <p>And another one.</p>
  
              <template #footer>
                  <p>Here's some contact info</p>
              </template>
          </Demo>
      </div>
  </template>
  ```

- 子组件

  ```html
  <template>
      <div>
          <header>
              <slot name="header"></slot>
          </header>
          <main>
              <slot></slot>
          </main>
          <footer>
              <slot name="footer"></slot>
          </footer>
      </div>
  </template>
  ```

#### 作用域插槽

简单说就是在使用插槽时可以向父组件推送值，见如下代码

- 子组件

  ```html
  <template>
      <div>
          <slot math="123" bookName="abc"></slot>
      </div>
  </template>
  ```

- 父组件

  ```html
  <template>
      <div>
          <Demo v-slot="{ math, bookName }">
              {{ math }} {{ bookName }}
          </Demo>
      </div>
  </template>
  ```

### 依赖注入

#### Prop逐级透传问题

因为在透传章节说过，如果一个父组件再给父组件传入属性时，会给子组件的根元素，如果子组件只引入一个组件，那么会再给它子组件的根元素，这样透传的链路非常长，可能会影响到更多这条路上的组件，所以出现了本小节的依赖注入

#### Provide && inject

创建三个文件成如下形式

- 上层组件

  ```vue
  <template>
      <div>
          <Demo></Demo>
      </div>
  </template>
  
  <script setup>
      import Demo from '../components/ppp/demo.vue';
      import { provide } from 'vue';
  
      provide('message', 'ym')
  </script>
  ```

- 中层组件

  ```vue
  <template>
      <grandSon></grandSon>
  </template>
  
  <script setup>
  import grandSon from './grandSon.vue';
  </script>
  ```

- 下层组件

  ```vue
  <template>
      <div>
          grandSon
      </div>
  </template>
  
  <script setup>
      
  import { inject } from 'vue'
  
  const message = inject('message')
  console.log(message);
  </script>
  ```

在上层组件中提供一个值，在他之内的所有组件都可以获取到此值无论几层嵌套

还可以在main.js中全局提供，这样在任何页面都可以注入。如果想让提供的数据为只读的，只需在提供时用readonly()包裹住传过来的值即可
