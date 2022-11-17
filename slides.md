---
theme: default
lineNumbers: false
title: End-to-End Type Safety with Nuxt and tRPC
layout: image-right
image: /bg-1.png
---


<div class="absolute top-10">
  <h1>End-to-End Type Safety <br /> with Nuxt and tRPC</h1>
</div>

<div class="flex justify-start items-center absolute bottom-10">
  <img class="rounded-full h-24 w-24" src="/avatar.png" alt="Cody Bontecou avatar">
  <div class="flex flex-col">
    <span class="ml-6 font-semibold text-lg">Cody Bontecou</span>
    <span class="ml-6 font-medium">Senior Fullstack Engineer at DEPT®</span>
  </div>
</div>

<!--
* I've been writing production-grade Vue code for four years.
* I chose Nuxt when possible.
* Majority of backend experience is in Python. Around six years.
* Quickly falling in love with Typescript.
* Tend to write everything in Typescript.
-->

---
layout: center
---


# What is type safety?

<p class="py-8">
Type safety is when the compiler validates types while compiling, throwing an error if you try to assign the wrong type to a variable.</p>

```ts {1-4|6-8|all}
interface User {
  firstName: string
  lastName: string
}

const fullName: string = (user: User) => {
  return `${user.firstName} ${user.lastName}`
}
```

<!--
* The reason why we use Typescript.
* Use the drawing tool to underline the error in multiplyByTen
-->

---
layout: two-cols
title: So... what is end-to-end type safety?
---

# So... what is end-to-end type safety?

<p class="py-8">
End-to-end type safety is having a single source of truth for your types across all layers of your application.
</p>

<!-- TODO: Get this image back -->
<img v-click class="w-screen" src="/db-server-client.png" alt="">

::right::

<h2 class="pb-2" v-click>How it's typically solved</h2>

<v-click>
```sql
// prisma.schema
model User {
  id        String @id @default(cuid())
  firstName String
  lastName  String
}
```
</v-click>

<!-- TODO: Write proper getUser endpoint -->
<!-- TODO: Define User interface in server -->
<v-click>
```ts
// server/api
module.exports = async function (fastify, opts) {
  fastify.get('/', async function (request, reply) {
    return 'this main'
  })
}
```
</v-click> 

<!-- TODO: Write proper fetch -->
<!-- TODO: Define User interface in frontend -->
<v-click>
```js
// pages/index.vue
<script setup lang="ts">
  const user: User = await fetch('/getUser')
  const fullName = `${user.firstName} ${user.lastName}`
</script>

<template>
{{ fullName }}
</template>
```
</v-click>

<!--
Each layer must maintain their type and communicate it to other developers.

Layers of a fullstack application may include:
* Database
** Prisma
** Relational DB (PostgreSQL, MySQL, SQLite)
* Server
** NestJS
** Nuxt
** tRPC
* Client
** Vue
** Nuxt (The reason we're all here)
-->

---
layout: two-cols
---

<h2 class="ml-2 mb-[2.75rem]">The problem...</h2>

<v-click>
```sql
// prisma.schema
model User {
  id        String @id @default(cuid())
  firstName String
  lastName  String
}
```
</v-click>

<v-click>
```sql
// prisma.schema
model User {
  id        String @id @default(cuid())
  fullName  String
}
```
</v-click>


::right::

<h2 v-click class="ml-2">What if a type is changed upstream?</h2>

<div v-click class="flex flex-col ml-2">
  <!-- TODO: Write proper getUser endpoint -->

  ```ts
  // server/api
  module.exports = async function (fastify, opts) {
    fastify.get('/', async function (request, reply) {
      return 'this main'
    })
  }
  ```

  <!-- TODO: Write proper fetch -->
  ```js
  // pages/index.vue
  <script setup lang="ts">
    const user: User = await fetch('/getUser')
    const fullName = `${user.firstName} ${user.lastName}`
  </script>

  <template>
  {{ fullName }}
  </template>
  ```
</div>

<v-click>
  <p class="text-red-500 font-semibold">This will error</p>
</v-click>

<!--
* Lets say the team managing the database decides to combine firstName and lastName into fullName

* Now every layer upstream will run into an error when trying to access values from the User object that no longer exist.
-->

---
layout: fact
---

<!-- TODO: Add little drumroll gif with v-click -->

<h1 class="text-lg font-semibold">tRPC to the rescue!</h1>

---
layout: image-right
image: /bg-2.png
---

<div class="flex h-full w-full flex-col items-center justify-center">
  <h1 class="font-semibold">Typescript Remote Procedure Call (tRPC)</h1>
  <p>tRPC is a method of exposing server functions to the client in a typesafe manner</p>
</div>

<!-- 
Worth mentioning it only works when the client and server are both written in Typescript.
-->

---
layout: two-cols
---

<div class="h-full flex flex-col justify-center">
  <h1>Dependencies</h1>

  ```bash
  npm install @trpc/client @trpc/server trpc-nuxt zod
  ```

  Add `trpc-nuxt` to your nuxt.config.ts:
  ```ts
  export default defineNuxtConfig({
    modules: ['trpc-nuxt/module'],
    typescript: {
      strict: true,
    },
  })
  ```
</div>

::right::

<div class="ml-4 h-full flex flex-col justify-center">

```bash
@trpc/client

- To make typesafe API calls from your client.
```

```bash
@trpc/server

- For tRPC endpoints and routers.
```


```bash
trpc-nuxt

- Nuxt module built build + maintained by wobsoriano
```

```bash
zod

- Server-side input validation
```
</div>


---
layout: two-cols
clicks: 2
---
<div class="flex h-full items-center justify-center">

```ts {1-7|10|11|all}
// server/trpc
import { initTRPC } from '@trpc/server'

// Avoid exporting the entire t-object since it's not very
// descriptive and can be confusing to newcomers used to
// meaning translation in i18n libraries.
const t = initTRPC.create()

// Base router and procedure helpers
export const router = t.router
export const procedure = t.procedure
```

</div>
::right::

<!-- TODO: All options -->
<!-- TODO: Make sure all ml in two-cols is equal to four -->
<div class="ml-4 flex flex-col h-full justify-center text-sm">
  <p v-click-hide="1">Create the t-object and extract the helpers you plan on using.</p>
  <ul v-click-hide="1">
  Options include
    <li>
      Router
    </li>
    <li>
      Procedures
    </li>
    <li>
      Middleware
    </li>
  </ul>
  <div v-click-hide="2">
    <p v-click="1">Router provides a space to collect related procedures with a shared namespace.</p>
  </div>
  <span v-click="2">Procedure can be viewed like rest endpoints that first validate the input using Zod, Yup, or Superstruct.</span>
</div>

---
layout: two-cols
clicks: 4
---

```ts {all|1-2|4-5|6-10|11-16|all}
import { z } from 'zod'
import { router, procedure } from '../trpc'

export const helloRouter = router({
  hello: procedure
    .input(
      z.object({
        text: z.string(),
      })
    )
    .query(({ input }) => {
      return {
        greeting: `hello ${input?.text ?? 'world'}`,
      }
    }),
})
```

::right::

<div v-click-hide="6" class="ml-4 flex flex-col h-full text-sm">
  <p v-click="1">
    Import the router and procedure created in <code>server/trpc</code>
  </p>
  <p v-click="2">
    Calling your router function and define your first procedure. You can think of this like an HTTP endpoint <code>api/trpc/hello</code>
  </p>
  <p v-click="3">
    Validate input using Zod. Here we are defining an input parameter named text that expects a string.
  </p>
  <p v-click="4">
    <code>.query()</code> is the same as an HTTP GET request. Here it is taking the input defined above and returning something based on that input.
  </p>
</div>

<!-- 
# Here's an example
#3 - Here we are defining an input parameter named text that expects a string.
Maybe live-code here?
-->

---
layout: two-cols
clicks: 3
---

```ts {all|4-5|11-19|all}
import { z } from 'zod'
import { router, procedure } from '../trpc'

export const authRouter = router({
  login: procedure
    .input(
      z.object({
        name: z.string(),
      })
    )
    .mutation(({ input }) => {
      // Here some login stuff would happen
      return {
        user: {
          name: input.name,
          role: 'ADMIN',
        },
      }
    }),
})
```

::right::

<div v-click-hide="6" class="ml-4 flex flex-col h-full text-sm">
  <p v-click="1">
    Here, we are creating a second router, <code>authRouter</code> with a login procedure
  </p>
  <p v-click="2">
    We're calling the <code>.mutation()</code> function which is the equivalent of an HTTP POST request.
  </p>
</div>

---
layout: image-right
image: /alternative.png
---

<div class="flex h-full w-full flex-col items-center justify-center">
  <h1 class="font-semibold">Think of it as</h1>
</div>

<!--
I understand it might feel like there is a lot to take in.
Just remember:
Zod (z): Validation
tRPC (t): HTTP-like routes
-->

---
layout: two-cols
---

<div class="flex w-full h-full items-center justify-center">

```ts
import { createNuxtApiHandler } from 'trpc-nuxt'
import { appRouter } from '@/server/trpc/routers'
// export API handler
export default createNuxtApiHandler({
  router: appRouter,
  createContext: () => ({}),
})
```


</div>
::right::

<div class="flex flex-col w-full h-full items-center justify-center">
<p>This is the entrypoint for your API and exposes the tRPC router.</p>

<p>This is where you can expose your API to middleware to manage things like CORS.</p>
</div>


---
layout: two-cols
clicks: 7
---

```ts {all|2-9|11-12}
// server/trpc/routers.ts
import { router } from '../trpc'
import { authRouter } from './authRouter'
import { helloRouter } from './helloRouter'

const appRouter = router({ 
  auth: authRouter, 
  hello: helloRouter,
})

// export type definition of API
export type AppRouter = typeof appRouter
```

<div class="ml-4 flex flex-col h-full text-sm">
  <p v-click="3">
    The AppRouter type signature is passed to trpc-nuxt's <code>createTRPCNuxtProxyClient</code> function.
  </p>
  <p v-click="7">
     Using <code>provide</code> to inject <code>trpc</code> to the client.
  </p>
</div>

::right::

<div class="ml-4 flex flex-col h-full text-sm">
  <p v-click="1">
    Now, we merge the routers into a centralize <code>appRouter</code>
  </p>
  <p v-click="2">
    The magic trpc sauce - <code>export/import type</code> - syntax introduced in <a src="https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-8.html">Typescript 3.8</a> 
  </p>

<div v-click="3">

```ts {4-7|8-12}
// plugins/trpc.ts
import { httpBatchLink } from '@trpc/client'
import { createTRPCNuxtProxyClient } from 'trpc-nuxt/client'
import type { AppRouter } from '@/server/trpc/routers'

export default defineNuxtPlugin(() => {
  const trpc = createTRPCNuxtProxyClient<AppRouter>()
  return {
    provide: {
      trpc,
    },
  }
})
```

</div>

</div>

<!-- 
tRPC shares type signatures, not data, code, or any runtime construct, between the server and client.
 -->

---
layout: fact
---

<h1 class="text-lg font-semibold">Now the fun begins</h1>

<!-- 
- We can now begin to enjoy our server to client and client to server type safety

- Live Code?
-->
