---
layout: post
title: "使用Kotlin进行全栈开发 Ktor+Kotlin/JS"
date: 2024-04-13T21:50:28+08:00
---

## 前言

本文将介绍如何使用 Kotlin 全栈技术栈`Ktor`+`Kotlin/JS`来构建一个简单的全栈应用。

## 准备工作

### 创建项目

首先我们需要创建一个`Kotlin`项目，之后继续在其中新建两个子项目，一个是`Kotlin/JS`项目，另一个是`Ktor`项目。

### 添加依赖和插件

这里我使用了`Gradle`的`calalog`，在项目中的`gradle`目录下创建一个`libs.versions.toml`文件，用于管理项目中的依赖版本。

```toml
[versions]
jimmer = "0.0.9"
kotlin = "1.9.23"
ktor = "2.3.9"
ksp = "1.9.23-1.0.20"
coroutines = "1.8.0"
serialization = "1.6.3"
wrappers = "1.0.0-pre.729"
logback = "1.5.3"
postgresql = "42.7.3"
hikari = "5.1.0"
koin = "3.5.6"

[libraries]
ktor-server-core = { module = "io.ktor:ktor-server-core-jvm", version.ref = "ktor" }
ktor-server-netty = { module = "io.ktor:ktor-server-netty-jvm", version.ref = "ktor" }
ktor-server-cors = { module = "io.ktor:ktor-server-cors", version.ref = "ktor" }
ktor-server-content-negotiation = { module = "io.ktor:ktor-server-content-negotiation", version.ref = "ktor" }
ktor-serialization-jsackson = { module = "io.ktor:ktor-serialization-jackson", version.ref = "ktor" }
ktor-server-config-yaml = { module = "io.ktor:ktor-server-config-yaml", version.ref = "ktor" }
logback = { module = "ch.qos.logback:logback-classic", version.ref = "logback" }
kotlinx-coroutines-core = { module = "org.jetbrains.kotlinx:kotlinx-coroutines-core", version.ref = "coroutines" }
kotlinx-serialization-json = { module = "org.jetbrains.kotlinx:kotlinx-serialization-json", version.ref = "serialization" }
kotlin-wrappers = { module = "org.jetbrains.kotlin-wrappers:kotlin-wrappers-bom", version.ref = "wrappers" }
kotlin-wrappers-react = { module = "org.jetbrains.kotlin-wrappers:kotlin-react" }
kotlin-wrappers-react-dom = { module = "org.jetbrains.kotlin-wrappers:kotlin-react-dom" }
kotlin-wrappers-emotion = { module = "org.jetbrains.kotlin-wrappers:kotlin-emotion" }
postgresql = { module = "org.postgresql:postgresql", version.ref = "postgresql" }
hikari = { module = "com.zaxxer:HikariCP", version.ref = "hikari" }
koin = { module = "io.insert-koin:koin-ktor", version.ref = "koin" }

[bundles]
api = ['ktor-server-core', 'ktor-server-netty', 'ktor-server-cors', 'ktor-server-content-negotiation', 'ktor-serialization-jsackson', 'ktor-server-config-yaml', 'logback', 'postgresql', 'hikari', 'koin']
app = ['kotlinx-coroutines-core', 'kotlinx-serialization-json', 'kotlin-wrappers-react', 'kotlin-wrappers-react-dom', 'kotlin-wrappers-emotion']

[plugins]
jimmer = { id = "cn.enaium.jimmer.gradle", version.ref = "jimmer" }
kotlin-jvm = { id = "org.jetbrains.kotlin.jvm", version.ref = "kotlin" }
kotlin-multiplatform = { id = "org.jetbrains.kotlin.multiplatform", version.ref = "kotlin" }
ktor = { id = "io.ktor.plugin", version.ref = "ktor" }
ksp = { id = "com.google.devtools.ksp", version.ref = "ksp" }
kotlin-plugin-serialization = { id = "org.jetbrains.kotlin.plugin.serialization", version.ref = "kotlin" }
```

之后我们分别在前端和后端项目中的`build.gradle.kts`文件中引入这些依赖和插件。

#### 后端

```kotlin
plugins {
    alias(libs.plugins.kotlin.jvm)
    alias(libs.plugins.ktor)
    alias(libs.plugins.ksp)
    alias(libs.plugins.jimmer)
    application
}

group = "cn.enaium"
version = "1.0.0"

application {
    mainClass = "cn.enaium.TodoKt"
    applicationDefaultJvmArgs = listOf("-Dio.ktor.development=${extra["development"] ?: "false"}")
}

dependencies {
    implementation(libs.bundles.api)
}
```

这里有一个配置，添加到`gradle.properties`文件中。

```properties
development=true
```

#### 前端

```kotlin
plugins {
    alias(libs.plugins.kotlin.multiplatform)
    alias(libs.plugins.kotlin.plugin.serialization)
}

kotlin {
    js {
        browser {
            commonWebpackConfig {
                cssSupport {
                    enabled.set(true)
                }
            }
        }
        binaries.executable()
    }

    sourceSets {
        val jsMain by getting {
            dependencies {
                implementation(project.dependencies.enforcedPlatform(libs.kotlin.wrappers))
                implementation(libs.bundles.app)
            }
        }
    }
}
```

这里需要将前端项目的`src/main`改为`src/jsMain`。

最后进入到根项目的`settings.gradle.kts`文件中添加以下代码。

```kotlin
pluginManagement {
    repositories {
        maven("https://maven.pkg.jetbrains.space/public/p/compose/dev")
        google()
        gradlePluginPortal()
        mavenCentral()
    }
}

dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
        maven("https://maven.pkg.jetbrains.space/public/p/compose/dev")
    }
}
```

还有`gradle.build.kts`文件中只保留以下代码。

```kotlin
plugins {
    alias(libs.plugins.kotlin.jvm) apply false
    alias(libs.plugins.kotlin.multiplatform) apply false
}
```

好了，现在我们的项目已经准备好了。

## 编写代码

### 后端

首先创建配置文件`src/main/resources/application.yml`。

```yaml
ktor:
  deployment:
    port: 8080
  application:
    modules:
      - cn.enaium.TodoKt.module
jdbc:
  driver: 'org.postgresql.Driver'
  url: 'jdbc:postgresql://localhost:5432/postgres?currentSchema=todo'
  username: 'postgres'
  password: 'postgres'
```

之后创建`logback`配置文件`src/main/resources/logback.xml`。

```xml
<configuration>
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{YYYY-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    <root level="trace">
        <appender-ref ref="STDOUT"/>
    </root>
    <logger name="org.eclipse.jetty" level="INFO"/>
    <logger name="io.netty" level="INFO"/>
</configuration>
```

还有创建数据库。

```sql
drop schema if exists todo cascade;
create schema todo;

drop table if exists todo.task;
create table todo.task
(
    id         uuid primary key,
    name       text not null,
    start_time timestamp default now(),
    end_time   timestamp
)
```

之后创建一个主类`cn.enaium.Todo`。

```kotlin
fun main(args: Array<String>) = EngineMain.main(args)
```

之后编写一个扩展函数`cn.enaium.Todo.module`。

```kotlin
fun Application.module() {

}
```

安装一些插件

#### Koin

```kotlin
install(Koin) {
    modules(module {
        single<ApplicationEnvironment> { environment }
    })
}
```

#### CORS

```kotlin
install(CORS) {
    allowMethod(HttpMethod.Options)
    allowMethod(HttpMethod.Post)
    allowMethod(HttpMethod.Get)
    allowHeader(HttpHeaders.AccessControlAllowOrigin)
    allowHeader(HttpHeaders.ContentType)
    anyHost()
}
```

#### Jackson

```kotlin
install(ContentNegotiation) {
    jackson {
        registerModules(ImmutableModule())
    }
}
```

#### Jimmer

接下来配置一下`Jimmer`。

```kotlin
fun sql(environment: ApplicationEnvironment): KSqlClient {
    return newKSqlClient {
        setConnectionManager {
            HikariPool(HikariConfig().apply {
                driverClassName = environment.config.property("jdbc.driver").getString()
                jdbcUrl = environment.config.property("jdbc.url").getString()
                username = environment.config.property("jdbc.username").getString()
                password = environment.config.property("jdbc.password").getString()
                maximumPoolSize = 10
                connectionTimeout = 30000
            }).connection.use {
                proceed(it)
            }
        }
        setDialect(PostgresDialect())
    }
}
```

之后添加到`Koin`中。

```kotlin
single<KSqlClient> { sql(get()) }
```

编写一个`Task`实体类。

```kotlin
package cn.enaium.entity

import org.babyfish.jimmer.sql.Entity
import org.babyfish.jimmer.sql.GeneratedValue
import org.babyfish.jimmer.sql.Id
import org.babyfish.jimmer.sql.Table
import org.babyfish.jimmer.sql.meta.UUIDIdGenerator
import java.util.*

/**
 * @author Enaium
 */
@Entity
@Table(name = "task")
interface Task {
    @Id
    @GeneratedValue(generatorType = UUIDIdGenerator::class)
    val id: UUID

    val name: String

    val startTime: Date

    val endTime: Date?
}
```

接下来就可以编写`Service`了。

```kotlin
package cn.enaium.service

import cn.enaium.entity.Task
import cn.enaium.entity.endTime
import cn.enaium.entity.startTime
import org.babyfish.jimmer.sql.kt.KSqlClient
import org.babyfish.jimmer.sql.kt.ast.expression.isNotNull

/**
 * @author Enaium
 */
class TodoServe(private val sql: KSqlClient) {
    fun getTasks(): List<Task> {
        return sql.createQuery(Task::class) {
            orderBy(table.endTime.isNotNull(), table.startTime)
            select(table)
        }.execute()
    }

    fun saveTask(task: Task) {
        sql.save(task)
    }
}
```

这里我们添加两个方法`getTasks`和`saveTask`，`getTasks`用于获取所有任务并按照创建时间和是否完成排序，`saveTask`用于保存任务，之后还是添加到`Koin`中。

```kotlin
single<TodoServe> { TodoServe(get()) }
```

之后我们在`module`添加路由。

```kotlin
val todoServe by inject<TodoServe>()

routing {
    get("/task") {
        call.respond(todoServe.getTasks())
    }
    post("/task") {
        todoServe.saveTask(call.receive())
        call.response.status(HttpStatusCode.OK)
    }
}
```

### 前端

首先在`src/jsMain/resources/index.html`中添加以下代码，这里需要注意的是`app.js`，这个文件名称需要和前端的项目名称一致。

```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Hello, Kotlin/JS!</title>
</head>
<body>
<div id="root"></div>
<script src="app.js"></script>
</body>
</html>
```

之后写一个`main`函数。

```kotlin
import react.dom.client.createRoot
import web.dom.document

/**
 * @author Enaium
 */
fun main() {
    val container = document.getElementById("root") ?: error("Couldn't find root container!")
    createRoot(container).render(App.create())
}

val App = FC {
    
}
```

然后就可以编写组件了。

首先需要创建两个`data`类，一个是`Task`，另一个是`TaskInput`，`Task`用于展示任务，`TaskInput`用于请求。

```kotlin
@Serializable
data class Task(val id: String, var name: String, val startTime: Long, val endTime: Long?) {
    fun copy(name: String = this.name, startTime: Long = this.startTime, endTime: Long? = this.endTime) =
        Task(id, name, startTime, endTime)

    fun toInput() = TaskInput(id, name, startTime, endTime)
}

@Serializable
data class TaskInput(
    val id: String? = null,
    val name: String? = null,
    val startTime: Long? = null,
    val endTime: Long? = null
)
```

之后编写请求函数，使用`fetch`发送请求。

```kotlin
val coroutine = CoroutineScope(window.asCoroutineDispatcher())

suspend fun fetchTasks(): List<Task> {
    window.fetch("http://localhost:8080/task").await().let {
        if (it.status != 200.toShort()) {
            throw Exception("Failed to fetch")
        }
        return Json.decodeFromDynamic<List<Task>>(it.json().await())
    }
}

suspend fun saveTask(task: TaskInput) {
    window.fetch(
        "http://localhost:8080/task",
        RequestInit(
            method = "POST",
            body = Json.encodeToString(TaskInput.serializer(), task),
            headers = json("Content-Type" to "application/json")
        )
    ).await().let {
        if (it.status != 200.toShort()) {
            throw Exception("Failed to save")
        }
    }
}
```

#### TaskItem

编写一个`TaskItem`组件，用于展示任务，编辑任务，完成任务，逻辑就是点击`Edit`按钮可以编辑任务，按`Enter`保存，按`Escape`取消，点击`Finish`按钮完成任务。

```kotlin
external interface TaskItemProps : Props {
    var task: Task
}

val TaskItem = FC<TaskItemProps> { props ->
    var editState by useState(false)

    var taskState by useState<TaskInput>()

    useEffect(listOf(taskState)) {
        taskState?.let {
            coroutine.launch {
                saveTask(it)
                window.location.reload()
            }
        }
    }

    div {
        if (editState) {
            input {
                defaultValue = props.task.name
                onKeyUp = {
                    if (it.asDynamic().key == "Enter") {
                        taskState = props.task.copy(name = it.target.asDynamic().value as String).toInput()
                        editState = false
                    }

                    if (it.asDynamic().key == "Escape") {
                        editState = false
                    }
                }
            }
        } else {
            div {
                css {
                    color = if (props.task.endTime == null) Color("red") else Color("green")
                }
                div {
                    +props.task.id
                }
                div {
                    +props.task.name
                }
                div {
                    +kotlin.js.Date(props.task.startTime).toLocaleString()
                    props.task.endTime?.let {
                        +" - "
                        +kotlin.js.Date(it).toLocaleString()
                    }
                }
            }

            button {
                +"Edit"
                onClick = {
                    editState = !editState
                }
            }
            button {
                +"Finish"
                onClick = {
                    taskState = props.task.copy(endTime = Date().getTime().toLong()).toInput()
                }
            }
        }
    }
}
```

#### App

最后编写`App`组件，获取任务列表，添加任务。
```kotlin
val App = FC {
    var tasksState by useState(emptyList<Task>())
    var taskState by useState<TaskInput>()

    useEffectOnce {
        coroutine.launch {
            tasksState = fetchTasks()
        }
    }

    useEffect(listOf(taskState)) {
        taskState?.let {
            coroutine.launch {
                saveTask(it)
                window.location.reload()
            }
        }
    }

    div {
        input {
            css {
                fontSize = 24.px
            }

            onKeyUp = {
                if (it.asDynamic().key == "Enter") {
                    taskState = TaskInput(name = it.target.asDynamic().value as String)
                }
            }
        }
        div {
            css {
                marginTop = 10.px
                display = Display.flex
                flexDirection = FlexDirection.column
                gap = 10.px
            }
            tasksState.forEach {
                TaskItem {
                    key = it.id
                    task = it
                }
            }
        }
    }
}
```

## 运行

前端和后端默认端口都是`8080`，所以先运行后端，之后运行前端。

后端使用`application`插件的`run`任务，前端使用`jsBrowserDevelopmentRun`任务。