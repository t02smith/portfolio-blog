---
title: Why You Should Use Nuxt Content
description: How using Nuxt Content can allow you to easily add and edit the content shown on your page using Markdown.
draft: false
recommended: true
badges: ["nuxt", "vue", "javascript"]
authors: [{ name: "Tom Smith", role: "Nuxt Fanboy", link: "/" }]
---

Let's start with some very simple questions. Do you:

- want to write your content in a user-friendly language like Markdown?
- want to have that content rendered on your website by dropping a file into a folder?
- want to allow your users to easily search through your content?

Chances are you answered yes to at least one of those questions. [Nuxt Content](https://content.nuxtjs.org/) is a library that will take any `.md`, `.yml`, `.json`, and `.csv` files kept inside the `content/` folder of your Nuxt application and render them as HTML at your desired route.

> 💡 This website uses Nuxt Content to power this page and the [projects](/projects) page

## 🚦 Where to Start

It is always best to check out the developer's [Get Started](https://content.nuxtjs.org/get-started) page for the most up-to-date instructions on how to get started.

Regardless, to add Nuxt Content to your site you:

1. Create a new Nuxt project

```bash{}[Run in your terminal]
npx nuxi@latest init my-nuxt-content-app
```

The open up your new project in the IDE/text editor of your choice.

2. Install your dependencies

```bash{}[Run in your terminal]
# for yarn users
yarn add --dev @nuxt/content
yarn install

# for npm users
npm i -D @nuxt/content
npm i
```

3. Update your `nuxt.config.ts` file

```typescript{}[@/nuxt.config.ts]
export default defineNuxtConfig({
    modules: [
      '@nuxt/content'
    ],
    content: {
      // https://content.nuxtjs.org/api/configuration
    }
})
```

4. Set up page routing

First we need to change `app.vue` to use Nuxt's built-in routing component so we can view our content.

```vue{}[@/app.vue]
<template>
  <NuxtPage />
</template>
```

5. Create your home page

We're going to be adding a home page which we'll use a bit later on in [🔎 Searching Content](#searching-content) for finding what blogs we have available on our website. For now just add some basic HTML to it.

```vue{}[@/pages/index.vue]
<template>
  <h1>Hello World</h1>
  <p>Welcome to my blog!</p>
</template>
```

6. Create a slug file to render your content

```vue{}[@/pages/[...slug].vue]
<template>
  <ContentDoc />
</template>
```

7. Add some content then find it

To give you an example create the following file:

```markdown{}[@/content/hello+world.md]
# Hello world

I can follow instructions!!
```

Then run your application with `yarn dev` or `npm run dev` and head to [http://localhost:3000/hello+world] and you should see:

![first page](/img/projects/nuxt-content/first-page.png)

## 🔎 Searching Content

One of the most powerful things Nuxt Content lets us do is to search our content using nothing but metadata that we write into our files, whilst also abstracting over any nasty HTTP requests you would oridnarily need to use.

### 💿 Adding Metadata

You can add whatever metadata you want to your files but some important fields are:

- **title** The title of the page that is injected into the page metadata.
- **description** A quick memo about what this content actually is.
- **draft** Whether or not this content is ready to show the world. If it isn't then the content won't be hosted.

In our markdown file from before it will look like:

```markdown{}[@/content/hello+world.md]
---
title: Hello World (the blog)
description: This is my first piece of content deployed on my website
recommended: true
---
# Hello world

I can follow instructions!!
```

### ✈️ Writing Queries

Nuxt Content offers the `queryContent` function which will allow us to query the metadata stored in our Markdown files. Take the following query:

```javascript{}[]
const posts = await queryContent("/")
    .where({
      # we will only fetch markdown files which have the metatag "recommended" set to "true"
      recommended: { $eq: true },
    })
    .only([
      # we want to fetch the following fields
      "_path", "title", "description"
    ])
    .limit(4) # limit to four content results
    .find();
```

If this were an SQL query it would look something like:

```sql{}[]
SELECT _path, title, description
FROM content_files
WHERE recommended = true
LIMIT 4;
```

### 🧱 Making our search

Head back to `@/pages/index.vue` and it's time we added our search functionality.

1. Add our state variables

```vue{}[@/pages/index.vue]
<template>
 ...
</template>
<script setup>

// The value entered by the user into the search bar
const searchValue = ref("");

// The output of our query
const searchResult = ref(null);

</script>
```

2. Add a query function

```vue{}[@/pages/index.vue]
<template>
 ...
</template>
<script setup>

// ...

// run this query and save the result in searchResult
async function searchForContent() {
  searchResult.value = await queryContent("/")
    .where({
      $or: [
        { title: { $icontains: searchValue.value } },
        { description: { $icontains: searchValue.value } },
      ]
    })
    .only(["_path", "title", "description"])
    .find();
}

</script>
```

3. Call our search function whenever `searchValue` changes

```vue{}[@/pages/index.vue]
<template>
 ...
</template>
<script setup>

// ...

// call searchForContent whenever searchValue changes
watch(searchValue, searchForContent)

</script>
```

4. Add some HTML to show to our users

```vue{}[@/pages/index.vue]
<template>
  <h1>Welcome to my blog</h1>
  <input type="text" v-model="searchValue" />
  <ul>
    <NuxtLink :to="post._path" v-if="searchResult" v-for="post in searchResult">
      <p>
        <strong>{{ post.title }}</strong>
      </p>
      <p>{{ post.description }}</p>
    </NuxtLink>
  </ul>
</template>
<script setup>
// ...
</script>
```

Overall your `@/pages/index.vue` file should look like:

```vue{}[@/pages/index.vue]
<template>
  <h1>Welcome to my blog</h1>
  <input type="text" v-model="searchValue" />
  <ul>
    <NuxtLink :to="post._path" v-if="searchResult" v-for="post in searchResult">
      <p>
        <strong>{{ post.title }}</strong>
      </p>
      <p>{{ post.description }}</p>
    </NuxtLink>
  </ul>
</template>
<script setup>
// The value entered by the user into the search bar
const searchValue = ref("");

// The output of our query
const searchResult = ref(null);

// run this query and save the result in searchResult
async function searchForContent() {
  searchResult.value = await queryContent("/")
    .where({
      $or: [
        { title: { $icontains: searchValue.value } },
        { description: { $icontains: searchValue.value } },
      ],
    })
    .only(["_path", "title", "description"])
    .find();
}

// call searchForContent whenever searchValue changes
watch(searchValue, searchForContent);
</script>
```

Then head on over to [http://localhost:3000](http://localhost:3000) and you now have a fully functioning search bar. Type `hello` in and our example blog post should come up.

![search bar](/img/projects/nuxt-content/search.png)

## 🎁 Wrapping Up

Congratulations you have just made a basic blog. Granted it is very ugly but it is relatively easy to style it (you can view the stylings I use for this site at [this link](https://github.com/t02smith/portfolio/blob/main/assets/style/components/markdown.scss)).

Some extensions for you to help improve:

1. Style the website (you can use CSS, SCSS, TailwindCSS, ...)
2. Upgrade your search by adding in sort and filter options
3. Upgrade your home page with recommended blog posts
