---
title: "使用SolidJS+SpringBoot写一个简单的个人博客"
date: 2023-05-23T12:24:52+08:00
---

## 前言

前端我们使用了`SolidJS`来进行开发,它是一个新出的前端框架,它的特点是使用`React Hooks`的方式来进行开发,并且它的`API`和`React`的`API`非常相似,所以如果你会`React`的话,那么你就会`SolidJS`.

后端我们使用了`SpringBoot`来进行开发,数据库我们使用了`MySQL`来进行开发,这里我使用的是`MariaDB`来进行开发,ORM 框架使用的是`Jimmer`,这个框架它支持`Java`和`Kotlin`,这里为了简单起见就使用`Java`开发,但实际使用`Koltin`会更方便.

## 前端环境

首先呢,需要安装一下`SolidJS`的模板,这里我使用的是`TypeScript`和`Bootstrap`的模板

```bash
npx degit solidjs/templates/ts-bootstrap website
cd website
pmpm install # or npm install
pmpm dev # or npm run dev
```

然后我们需要安装一下`SolidJS`的路由,这是一个官方的路由

```bash
pnpm install @solidjs/router
```

然后我们需要安装一下`SolidJS`的状态管理,这里我使用的是`zustand`

```bash
pnpm install zustand solid-zustand
```

然后我们需要安装一下`Immer`用于修改不可变数据

```bash
pnpm install solid-immer
```

然后我们需要安装`solid-toast`,这是一个用于显示提示的库

```bash
pnpm install solid-toast
```

最后我们需要安装`TanStack Query`,这是一个用于进行网络请求的库,同时支持`React`,`Vue`,`Svelte`当然也支持`SolidJS`

```bash
pnpm install @tanstack/solid-query
```

### 编写前端基础代码

现在我们开始编写前端代码

首先我们需要删掉模板中自带的样式,并且删除`App.tsx`中的内容,然后我们需要在`App.tsx`中引入`Router`组件,并且使用`useRoutes`来进行路由的渲染,这里我们需要传入一个路由数组,这个数组中包含了我们的路由信息,然后我们需要在`App.tsx`中使用`Router`组件来进行路由的渲染

```tsx
import type { Component } from "solid-js"
import { Router, useRoutes } from "@solidjs/router"
import routes from "./router" //这里是路由数组
import { Toaster } from "solid-toast" //这个组件用于显示提示

const App: Component = () => {
  const Routes = useRoutes(routes)

  return (
    <>
      <Toaster />
      <Router>
        <Routes />
      </Router>
    </>
  )
}
```

之后新建`views`目录,在其中新家一个`Login.tsx`文件,这个文件用于渲染登录页面,然后我们需要在路由数组中添加这个路由,路由文件就是在`src`目录下新建一个`router`目录,然后在其中新建一个`index.ts`文件,这个文件用于导出路由数组

```tsx
import { RouteDefinition } from "@solidjs/router"
import Login from "../views/Login"

const routes: RouteDefinition[] = [
  {
    path: "/login",
    component: () => <Login /> //这里有个坑,如果不加上() =>,那么会报错,也可以直接使用Login比如component: Login
  }
]

export default routes
```

好了现在来写用户的`session`状态的管理,我们需要在`src`目录下新建一个`store`目录,然后在其中新建一个`index.ts`文件,这个文件用于导出`session`状态

```tsx
import create from "solid-zustand" //需要注意的是这里使用的是solid-zustand,而不是zustand
import { persist } from "zustand/middleware"

//这里定义了session的类型
interface Session {
  token?: string
  id?: string
  setToken: (token: string) => void
  setId: (id: string) => void
}

//这里使用zustand的persist中间件来进行持久化
export const useSessionStore = create(
  persist<Session>(
    (set, get) => ({
      setToken: (token: string) => set({ token }),
      setId: (id: string) => set({ id })
    }),
    {
      name: "session-store",
      getStorage: () => localStorage
    }
  )
)
```

## 后端环境

首先打开`IntelliJ IDEA`然后选择`Spring Initializr`,语言选择`Java`,构建工具选择`Gradle`,脚本使用`Kotlin`,因为`kts`比`groovy`更好用,剩下的自己看着改改,`JDK`使用`17`,因为`SpringBoot3`只能使用`JDK17`,然后点击`Next`,依赖选择`SpringWeb`,`MariaDB Driver`,`Lombok`,上面这些步骤使用`start.spring.io`也可以完成,然后点击`Create`,然后等待`Gradle`下载依赖,下载完成后就可以开始编写代码了

首先引入`Jimmer`的相关依赖

```kotlin
dependencies {
implementation("org.babyfish.jimmer:jimmer-spring-boot-starter:0.7.67")//这个是Jimmer的SpringBoot的Starter
annotationProcessor("org.babyfish.jimmer:jimmer-apt:0.7.67")//这个是Jimmer的APT,用于生成代码
}
```

接着引入`SaToken`的依赖

```kotlin
implementation("cn.dev33:sa-token-spring-boot3-starter:1.34.0")
```

然后我们需要在`application.properties`中配置一下所需要的配置

```properties
#数据库配置,这里使用的是MariaDB,因为我的数据库在虚拟机中,所以这里使用的是虚拟机的IP地址
spring.datasource.driver-class-name=org.mariadb.jdbc.Driver
spring.datasource.username=root
spring.datasource.password=root
spring.datasource.url=jdbc:mariadb://192.168.56.101:3306/blog
#Jimmer配置,这里配置了生成Typescript的路径,以及是否显示SQL语句,以及是否美化SQL语句
jimmer.client.ts.path=/typescript.zip
jimmer.show-sql=true
jimmer.pretty-sql=true
#SaToken配置,这里配置了token的名称
sa-token.token-name=token
```

需要注意一下,`IntelliJ IDEA`默认对`properties`文件的编码是`ISO-8859-1`,所以我们需要修改一下编码,修改方法是在`File`->`Settings`->`Editor`->`File Encodings`中修改`Default encoding for properties files`的编码为`UTF-8`,要不然中文会乱码

### 设计数据库

首先一个简单的个人博客包括了文章,评论,用户,分类

```sql
create table blog.category
(
    id   uuid        not null
        primary key,
    name varchar(32) not null
);

create table blog.user
(
    id       uuid        not null
        primary key,
    username varchar(18) not null,
    password varchar(18) not null
);

create table blog.post
(
    id          uuid        not null
        primary key,
    user_id     uuid        not null,
    category_id uuid        not null,
    title       varchar(50) not null,
    content     text        not null,
    constraint post_category_id_fk
        foreign key (category_id) references blog.category (id),
    constraint post_user_id_fk
        foreign key (user_id) references blog.user (id)
);

create table blog.reply
(
    id      uuid          not null
        primary key,
    user_id uuid          not null,
    post_id uuid          not null,
    content varchar(1000) not null,
    constraint reply_post_id_fk
        foreign key (post_id) references blog.post (id),
    constraint reply_user_id_fk
        foreign key (user_id) references blog.user (id)
);
```

## 后端编写

### 定义实体类

在`Jimmer`中定义实体类需要使用接口,如果你使用的是自增主键,那么你需要使用`@GeneratedValue(strategy = GenerationType.IDENTITY)`

```java
import org.babyfish.jimmer.sql.Entity;
import org.babyfish.jimmer.sql.GeneratedValue;
import org.babyfish.jimmer.sql.Id;
import org.babyfish.jimmer.sql.Table;
import org.babyfish.jimmer.sql.meta.UUIDIdGenerator;
import org.jetbrains.annotations.NotNull;

@Entity//这个注解用于标记这个接口是一个实体类
@Table(name = "user")//如果表名比较特殊,那么可以使用这个注解来指定表名
public interface User {
    @Id//这个注解用于标记这个属性是主键
    @GeneratedValue(generatorType = UUIDIdGenerator.class)//这个注解用于标记这个属性的值是自动生成的,这里使用的是UUID
    UUID id();


    @NotNull//这个注解用于标记这个属性的值不可为空,推荐使用Jetbrains的注解
    String username();

    @NotNull
    String password();
}
```

接着定义其他的实体类

```java
@Entity
@Table(name = "category")
public interface Category {
    @Id
    @GeneratedValue(generatorType = UUIDIdGenerator.class)
    UUID id();

    String name();
}
```

```java
@Entity
@Table(name = "post")
public interface Post {
    @Id
    @GeneratedValue(generatorType = UUIDIdGenerator.class)
    UUID id();

    UUID userId();

    UUID categoryId();

    @NotNull
    String title();

    @NotNull
    String content();
}
```

```java
@Entity
@Table(name = "reply")
public interface Reply {
    @Id
    @GeneratedValue(generatorType = UUIDIdGenerator.class)
    UUID id();

    UUID userId();

    UUID postId();

    @NotNull
    String content();
}
```

定义好实体类之后需要运行一下项目,这样`Jimmer`就会生成代码,`Draft`,`Fetcher`,`Props`,`Table`和`TableEx`之类的,找到项目下的`build/generated/sources/annotationProcessor`位置,你就会发现这些生成的代码,如果使用的是`Kotlin`生成的类会和这些有点不同,但本篇文章使用的是`Java`,所以就不多说了,因为虽然本质上一样,但还是有点不同的

如果你使用的是 Linux 环境,那么你可以像我一样为每个实体类加上一个`Table`注解,那是因为`Linux`的名称是区分大小写的,而`Windows`的名称是不区分大小写的,`Jimmer`默认是会将所有的名称转换为大写,你也可以直接使用配置来修改这个行为

```java
@Bean
public DatabaseNamingStrategy databaseNamingStrategy() {
    return DefaultDatabaseNamingStrategy.LOWER_CASE;//这里使用的是小写
}
```

### 定义 Repository

`Jimmer`和`JPA`一样,同样拥有`Repository`,因为`Jimmer`同时支持`Java`和`Kotlin`,所以这里的`Repository`是`JRepository`,这个接口继承了`JPA`的`JpaRepository`,所以它拥有`JPA`的所有功能,泛型的第一个参数是实体类的类型,第二个参数是主键的类型,这里我的主键使用的是`UUID`,所以这里的类型是`UUID`

```java
import cn.enaium.blog.model.entity.User;
import org.babyfish.jimmer.spring.repository.JRepository;
import org.springframework.stereotype.Repository;

import java.util.UUID;

@Repository//这个注解用于标记这个接口是一个Repository,这个注解是SpringData提供的,也可以不用这个注解,SpringBoot会自动扫描这个接口
public interface UserRepository extends JRepository<User, UUID> {
}
```

接着定义其他的`Repository`

```java
@Repository
public interface CategoryRepository extends JRepository<Category, UUID> {
}
```

```java
@Repository
public interface PostRepository extends JRepository<Post, UUID> {
}
```

```java
@Repository
public interface ReplyRepository extends JRepository<Reply, UUID> {
}
```

现在最基本的代码就写完了,还有一部分没有写,之后一遍写`Controller`时再写,现在我们需要编写一下`Controller`的代码

### 编写 Controller

#### SessionController

`SessionController`用来处理用户的登录和和退出登录

```java
@RestController
public class SessionController {
    @PutMapping("/sessions/")
    public void login() {

    }

    @DeleteMapping("/sessions/")
    public void logout() {

    }
}
```

既然要登录,那么就需要一个登录的请求体,这里我使用的是`JSON`来进行传输,所以我们需要定义一个`UserInput`类,这个类用于接收`JSON`数据

```java
@Data
public class UserInput {
    @Nullable
    private final UUID id;
    @Nullable
    private final String username;
    @Nullable
    private final String password;
}
```

接着将`UserRepository`注入到`SessionController`中

```java
@AllArgsConstructor//这个注解用于标记这个类的构造函数是全参构造函数,这个注解是Lombok提供的
public class SessionController {
    private final UserRepository userRepository;//这里使用的是构造函数注入,也可以使用字段注入,但是不推荐使用字段注入
}
```

首先要在`UserRepository`中编写一个`findByUsername`方法,这个方法用于根据用户名查询用户

```java
@Repository
public interface UserRepository extends JRepository<User, UUID> {
    User findByUsername(String username);//这里的方法名是有规则的,如果是根据用户名查询,那么方法名就是findByUsername,如果是根据用户名和密码查询,那么方法名就是findByUsernameAndPassword
}
```

然后我们需要在`SessionController`中编写`login`方法,简单的登录逻辑,如果用户不存在,那么就抛出一个`NullPointerException`,如果密码错误,那么就抛出一个`IllegalArgumentException`,如果登录成功,那么就返回一个`LoginResponse`对象,这个对象包含了`token`和`id`

```java
@PutMapping("/sessions/")
public LoginResponse login(@RequestBody UserInput userInput) {
    final var user = userRepository.findByUsername(userInput.getUsername());
    if (user == null) {
        throw new NullPointerException("User not found");
    }
    if (!user.password().equals(userInput.getPassword())) {
        throw new IllegalArgumentException("Password error");
    }
    return new LoginResponse(StpUtil.createLoginSession(user.id()), user.id());
}
```

```java
@Data
public class LoginResponse {
    private final String token;
    private final UUID id;
}
```

然后我们需要编写`logout`方法,这个方法用于退出登录,这个方法很简单,就是调用`SaToken`的`logout`方法

```java
@DeleteMapping("/sessions/")
public void logout() {
    StpUtil.logout();
}
```

#### UserController

接着我们需要编写`UserController`,这个类用于处理用户的注册

```java
@RestController
@AllArgsConstructor
public class UserController {

    private final UserRepository userRepository;

    @PutMapping("/users/")
    @ResponseStatus(HttpStatus.OK)//这个注解用于标记这个方法的响应状态码,这个注解是Spring提供的
    public void register(@RequestBody UserInput userInput) {
        //如果用户已经存在,那么就抛出一个异常
        if (userRepository.findByUsername(userInput.getUsername()) != null) {
            throw new IllegalArgumentException("User already exists");
        }

        //插入用户,这里使用的是Jimmer的UserDraft,这个类用于创建一个User对象,这个对象是一个Draft对象,这个对象可以修改,因为在Jimmer中实体类是不可变的,所以需要使用Draft,对于使用过Immer的人来说,这个Draft就是一个Immer的Draft,这里使用的是produce方法,这个方法用于创建一个Draft对象,这个方法的参数是一个Consumer,这个Consumer用于修改Draft对象,这里Jimmer借鉴了Immer
        userRepository.insert(UserDraft.$.produce(draft -> {
            draft.setUsername(userInput.getUsername());
            draft.setPassword(userInput.getPassword());
        }));
    }
}
```

#### CategoryController

然后我们需要编写`CategoryController`,这个类用于处理分类的查询

```java
@RestController
@AllArgsConstructor
public class CategoryController {
    private final CategoryRepository categoryRepository;

    @GetMapping("/categories/")
    public List<Category> findCategories() {
        return categoryRepository.findAll();
    }
}
```

#### PostController

然后我们需要编写`PostController`,这个类用于处理文章的查询,这里会使用`Jimmer`的`Fetcher`功能,这个功能用于查询关联表的数据,这里我们需要查询文章的分类,所以需要使用`Fetcher`功能

首先定义`PostInput`类,这个类用于接收`JSON`数据

```java
@Data
public class PostInput {
    @Nullable
    private final UUID id;
    @Nullable
    private final String title;
    @Nullable
    private final String content;
    @Nullable
    private final UUID categoryId;
}
```

接着设置表的关联关系,表的关联在表的实体类中进行设置,需要注意的是,关联建立后,之前生成的代码是不会更新的,所以需要手动更新一下,这里我使用的是`IntelliJ IDEA`的`Rebuild Project`功能,这个功能会重新生成代码,或者使用`Gradle`的`build`功能也可以,这里我使用的是`IntelliJ IDEA`的`Rebuild Project`功能

首先`Post`表和`Category`表是多对一的关系,所以在`Post`表中设置一个`category`方法,之后再`Category`实体类中定义一个`posts`方法,这个方法表示这个分类下的所有文章

```java
@Entity
@Table(name = "post")
public interface Post {
    @Id
    @GeneratedValue(generatorType = UUIDIdGenerator.class)
    UUID id();

    UUID userId();

    UUID categoryId();

    @ManyToOne//这个注解用于标记这个属性是多对一的关系
    Category category();

    @NotNull
    String title();

    @NotNull
    String content();
}
```

```java
@Entity
@Table(name = "category")
public interface Category {
    @Id
    @GeneratedValue(generatorType = UUIDIdGenerator.class)
    UUID id();

    @NotNull
    String name();

    @OneToMany(mappedBy = "category")//这个注解用于标记这个属性是一对多的关系,这里的mappedBy表示这个属性在另一个实体类中的名称
    List<Post> posts();
}
```

之后我们开始写`PostController`的代码,在编写之前响一下,都需要写哪些方法,这里我列出来

- 查询所有文章(包含分类),用于渲染首页,但这里不需要文章的内容,所以不需要查询文章的内容,这里需要使用`Fetcher`功能
- 根据文章 ID 查询文章,用于渲染文章详情页,这里需要查询文章的内容,所以需要使用`Fetcher`功能
- 根据分类 ID 查询文章,用于渲染分类页,但这里不需要文章的内容,这里需要使用`Fetcher`功能

以上就是基本需要的方法,所以这里我们需要定义 2 个`Fetcher`,一个是不包含文章内容的`Fetcher`,一个是包含文章内容的`Fetcher`

```java
public class PostController {
    //这个Fetcher用于查询所有文章,但不包含文章内容
    public static final PostFetcher DEFAULT_POST = PostFetcher.$
            .allScalarFields()//这个方法用于查询所有的标量字段,标量字段就是不是关联表的字段
            .content(false)//这个方法用于设置是否查询文章内容,这里设置为false,表示不查询文章内容
            .category(//这个方法用于设置查询文章的分类,这里使用的是一个CategoryFetcher,这个Fetcher用于查询分类
                    CategoryFetcher.$
                            .allScalarFields()//这个方法用于查询所有的标量字段,标量字段就是不是关联表的字段
            );

    //这个Fetcher用于查询所有文章,包含文章内容
    public static final PostFetcher FULL_POST = PostFetcher.$
            .allScalarFields()//这个方法用于查询所有的标量字段,标量字段就是不是关联表的字段
            .category(//这个方法用于设置查询文章的分类,这里使用的是一个CategoryFetcher,这个Fetcher用于查询分类
                    CategoryFetcher.$
                            .allScalarFields()//这个方法用于查询所有的标量字段,标量字段就是不是关联表的字段
            );
}
```

接着写接口

```java
@RestController
@AllArgsConstructor
public class PostController {
    private final PostRepository repository;

    //这个方法用于创建文章
    @PutMapping("/posts/")
    @ResponseStatus(HttpStatus.OK)
    public void createPost(@RequestBody PostInput postInput) {
        repository.insert(PostDraft.$.produce(draft -> {
            draft.setUserId(UUID.fromString(StpUtil.getLoginIdAsString()));
            draft.setTitle(postInput.getTitle());
            draft.setContent(postInput.getContent());
            draft.setCategoryId(postInput.getCategoryId());
        }));
    }

    //这个方法用于查询所有文章,但不包含文章内容,FetchBy注解用于标记这个方法使用的是哪个Fetcher,其实就是为了生成Typescript代码时使用的,这个注解是Jimmer提供的,这个注解的参数是一个字符串,这个字符串就是Fetcher的名称,这个名称就是在哪个类中定义的,这里是在PostController中定义的,默认是在当前类中定义的,如果不想在当前类中定义,那么就需要使用ownerType参数来指定
    @GetMapping("/posts/")
    public Page<@FetchBy(value = "FULL_POST", ownerType = PostController.class) Post> findPosts(@RequestParam(defaultValue = "0") Integer page, @RequestParam(defaultValue = "10") Integer size) {
        return repository.findAll(PageRequest.of(page, size), DEFAULT_POST);//这里使用的是findAll方法,这个方法用于查询所有的实体类,这个方法的第二个参数是一个Fetcher,这个Fetcher用于查询关联表的数据,这里使用的是DEFAULT_POST,表示查询文章的分类,但不查询文章的内容,这个方法是Jimmer提供的
    }

    //这个方法用于查询所有文章,包含文章内容
    @GetMapping("/posts/{id}")
    public @FetchBy("FULL_POST") Post findPost(@PathVariable UUID id) {
        return repository.findNullable(id, FULL_POST);//这里使用的是findNullable方法,这个方法用于查询一个实体类,如果查询不到,那么就返回null,如果查询到了,那么就返回这个实体类,这个方法的第二个参数是一个Fetcher,这个Fetcher用于查询关联表的数据,这里使用的是FULL_POST,表示查询文章的分类,同时查询文章的内容,这个方法是Jimmer提供的
    }

    //这个方法用于查询分类下的所有文章,但不包含文章内容
    @GetMapping("/categories/{categoryId}/posts/")
    public Page<@FetchBy("DEFAULT_POST") Post> findPostsByCategory(@PathVariable UUID categoryId, @RequestParam(defaultValue = "0") Integer page, @RequestParam(defaultValue = "10") Integer size) {
        return repository.findAllByCategoryId(PageRequest.of(page, size), categoryId, DEFAULT_POST);
    }
}
```

接着看下`PostRepository`中的代码

```java
@Repository
public interface PostRepository extends JRepository<Post, UUID> {
    //这个方法根据分类ID查询文章,具体的Fetcher在方法的参数中指定
    Page<Post> findAllByCategoryId(Pageable pageable, UUID categoryId, Fetcher<Post> fetcher);
}
```

#### ReplyController

然后我们需要编写`ReplyController`,这个类用于处理评论的查询,这里会使用`Jimmer`的`Fetcher`功能,这个功能用于查询关联表的数据,这里我们需要查询评论的用户,所以需要使用`Fetcher`功能

首先定义`ReplyInput`类,这个类用于接收`JSON`数据

```java
@Data
public class ReplyInput {
    @Nullable
    private final UUID id;

    @Nullable
    private final UUID userId;

    @Nullable
    private final UUID postId;

    @Nullable
    private final String content;
}
```

接着设置表的关联关系,表的关联在表的实体类中进行设置

```java
@Entity
@Table(name = "reply")
public interface Reply {
    @Id
    @GeneratedValue(generatorType = UUIDIdGenerator.class)
    UUID id();

    UUID userId();

    @ManyToOne//这个注解用于标记这个属性是多对一的关系
    User user();

    UUID postId();

    @NotNull
    String content();
}
```

```java
@Entity
@Table(name = "user")
public interface User {
    @Id
    @GeneratedValue(generatorType = UUIDIdGenerator.class)
    UUID id();

    @NotNull
    String username();

    @NotNull
    String password();

    @OneToMany(mappedBy = "user")//这个注解用于标记这个属性是一对多的关系,这里的mappedBy表示这个属性在另一个实体类中的名称
    List<Reply> replies();
}
```

接着写接口

```java
@RestController
@AllArgsConstructor
public class ReplyController {

    private final ReplyRepository replyRepository;

    @PutMapping("/replies/")
    public void createReply(@RequestBody ReplyInput replyInput) {
        replyRepository.insert(ReplyDraft.$.produce(draft -> {
            draft.setUserId(UUID.fromString(StpUtil.getLoginIdAsString()));
            draft.setContent(replyInput.getContent());
            draft.setPostId(replyInput.getPostId());
        }));
    }

    @GetMapping("/posts/{postId}/replies/")
    public Page<@FetchBy("DEFAULT_REPLY") Reply> findRepliesByPost(@PathVariable UUID postId, @RequestParam(defaultValue = "0") Integer page, @RequestParam(defaultValue = "10") Integer size) {
        return replyRepository.findAllByPostId(PageRequest.of(page, size), postId, DEFAULT_REPLY);
    }

    //这个方法用于查询所有评论,包含评论的用户,但不包含用户的密码
    public static final ReplyFetcher DEFAULT_REPLY = ReplyFetcher.$.allScalarFields().user(UserFetcher.$.username());
}
```

#### ExceptionController

现在编写全局异常处理,这个异常处理用于处理`NullPointerException`和`IllegalArgumentException`等异常

```java
@ControllerAdvice//这个注解用于标记这个类是一个全局异常处理类
public class ExceptionController {
    @ExceptionHandler(Exception.class)//这个注解用于标记这个方法是一个异常处理方法,这里的参数是异常的类型,因为要处理所有的异常,所以这里使用的是Exception
    public ResponseEntity<String> exception(Exception exception) {
        if (exception instanceof NullPointerException) {
            //这里返回的是一个ResponseEntity,这个对象包含了状态码和响应体,这里我们返回的是404状态码和异常信息
            return ResponseEntity.status(HttpStatus.NOT_FOUND).body(exception.getMessage());
        } else if (exception instanceof IllegalArgumentException) {
            //这里返回的是一个ResponseEntity,这个对象包含了状态码和响应体,这里我们返回的是400状态码和异常信息
            return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(exception.getMessage());
        } else if (exception instanceof NotLoginException) {
            //这里返回的是一个ResponseEntity,这个对象包含了状态码和响应体,这里我们返回的是401状态码和异常信息
            return ResponseEntity.status(HttpStatus.UNAUTHORIZED).body(exception.getMessage());
        }
        //这里返回的是一个ResponseEntity,这个对象包含了状态码和响应体,这里我们返回的是500状态码和异常信息
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(exception.getMessage());
    }
}
```

最后我们要开启后端的`CORS`,这个功能用于解决跨域问题,这里我使用的是拦截器来实现的,首先我们需要编写一个`CorsInterceptor`类,这个类用于拦截所有的请求,然后设置响应头

```java
@Component
public class CorsInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(@NotNull HttpServletRequest request, @NotNull HttpServletResponse response, @NotNull Object handler) throws Exception {
        response.setHeader(HttpHeaders.ACCESS_CONTROL_ALLOW_METHODS, "*");
        response.setHeader(HttpHeaders.ACCESS_CONTROL_ALLOW_HEADERS, "*");
        response.setHeader(HttpHeaders.ACCESS_CONTROL_ALLOW_ORIGIN, "*");
        response.setHeader(HttpHeaders.ACCESS_CONTROL_ALLOW_CREDENTIALS, "true");
        return HandlerInterceptor.super.preHandle(request, response, handler);
    }
}
```

然后我们需要在`WebMvcConfigurer`中注册这个拦截器

```java
@Configuration
@AllArgsConstructor
public class WebMvcConfiguration implements WebMvcConfigurer {
    private final CorsInterceptor corsInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(corsInterceptor);
    }
}
```

## 前端编写

### 生成前端的代码

我们在后端定义了那么多的实体类,那么我们需要将这些实体类生成`TypeScript`的代码,这里我们使用的是`Jimmer`自带的功能,在之前的配置中我们配置了`jimmer.client.ts.path=/typescript.zip`,这个配置用于指定生成的`TypeScript`代码的路径,我们可以手动访问这个地址下载一个压缩包,然后解压这个压缩包,然后将其中的`typescript`文件夹复制到`src`目录下,也可以写一个脚本来自动完成这些工作,这里我使用了`PowerShell`来完成这些工作,在和`src`的同级目录下新建一个`scripts`目录,然后在其中新建一个`generated.ps1`文件,这个文件用于生成`TypeScript`代码

```powershell
$sourceUrl = "http://localhost:8080/typescript.zip"
$tempDir = Join-Path $(Get-Item -Path $env:TEMP).FullName (New-Guid).ToString()
$generatedPath = Join-Path (Split-Path $PSScriptRoot) "src/__generated"

# 下载源码并解压
if (-not (Test-Path $tempDir)) {
    New-Item -ItemType Directory -Path $tempDir | Out-Null
}

Write-Host "Downloading $sourceUrl to $tempDir..."
Invoke-WebRequest -Uri $sourceUrl -OutFile "$tempDir/source.zip"
Write-Host "Extracting files from $tempDir/source.zip..."
Expand-Archive -Path "$tempDir/source.zip" -DestinationPath $tempDir

# 删除已有的生成路径
if (Test-Path $generatedPath) {
    Write-Host "Deleting existing generated path $generatedPath..."
    Remove-Item -Path $generatedPath -Recurse -Force
}

# 移动文件
Write-Host "Moving files from $tempDir to $generatedPath..."
Move-Item -Path $tempDir -Destination $generatedPath

# 删除源码
Remove-Item -Path "$generatedPath/source.zip"

Write-Host "API code is refreshed successfully."
```

如果是对`Typescript`代码比较熟悉的话,看这些生成的代码应该没什么问题,但这里时间有限,所以我就不详细讲解了

### 编写前端代码

在生成了代码之后,我们还需要为`fetch`创建一个拦截器,这个拦截器用于在每次请求时添加`token`,在`common`目录中新建一个`ApiInstance.ts`文件,这个文件用于创建`fetch`的实例

```ts
import { Api } from "../__generated"
import { useSessionStore } from "../store"

//创建一个Api实例
export const api = new Api(async ({ uri, method, body }) => {
  //获取token
  const token = useSessionStore().token as string | undefined
  //发送请求
  const response = await fetch(`http://localhost:8080${uri}`, {
    method,
    body: body !== undefined ? JSON.stringify(body) : undefined,
    headers: {
      "content-type": "application/json;charset=UTF-8",
      ...(token !== undefined && token !== "" ? { token } : {})
    }
  })
  //如果响应状态码不是200,那么就抛出一个错误
  if (response.status !== 200) {
    throw await response.text()
  }
  //获取响应体
  const text = await response.text()
  //如果响应体为空,那么就返回null
  if (text.length === 0) {
    return null
  }
  //返回响应体
  return JSON.parse(text)
})
```

### 编写注册页面

在`views`目录下新建一个`Register.tsx`文件,这个文件用于渲染注册页面

```tsx
import { createImmerSignal } from "solid-immer"
import { UserInput } from "../__generated/model/static"
import { api } from "../common/ApiInstance"
import toast from "solid-toast"
import { Link, Route, useNavigate } from "@solidjs/router"

const Register = () => {
  const navigate = useNavigate() //这个函数用于跳转路由

  let formRef //这个引用用于获取表单元素

  const [form, setForm] = createImmerSignal<UserInput>({}) //这个函数用于创建一个信号,这个信号用于存储表单数据

  const submit = (e) => {
    //阻止默认事件
    e.preventDefault()
    e.stopPropagation()

    //如果表单验证通过,那么就发送请求
    if (formRef.checkValidity()) {
      api.userController
        .register({ body: form() })
        .then(() => {
          //如果注册成功,那么就跳转到登录页面
          toast.success("Registered successfully")
          navigate("/login")
        })
        .catch((err) => {
          //如果注册失败,那么就显示错误信息
          toast.error(err)
        })
    }

    //这里需要手动调用表单的验证方法,因为我们使用的是原生的表单验证,而不是使用第三方库
    formRef.classList.add("was-validated")
  }

  return (
    <div class="vh-100 d-flex justify-content-center align-items-center">
      <div class="card p-5">
        <form ref={formRef} class="needs-validation" style={{ width: "18rem", height: "14rem" }} novalidate>
          <div class="d-flex flex-column justify-content-between h-100">
            <div>
              <label for="validationCustom01" class="form-label">
                Username
              </label>
              <input
                type="text"
                class="form-control"
                id="validationCustom01"
                value={form().username ?? ""}
                required
                onInput={(e) => setForm((draft) => (draft.username = e.currentTarget.value))}
              />
              <div class="valid-feedback">Looks good!</div>
              <div class="invalid-feedback">Please enter your username.</div>
            </div>
            <div>
              <label for="validationCustom02" class="form-label">
                Password
              </label>
              <input
                type="text"
                class="form-control"
                id="validationCustom02"
                required
                value={form().password ?? ""}
                onInput={(e) => setForm((draft) => (draft.password = e.currentTarget.value))}
              />
              <div class="valid-feedback">Looks good!</div>
              <div class="invalid-feedback">Please enter your password.</div>
            </div>
            <Link href="/login">Login</Link>
            {/* 这里使用了Link组件,这个组件用于跳转路由 */}
            <button class="btn btn-primary" type="submit" onClick={submit}>
              Register
            </button>
          </div>
        </form>
      </div>
    </div>
  )
}

export default Register
```

### 编写登录页面

在之前已经创建好的`Login.tsx`中编写登录页面

```tsx
import { createImmerSignal } from "solid-immer"
import { useSessionStore } from "../store"
import { UserInput } from "../__generated/model/static"
import { api } from "../common/ApiInstance"
import toast from "solid-toast"
import { Link, Route, useNavigate } from "@solidjs/router"

const Login = () => {
  const navigate = useNavigate()

  const session = useSessionStore() //这个函数用于获取session状态

  let formRef

  const [form, setForm] = createImmerSignal<UserInput>({})

  const submit = (e) => {
    e.preventDefault()
    e.stopPropagation()

    if (formRef.checkValidity()) {
      api.sessionController
        .login({ body: form() })
        .then((data) => {
          //如果登录成功,那么就将token和id存储到session中,然后跳转到首页
          session.setToken(data.token)
          session.setId(data.id)
          navigate("/")
        })
        .catch((err) => {
          toast.error(err)
        })
    }

    formRef.classList.add("was-validated")
  }

  return (
    <div class="vh-100 d-flex justify-content-center align-items-center">
      <div class="card p-5">
        <form ref={formRef} class="needs-validation" style={{ width: "18rem", height: "14rem" }} novalidate>
          <div class="d-flex flex-column justify-content-between h-100">
            <div>
              <label for="validationCustom01" class="form-label">
                Username
              </label>
              <input
                type="text"
                class="form-control"
                id="validationCustom01"
                value={form().username ?? ""}
                required
                onInput={(e) => setForm((draft) => (draft.username = e.currentTarget.value))}
              />
              <div class="valid-feedback">Looks good!</div>
              <div class="invalid-feedback">Please enter your username.</div>
            </div>
            <div>
              <label for="validationCustom02" class="form-label">
                Password
              </label>
              <input
                type="text"
                class="form-control"
                id="validationCustom02"
                required
                value={form().password ?? ""}
                onInput={(e) => setForm((draft) => (draft.password = e.currentTarget.value))}
              />
              <div class="valid-feedback">Looks good!</div>
              <div class="invalid-feedback">Please enter your password.</div>
            </div>
            <Link href="/register">Register</Link>
            <button class="btn btn-primary" type="submit" onClick={submit}>
              Login
            </button>
          </div>
        </form>
      </div>
    </div>
  )
}

export default Login
```

### 编写首页布局

在`src`目录下新建一个`layouts`目录,之后在里面创建一个`HomeLayout.tsx`文件,这就是首页的布局,这个布局包含了一个导航栏

```tsx
import { Outlet } from "@solidjs/router"
import Nav from "../components/Nav"

const HomeLayout = () => {
  return (
    <div>
      <div class="container">
        <Nav />
      </div>
      <div class="container">
        {/* 这里使用了Outlet组件,这个组件用于渲染子路由 */}
        <Outlet />
      </div>
    </div>
  )
}

export default HomeLayout
```

然后在`src`目录下新建一个`components`目录,在其中新建一个`Nav.tsx`文件,这个文件用于渲染导航栏,`Logo`可以从`SolidJS`的官网下载,然后放到`assets`目录下,直接把官网上的扒下来就行

```tsx
import { Link } from "@solidjs/router"
import Logo from "../assets/solid.svg"

const Nav = () => {
  return (
    <div>
      <nav class="navbar navbar-expand-lg navbar-light">
        <img src={Logo} alt="" width="30" height="24" class="d-inline-block align-text-top" />
        <span class="navbar-brand mb-0 h1">Solid</span>
        <ul class="navbar-nav me-auto mb-2 mb-lg-0">
          <li class="nav-item">
            <Link class="nav-link" href="/">
              Posts
            </Link>
          </li>
          <li class="nav-item">
            <Link class="nav-link" href="/category">
              Category
            </Link>
          </li>
          <li class="nav-item">
            <Link class="nav-link" href="/write">
              Write
            </Link>
          </li>
        </ul>
      </nav>
    </div>
  )
}

export default Nav
```

之后分别在`views`目录中新建`Posts.tsx`,`Category.tsx`,`Write.tsx`文件,这些文件用于渲染文章列表,分类列表,写文章页面

```tsx
const Posts = () => {
  return <div>Post</div>
}
export default Posts
```

```tsx
const Category = () => {
  return <div>Category</div>
}

export default Category
```

```tsx
const Write = () => {
  return () => <div>Write</div>
}

export default Write
```

然后我们需要在`router`目录中编写路由数组

```tsx
const routes: RouteDefinition[] = [
  {
    path: "/login",
    component: () => <Login />
  },
  {
    path: "/register",
    component: () => <Register />
  },
  {
    path: "/",
    component: () => <HomeLayout />,
    children: [
      {
        path: "/",
        component: () => <Posts />
      },
      {
        path: "/category",
        component: () => <Category />
      },
      {
        path: "/write",
        component: () => <Write />
      }
    ]
  }
]
```

### 编写发布文章页面

在编写页面之前,需要将`TanStack Query`的`QueryClientProvider`组件放到`App.tsx`中

```tsx
import type { Component } from "solid-js"
import { Router, useRoutes } from "@solidjs/router"
import routes from "./router"
import { Toaster } from "solid-toast"
import { QueryClient, QueryClientProvider } from "@tanstack/solid-query"

const App: Component = () => {
  const queryClient = new QueryClient()
  const Routes = useRoutes(routes)

  return (
    <>
      <Toaster />
      {/* 这里使用了QueryClientProvider组件,这个组件用于提供QueryClient,这个组件必须放在Router组件的外面 */}
      <QueryClientProvider client={queryClient}>
        <Router>
          <Routes />
        </Router>
      </QueryClientProvider>
    </>
  )
}

export default App
```

首先需要明确一点,`TanStack Query`在`SolidJS`中的响应并不支持解构

不支持这样

```ts
const { isLoading, error, data } = useQuery(...)
```

只能这样

```ts
const query = createQuery(...)
```

然后在`Write.tsx`中编写发布文章页面

在开始之前,需要向`Category`中插入 3 条数据

```sql
insert into category(id, name)
values (uuid(), 'Category 1');
insert into category(id, name)
values (uuid(), 'Category 2');
insert into category(id, name)
values (uuid(), 'Category 3');
```

发帖页面的代码如下

```tsx
import { createQuery } from "@tanstack/solid-query"
import { api } from "../common/ApiInstance"
import { PostInput } from "../__generated/model/static"
import { createImmerSignal } from "solid-immer"
import { For, Match, Switch } from "solid-js"
import { useNavigate } from "@solidjs/router"
import toast from "solid-toast"

const Write = () => {
  const navigate = useNavigate()

  const [form, setForm] = createImmerSignal<PostInput>({}) //这个函数用于创建一个信号,这个信号用于存储表单数据

  let formRef

  const categories = createQuery({
    queryKey: () => ["category"],
    queryFn: () => api.categoryController.findCategories()
  })

  const submit = (e) => {
    e.preventDefault()
    e.stopPropagation()

    if (formRef.checkValidity()) {
      api.postController
        .createPost({ body: form() })
        .then(() => {
          toast.success("Publish success")
          navigate("/")
        })
        .catch((err) => {
          toast.error(err)
        })
    }

    formRef.classList.add("was-validated")
  }

  return () => (
    <div>
      <form ref={formRef} class="needs-validation" novalidate>
        <div>
          <label class="form-label">Title</label>
          <input
            type="text"
            class="form-control"
            required
            onInput={(e) => setForm((draft) => (draft.title = e.currentTarget.value))}
          />
          <div class="valid-feedback">Looks good!</div>
          <div class="invalid-feedback">Please enter title.</div>
        </div>
        <div>
          <label class="form-label">Content</label>
          <textarea
            class="form-control"
            style={{ height: "16rem" }}
            required
            onInput={(e) => setForm((draft) => (draft.content = e.currentTarget.value))}
          />
          <div class="valid-feedback">Looks good!</div>
          <div class="invalid-feedback">Please enter content.</div>
        </div>
        <div>
          <label class="form-label">Category</label>
          {/* 这里使用了Switch组件,这个组件用于根据条件渲染不同的内容 */}
          <Switch>
            {/* 这里使用了Match组件,这个组件用于匹配条件,如果条件为true,那么就渲染加载 */}
            <Match when={categories.isLoading}>
              <div class="spinner-border" role="status">
                <span class="visually-hidden">Loading...</span>
              </div>
            </Match>
            <Match when={categories.isError}>Error</Match>
            <Match when={categories.isSuccess}>
              <select
                class="form-select"
                value={form().categoryId}
                onInput={(e) =>
                  setForm((draft) => {
                    draft.categoryId = e.currentTarget.value
                  })
                }
                required
              >
                <For each={categories.data}>{(item) => <option value={item.id}>{item.name}</option>}</For>
              </select>
            </Match>
          </Switch>

          <div class="valid-feedback">Looks good!</div>
          <div class="invalid-feedback">Please select category.</div>
        </div>
        <button class="btn btn-primary" type="submit" onClick={submit}>
          Publish
        </button>
      </form>
    </div>
  )
}

export default Write
```

### 编写文章详情页面

在`views`目录下创建`Post.tsx`文件

```tsx
import { useParams } from "@solidjs/router"
import { createQuery } from "@tanstack/solid-query"
import { api } from "../common/ApiInstance"
import { Match, Switch } from "solid-js"

const Post = () => {
  //这里使用了useParams函数,这个函数用于获取路由参数
  const params = useParams()

  const post = createQuery({
    queryKey: () => ["post", params.id],
    queryFn: () => api.postController.findPost({ id: params.id })
  })

  return (
    <div>
      <Switch>
        <Match when={post.isLoading}>
          <div class="spinner-border" role="status">
            <span class="visually-hidden">Loading...</span>
          </div>
        </Match>
        <Match when={post.isError}>Error</Match>
        <Match when={post.isSuccess}>
          <div>{post.data.title}</div>
          <div>{post.data.category.name}</div>
          <div>{post.data.content}</div>
        </Match>
      </Switch>
    </div>
  )
}

export default Post
```

之后在`router`目录中添加一个路由,布局是`HomeLayout`,路径是`/post/:id`,组件是`Post`

```tsx
{
  path: "/post/:id",
  component: () => <Post />
}
```

### 编写文章列表页面

在`Posts.tsx`中编写文章列表页面,这里使用了`RequestOf`,这个类型用于获取请求的参数类型,这个类型是`Jimmer`自动生成的,比如`RequestOf<typeof api.postController.findPosts>`的类型就是`{ page?: number; size?: number; }`

```tsx
import { createImmerSignal } from "solid-immer"
import { RequestOf } from "../__generated"
import { api } from "../common/ApiInstance"
import { createQuery } from "@tanstack/solid-query"
import { For, Match, Switch } from "solid-js"
import { Link } from "@solidjs/router"

const Posts = () => {
  const [options, setOptions] = createImmerSignal<RequestOf<typeof api.postController.findPosts>>({})

  const posts = createQuery({
    queryKey: () => ["posts", options()],
    queryFn: () => api.postController.findPosts(options())
  })

  return (
    <>
      <Switch>
        <Match when={posts.isLoading}>
          <div class="spinner-border" role="status">
            <span class="visually-hidden">Loading...</span>
          </div>
        </Match>
        <Match when={posts.isError}>Error</Match>
        <Match when={posts.isSuccess}>
          <ul class="list-group">
            <For each={posts.data.content}>
              {(post) => (
                <li class="list-group-item">
                  {/* 这里使用了Link组件,这个组件用于跳转路由,这里使用了post.id作为参数 */}
                  <Link href={`/post/${post.id}`}>{post.title}</Link>
                </li>
              )}
            </For>
          </ul>
          {/* 分页 */}
          <nav aria-label="Page navigation example">
            <ul class="pagination">
              <li class="page-item">
                <div
                  class="page-link"
                  onClick={() => {
                    if (options().page ?? 0 > 0)
                      setOptions((draft) => {
                        draft.page = (options().page ?? 0) - 1
                      })
                  }}
                >
                  Previous
                </div>
              </li>
              {/* 这里使用了For组件,一共有几页就渲染几页,这里使用了Array.from方法,这个方法用于将一个数字转换为数组,比如Array.from({length: 3})的结果就是[1,2,3] */}
              <For each={Array.from({ length: posts.data.totalPages }, (_, i) => i + 1)}>
                {(page, index) => (
                  <li class="page-item">
                    <div
                      class="page-link"
                      classList={{ active: (options().page ?? 0) === index() }}
                      onClick={() =>
                        setOptions((draft) => {
                          draft.page = index()
                        })
                      }
                    >
                      {page}
                    </div>
                  </li>
                )}
              </For>
              <li class="page-item">
                <div
                  class="page-link"
                  onClick={() => {
                    if ((options().page ?? 0) < posts.data.totalPages - 1)
                      setOptions((draft) => {
                        draft.page = (options().page ?? 0) + 1
                      })
                  }}
                >
                  Next
                </div>
              </li>
            </ul>
          </nav>
        </Match>
      </Switch>
    </>
  )
}
export default Posts
```

### 编写分类列表页面

在`Category.tsx`中编写分类列表页面,这里为了方便,我直接将`Posts.tsx`中的代码复制过来了,然后修改了一下请求参数

```tsx
import { createImmerSignal } from "solid-immer"
import { RequestOf } from "../__generated"
import { createQuery } from "@tanstack/solid-query"
import { api } from "../common/ApiInstance"
import { Link, useParams } from "@solidjs/router"
import { Switch, Match, For } from "solid-js"

const CategoryPosts = () => {
  const params = useParams()

  const [options, setOptions] = createImmerSignal<RequestOf<typeof api.postController.findPostsByCategory>>({
    categoryId: params.categoryId
  })

  const posts = createQuery({
    queryKey: () => ["posts", options()],
    queryFn: () => api.postController.findPostsByCategory(options())
  })

  return (
    <Switch>
      <Match when={posts.isLoading}>
        <div class="spinner-border" role="status">
          <span class="visually-hidden">Loading...</span>
        </div>
      </Match>
      <Match when={posts.isError}>Error</Match>
      <Match when={posts.isSuccess}>
        <ul class="list-group">
          <For each={posts.data.content}>
            {(post) => (
              <li class="list-group-item">
                <Link href={`/post/${post.id}`}>{post.title}</Link>
              </li>
            )}
          </For>
        </ul>
        <nav aria-label="Page navigation example">
          <ul class="pagination">
            <li class="page-item">
              <div
                class="page-link"
                onClick={() => {
                  if (options().page ?? 0 > 0)
                    setOptions((draft) => {
                      draft.page = (options().page ?? 0) - 1
                    })
                }}
              >
                Previous
              </div>
            </li>
            <For each={Array.from({ length: posts.data.totalPages }, (_, i) => i + 1)}>
              {(page, index) => (
                <li class="page-item">
                  <div
                    class="page-link"
                    classList={{ active: (options().page ?? 0) === index() }}
                    onClick={() =>
                      setOptions((draft) => {
                        draft.page = index()
                      })
                    }
                  >
                    {page}
                  </div>
                </li>
              )}
            </For>
            <li class="page-item">
              <div
                class="page-link"
                onClick={() => {
                  if ((options().page ?? 0) < posts.data.totalPages - 1)
                    setOptions((draft) => {
                      draft.page = (options().page ?? 0) + 1
                    })
                }}
              >
                Next
              </div>
            </li>
          </ul>
        </nav>
      </Match>
    </Switch>
  )
}

export default CategoryPosts
```

之后在`router`目录中添加一个路由,布局是`HomeLayout`,路径是`/category/:categoryId`,组件是`CategoryPosts`

```tsx
{
  path: "/category/:categoryId",
  component: () => <CategoryPosts />
}
```

### 编写评论列表组件

在`components`目录下新建一个`Replys.tsx`文件,这个文件用于渲染评论列表

```tsx
import { createQuery } from "@tanstack/solid-query"
import { createImmerSignal } from "solid-immer"
import { Component, For, Match, Switch } from "solid-js"
import { RequestOf } from "../__generated"
import { api } from "../common/ApiInstance"
import { ReplyInput } from "../__generated/model/static"
import toast from "solid-toast"

const Replys: Component<{ postId: string }> = ({ postId }) => {
  const [options, setOptions] = createImmerSignal<RequestOf<typeof api.replyController.findRepliesByPost>>({ postId })

  const replies = createQuery({
    queryKey: () => ["replies", options()],
    queryFn: () => api.replyController.findRepliesByPost(options())
  })

  let formRef

  const [form, setForm] = createImmerSignal<ReplyInput>({ postId })

  const submit = (e) => {
    e.preventDefault()
    e.stopPropagation()

    if (formRef.checkValidity()) {
      api.replyController
        .createReply({
          body: form()
        })
        .then(() => {
          toast.success("Reply created")
        })
        .catch(() => {
          toast.error("Reply creation failed")
        })
    }

    formRef.classList.add("was-validated")
  }

  return (
    <div>
      <div class="card">
        <form ref={formRef} class="needs-validation">
          <div>
            <label class="form-label">Content</label>
            <textarea
              class="form-control"
              value={form().content ?? ""}
              required
              onInput={(e) => setForm((draft) => (draft.content = e.currentTarget.value))}
            />
            <div class="valid-feedback">Looks good!</div>
            <div class="invalid-feedback">Please enter your reply.</div>
          </div>
          <button class="btn btn-primary" type="submit" onClick={submit}>
            Reply
          </button>
        </form>
      </div>
      <Switch>
        <Match when={replies.isLoading}>
          <div class="spinner-border" role="status">
            <span class="visually-hidden">Loading...</span>
          </div>
        </Match>
        <Match when={replies.isError}>Error</Match>
        <Match when={replies.isSuccess}>
          <ul class="list-group">
            <For each={replies.data.content}>
              {(reply) => (
                <li class="list-group-item">
                  <div class="d-flex justify-content-between">
                    <div>User:{reply.user.username}</div>
                    <div>{reply.content}</div>
                  </div>
                </li>
              )}
            </For>
          </ul>
          <nav aria-label="Page navigation example">
            <ul class="pagination">
              <li class="page-item">
                <div
                  class="page-link"
                  onClick={() => {
                    if (options().page ?? 0 > 0)
                      setOptions((draft) => {
                        draft.page = (options().page ?? 0) - 1
                      })
                  }}
                >
                  Previous
                </div>
              </li>
              <For each={Array.from({ length: replies.data.totalPages }, (_, i) => i + 1)}>
                {(page, index) => (
                  <li class="page-item">
                    <div
                      class="page-link"
                      classList={{ active: (options().page ?? 0) === index() }}
                      onClick={() =>
                        setOptions((draft) => {
                          draft.page = index()
                        })
                      }
                    >
                      {page}
                    </div>
                  </li>
                )}
              </For>
              <li class="page-item">
                <div
                  class="page-link"
                  onClick={() => {
                    if ((options().page ?? 0) < replies.data.totalPages - 1)
                      setOptions((draft) => {
                        draft.page = (options().page ?? 0) + 1
                      })
                  }}
                >
                  Next
                </div>
              </li>
            </ul>
          </nav>
        </Match>
      </Switch>
    </div>
  )
}

export default Replys
```

最后将这个组件放到`Post.tsx`中

```tsx
<Replys postId={params.id} />
```

好了,到这里我们的博客就完成了,如果你想要看完整的代码,可以去[GitHub](https://github.com/Enaium-Learn/solid-blog)上看