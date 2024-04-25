---
layout: post
title: "1小时学会SpringBoot3+Vue3前后端分离开发"
date: 2024-04-23T20:32:11+08:00
---

## 引言

大家可能刚学会`Java`和`Vue`之后都会想下一步是什么？那么就先把`SpringBoot`和`Vue`结合起来，做一个前后端分离的项目吧。

## 准备工作

首先你需要懂得`Java`和`Vue`的基础知识，环境这里就不多说了，直接开始。

## 创建 SpringBoot 项目

使用`IDEA`旗舰版的可以直接使用自带`Spring Initializr`创建项目，其他的可以使用[Spring Initializr](https://start.spring.io/)创建项目。

语言选择`Java`，类型选择`Gradle-Kotlin`，`Java`选择 21，其他的都随便填。

![20240423215241](https://s2.loli.net/2024/04/23/OCrh1epGw5UDq24.png)

![20240423210415](https://s2.loli.net/2024/04/23/4nLkO3TgEdtpFsa.png)

接下来选择依赖，这里选择`web`，`lombok`,数据库选择`PostgreSQL`，如果你使用的是`MySQL`就选它

![20240423210635](https://s2.loli.net/2024/04/23/4GF2vzm5CtlaJT8.png)

![20240423210727](https://s2.loli.net/2024/04/23/CDSJkhpmK52yEgv.png)

之后点击创建自动打开项目，或者点击生成打开下载的项目

![20240423210848](https://s2.loli.net/2024/04/23/6Y2qCyk8OGbhrc1.png)
![20240423210903](https://s2.loli.net/2024/04/23/Y54rbSCx8QUAycW.png)

之后等待项目的依赖下载完成就好了

如果需要配置镜像那就在`repositories`中最上面添加腾讯云的镜像

```kotlin
repositories {
    maven {
        url = uri("https://mirrors.cloud.tencent.com/nexus/repository/maven-public")
    }
    mavenCentral()
}
```

首先我们需要创建数据库，比如一个图书管理系统，需要有一张图书表，有一些字段，比如标题、作者、创建时间、等等。

我们使用数据库管理工具来创建一个表吧。

![20240423225934](https://s2.loli.net/2024/04/23/pz61HG9EyuQLWUl.png)

注意这里使用的是`Postgres`，如果是`MySQL`类型略有不同。

之后我们就可以创建实体类了，这里需要先引入`ORM`框架依赖，这里我为了方便引入写了一个`Gradle`插件，把它写入到`plugins`中，接着在刷新一下项目就可以继续编写代码了。

```kotlin
plugins {
    // 省略其他插件...
    id("cn.enaium.jimmer.gradle") version "0.0.11"
}
```

我们创建一个接口`Book`添加一个`Entity`和`Table`注解，之后添加一些方法，名称就是根据数据库中字段名称一样，只不过要把蛇形命名改为小驼峰。

```java
@Entity
@Table(name = "book")
public interface Book {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    int id();

    String title();

    String author();

    LocalDateTime createTime();
}
```

写完之后，我们按下编译的快捷键(默认是 Ctrl+F9)，之后就可以编写接口了。

```java
@RestController
@RequiredArgsConstructor
public class BookController {

    private final JSqlClient sql;

    @GetMapping("/book")
    public List<Book> getBooks() {
        return sql.createQuery(Tables.BOOK_TABLE).select(Tables.BOOK_TABLE).execute();
    }

    @PostMapping("/book")
    public void saveBook(@RequestBody Book book) {
        sql.save(book);
    }
}
```

使用`RequiredArgsConstructor`注解可以为被`final`修饰的字段生成构造方法，这样就不用手动写构造方法了。

`getBooks`用于获取图书列表，首先使用`createQuery`创建一个查询，传入一张表，类似于`from book`，接着使用`select`选择所有字段，类似于`select id, name, author, createTime`，最后使用`execute`执行查询。
`saveBook`用于保存图书，使用`save`方法保存图书。

接下来需要配置允许跨域，这里使用`CORS`，在`SpringBoot`中配置`CORS`很简单，只需实现`WebMvcConfigurer`接口的`addCorsMappings`方法即可。

```java
@Configuration
public class WebMvcConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedMethods("*")
                .allowedOrigins("*")
                .allowedHeaders("*");
    }
}
```

最后我们配置一下数据库链接。

```properties
spring.datasource.driver-class-name=org.postgresql.Driver
spring.datasource.url=jdbc:postgresql://localhost:5432/postgres?currentSchema=sbv
spring.datasource.username=postgres
spring.datasource.password=postgres
jimmer.dialect=org.babyfish.jimmer.sql.dialect.PostgresDialect
```

如果是`MySQL`就把`PostgresDialect`改为`MySqlDialect`

这样我们的后端就写完了，接下来我们开始写前端。

## 创建 Vue 项目

这里使用`pnpm create vue`来创建，如果没有安装`pnpm`可以使用`npm install -g pnpm`来安装。

![20240423215715](https://s2.loli.net/2024/04/23/5ojwLOBYrxkClK8.png)

名称随便，之后使用`Typescript`和`Vue Router`剩下的选否。

之后使用命令`pnpm install`安装依赖，并删除`src`下的所有文件。

编写`App.vue`

```vue
<script setup lang="ts"></script>

<template>
  <h1>Vue 3 + Vite + TypeScript</h1>
</template>
```

编写`main.ts`

```vue
import { createApp } from "vue"; import App from "./App.vue"; const app = createApp(App); app.mount("#app");
```

这里我使用`naive ui`，使用命令安装`pnpm add -D naive-ui`，之后使用自动导入配置。

安装这两个插件`pnpm add -D unplugin-auto-import unplugin-vue-components`

之后修改`vite.config.ts`文件

```ts
import { fileURLToPath, URL } from "node:url"

import { defineConfig } from "vite"
import vue from "@vitejs/plugin-vue"
import AutoImport from "unplugin-auto-import/vite"
import Components from "unplugin-vue-components/vite"
import { NaiveUiResolver } from "unplugin-vue-components/resolvers"

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [
    vue(),
    AutoImport({
      imports: [
        "vue",
        {
          "naive-ui": ["useDialog", "useMessage", "useNotification", "useLoadingBar"]
        }
      ]
    }),
    Components({
      resolvers: [NaiveUiResolver()]
    })
  ],
  resolve: {
    alias: {
      "@": fileURLToPath(new URL("./src", import.meta.url))
    }
  }
})
```

然后就可以继续编写页面了，首先在`views`中编写两个页面一个用来获取所有的图书，一个用来添加图书。

```vue
<script setup lang="ts">
import type { DataTableColumns } from "naive-ui"
import { ref } from "vue"

interface Book {
  id: number
  title: string
  author: string
  createTime: string
}

const columns: DataTableColumns<Book> = [
  {
    title: "ID",
    key: "id"
  },
  {
    title: "Title",
    key: "title"
  },
  {
    title: "Author",
    key: "author"
  },
  {
    title: "Create Time",
    key: "createTime"
  }
]

const books = ref<Book[]>([])

fetch("http://localhost:8080/book")
  .then((response) => response.json())
  .then((data: Book[]) => {
    books.value = data
  })
</script>

<template>
  <n-data-table :columns="columns" :data="books" />
</template>
```

```vue
<script setup lang="ts">
import { ref } from "vue"
import { useMessage, type FormInst } from "naive-ui"

interface BookInput {
  title?: string
  author?: string
}

const message = useMessage()

const formRef = ref<FormInst | null>(null)
const bookInput = ref<BookInput>({})

const save = () => {
  formRef.value?.validate().then((valid) => {
    if (valid) {
      fetch("http://localhost:8080/book", {
        method: "POST",
        headers: {
          "Content-Type": "application/json"
        },
        body: JSON.stringify(bookInput.value)
      }).then(() => {
        message.success("Book saved")
      })
    }
  })
}
</script>

<template>
  <n-form ref="formRef" :model="bookInput">
    <n-form-item label="Title" path="title" :rule="[{ required: true, message: 'Please input title' }]">
      <n-input v-model:value="bookInput.title" />
    </n-form-item>
    <n-form-item label="Author" path="author" :rule="[{ required: true, message: 'Please input author' }]">
      <n-input v-model:value="bookInput.author" />
    </n-form-item>
    <n-button type="primary" @click="save">Save</n-button>
  </n-form>
</template>
```

之后编写布局，在`layouts`下编写一个，`BookLayout.vue`，我们使用左侧一栏来选择页面，右侧来展示页面。

```vue
<script setup lang="ts"></script>

<template>
  <n-layout has-sider>
    <n-layout-sider content-style="padding: 24px;">
      <ul>
        <li>
          <RouterLink to="/book">Book</RouterLink>
        </li>
        <li>
          <RouterLink to="/book/create">Create Book</RouterLink>
        </li>
      </ul>
    </n-layout-sider>
    <n-layout>
      <n-layout-content content-style="padding: 24px;">
        <RouterView />
      </n-layout-content>
    </n-layout>
  </n-layout>
</template>
```

之后创建一个`router`目录，编写`index.ts`文件

```ts
import BookLayout from "@/layouts/BookLayout.vue"
import Books from "@/views/Books.vue"
import SaveBook from "@/views/SaveBook.vue"
import { createRouter, createWebHistory } from "vue-router"

const router = createRouter({
  history: createWebHistory(),
  routes: [
    {
      path: "/book",
      component: BookLayout,
      children: [
        {
          path: "",
          component: Books
        },
        {
          path: "create",
          component: SaveBook
        }
      ]
    }
  ]
})

export default router
```

之后在`main.ts`中引入`router`

```ts
import router from "./router"

// 省略其他代码...

app.use(router)
```

最后在`App.vue`中使用`RouterView`

```vue
<template>
  <NMessageProvider>
    <RouterView />
  </NMessageProvider>
</template>
```

这样我们的前端就写完了，接下来我们启动项目。