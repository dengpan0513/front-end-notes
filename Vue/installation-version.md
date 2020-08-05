# 理解 Vue 不同版本

### 背景

在使用 `@vue/cli` 创建项目的过程中，在 `src/main.js` 中作如下修改：

```javascript
// main.js
import Vue from 'vue'

Vue.config.productionTip = false

new Vue({
  // render: h => h(App),
  data: {
    message: 'hello Vue'
  }
}).$mount('#app')
```

同时在 `public/index.html` 页面中尝试着使用 Vue 的文本插值来进行数据绑定，新增如下代码：

```html
<!-- index.html -->
<div id="app">
  {{ message }}
</div>
```

然后使用 `npm run serve` 启动服务，打开页面却发现页面中无内容显示，同时控制台中有如下图所示的报错信息：

![报错信息](./imgs/vue-version-error.jpg)

以上报错信息表示当前正在使用的是没有模板编译器的只包含运行时的 Vue 版本，解决办法要不使用 `render` 函数，要不使用包含编译器的 Vue 版本。那接下来就解决这个报错信息

### 解决报错

##### 术语解释

通过 Vue 文档可以知道其有很多不同的 Vue.js 构建版本，但大体上可分为**完整版**、**只包含运行时版本**这两个版本，其中有几个术语如下：

- **完整版：同时包含编译器和运行时的版本**
- **编译器：用来将模板字符串编译成为 JavaScript 渲染函数的代码**
- **运行时：用来创建 Vue 实例、渲染并处理虚拟 DOM 等的代码。基本上是完整版中出去编译器后的一切**

##### 使用完整版

解决上面报错的方法之一就是使用完整版的 Vue，这里我们通过 CDN 来引入，在 `public/index.html` 中新增如下代码：

```html
<script src="https://cdn.bootcss.com/vue/2.6.11/vue.js"></script>
```

同时在 `src/main.js` 中将代码作如下修改：

```javascript
const Vue = window.Vue
new Vue({
  el: '#app',
  data: {
    message: 'hello Vue'
  }
})
```

这样便可以解决报错信息，并且页面中也会显示 `message` 属性的值。在完整版中，使用 **`template`** 也可以达到同样的效果，如下代码：

```javascript
new Vue({
  el: '#app',
  template: `<div>{{ message }}</div>`,
  data: {
    message: 'hello Vue'
  }
})
```

##### 使用只包含运行时版

要解决只包含运行时版本下的报错，方法就是使用 `render` 函数，此时 `public/index.html` 代码如下

```html
<div id="app"></div>
```

而在 `src/main.js` 中代码如下

```javascript
new Vue({
  el: '#app',
  render (h) {
    return h('div', this.message)
  },
  data: {
    message: 'hello Vue'
  }
})
```

这样便解决了控制台的报错

### 完整版 VS 只包含运行时版

通过上述两种解决报错的方法，可以发现使用完整版的方法会在代码书写上会更加直观方便。那为什么 `@vue/cli` 中使用的是只包含运行时版呢？原因就在于：

> 只包含运行时版相比完整版的体积要小30%左右

比如完整版体积为100k，那么只包含运行时版的体积大约是70k。而完整版体积大的这一部分就是 `compiler` 的体积，`compiler` 将模板字符串编译成为 JavaScript 渲染函数，比如如下的模板字符串：

```html
<div id="app">
  {{ n }}
  <button @click="add">+1</button>
</div>
```

那么从上述 `compiler` 的作用看出，**无论是直接在HTML 里还是使用 template 选项完整版都支持，而只包含运行时版则都不支持**。在下面两种情况下都需要编译器：

- **挂载到一个元素上并以其 DOM 内部的 HTML 作为模板**，代码如下

  ```html
  <!-- 需要编译器 -->
  <div id="app">
    {{ message }}
  </div>
  ```

- **使用 `template` 选项**，代码如下

  ```javascript
  // 需要编译器
  new Vue({
    el: '#app',
    template: '<div>{{ message }}</div>',
    data: {
      message: 'hello Vue'
    }
  })
  ```

### 总结

- 完整版：保证开发体验，开发者可以直接在 `*.vue` 文件里使用模板语法，在代码书写上直观方便，但是不支持 `render` 函数，导致体积较大
- 只包含运行时版：保证用户体验，用户下载的 JS 文件体积更小，但是只支持 `render` 函数，所以在开发上不是很方便

- 最佳实践：使用 webpack 等打包工具，总是使用**只包含运行时版**，配合着 `vue-loader` 使用：
  - 开发时用完整版的方式写代码
  - 项目进行构建时 `vue-loader` 将模板字符串编译成为 JavaScript 的渲染函数
  - 最终打好的包里就不需要编译器，所以只用运行时版即可


