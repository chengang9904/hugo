+++
date = '2026-01-05T18:12:04+08:00'
draft = true
title = ''
+++


## 前言

本文是《Vue.js 从入门到实战》视频课程的配套文章，课程详细讲解了如何使用 Vue.js 框架开发一个完整的 TodoMVC 应用。如果你更喜欢视频学习，可以点击下面的链接观看完整课程：
[视频课程](https://www.bilibili.com/video/BV1714y167Sn/?spm_id_from=333.788.player.switch&vd_source=63362d5603e89561480602ac7231f43f&trackid=web_related_0.router-related-2206146-hhltv.1767545870122.860)

## 项目介绍

TodoMVC 是一个经典的前端示例项目，旨在展示各种前端框架和库如何实现相同的功能。通过构建一个简单的待办事项应用，开发者可以比较不同技术栈的优缺点，并学习最佳实践。

## 搭建vue

在开始编码之前，我们需要确保已经安装了 Node.js 和 Vue CLI。如果还没有安装，可以参考以下步骤：

1. 安装 Node.js：访问 [Node.js 官网](https://nodejs.org/) 下载并安装最新的 LTS 版本。
2. 安装 Vue CLI：打开终端或命令提示符，运行以下命令安装 Vue CLI：

   ```bash
   npm install -g @vue/cli
   ```

3. 创建 Vue 项目：运行以下命令创建一个新的 Vue 项目：

    ```bash
    npm create vue@2
    ```

4. 选择项目配置：在提示中选择默认配置，等待项目创建完成。
5. 进入项目目录：运行以下命令进入项目目录：

   ```bash
   cd your-project-name
   npm install
   ```

6. 启动开发服务器：运行以下命令启动开发服务器：

   ```bash
   npm run serve
   ```

现在，你已经成功搭建了一个 Vue.js 项目，接下来我们将开始构建 TodoMVC 应用的各个部分。

## 项目结构

在开始编码之前，我们先了解一下项目的基本结构。一个典型的 Vue.js 项目包含以下几个主要目录和文件：

```
your-project-name/
├── public/                 # 静态资源文件夹
│   └── index.html          # 入口 HTML 文件
├── src/                    # 源代码文件夹
│   ├── assets/             # 静态资源（图片、样式等）
│   ├── components/         # Vue 组件文件夹
│   ├── App.vue             # 根组件
│   └── main.js             # 入口 JavaScript 文件
├── package.json            # 项目配置文件
└── README.md               # 项目说明文件
```

## 拷贝模板

为了节省时间，我们可以直接使用现成的 TodoMVC Vue 模板。你可以从 [TodoMVC GitHub 仓库](<https://github.com/tastejs/todomvc-app-template）下载> Vue 版本的模板，或者使用以下命令克隆仓库：

```bash
git clone git@github.com:tastejs/todomvc-app-template.git
```

我们只拷贝 index.html 中我们需要的部分，然后样式通过链接导入

css 文件通过链接导入

```
@import 'https://unpkg.com/todomvc-app-css@2.4.1/index.css';
```

然后删掉script中的内容

在main.js 中删掉css的导入

最后搂一眼，差不多就是基础的样式搭建好了

## 编写代码

首先需要一个响应式的数据结构来存储待办事项列表。我们可以在 Vue 组件的 `data` 选项中定义一个数组来保存待办事项。


### 编写模拟的数据


```javascript
data() {
  return {
    todos: [
        { id: 1, title: '学习 Vue.js', completed: true  },
        { id: 2, title: '构建 TodoMVC 应用', completed: false },
    ],
  };
}
```

### 渲染待办事项列表
找到<li>标签，使用v-for指令遍历todos数组，动态生成待办事项列表。

```html
<li v-for="todo in todos" :key="todo.id" :class="{ completed: todo.completed }">
```

### 绑定待办事项数据

使用v-model指令实现双向绑定。

label标签中显示待办事项的标题，可以用插值表达式来显示todo.title，也可以使用v-text指令。

```
<div class="view">
    <input class="toggle" type="checkbox" v-model="todo.completed">
    <label>{{ todo.title }}</label>
    <button class="destroy"></button>
</div>
```

### ToggleAll

实现全选和取消全选功能。我们可以使用计算属性来判断是否所有待办事项都已完成，并通过一个方法来切换所有待办事项的状态。

```javascript
		toggleAll(e) {
			this.todos.forEach(todo => {
				todo.completed = e.target.checked;
			});
		},
``` 

### 过滤 

实现待办事项的过滤功能。我们可以使用计算属性来根据当前的过滤条件返回不同的待办事项列表。

添加一个visibility属性来存储当前的过滤条件：

添加mounted钩子来监听hash变化：

```javascript
  methods: {
		onHashChange() {
			const hash = window.location.hash.replace(/#\/?/, '')
			if (['all', 'active', 'completed'].includes(hash)) {
				this.visibility = hash
			} else {
				window.location.hash = ''
				this.visibility = 'all'
			}
		},
  },

	mounted() {
		window.addEventListener('hashchange', this.onHashChange);
		this.onHashChange();
	}
```


然后使用计算属性filteredTodos来根据visibility属性过滤待办事项：


```javascript
computed: {
    filteredTodos() {
    if (this.visibility === 'all') {
      return this.todos;
    } else if (this.visibility === 'active') {
      return this.todos.filter(todo => !todo.completed);
    } else if (this.visibility === 'completed') {
      return this.todos.filter(todo => todo.completed);
    }
  },
}
```

最后，在模板中使用filteredTodos来渲染待办事项列表：

### 计算未完成事项数

我们可以使用计算属性来计算未完成事项的数量：

```javascript
computed: {
  remaining() {
    return this.todos.filter(todo => !todo.completed).length;
  },
}
``` 

然后在模板中显示未完成事项数：

### 清除已完成事项

我们可以添加一个方法来清除所有已完成的待办事项：

```javascript
methods: {
  clearCompleted() {
    this.todos = this.todos.filter(todo => !todo.completed);
  },
}
```


然后在模板中绑定点击事件：

### 新增删除待办事项


```javascript
  methods: {
		addTodo(e) {
			const title = e.target.value.trim();
			if (title) {
				this.todos.push({
					id: this.todos.length + 1,
					title,
					completed: false
				});
				e.target.value = '';
			}
		}, 
    removeTodo(todo) {
			this.todos = this.todos.filter(t => t.id !== todo.id);
		},
  }
```

### 移除已完成事项

```javascript
		clearCompleted() {
			this.todos = this.todos.filter(todo => !todo.completed);
		}
```

### 细节处理

`items-left` 显示未完成事项数，使用计算属性 `remaining` 来动态计算未完成事项的数量。

```javascript
computed: {
  remaining() {
    return this.todos.filter(todo => !todo.completed).length;
  },
}
```

然后在模板中显示未完成事项数：

```html
<span class="todo-count">
  <strong>{{ remaining }}</strong> item{{ remaining !== 1 ? 's' : '' }} left
</span>
```

如果没有待办事项时，隐藏底部栏和待办事项列表，可以使用 `v-if` 指令来实现：

```html
<footer class="footer" v-if="todos.length > 0">
```

或者

```html
		<footer v-show="todos.length" class="footer">
``` 


## 总结

通过本文的介绍，我们学习了如何使用 Vue.js 框架构建一个完整的 TodoMVC 应用。我们涵盖了从项目搭建、数据管理、事件处理到视图渲染的各个方面。希望这篇文章能帮助你更好地理解 Vue.js 的核心概念和实战技巧。如果你对 Vue.js 有更深入的兴趣，建议继续学习官方文档和相关课程，提升你的前端开发技能。

[我的代码仓库地址](https://github.com/chengang9904/todomvc)