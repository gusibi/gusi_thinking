---
date: '2024-10-01T11:26:33+08:00'
lastmod: '2024-10-01T11:26:33+08:00'
draft: false
title: Nuxt quickstart 新手友好版
slug: nuxt-quickstart-beginner
description: 嗨，欢迎来到 Nuxt 3 的世界！👋 如果你是 Web 开发新手，或者刚刚接触 Nuxt，不用担心，这份指南会带你一步步了解 Nuxt
  3 的基础知识。我们会用简单的语言和实例来解释每个概念，让你轻松入门这个强大的 Vue.js 框架。
tags:
- 技术
categories:
- 技术
comment: false
toc: false
---

> nuxt.js quickstart
>
> 将这个nuxt.js quickstart ，请你扮演 nuxt 开发者，用更像是一个面向新手的博客的文风润色一下，如果有地方不详细，可以更详细的解释一下，请注意，要保留当前的目录结构，只需要润色



# Nuxt 3 新手友好指南: 从零开始构建你的第一个应用

嗨，欢迎来到 Nuxt 3 的世界！👋 如果你是 Web 开发新手，或者刚刚接触 Nuxt，不用担心，这份指南会带你一步步了解 Nuxt 3 的基础知识。我们会用简单的语言和实例来解释每个概念，让你轻松入门这个强大的 Vue.js 框架。准备好开始你的 Nuxt 3 之旅了吗？让我们开始吧！

## 1. 创建你的第一个 Nuxt 3 应用

首先, 让我们从创建一个全新的 Nuxt 3 项目开始我们的冒险吧! 打开你最喜欢的终端, 跟着我一起敲几行命令:

```bash
npx nuxi init my-awesome-nuxt-app
cd my-awesome-nuxt-app
npm install
```

哇! 就这么简单, 你已经创建了你的第一个 Nuxt 3 项目。但是等等, 这些命令到底做了什么? 让我来解释一下:

1. `npx nuxi init my-awesome-nuxt-app`: 这行命令告诉 `npx` (一个 npm 包运行器) 使用 `nuxi` (Nuxt 3 的命令行工具) 来初始化一个新项目。`my-awesome-nuxt-app` 是你项目的名字, 当然, 你可以换成任何你喜欢的名字!

2. `cd my-awesome-nuxt-app`: 这个命令让我们进入刚刚创建的项目文件夹。`cd` 是 "change directory" 的缩写, 就是切换目录的意思。

3. `npm install`: 这个命令会安装项目所需的所有依赖包。想象一下你在组装一个模型, 这个命令就是在给你准备所有需要的零件。

现在, 让我们看看我们的项目结构:

```
my-awesome-nuxt-app/
  ├── app.vue
  ├── nuxt.config.ts
  └── package.json
```

这些文件都是干什么用的呢? 让我来给你介绍一下这些新朋友:

- `app.vue`: 这是你的应用的主组件, 所有的页面都会在这里渲染。它就像是你的应用的"外壳"。
- `nuxt.config.ts`: 这是 Nuxt 的配置文件, 你可以在这里自定义 Nuxt 的行为。把它想象成你的应用的"控制面板"。
- `package.json`: 这个文件包含了项目的元数据和依赖信息。它就像是你项目的"身份证"。

让我们看看 `app.vue` 的内容:

```vue
<template>
  <div>
    <h1>欢迎来到 Nuxt 3!</h1>
    <NuxtPage />
  </div>
</template>
```

这段代码做了两件事:
1. 显示一个欢迎标题。
2. `<NuxtPage />` 组件会根据当前的 URL 自动加载并显示相应的页面内容。它就像是一个神奇的占位符, Nuxt 会自动填充它。

恭喜你! 你已经成功创建了你的第一个 Nuxt 3 应用。接下来, 让我们看看如何运行它并进行调试。

## 2. 调试模式: 你的开发好帮手

在开发过程中, 能够快速看到变化并及时发现错误是非常重要的。Nuxt 3 默认在开发模式下启用了调试功能, 让我们来体验一下:

```bash
npm run dev
```

输入这个命令后, 你会看到终端输出一些信息, 最后会告诉你应用运行在哪个地址上, 通常是 `http://localhost:3000`。

现在, 打开你的浏览器, 输入这个地址, 你就能看到你的 Nuxt 3 应用啦!🎉

但是等等, 调试模式到底给我们带来了什么好处呢?

1. **热重载**: 当你修改代码时, 浏览器会自动刷新, 显示最新的变化。你再也不需要手动刷新页面了!

2. **Vue Devtools**: 如果你安装了 Vue Devtools 浏览器扩展, 你可以使用它来检查你的组件结构、状态等。

3. **详细的错误信息**: 如果你的代码中有错误, Nuxt 会在浏览器控制台和终端中显示详细的错误信息, 帮助你快速定位问题。

想象一下, 调试模式就像是你的个人助理, 时刻关注着你的代码, 一旦发现问题就立即报告。是不是感觉开发变得轻松多了?

## 3. HTML 转义: 安全第一!

在 Web 开发中, 安全性是一个非常重要的话题。其中一个常见的安全问题是跨站脚本攻击 (XSS)。不用担心, Nuxt 3 已经为你考虑到了这一点!

Nuxt 3 使用 Vue 3, 它会自动对插值中的 HTML 进行转义。这听起来可能有点复杂, 让我用一个例子来解释:

```vue
<template>
  <div>{{ userInput }}</div>
</template>

<script setup>
const userInput = '<script>alert("这是一个 XSS 攻击!")</script>';
</script>
```

在这个例子中:
- `<script setup>`: 这是 Vue 3 的组合式 API 语法, 用于定义组件的逻辑。
- `const userInput`: 我们定义了一个包含潜在危险 HTML 的变量。

如果 Nuxt 不进行 HTML 转义, 这段代码可能会执行一个弹窗。但是不用担心, Nuxt 会自动将这些特殊字符转换为安全的形式。

当这段代码渲染时, `<script>` 标签会被转义, 显示为纯文本, 而不是被作为 JavaScript 执行。用户会在页面上看到 `<script>alert("这是一个 XSS 攻击!")</script>` 这样的文本, 而不是弹出一个警告框。

这就像是 Nuxt 为你的应用安装了一个安全过滤器, 确保用户输入的内容不会意外地变成可执行的代码。

记住, 虽然 Nuxt 提供了这层保护, 但在处理用户输入时仍然需要保持警惕。安全性是一个复杂的话题, 这只是其中的一个方面。

## 4. 路由: 让你的应用四通八达

在 Nuxt 3 中, 路由就像是你应用的地图, 告诉 Nuxt 在用户访问不同 URL 时应该显示什么内容。最棒的是, Nuxt 使用了一个叫做"基于文件的路由系统", 这意味着你的文件结构决定了你的路由! 听起来有点神奇, 对吧? 让我们一起探索一下。

首先, 创建一个 `pages` 文件夹, 然后在里面添加一些 `.vue` 文件:

```
pages/
  ├── index.vue
  └── about.vue
```

就这么简单, 你已经创建了两个路由:
- `/`: 对应 `pages/index.vue`
- `/about`: 对应 `pages/about.vue`

让我们看看 `index.vue` 的内容可能是什么样的:

```vue
<template>
  <div>
    <h1>欢迎来到我的 Nuxt 3 应用!</h1>
    <p>这是首页。</p>
  </div>
</template>
```

而 `about.vue` 可能是这样的:

```vue
<template>
  <div>
    <h1>关于我们</h1>
    <p>我们是一个使用 Nuxt 3 的超棒团队!</p>
  </div>
</template>
```

但是等等, 如果我们想要动态路由呢? 比如, 一个用户个人资料页面, URL 中包含用户 ID? Nuxt 也为我们考虑到了这一点!

创建一个文件 `pages/users/[id].vue`:

```vue
<template>
  <div>
    <h1>用户个人资料</h1>
    <p>用户 ID: {{ $route.params.id }}</p>
  </div>
</template>

<script setup>
const route = useRoute();
console.log(route.params.id);
</script>
```

这里发生了什么?
- `[id].vue`: 方括号表示这是一个动态参数。
- `{{ $route.params.id }}`: 在模板中, 我们可以通过 `$route.params` 访问路由参数。
- `useRoute()`: 这是一个组合式函数, 用于在 `<script setup>` 中访问当前路由。

现在, 当用户访问 `/users/1`, `/users/2` 等 URL 时, 都会显示这个页面, 而 `id` 参数会被传递给组件。

是不是感觉路由变得超级简单? 你只需要创建文件, Nuxt 就会自动为你处理所有的路由逻辑。这就是 Nuxt 的魔力!

## 5. 静态文件: 处理图片和其他资源

在 Web 开发中, 处理静态文件如图片、字体或其他资源文件是非常常见的需求。Nuxt 3 为我们提供了一个简单的方法来处理这些文件。

首先, 在你的项目根目录下创建一个 `public` 文件夹。这个文件夹很特别, 因为 Nuxt 会直接将其中的内容映射到你的网站根目录。

让我们来试试看:

1. 在 `public` 文件夹中创建一个 `images` 文件夹。
2. 把一张名为 `logo. png` 的图片放入 `images` 文件夹。

现在你的文件结构可能看起来像这样:

```
my-awesome-nuxt-app/
  ├── public/
  │   └── images/
  │       └── logo. png
  ├── app. vue
  ├── nuxt. config. ts
  └── package. json
```

太棒了! 现在你可以在你的 Vue 组件中使用这张图片了。让我们修改一下 `app. vue`:

```vue
<template>
  <div>
    <img src="/images/logo. png" alt="我的 Logo">
    <h 1>欢迎来到我的 Nuxt 3 应用!</h 1>
    <NuxtPage />
  </div>
</template>
```

注意看 `src` 属性:
- `/images/logo. png`: 这个路径是相对于服务器根路径的, 不需要包含 `public`。

当你运行你的应用时, 你应该能看到 logo 图片显示在页面上了。是不是很简单?

这种方法不仅适用于图片, 还适用于任何你想直接提供给浏览器的静态文件, 比如 `robots. txt`, `favicon. ico` 等。

这种方法有几个好处：

1. 简单直观：你不需要记住复杂的路径，只需要记住文件在 `public` 目录中的位置。
2. 性能优化：Nuxt 会自动处理这些静态文件，确保它们能够高效地被加载。
3. 部署友好：当你部署你的应用时，这些文件会被自动处理，你不需要额外的配置。

记住，`public` 目录不仅仅用于图片。你可以在这里放置任何类型的静态文件，比如字体文件、robots.txt、favicon.ico 等。Nuxt 会确保这些文件都能被正确访问。


## 6. 渲染模板:组件化的魔力

在现代前端开发中,组件化是一个非常重要的概念。它允许我们将界面拆分成小的、可重用的部分,就像乐高积木一样。Nuxt 3 完全拥抱了这个理念,让我们来看看如何创建和使用组件。

首先,让我们创建一个简单的用户卡片组件。在项目根目录下创建一个 `components` 文件夹,然后在里面创建 `UserCard.vue` 文件:

```vue
<template>
  <div class="user-card">
    <img :src="user.avatar" :alt="user.name" class="user-avatar">
    <h2>{{ user.name }}</h2>
    <p>{{ user.email }}</p>
  </div>
</template>

<script setup>
defineProps(['user']);
</script>

<style scoped>
.user-card {
  border: 1px solid #ccc;
  border-radius: 8px;
  padding: 16px;
  margin: 16px 0;
  text-align: center;
}
.user-avatar {
  width: 100px;
  height: 100px;
  border-radius: 50%;
}
</style>
```

让我解释一下这个组件的各个部分:

1. `<template>`: 这里定义了组件的 HTML 结构。我们使用了一些动态绑定(`:src` 和 `:alt`)和插值(`{{ }}`)来显示用户信息。

2. `<script setup>`: 这是 Vue 3 的组合式 API 语法。`defineProps(['user'])` 声明了这个组件接受一个 `user` 属性。

3. `<style scoped>`: 这里定义了组件的样式。`scoped` 属性确保这些样式只应用于这个组件,不会影响其他地方。

现在,让我们在一个页面中使用这个组件。修改 `pages/index.vue`:

```vue
<template>
  <div>
    <h1>用户列表</h1>
    <UserCard v-for="user in users" :key="user.id" :user="user" />
  </div>
</template>

<script setup>
const users = ref([
  { id: 1, name: '张三', email: 'zhangsan@example.com', avatar: 'https://placekitten.com/100/100' },
  { id: 2, name: '李四', email: 'lisi@example.com', avatar: 'https://placekitten.com/101/101' },
]);
</script>
```

这里发生了什么?

1. 我们导入并使用了 `UserCard` 组件。注意,我们不需要显式导入它 - Nuxt 会自动导入 `components` 文件夹中的组件。

2. 我们使用 `v-for` 指令来循环渲染用户列表。每个 `UserCard` 都接收一个 `user` 对象作为属性。

3. `:key="user.id"` 为每个列表项提供了一个唯一的 key,这有助于 Vue 优化渲染。

4. `ref([...])` 创建了一个响应式数组,包含用户数据。在实际应用中,这些数据可能来自 API 调用。

现在,当你运行你的应用时,你应该能看到一个漂亮的用户列表,每个用户都显示在一个卡片中。

组件化的好处是什么?

1. **可重用性**: 你可以在应用的任何地方重复使用 `UserCard` 组件。
2. **可维护性**: 如果你需要修改用户卡片的外观,你只需要修改一个地方。
3. **可测试性**: 你可以单独测试 `UserCard` 组件,确保它在各种情况下都能正确工作。

这就是组件化的魔力!它让你的代码更加整洁、高效,并且易于管理。随着你的应用成长,你会发现组件化成为你构建复杂界面的强大工具。

## 7. 状态管理:保持数据的整洁

当你的应用变得更复杂时,管理状态(即你的应用数据)变得越来越重要。Nuxt 3 提供了一种简单而强大的方式来管理状态,这就是 "Nuxt State"。

让我们通过一个简单的计数器例子来看看如何使用它。

首先,创建一个新文件 `composables/useState.js`:

```javascript
export const useCounter = () => useState('counter', () => 0);
```

这段代码做了什么?

- `useState`: 这是 Nuxt 3 提供的函数,用于创建一个响应式状态。
- `'counter'`: 这是状态的唯一标识符。如果多个组件使用相同的标识符,它们会共享同一个状态。
- `() => 0`: 这是状态的初始值。在这里,我们的计数器从 0 开始。

现在,让我们在组件中使用这个状态。创建一个新的组件 `components/Counter.vue`:

```vue
<template>
  <div>
    <p>计数: {{ count }}</p>
    <button @click="increment">增加</button>
    <button @click="decrement">减少</button>
  </div>
</template>

<script setup>
const count = useCounter();

function increment() {
  count.value++;
}

function decrement() {
  count.value--;
}
</script>
```

让我们来解析这个组件:

1. 我们使用 `useCounter()` 来获取计数器的状态。
2. `count.value` 用于访问和修改状态值。注意 `.value` 的使用,这是因为 `count` 是一个响应式引用。
3. `increment` 和 `decrement` 函数分别用于增加和减少计数。

现在,你可以在任何页面或组件中使用这个 `Counter` 组件。例如,修改 `pages/index.vue`:

```vue
<template>
  <div>
    <h1>Nuxt 3 计数器示例</h1>
    <Counter />
    <Counter /> <!-- 注意,我们使用了两次 Counter 组件 -->
  </div>
</template>
```

当你运行这个应用时,你会看到两个计数器。有趣的是,它们共享同一个状态!如果你增加其中一个计数器,另一个也会跟着变化。这就是共享状态的强大之处。

这种方法的优点是什么?

1. **简单**: 你不需要设置复杂的状态管理库。
2. **灵活**: 你可以轻松地在不同组件之间共享状态。
3. **响应式**: 状态的变化会自动反映在使用它的所有地方。

状态管理可能看起来有点抽象,但它是构建大型、复杂应用的关键。随着你的 Nuxt 3 之旅继续,你会发现越来越多使用共享状态的场景。

记住,虽然这种方法对于简单到中等复杂度的应用来说已经足够了,但对于更复杂的状态管理需求,Nuxt 3 也完全兼容像 Pinia 这样的状态管理库。

好啦,现在你已经掌握了 Nuxt 3 中状态管理的基础知识。准备好探索更多高级特性了吗?让我们继续我们的 Nuxt 3 冒险之旅吧！

## 8. 异步数据获取:与后端对话

在实际应用中,我们经常需要从服务器获取数据。Nuxt 3 提供了两个强大的组合式函数来处理异步数据获取: `useFetch` 和 `useAsyncData`。今天,我们主要看看 `useFetch`，因为它更简单直接。

让我们创建一个简单的博客文章列表页面来展示如何使用 `useFetch`。

创建一个新文件 `pages/posts.vue`:

```vue
<template>
  <div>
    <h1>博客文章</h1>
    <div v-if="pending">加载中...</div>
    <div v-else-if="error">错误: {{ error.message }}</div>
    <ul v-else>
      <li v-for="post in posts" :key="post.id">
        {{ post.title }}
      </li>
    </ul>
  </div>
</template>

<script setup>
const { data: posts, pending, error } = await useFetch('https://jsonplaceholder.typicode.com/posts')
</script>
```

让我们来解析这段代码:

1. `useFetch`: 这个函数用于从 API 获取数据。在这个例子中,我们使用了一个免费的在线 API 来获取博客文章数据。

2. `{ data: posts, pending, error }`: 这里我们使用了解构赋值。`useFetch` 返回多个值,我们只取我们需要的。
   - `data` 被重命名为 `posts`,包含了 API 返回的数据。
   - `pending` 表示数据是否正在加载。
   - `error` 如果发生错误,这里会包含错误信息。

3. `v-if`, `v-else-if`, `v-else`: 这些是 Vue 的条件渲染指令。我们用它们来显示不同的内容,基于数据的加载状态。

4. `v-for="post in posts"`: 这个指令用于循环渲染文章列表。

这个组件做了什么?当页面加载时,它会:

1. 显示"加载中..."消息。
2. 从 API 获取博客文章数据。
3. 如果成功,显示文章列表。
4. 如果失败,显示错误消息。

`useFetch` 的美妙之处在于它自动处理了很多复杂的情况:

- 它支持服务器端渲染(SSR),这意味着数据可以在服务器端获取,提高了首次加载速度。
- 它自动处理了加载状态和错误处理。
- 它默认情况下会缓存结果,避免不必要的重复请求。

小贴士:在实际应用中,你可能想要添加更多的交互性,比如点击文章标题跳转到文章详情页。你可以结合我们之前学的路由知识来实现这个功能!

异步数据获取是构建现代 web 应用的关键部分。通过 Nuxt 3 的 `useFetch`,它变得简单而强大。记住,实践是掌握这些概念的最好方法。尝试修改这个例子,也许获取不同类型的数据,或者添加一些额外的功能?

好的,让我们继续探索 Nuxt 3 的更多特性!

## 9. 自定义错误页面:优雅地处理错误

在任何应用中,错误都是不可避免的。但是,我们可以通过创建一个友好的错误页面来改善用户体验。Nuxt 3 使这变得非常简单。

在你的项目根目录下创建一个 `error.vue` 文件:

```vue
<template>
  <div class="error-page">
    <h1>哎呀!出错了</h1>
    <p>{{ error.message }}</p>
    <button @click="handleError">返回首页</button>
  </div>
</template>

<script setup>
const props = defineProps({
  error: Object
})

const handleError = () => {
  clearError({ redirect: '/' })
}
</script>

<style scoped>
.error-page {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100vh;
  text-align: center;
}
</style>
```

让我们来解析这个组件:

1. `defineProps`: 这个函数用来定义组件的 props。在这里,我们期望接收一个 `error` 对象。

2. `error.message`: 这会显示错误的具体信息。

3. `handleError`: 这个函数用 `clearError` 清除错误,并将用户重定向到首页。

4. `<style scoped>`: 这些样式让错误页面居中显示,看起来更美观。

现在,每当你的应用遇到错误时,Nuxt 都会自动显示这个错误页面。这不仅包括你的代码中的错误,还包括 404 "页面未找到" 这样的错误。

要测试这个错误页面,你可以故意在某个组件中引入一个错误。例如,修改 `pages/index.vue`:

```vue
<template>
  <div>
    <h1>首页</h1>
    {{ nonExistentVariable.property }}
  </div>
</template>
```

当你访问首页时,你会看到我们刚刚创建的错误页面,因为 `nonExistentVariable` 不存在。

自定义错误页面的好处是什么?

1. **提升用户体验**: 友好的错误信息可以减少用户的沮丧感。
2. **保持品牌一致性**: 你可以设计与你的应用风格一致的错误页面。
3. **提供解决方案**: 你可以在错误页面上提供有用的链接或操作,帮助用户解决问题。

记住,虽然我们希望用户永远不会看到错误页面,但当错误确实发生时,一个设计良好的错误页面可以大大改善用户体验。

## 10. 中间件：你的路由守卫英雄 

想象一下，中间件就像是站在你的页面门口的保安。它可以检查访客的"通行证"，决定是否让他们进入页面。超级酷，对吧？

来看看如何创建一个简单的身份验证中间件：

1. 创建文件 `middleware/auth.js`：

```javascript
export default defineNuxtRouteMiddleware((to, from) => {
  // 假设我们有一个 isLoggedIn 变量来检查用户是否登录
  const isLoggedIn = true; // 在实际应用中，这应该是一个真实的身份验证检查

  // 如果用户没有登录，而且想要访问的不是登录页面
  if (!isLoggedIn && to.path !== '/login') {
    // 那就把他们送到登录页面吧！
    return navigateTo('/login');
  }
});
```

这段代码是不是很有意思？它就像一个小小的门卫，检查用户是否有"VIP 通行证"（已登录）。如果没有，就会礼貌地把他们带到登录页面。

那么，如何使用这个中间件呢？有两种方式：

1. 全局应用：在 `nuxt.config.ts` 中添加：

```typescript
export default defineNuxtConfig({
  router: {
    middleware: ['auth']
  }
});
```

这样，每个页面都会经过这个中间件的检查。

2. 在特定页面使用：

```vue
<script setup>
definePageMeta({
  middleware: 'auth'
});
</script>
```

这样只有这个页面会使用这个中间件。很灵活，对吧？

## 11. 插件：给你的 Nuxt 应用超能力 🦸‍♀️

插件就像是给你的 Nuxt 应用装上了新的超能力。它们可以在应用启动时添加全局功能。让我们来创建一个简单的插件：

1. 创建文件 `plugins/myPlugin.js`：

```javascript
export default defineNuxtPlugin((nuxtApp) => {
  nuxtApp.provide('myPlugin', {
    sayHello: (name) => `你好，${name}！欢迎来到 Nuxt 3 的世界！`
  });
});
```

这个插件添加了一个全局可用的 `sayHello` 方法。很贴心，对吧？

现在，你可以在任何组件中使用这个插件：

```vue
<script setup>
const { $myPlugin } = useNuxtApp();
const message = $myPlugin.sayHello('努克斯特');
</script>

<template>
  <div>{{ message }}</div>
</template>
```

运行这段代码，你会看到一条友好的问候消息："你好，努克斯特！欢迎来到 Nuxt 3 的世界！"

是不是感觉给应用添加新功能变得超级简单？

## 12. SSR 和 SSG：让你的网站飞起来 🚀

Nuxt 3 支持服务器端渲染（SSR）和静态网站生成（SSG）。这听起来可能有点复杂，但别担心，我们来简单解释一下：

- SSR：每次有人访问你的网站时，服务器都会即时生成页面。这对于需要实时数据的网站很有用。
- SSG：预先生成所有页面，访客直接获取静态文件。这对于内容不经常变化的网站来说，加载速度超快！

要配置这些功能，只需在 `nuxt.config.ts` 中添加几行代码：

```typescript
export default defineNuxtConfig({
  ssr: true, // 启用服务器端渲染
  target: 'static' // 或 'server'，用于配置部署目标
});
```

如果你想生成一个完全静态的网站，只需运行：

```bash
npm run generate
```

这个命令会生成一个可以部署到任何静态托管服务的网站。超级方便，对吧？

记住，Rome wasn't built in a day（罗马不是一天建成的）。同样，掌握这些高级特性也需要时间和练习。从小处着手，慢慢来，你会发现 Nuxt 3 的强大之处！

如果你对任何部分感到困惑，或者想深入了解某个主题，别害羞，尽管问我！学习新东西最好的方式就是保持好奇心和提问。

接下来，我们将探讨 Nuxt 3 的多语言支持和部署策略。准备好了吗？让我们继续我们的 Nuxt 3 冒险之旅吧！🚀

## 13. 多国语言支持：让你的应用说多种语言 🌍

在这个全球化的时代，让你的应用支持多种语言是一个很酷的功能。虽然 Nuxt 3 没有内置的国际化解决方案，但我们可以使用 `@nuxtjs/i18n` 模块来实现这个功能。让我们一步步来：

1. 首先，安装 `@nuxtjs/i18n` 模块：

```bash
npm install @nuxtjs/i18n@next
```

2. 然后，在 `nuxt.config.ts` 中配置它：

```typescript
export default defineNuxtConfig({
  modules: ['@nuxtjs/i18n'],
  i18n: {
    locales: [
      { code: 'en', iso: 'en-US', file: 'en.json' },
      { code: 'zh', iso: 'zh-CN', file: 'zh.json' },
    ],
    defaultLocale: 'en',
    langDir: 'locales/',
    strategy: 'prefix_except_default',
  }
})
```

这段配置看起来可能有点复杂，让我解释一下：
- `locales`: 这里定义了我们支持的语言。在这个例子中，我们支持英语和中文。
- `defaultLocale`: 设置默认语言为英语。
- `langDir`: 指定存放语言文件的目录。
- `strategy`: 定义 URL 策略。这里使用 'prefix_except_default'，意味着除了默认语言外，其他语言的 URL 会带有语言前缀。比如，中文页面的 URL 可能是 `/zh/about`。

3. 创建语言文件：

在 `locales` 目录下创建 `en.json` 和 `zh.json`：

```json
// en.json
{
  "welcome": "Welcome to my awesome Nuxt 3 app!",
  "about": "About"
}

// zh.json
{
  "welcome": "欢迎来到我超酷的 Nuxt 3 应用！",
  "about": "关于"
}
```

4. 在组件中使用：

```vue
<template>
  <div>
    <h1>{{ $t('welcome') }}</h1>
    <nuxt-link :to="localePath('about')">{{ $t('about') }}</nuxt-link>
    <button @click="switchLanguage">Switch Language</button>
  </div>
</template>

<script setup>
const { t, locale } = useI18n();
const switchLanguage = () => {
  locale.value = locale.value === 'en' ? 'zh' : 'en';
};
</script>
```

这里的 `$t` 函数用于翻译文本，`localePath` 用于生成本地化的路由路径。`switchLanguage` 函数允许用户切换语言。

就是这样！现在你的应用可以说多种语言了。是不是感觉你的应用一下子变得更国际化了？

## 14. 部署：让你的 Nuxt 3 应用飞向云端 ☁️

创建了一个很棒的应用后，下一步就是让全世界都能看到它。这就是部署的过程。Nuxt 3 提供了多种部署选项，让我们一起看看：

### a. 静态托管：简单又快速

如果你的应用不需要服务器端渲染或 API 路由，静态托管是一个很好的选择。

1. 生成静态文件：

```bash
npm run generate
```

这个命令会在 `.output/public` 目录生成静态文件。

2. 部署：
   你可以将 `.output/public` 目录上传到任何静态文件托管服务，比如 Netlify, Vercel, 或者 GitHub Pages。就这么简单！

### b. Node. js 服务器：更多的控制权

如果你需要服务器端渲染或 API 路由，可以选择部署到 Node. js 环境。

1. 构建应用：

```bash
npm run build
```

2. 启动服务器：

```bash
node .output/server/index.mjs
```

你可以将这个部署到任何支持 Node. js 的平台，如 Heroku 或 DigitalOcean。

### c. 使用 PM 2：让你的应用永不停歇

对于生产环境，你可能想使用进程管理器如 PM 2：

```bash
npm install -g pm2
pm2 start .output/server/index.mjs
```

PM 2 可以帮助你管理和监控你的应用，确保它始终运行。

### d. Docker：打包整个环境

Docker 允许你将应用和其整个运行环境打包在一起。创建一个 `Dockerfile`：

```dockerfile
FROM node:16

WORKDIR /app

COPY . .

RUN npm install
RUN npm run build

EXPOSE 3000

CMD ["node", ".output/server/index.mjs"]
```

然后构建和运行 Docker 镜像：

```bash
docker build -t my-nuxt-app .
docker run -p 3000:3000 my-nuxt-app
```

这样，无论在哪里运行，你的应用都会在相同的环境中运行。

### e. Serverless 部署：按需缩放

Nuxt 3 也支持 Serverless 部署。例如，部署到 Vercel 非常简单：

1. 安装 Vercel CLI: `npm i -g vercel`
2. 运行 `vercel` 命令并按照提示操作

就是这么简单！

### 环境变量：保护你的秘密 🤫

在部署时，正确管理环境变量很重要。在 Nuxt 3 中，你可以使用 `.env` 文件或运行时配置。

创建 `.env` 文件：

```
API_BASE_URL=https://api.example.com
```

在 `nuxt.config.ts` 中：

```typescript
export default defineNuxtConfig({
  runtimeConfig: {
    apiSecret: '', // 只在服务器端可用
    public: {
      apiBase: '' // 客户端也可用
    }
  }
})
```

使用环境变量：

```javascript
const config = useRuntimeConfig()
console.log(config.apiSecret) // 仅在服务器端
console.log(config.public.apiBase) // 客户端和服务器端
```

这样，你就可以安全地管理你的 API 密钥和其他敏感信息了。

记住，选择哪种部署方式取决于你的应用需求和你的团队技能。不要害怕尝试不同的方法，找到最适合你的那个！

好了，我们的 Nuxt 3 高级特性之旅到此结束。你现在已经掌握了中间件、插件、多语言支持和部署的基础知识。这些工具会让你的 Nuxt 3 开发之路变得更加顺畅。

记住，学习是一个持续的过程。不要期望一次就掌握所有内容。慢慢来，一步一步地实践这些概念。很快，你就会成为 Nuxt 3 的高手！

如果你有任何问题，或者想更深入地了解某个主题，随时问我。学习新技术的过程应该是有趣的，所以享受你的 Nuxt 3 之旅吧！🚀🎉
## 相关链接 ：

1. [https://nuxt.com/docs/getting-started/introduction](https://nuxt.com/docs/getting-started/introduction)
