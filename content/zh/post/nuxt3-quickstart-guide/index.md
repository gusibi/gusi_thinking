---
date: '2024-10-01T12:18:08+08:00'
lastmod: '2024-10-01T12:18:08+08:00'
draft: false
title: Nuxt 3 quickstart guide
slug: nuxt3-quickstart-guide
description: Certainly! I'd be happy to translate this Nuxt 3 quickstart guide into
  English for you.
tags:
- 技术
categories:
- 技术
comment: false
toc: false
---

> Nuxt 3 quickstart guide


Certainly! I'd be happy to translate this Nuxt 3 quickstart guide into English for you. Here's the translated version:

# Nuxt 3 Beginner-Friendly Guide: Building Your First Application from Scratch

Hey there! Welcome to the world of Nuxt 3! 👋 If you're new to web development or just getting started with Nuxt, don't worry. This guide will walk you through the basics of Nuxt 3 step by step. We'll explain each concept using simple language and examples, making it easy for you to get started with this powerful Vue. js framework. Ready to begin your Nuxt 3 journey? Let's dive in!

## 1. Creating Your First Nuxt 3 Application

First, let's start our adventure by creating a brand new Nuxt 3 project! Open your favorite terminal and follow along with these commands:

```bash
npx nuxi init my-awesome-nuxt-app
cd my-awesome-nuxt-app
npm install
```

Wow! Just like that, you've created your first Nuxt 3 project. But wait, what did these commands actually do? Let me explain:

1. `npx nuxi init my-awesome-nuxt-app`: This command tells `npx` (an npm package runner) to use `nuxi` (Nuxt 3's command-line tool) to initialize a new project. `my-awesome-nuxt-app` is the name of your project, but of course, you can change it to anything you like!

2. `cd my-awesome-nuxt-app`: This command lets us enter the newly created project folder. `cd` stands for "change directory".

3. `npm install`: This command installs all the dependencies needed for your project. Think of it as gathering all the necessary parts to assemble a model.

Now, let's take a look at our project structure:

```
my-awesome-nuxt-app/
  ├── app.vue
  ├── nuxt.config.ts
  └── package.json
```

What are these files for? Let me introduce you to these new friends:

- `app.vue`: This is the main component of your application, where all pages will be rendered. Think of it as the "shell" of your app.
- `nuxt.config.ts`: This is Nuxt's configuration file where you can customize Nuxt's behavior. Think of it as your app's "control panel".
- `package.json`: This file contains metadata about your project and its dependencies. It's like your project's "ID card".

Let's take a look at the contents of `app.vue`:

```vue
<template>
  <div>
    <h1>Welcome to Nuxt 3!</h1>
    <NuxtPage />
  </div>
</template>
```

This code does two things:
1. Displays a welcome heading.
2. The `<NuxtPage />` component automatically loads and displays the content of the corresponding page based on the current URL. It's like a magical placeholder that Nuxt will automatically fill in.

Congratulations! You've successfully created your first Nuxt 3 application. Next, let's see how to run it and debug it.

## 2. Debug Mode: Your Development Sidekick

During development, being able to quickly see changes and catch errors is crucial. Nuxt 3 enables debugging features by default in development mode. Let's experience it:

```bash
npm run dev
```

After entering this command, you'll see some information in the terminal, and finally, it will tell you which address your application is running on, usually `http://localhost:3000`.

Now, open your browser, enter this address, and you'll see your Nuxt 3 application! 🎉

But wait, what benefits does debug mode actually bring us?

1. **Hot Reloading**: When you modify your code, the browser will automatically refresh to show the latest changes. No more manual page refreshing!

2. **Vue Devtools**: If you've installed the Vue Devtools browser extension, you can use it to inspect your component structure, state, and more.

3. **Detailed Error Messages**: If there are errors in your code, Nuxt will display detailed error messages in both the browser console and terminal, helping you quickly locate issues.

Imagine debug mode as your personal assistant, constantly watching your code and immediately reporting any problems. Doesn't that make development feel much easier?

## 3. HTML Escaping: Safety First!

In web development, security is a very important topic. One common security issue is cross-site scripting (XSS) attacks. Don't worry, Nuxt 3 has already considered this for you!

Nuxt 3 uses Vue 3, which automatically escapes HTML in interpolations. This might sound a bit complex, so let me explain with an example:

```vue
<template>
  <div>{{ userInput }}</div>
</template>

<script setup>
const userInput = '<script>alert("This is an XSS attack!")</script>';
</script>
```

In this example:
- `<script setup>`: This is Vue 3's Composition API syntax, used to define component logic.
- `const userInput`: We defined a variable containing potentially dangerous HTML.

If Nuxt didn't perform HTML escaping, this code might execute a popup. But don't worry, Nuxt will automatically convert these special characters into safe forms.

When this code is rendered, the `<script>` tag will be escaped and displayed as plain text, rather than being executed as JavaScript. Users will see text like `<script>alert("This is an XSS attack!")</script>` on the page, instead of a popup alert.

It's like Nuxt has installed a security filter for your application, ensuring that user input doesn't accidentally become executable code.

Remember, although Nuxt provides this layer of protection, you should still remain vigilant when handling user input. Security is a complex topic, and this is just one aspect of it.

## 4. Routing: Making Your Application Accessible

In Nuxt 3, routing is like a map for your application, telling Nuxt what content to display when users visit different URLs. The best part is, Nuxt uses a "file-based routing system", which means your file structure determines your routes! Sounds a bit magical, right? Let's explore together.

First, create a `pages` folder, then add some `.vue` files inside:

```
pages/
  ├── index.vue
  └── about.vue
```

Just like that, you've created two routes:
- `/`: corresponds to `pages/index.vue`
- `/about`: corresponds to `pages/about.vue`

Let's see what the content of `index.vue` might look like:

```vue
<template>
  <div>
    <h1>Welcome to my Nuxt 3 app!</h1>
    <p>This is the home page.</p>
  </div>
</template>
```

And `about.vue` might look like this:

```vue
<template>
  <div>
    <h1>About Us</h1>
    <p>We're an awesome team using Nuxt 3!</p>
  </div>
</template>
```

But wait, what if we want dynamic routes? For example, a user profile page with the user ID in the URL? Nuxt has thought of this too!

Create a file `pages/users/[id].vue`:

```vue
<template>
  <div>
    <h1>User Profile</h1>
    <p>User ID: {{ $route.params.id }}</p>
  </div>
</template>

<script setup>
const route = useRoute();
console.log(route.params.id);
</script>
```

What's happening here?
- `[id].vue`: The square brackets indicate this is a dynamic parameter.
- `{{ $route.params.id }}`: In the template, we can access route parameters via `$route.params`.
- `useRoute()`: This is a composable function used to access the current route in `<script setup>`.

Now, when users visit `/users/1`, `/users/2`, etc., this page will be displayed, and the `id` parameter will be passed to the component.

Doesn't routing feel super simple? You just need to create files, and Nuxt will automatically handle all the routing logic for you. This is the magic of Nuxt!

## 5. Static Files: Handling Images and Other Assets

In web development, handling static files like images, fonts, or other asset files is very common. Nuxt 3 provides us with a simple way to handle these files.

First, create a `public` folder in your project root directory. This folder is special because Nuxt will directly map its contents to your website's root directory.

Let's try it out:

1. Create an `images` folder inside the `public` folder.
2. Put an image named `logo.png` into the `images` folder.

Now your file structure might look like this:

```
my-awesome-nuxt-app/
  ├── public/
  │   └── images/
  │       └── logo.png
  ├── app.vue
  ├── nuxt.config.ts
  └── package.json
```

Great! Now you can use this image in your Vue components. Let's modify `app.vue`:

```vue
<template>
  <div>
    <img src="/images/logo.png" alt="My Logo">
    <h1>Welcome to my Nuxt 3 app!</h1>
    <NuxtPage />
  </div>
</template>
```

Notice the `src` attribute:
- `/images/logo.png`: This path is relative to the server root path, no need to include `public`.

When you run your application, you should see the logo image displayed on the page. Isn't it simple?

This method not only works for images but also for any static files you want to serve directly to the browser, such as `robots.txt`, `favicon.ico`, etc.

This approach has several benefits:

1. Simple and intuitive: You don't need to remember complex paths, just remember the file's location in the `public` directory.
2. Performance optimization: Nuxt will automatically handle these static files, ensuring they can be loaded efficiently.
3. Deployment-friendly: When you deploy your application, these files will be automatically processed, you don't need extra configuration.

Remember, the `public` directory is not just for images. You can place any type of static file here, such as font files, robots. txt, favicon. ico, etc. Nuxt will ensure that all these files can be accessed correctly.

## 6. Rendering Templates: The Magic of Components

In modern frontend development, componentization is a very important concept. It allows us to break down the interface into small, reusable parts, just like Lego blocks. Nuxt 3 fully embraces this idea, let's see how to create and use components.

First, let's create a simple user card component. Create a `components` folder in the project root directory, then create a `UserCard.vue` file inside:

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

Let me explain the different parts of this component:

1. `<template>`: This defines the HTML structure of the component. We used some dynamic bindings (`:src` and `:alt`) and interpolation (`{{ }}`) to display user information.

2. `<script setup>`: This is Vue 3's Composition API syntax. `defineProps(['user'])` declares that this component accepts a `user` prop.

3. `<style scoped>`: This defines the styles for the component. The `scoped` attribute ensures these styles only apply to this component and don't affect other places.

Now, let's use this component in a page. Modify `pages/index.vue`:

```vue
<template>
  <div>
    <h1>User List</h1>
    <UserCard v-for="user in users" :key="user.id" :user="user" />
  </div>
</template>

<script setup>
const users = ref([
  { id: 1, name: 'John Doe', email: 'john@example.com', avatar: 'https://placekitten.com/100/100' },
  { id: 2, name: 'Jane Smith', email: 'jane@example.com', avatar: 'https://placekitten.com/101/101' },
]);
</script>
```

What's happening here?

1. We imported and used the `UserCard` component. Note that we don't need to explicitly import it - Nuxt will automatically import components from the `components` folder.

2. We use the `v-for` directive to loop through and render the user list. Each `UserCard` receives a `user` object as a prop.

3. `:key="user.id"` provides a unique key for each list item, which helps Vue optimize rendering.

4. `ref([...])` creates a reactive array containing user data. In a real application, this data might come from an API call.

Now, when you run your application, you should see a nice list of users, each displayed in a card.

What are the benefits of componentization?

1. **Reusability**: You can reuse the `UserCard` component anywhere in your application.
2. **Maintainability**: If you need to modify the appearance of the user card, you only need to change it in one place.
3. **Testability**: You can test the `UserCard` component in isolation, ensuring it works correctly in various scenarios.

This is the magic of componentization! It makes your code cleaner, more efficient, and easier to manage. As your application grows, you'll find componentization becomes a powerful tool for building complex interfaces.

## 7. State Management: Keeping Your Data Tidy

As your application becomes more complex, managing state (i.e., your application data) becomes increasingly important. Nuxt 3 provides a simple yet powerful way to manage state, called "Nuxt State".

Let's look at how to use it through a simple counter example.

First, create a new file `composables/useState.js`:

```javascript
export const useCounter = () => useState('counter', () => 0);
```

What does this code do?

- `useState`: This is a function provided by Nuxt 3 for creating a reactive state.
- `'counter'`: This is a unique identifier for the state. If multiple components use the same identifier, they will share the same state.
- `() => 0`: This is the initial value of the state. Here, our counter starts at 0.

Now, let's use this state in a component. Create a new component `components/Counter.vue`:

```vue
<template>
  <div>
    <p>Count: {{ count }}</p>
    <button @click="increment">Increase</button>
    <button @click="decrement">Decrease</button>
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

Let's break down this component:

1. We use `useCounter()` to get the counter state.
2. `count.value` is used to access and modify the state value. Note the use of `.value`, this is because `count` is a reactive reference.
3. `increment` and `decrement` functions are used to increase and decrease the count respectively.

Now, you can use this `Counter` component in any page or component. For example, modify `pages/index.vue`:

```vue
<template>
  <div>
    <h1>Nuxt 3 Counter Example</h1>
    <Counter />
    <Counter /> <!-- Note that we used the Counter component twice -->
  </div>
</template>
```

When you run this application, you'll see two counters. Interestingly, they share the same state! If you increase one counter, the other will change too. This is the power of shared state.

What are the advantages of this approach?

1. **Simple**: You don't need to set up complex state management libraries.
2. **Flexible**: You can easily share state between different components.
3. **Reactive**: Changes in state are automatically reflected everywhere it's used.

State management might seem a bit abstract, but it's key to building large, complex applications. As you continue your Nuxt 3 journey, you'll find more and more scenarios where shared state is useful.

Remember, while this approach is sufficient for simple to moderately complex applications, Nuxt 3 is also fully compatible with state management libraries like Pinia for more complex state management needs.

## 8. Asynchronous Data Fetching: Talking to the Backend

In real-world applications, we often need to fetch data from a server. Nuxt 3 provides two powerful composables for handling asynchronous data fetching: `useFetch` and `useAsyncData`. Today, we'll mainly look at `useFetch` as it's simpler and more straightforward.

Let's create a simple blog post list page to demonstrate how to use `useFetch`.

Create a new file `pages/posts.vue`:

```vue
<template>
  <div>
    <h1>Blog Posts</h1>
    <div v-if="pending">Loading...</div>
    <div v-else-if="error">Error: {{ error.message }}</div>
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

Let's break down this code:

1. `useFetch`: This function is used to fetch data from an API. In this example, we're using a free online API to get blog post data.

2. `{ data: posts, pending, error }`: Here we're using destructuring assignment. `useFetch` returns multiple values, and we're only taking what we need.
   - `data` is renamed to `posts`, containing the data returned by the API.
   - `pending` indicates whether the data is still loading.
   - `error` will contain error information if something goes wrong.

3. `v-if`, `v-else-if`, `v-else`: These are Vue's conditional rendering directives. We use them to display different content based on the loading state of the data.

4. `v-for="post in posts"`: This directive is used to loop through and render the list of posts.

What does this component do? When the page loads, it will:

1. Display a "Loading..." message.
2. Fetch blog post data from the API.
3. If successful, display the list of posts.
4. If there's an error, display the error message.

The beauty of `useFetch` is that it automatically handles many complex scenarios:

- It supports server-side rendering (SSR), meaning data can be fetched on the server side, improving initial load times.
- It automatically handles loading states and error handling.
- It caches results by default, avoiding unnecessary repeated requests.

Pro tip: In a real application, you might want to add more interactivity, like clicking on a post title to navigate to a post details page. You can combine this with the routing knowledge we learned earlier to achieve this!

Asynchronous data fetching is a crucial part of building modern web applications. With Nuxt 3's `useFetch`, it becomes simple yet powerful. Remember, practice is the best way to master these concepts. Try modifying this example, perhaps fetching different types of data, or adding some extra functionality?

Alright, let's continue exploring more features of Nuxt 3!

## 9. Custom Error Page: Handling Errors Gracefully

Errors are inevitable in any application. However, we can improve the user experience by creating a friendly error page. Nuxt 3 makes this very simple.

Create an `error.vue` file in your project root directory:

```vue
<template>
  <div class="error-page">
    <h1>Oops! Something went wrong</h1>
    <p>{{ error.message }}</p>
    <button @click="handleError">Return to Home</button>
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

Let's break down this component:

1. `defineProps`: This function is used to define the component's props. Here, we expect to receive an `error` object.

2. `error.message`: This will display the specific error message.

3. `handleError`: This function uses `clearError` to clear the error and redirect the user to the home page.

4. `<style scoped>`: These styles center the error page content, making it look more appealing.

Now, whenever your application encounters an error, Nuxt will automatically display this error page. This includes not only errors in your code but also 404 "Page Not Found" errors.

To test this error page, you can intentionally introduce an error in one of your components. For example, modify `pages/index.vue`:

```vue
<template>
  <div>
    <h1>Home Page</h1>
    {{ nonExistentVariable.property }}
  </div>
</template>
```

When you visit the home page, you'll see our custom error page because `nonExistentVariable` doesn't exist.

What are the benefits of a custom error page?

1. **Improved User Experience**: Friendly error messages can reduce user frustration.
2. **Brand Consistency**: You can design an error page that matches your application's style.
3. **Provide Solutions**: You can include helpful links or actions on the error page to help users resolve the issue.

Remember, while we hope users never see the error page, a well-designed error page can greatly improve the user experience when errors do occur.

## 10. Middleware: Your Route Guard Hero

Imagine middleware as a security guard standing at the door of your pages. It can check the visitor's "pass" and decide whether to let them enter the page. Super cool, right?

Let's see how to create a simple authentication middleware:

1. Create a file `middleware/auth.js`:

```javascript
export default defineNuxtRouteMiddleware((to, from) => {
  // Assume we have an isLoggedIn variable to check if the user is logged in
  const isLoggedIn = true; // In a real app, this should be an actual authentication check

  // If the user is not logged in and trying to access a page other than the login page
  if (!isLoggedIn && to.path !== '/login') {
    // Let's send them to the login page!
    return navigateTo('/login');
  }
});
```

Isn't this code interesting? It's like a little doorkeeper, checking if the user has a "VIP pass" (is logged in). If not, it politely escorts them to the login page.

So, how do we use this middleware? There are two ways:

1. Apply globally: Add to `nuxt.config.ts`:

```typescript
export default defineNuxtConfig({
  router: {
    middleware: ['auth']
  }
});
```

This way, every page will go through this middleware check.

2. Use on specific pages:

```vue
<script setup>
definePageMeta({
  middleware: 'auth'
});
</script>
```

This way, only this page will use the middleware. Pretty flexible, right?

## 11. Plugins: Give Your Nuxt App Superpowers 🦸‍♀️

Plugins are like giving your Nuxt application new superpowers. They can add global functionality when your app starts up. Let's create a simple plugin:

1. Create a file `plugins/myPlugin.js`:

```javascript
export default defineNuxtPlugin((nuxtApp) => {
  nuxtApp.provide('myPlugin', {
    sayHello: (name) => `Hello, ${name}! Welcome to the world of Nuxt 3!`
  });
});
```

This plugin adds a globally available `sayHello` method. Pretty neat, right?

Now, you can use this plugin in any component:

```vue
<script setup>
const { $myPlugin } = useNuxtApp();
const message = $myPlugin.sayHello('Nuxter');
</script>

<template>
  <div>{{ message }}</div>
</template>
```

Run this code, and you'll see a friendly greeting message: "Hello, Nuxter! Welcome to the world of Nuxt 3!"

Doesn't it feel super easy to add new functionality to your app?

## 12. SSR and SSG: Make Your Website Fly 🚀

Nuxt 3 supports both Server-Side Rendering (SSR) and Static Site Generation (SSG). This might sound complicated, but don't worry, let's break it down:

- SSR: The server generates the page every time someone visits your website. This is useful for websites that need real-time data.
- SSG: All pages are pre-generated, and visitors get static files. This is super fast for websites where content doesn't change often!

To configure these features, just add a few lines to your `nuxt.config.ts`:

```typescript
export default defineNuxtConfig({
  ssr: true, // Enable server-side rendering
  target: 'static' // or 'server', to configure deployment target
});
```

If you want to generate a fully static website, just run:

```bash
npm run generate
```

This command will generate a website that you can deploy to any static hosting service. Super convenient, right?

Remember, Rome wasn't built in a day. Similarly, mastering these advanced features takes time and practice. Start small, take it slow, and you'll discover the power of Nuxt 3!

If you're confused about any part or want to dive deeper into a topic, don't be shy, just ask me! The best way to learn new things is to stay curious and ask questions.

Next, we'll explore Nuxt 3's multilingual support and deployment strategies. Are you ready? Let's continue our Nuxt 3 adventure! 🚀

## 13. Multilingual Support: Make Your App Speak Multiple Languages 🌍

In this globalized era, having your app support multiple languages is a cool feature. Although Nuxt 3 doesn't have a built-in internationalization solution, we can use the `@nuxtjs/i18n` module to achieve this functionality. Let's go through it step by step:

1. First, install the `@nuxtjs/i18n` module:

```bash
npm install @nuxtjs/i18n@next
```

2. Then, configure it in `nuxt.config.ts`:

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

This configuration might look a bit complex, let me explain:
- `locales`: This defines the languages we support. In this example, we support English and Chinese.
- `defaultLocale`: Sets the default language to English.
- `langDir`: Specifies the directory where language files are stored.
- `strategy`: Defines the URL strategy. Here, we use 'prefix_except_default', which means URLs for languages other than the default will have a language prefix. For example, a Chinese page URL might be `/zh/about`.

3. Create language files:

Create `en.json` and `zh.json` in the `locales` directory:

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

4. Use in components:

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

Here, the `$t` function is used for text translation, `localePath` is used to generate localized route paths. The `switchLanguage` function allows users to switch languages.

That's it! Now your app can speak multiple languages. Doesn't it feel like your app just became more international?

## 14. Deployment: Let Your Nuxt 3 App Soar to the Cloud ☁️

After creating an awesome app, the next step is to let the whole world see it. This is the deployment process. Nuxt 3 offers multiple deployment options, let's take a look:

### a. Static Hosting: Simple and Fast

If your app doesn't need server-side rendering or API routes, static hosting is a great choice.

1. Generate static files:

```bash
npm run generate
```

This command will generate static files in the `.output/public` directory.

2. Deploy:
   You can upload the `.output/public` directory to any static file hosting service, such as Netlify, Vercel, or GitHub Pages. It's that simple!

### b. Node. js Server: More Control

If you need server-side rendering or API routes, you can choose to deploy to a Node. js environment.

1. Build the app:

```bash
npm run build
```

2. Start the server:

```bash
node .output/server/index.mjs
```

You can deploy this to any platform that supports Node. js, such as Heroku or DigitalOcean.

### c. Using PM 2: Keep Your App Running Non-Stop

For production environments, you might want to use a process manager like PM 2:

````bash
npm install -g pm2
pm2 start .output/server/index.mjs
````

PM 2 can help you manage and monitor your application, ensuring it's always running.

### d. Docker: Package the Entire Environment

Docker allows you to package your application along with its entire runtime environment. Create a `Dockerfile`:

````dockerfile
FROM node:16

WORKDIR /app

COPY . .

RUN npm install
RUN npm run build

EXPOSE 3000

CMD ["node", ".output/server/index.mjs"]
````

Then build and run the Docker image:

````bash
docker build -t my-nuxt-app .
docker run -p 3000:3000 my-nuxt-app
````

This way, your application will run in the same environment no matter where it's deployed.

### e. Serverless Deployment: Scale on Demand

Nuxt 3 also supports serverless deployment. For example, deploying to Vercel is very simple:

1. Install Vercel CLI: `npm i -g vercel`
2. Run the `vercel` command and follow the prompts

It's that easy!

#### Environment Variables: Protect Your Secrets 🤫

When deploying, it's important to manage environment variables correctly. In Nuxt 3, you can use `.env` files or runtime configuration.

Create a `.env` file:

````
API_BASE_URL=https://api.example.com
````

In `nuxt.config.ts`:

````typescript
export default defineNuxtConfig({
  runtimeConfig: {
    apiSecret: '', // Only available on the server side
    public: {
      apiBase: '' // Available on both client and server side
    }
  }
})
````

Using environment variables:

````javascript
const config = useRuntimeConfig()
console.log(config.apiSecret) // Only on server side
console.log(config.public.apiBase) // On client and server side
````

This way, you can safely manage your API keys and other sensitive information.

Remember, choosing which deployment method to use depends on your application's needs and your team's skills. Don't be afraid to try different methods to find the one that suits you best!

Well, our journey through Nuxt 3's advanced features ends here. You now have a grasp of middleware, plugins, multilingual support, and deployment basics. These tools will make your Nuxt 3 development journey smoother.

Remember, learning is an ongoing process. Don't expect to master everything at once. Take it slow, practice these concepts step by step. Soon, you'll become a Nuxt 3 expert!

If you have any questions, or want to dive deeper into a topic, feel free to ask me. Learning new technology should be fun, so enjoy your Nuxt 3 journey! 🚀🎉


## 相关链接 ：

1. []()
