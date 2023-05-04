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

## 逻辑复用

### 组合式函数

在完成项目时，常常有一部分代码是可以复用的，我们可以把它单独维护到一个文件内，再在需要使用的页面引入。和vue2中的mixin做的是类似的事情但和mixin的用法不同。这样做的好处不言而域，更少的编写重复的代码，更快的开发周期，更清晰的代码结构和更简单的后续维护。

具体实现就是通过ref，watch等vue中的组合式API去在一个单独的文件中书写共用逻辑，在需要的页面引用即可。但还有几点约定需要记住：

- 命名必须使用驼峰命名法并且以"use"开头。由于到了企业，很多代码都是公用的，别人还要看我们写的代码，所以要养成遵守规定的习惯。
- 如果传过来的参数是一个ref，我们要处理好兼容问题，使用`unref()`可以解决此问题
- 我们一直使用的都是ref()因为推荐的约定是组合是函数始终返回一个包含多个ref的普通非相应式对象，这样组件在解构之后仍可以保证是响应式的

#### 副作用

组合式函数可以执行副作用，但要注意以下规则：

- 如果使用了SSR，则确保组件在组建挂载后才调用生命周期钩子中执行DOM相关的副作用
- 确保在`onUnmounted`时清理副作用。举例来说，如果一个组合是函数设置了一个事件监听器，他就应该在`onUnmounted`中被移除

#### 使用限制

组合式函数一定要在`<script setup>`或者`setup()`钩子中，应始终被同步地调用。这个限制是为了让vue能够确定当前执行的是哪个组件实例，可以将生命周期钩子注册到该组件实例上，将计算属性和监听器注册到该组件实例上，以便在该组件被卸载时停止监听，避免内存泄漏

#### 与其他模式比较

##### 和mixin比较

mixin和组合式函数相比有三个短板：

- **不清晰的数据来源**：当使用了多个mixin时，实例上的数据来源于哪个mixin变得不明显，这使代码维护很困难
- **命名空间冲突**：多个来自不同作者地mixin可能会注册相同地属性名，造成命名冲突
- **隐式地跨mixin交流**：多个mixin需要依赖共享属性名来进行相互作用，这使得它们隐式地耦合在一起

##### 和React Hooks的对比

组合式API一部分灵感就来自于Reack Hooks，但vue的组合是函数时基于vue细粒度的相应式系统，这和React hooks的执行模型有本质不同

### 自定义指令

需要使用directive来创建自定义指令，在main.js写入如下代码

```js
app.directive('color', (el, binding) => {
    el.style.color = binding.value
})
```

在页面中使用

```html
<span v-color="'red'">123</span>
```

这样123的字体会变为红色

如果在组件上使用自定义指令，会发生之前章节提到过的透传。并且最好不在组件上使用自定义指令

### 插件

插件是一种能为Vue添加全局功能的工具代码，发挥作用的常见场景有：

- 通过`app.component()`和`app.directive()`注册一到多个全局组件或自定义指令
- 通过`app.provide()`使一个资源可被注入进整个应用
- 向`app.config.globalProperties`中添加一些全局实例属性和方法

#### 编写一个插件

在plugins文件夹下创建一个文件，里面写入此方法

```js
export default {
    install: (app, options) => {
        app.config.globalProperties.$translate = (key) => {
            return key.split('.').reduce((o,i) => {
                console.log(o,i);
                if(o) return o[i]
            }, options)
        }
    }
}
```

install接受的两个参数为app.use使传入的，` app.config.globalProperties`是将translate挂载到vue原型上，key为页面上传进的参数，以下为main.js代码

```js
import i18n from "./plugins/i18n";
app.use(i18n, {
    greetings: {
        hello: 'Bonjour!'
    }
})
```

现在页面就可以使用了， 会出现Bonjour!

```vue
<h1>{{ $translate('greetings.hello') }}</h1>
```

#### 插件中的provide/inject

在插件中我们可以全局provide

```js
export default {
    install: (app, options) => {
        app.config.globalProperties.$translate = (key) => {
            return key.split('.').reduce((o,i) => {
                if(o) return o[i]
            }, options)
        }
        app.provide('i18n', options)
    }
}
```

然后在页面中获取即可

```js
import { inject } from 'vue';
const i18n = inject('i18n');
console.log(i18n);
```

## 内置组件

### Transition

用于组件在切换时发生的动画，由于和vue2大同小异，本章只记录重要的知识点，当作复习

#### CSS过渡class

- `v-enter-from`：

  进入动画的起始状态，在元素插入之前添加，在元素插入完成后的下一帧移除。

- `v-enter-active`：

  进入动画的生效状态，应用于整个进入动画阶段。在元素被插入之前添加，在过渡或动画完成之后移除。这个class可以被用来定义进入动画的持续时间、延迟与速度曲线类型

- `v-enter-to`：

  进入动画的结束状态。在元素插入完成后的下一帧被添加，在过渡或动画完成之后移除。

- `v-leave-from`：

  离开动画的起始状态。在离开过渡效果被触发时立即添加，在一帧后被移除

- `v-leave-active`：

  离开动画的生效状态。应用于整个离开动画阶段。在离开过渡效果被触发时立即添加，在过渡或动画完成之后移除。这个 class 可以被用来定义离开动画的持续时间、延迟与速度曲线类型。

- `v-leave-to`：

  离开动画的结束状态。在一个离开动画被触发后的下一帧被添加，在过渡或动画完成之后移除

#### 为过渡效果命名

在`<Transition></Transition>`添加name属性，则css部分代码的class要改成`修改后的name值-enter-to`

#### CSS的animation和transition

一般过渡都是搭配css原生动画和帧动画去使用，可以使触发的动画更加的圆滑

#### 性能考量

使用transform和opacity之类的属性制作动画是非常高效的，因为它们在动画过程中不会影响到DOM结构，因此不会每一帧都出发昂贵的css布局重新计算。并且大多数的现代浏览器都可以在执行transform动画时利用GPU进行硬件加速

#### js钩子

我们可以通过监听`<Transition>`组件事件的方式在过渡过程中挂上钩子函数

```html
<Transition
  @before-enter="onBeforeEnter"
  @enter="onEnter"
  @after-enter="onAfterEnter"
  @enter-cancelled="onEnterCancelled"
  @before-leave="onBeforeLeave"
  @leave="onLeave"
  @after-leave="onAfterLeave"
  @leave-cancelled="onLeaveCancelled"
>
  <!-- ... -->
</Transition>
```

```js
// 在元素被插入到 DOM 之前被调用
// 用这个来设置元素的 "enter-from" 状态
function onBeforeEnter(el) {}

// 在元素被插入到 DOM 之后的下一帧被调用
// 用这个来开始进入动画
function onEnter(el, done) {
  // 调用回调函数 done 表示过渡结束
  // 如果与 CSS 结合使用，则这个回调是可选参数
  done()
}

// 当进入过渡完成时调用。
function onAfterEnter(el) {}
function onEnterCancelled(el) {}

// 在 leave 钩子之前调用
// 大多数时候，你应该只会用到 leave 钩子
function onBeforeLeave(el) {}

// 在离开过渡开始时调用
// 用这个来开始离开动画
function onLeave(el, done) {
  // 调用回调函数 done 表示过渡结束
  // 如果与 CSS 结合使用，则这个回调是可选参数
  done()
}

// 在离开过渡完成、
// 且元素已从 DOM 中移除时调用
function onAfterLeave(el) {}

// 仅在 v-show 过渡中可用
function onLeaveCancelled(el) {}
```

#### 可复用的过渡效果

可以把`<Transition>`组件包装成一个组件来复用

#### 出现时过渡

如果想在某个节点初次渲染时应用一个效果，可以在标签中添加`appear`属性

#### 元素间过渡

除了单个元素切换，还可以用v-else-if来完成多组件间切换，但要确保任意时刻都要有元素被渲染

#### 过渡模式

在某些时候我们可能需要在组件的切换过程中需要等上一个组件动画结束再开始下一个组件，我们自己来控制是很困难的，但可以添加`mode="out-in"`即可实现

### TransitionGroup

它是用来对v-for列表中的元素插入、移除顺序改变添加动画效果

#### 和Transition的区别

它们两个支持基本相同的props，CSS过渡class和js钩子监听器，但有以下几点区别：

- 默认情况下它不会渲染一个容器元素，但可以通过传入tag="元素标签名"来指定一个元素进行渲染
- 过渡模式在这里不可用，因为不再是互斥的元素之间的切换
- 列表中的每个元素都必须有个独一无二的attribute
- CSS过渡class会被应用再列表内的元素上，而不是容器元素上

例子就不举了，[vue官网](https://cn.vuejs.org/guide/built-ins/transition-group.html)在上可以看到很多例子

### keepAlive

它可以在多个组件动态切换时缓存被移除的组件实例

#### 基本使用

之前章节提到过动态组件，在切换之后不会保留，如下列代码

```vue
<template>
    <div>
        <button @click="tabName = 'demo'">切换组件demo</button>
        <button @click="tabName = 'grandSon'">切换组件grandSon</button>
        <component :is="obj[tabName]"></component>
    </div>
</template>

<script setup>
import demo from '@/components/ppp/demo.vue'
import grandSon from '@/components/ppp/grandSon'
import { ref } from 'vue';
let obj = {
    'demo': demo,
    'grandSon': grandSon
}
let tabName = ref('demo')

</script>
```

demo组件中写有变量加一逻辑，每次加一后切换组件，在切换回demo都会变为零，而是用keepAlive就可以缓存demo这个组件，如下

```html
<keep-alive>
	<component :is="obj[tabName]"></component>
</keep-alive>
```

#### 包含/排除

由于keepAlive默认会缓存内部所有的组件实例，我们可以通过include和exclude属性来定制该行为，这两个属性的值都可以是一个以英文逗号分隔的字符串，一个正则表达式或者包含这两种类型的一个数组

```html
<!-- 以英文逗号分隔的字符串 -->
<KeepAlive include="a,b">
  <component :is="view" />
</KeepAlive>

<!-- 正则表达式 (需使用 `v-bind`) -->
<KeepAlive :include="/a|b/">
  <component :is="view" />
</KeepAlive>

<!-- 数组 (需使用 `v-bind`) -->
<KeepAlive :include="['a', 'b']">
  <component :is="view" />
</KeepAlive>
```

它会根据组件的name选项自动匹配，现在会自动生成对应的name选项了，如下

```html
<keep-alive include="grandSon">
	<component :is="obj[tabName]"></component>
</keep-alive>
```

这样demo组件就不会被缓存了

#### 最大缓存实例数

通过传入max属性可以限制可被缓存的最大组件实例数，在设置此属性后类似于一个LRU缓存(最近最少使用算法)，如果缓存的实例数量即将超过指定的哪个最大数量，则最久没有被访问的缓存实例将会被销毁，为新的实例腾出空间，使用如下

```html
<KeepAlive :max="10">
  <component :is="activeComponent" />
</KeepAlive>
```

#### 缓存实例的生命周期

当一个组件实例从DOM上移除但因为被keepAlive缓存时，他将变为不活跃状态而不是被卸载，当一个组件实例作为缓存树一部分插入到DOM树中，它将重新被激活。一个持续存在的组件可以通过`onActivated()`和`onDeactivated()`注册相应的两个状态的生命周期钩子

- `onActivated()`：组件首次挂载和每次重新被激活时都会调用
- `onDeactivated()`：组件进入缓存时或者被卸载时都会调用

### TelePort

可以将组件内部的一部分模板传送到该组件DOM结构外层的位置

#### 基本用法

比如我们想点击一个按钮弹出模态框，而模态框和按钮的组件没有直接关系，只是需要触发，这时就可以把模态框传入到body之下即可，再次不做过多演示了，因为和vue2基本类似

#### 搭配组件使用

Teleport只改变了渲染的DOM结构，并不会组件间的逻辑关系。如果Teleport包含了一个组件，那么该组件始终和这个使用了它的组件保持料机上的父子关系，并且传出的props和emit也会照常工作，提供与注入也一样

#### 禁用Teleport

一些场景可能需要视情况禁用Teleport，如下书写即可

```html
<Teleport :disabled="isMobile">
  ...
</Teleport>
```

### Suspense

由于不太完善，等待完善后再书写

## 应用模块化

### 单文件组件

单文件组件顾名思义就是`*.vue`文件，英文Single-File Component，简称SFC，是一种特殊的文件格式，是我们能将一个vue组件模板逻辑和样式封装在单个文件里，见如下代码

```vue
<template>
  <p class="greeting">{{ greeting }}</p>
</template>
<script setup>
import { ref } from 'vue'
const greeting = ref('Hello World!')
</script>
<style>
.greeting {
  color: red;
  font-weight: bold;
}
</style>
```

#### 为什么使用SFC

使用SFC有以下几种优点：

- 使用熟悉的HTML，CSS和JS语法编写模块化的组件
- 让本来就强相关的关注点自然内聚
- 预编译模板，避免编译时的运行开销
- 组件作用域的CSS
- 在组合式API时语法更简单
- 通过交叉分析模板和逻辑代码能进行更多编译时优化
- 更好的IDE支持，提供自动补全和对模板中表达式的类型检查
- 开箱即用的模块热更新支持

SFC是vue框架提供的一个功能，以下场景为使用SFC的推荐场景：

- 单页面应用(SPA)
- 静态站点生成(SSG)
- 任何值得引入构建步骤以获得更好的开发体验的项目

但是一些轻量的场景，SFC显得多余，比如只需要给静态HTML做一点交互，则没必要使用SFC

#### SFC是如何工作的

直接再需要使用组件的页面正常引用即可

#### 如何看待关注点分离

关注点分离就是化繁为简，也就是认为SFC将前端三剑客集成到一处有失妥当，应当分离开。

针对这一点我们要达成共识：**前端开发的关注点不是完全基于文件类型分离的**。前端工程化的目的是要能够更好的维护代码。如上述所说并不能帮助我们再日常开发中提升效率。在现代的UI开发中，我们发现与其将代码库划分为三个巨大的层，相互交织在一起，不如将它们划分为松散耦合的组件，再按需组合起来。在一个组件中，把这三者放在一起，实际上会使组件更具有内聚性和可维护性

### 工具链

#### 项目脚手架

##### Vite

Vite是一个轻量级的、速度极快的构建工具，对Vue SFC提供第一优先级支持

##### Vue CLI

Vue CLI是官方提供的基于webpack的Vue工具链，现在处于维护模式。官方推荐使用Vite开始新的项目，除非当前项目依赖特定的webpack的特性。大多数情况，Vite将提供更优秀的开发体验

#### IDE支持

- 推荐使用VSCode，配合Vue语言特性(Volar)插件，该插件提供了语法高亮、TypeScript支持，以及模板内表达式与组件props的只能提示
- WebStorm同样也为Vue的单文件组件提供了很好的内置支持

#### 浏览器开发插件

devtools，就是大家喜闻乐见的vue开发者工具

#### 测试

- Cypress推荐用于 E2E测试，也可以通过Cypress组件测试运行器，来给Vue SFC作单文件组件测试
- Vitest是一个追求更快运行速度的测是运行器，由vue/vite团队成员开发，主要基于vite的应用设计，可以为组件提供即时相应的测式反馈
- Jest可以通过vite-jest配合vite使用，不过只推荐在你已经有一套基于jest的测试集，且想要迁移到基于vite的开发配置时使用，因为vitest也能够提供类似的功能，且后者与vite的集成更方便高效

#### 代码规范

eslint是通用的

#### 格式化

Volar有格式化功能，Prettier也提供了内置的格式化支持

#### SFC自定义块集成

自定义块被编译成导出到同一Vue文件的不同请求查询。这取决于底层构建工具如何处理这类导入请求。

- 如果使用Vite，需使用一个自定义Vite插件将自定义块转换为可执行的js代码
- 如果使用Vue CLI或只是webpack，需要使用一个loader来配置如何转换匹配到的自定义块

#### 底层库

##### `@vue/compiler-sfc`

这个包是Vue核心monorepo的一部分，并始终和vue主包版本号保持一致，它成为vue主包的一个依赖被代理到了vue/compiler-sfc目录下，因此你无需单独安装它。这个包本身提供了处理SFC的底层的功能，并只适用于需要支持VueSFC相关工具链的开发者

##### `@vitejs/plugin-vue`

为Vite提供Vue SFC支持的官方插件

##### `vue-loader`

为webpack提供Vue SFC支持的官方loader

### 路由

#### 客户端 vs 服务端路由

服务端路由是指服务器根据用户访问的URL路径返回不同的响应结果。当我们在一个传统的服务端渲染的web应用中点击一个链接时，浏览器会从服务端获得全新的HTML，然后重新加载整个页面

然而，在单页面应用中，客户端的js可以拦截页面的跳转请求，动态获取新的数据，然后再无需重新加载的情况下更新当前页面。这样通常可以带来更顺滑的用户体验，尤其是在更偏向应用的场景下，因为这类场景下用户通常会在很长的一段事件中做出多次交互

在这类单页应用里，路由是在客户端执行的。一个客户端路由器的职责就是利用诸如HisroryAPI或是hashChange事件这样的浏览器API来管理应用当前应该渲染的视图

#### 实现一个简单的路由

创建两个组件引入到一个组件中，见如下代码书写即可

```vue
<template>
    <a href="#/">Home</a>
    <a href="#/demo">Demo</a>
    <component :is="currentView"></component>
</template>

<script setup>
    import { computed, ref } from 'vue'
    import demo from '../components/ppp/demo.vue';
    import Home from './Home.vue';

    const routes = {
        '/': Home,
        '/demo': demo
    }
    const currentPath = ref(window.location.hash);
    window.addEventListener('hashchange', () => {
        currentPath.value = window.location.hash;
        console.log(currentPath);
    })
    const currentView = computed(() => {
        return routes[currentPath.value.slice(1) || '/']
    })
</script>
```

### 状态管理

意思就是再单一的页面，别的页面想要更改当前页面数据，如果是父组件改变子组件还比较简单，但如果是没有关系的两个组件就会很复杂。说到这大家都能想到了，就是vuex。

vuex是vue官方针对状态管理而去编写的一个库，但现在vue3的出现使得另一个状态管理库将要迭代vuex，也就是Pinia(绰号大菠萝)。

#### pinia

用现成的响应式API实现状态管理是可行的，但如果是大规模的生产应用还需要考虑以下事项：

- 更强的团队协作约定
- 与Vue DevTools集成，包括时间轴、组件内部审查和时间旅行调试
- 模块热更新
- 服务端渲染支持

Pinia就实现了上述需求，由vue核心团队维护。Pinia在生态系统能够和vuex承担相同的职责且能做的更好，因此我们建议使用Pinia。事实上Pinia最初是为了探索Vuex的下一个版本而开发的，而最终发现，Pinia已经实现了Vuex5中的大部分内容，所以将其作为了新的官方推荐。相比于Vuex，Pinia提供了更简洁直接的API，并提供了组合式API的风格，最重要的事，再使用TS时它提供了更完善的类型推导

### 测试

测试的意义在于预防代码中可能出现的意想不到的bug，长期测试可以保证代码的见状性并且能帮助自己和团队更快速，更自信的构建复杂的vue项目。

#### 测试要做什么

用白话讲就是检测所写的函数是否按照我们的预期执行

#### 测试的类型

基本分为以下三种

- 单元测试：检查给定函数、类或组合式函数的输入是否产生预期的输出或副作用
- 组件测试：检查组件是否正常挂载和渲染、是否可以互动，以及表现是否符合预期。这些测试比单元测试导入了更多的代码，更复杂，需要更多时间来执行
- 端到端测试：检查跨越多个页面的功能，并对生产构建的vue应用进行实际的网络请求。这些测试通常涉及到建立一个数据库或其他后端

以下拿单元测试举个简单的例子

#### 单元测试

创建一个vite搭建的vue项目，并且安装vitest(vue核心团队维护的测试工具)

在vite.config.js中添加如下配置

```js
/// <reference types="vitest"/>

import { defineConfig } from "vitest/config";

export default defineConfig({
    test: {
        // ...
    }
})
```

在package.json中添加启动命令

```json
"scripts": {
    "test": "vitest",
    "coverage": "vitest run --coverage"
  },
```

创建一个函数sum.js，抛出了一个increment函数

```js
export function increment(current, max = 10) {
    if (current < max) {
        return current + 1
    }
    return current
}
```

创建一个sum.test.js文件用于测试，代码如下

```js
import { increment } from '../src/components/sum'

import { describe, expect, it, test } from 'vitest'

describe('increment', () => {
    it('increments the current number by 1', () => {
        expect(increment(0, 10)).toBe(1)
    })

    it('does not increment the current number over the max', () => {
        expect(increment(10, 10)).toBe(10)
    })

    it('has a default max of 10', () => {
        expect(increment(10)).toBe(10)
    })
})
```

引入的describe是一个作用域，expect是断言，it或者test都是定义一种测试期望的方法，toBe用于断言基础对象是否相等。到这可能就看不懂什么意思了，用白话说一下。

describe就是划分一个区域，只测试这一个函数。it或者test可以接受一条测试，第一个参数是测试名称，第二个参数是测试用例。expect传入一个函数可以获取函数的返回值，再和toBe里面的参数作比较，相等则测试成功，不相等则测试失败

最后可以运行以下npm run test看一下测试结果

### 服务端渲染

已经多次记录过，不再赘述
