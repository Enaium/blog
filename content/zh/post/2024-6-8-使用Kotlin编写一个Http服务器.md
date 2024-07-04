---
layout: post
title: "使用Kotlin编写一个Http服务器"
date: 2024-06-08T20:04:11+08:00
---

## 引言

在本文中，我们将使用 Kotlin 编写一个简单的 HTTP 服务器。我们将使用 Java 的 `ServerSocket` 类来实现这个服务器。我们将创建一个简单的服务器，它将监听端口 8000，并在接收到请求时返回一个简单的响应。

## Http 的格式

HTTP 请求和响应都是文本格式的。HTTP 请求由请求行、请求头和请求体组成。HTTP 响应由状态行、响应头和响应体组成。

具体可以到 [MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Messages) 查看。

## 代码实现

首先我们需要创建一个`Method`枚举和一个`Version`枚举，用于表示请求的方法和版本。

```kotlin
enum class Method {
    GET,
    POST,
    UNKNOWN;

    companion object {
        fun parse(method: String): Method =
            when (method) {
                "GET" -> GET
                "POST" -> POST
                else -> UNKNOWN
            }
    }
}
```

```kotlin
enum class Version {
    HTTP_1_1,
    UNKNOWN;

    companion object {
        fun parse(version: String): Version = when (version) {
            "HTTP/1.1" -> HTTP_1_1
            else -> UNKNOWN
        }
    }

    override fun toString(): String {
        return when (this) {
            HTTP_1_1 -> "HTTP/1.1"
            UNKNOWN -> "UNKNOWN"
        }
    }
}
```

然后我们创建一个`HttpRequest`类，用于表示 HTTP 请求。

```kotlin
data class HttpRequest(
    val method: Method,
    val path: String,
    val version: Version,
    val headers: Map<String, String>,
    val body: String
)
```

接着我们为`HttpRequest`类添加一个静态方法`parse`，用于解析 HTTP 请求。

```kotlin
companion object {
    fun parse(reader: BufferedReader): HttpRequest {
    }
}
```

在`parse`方法中，我们首先读取请求行，然后读取请求头，最后读取请求体。

首先我们先来定义一些默认值。

```kotlin
var method = Method.UNKNOWN
var path = ""
var version = Version.UNKNOWN
val headers = mutableMapOf<String, String>()
var body = ""
```

然后我们读取请求的所有数据，不过需要注意这里不不能使用`readText`和`readLines`，这些方法会导致`Socket`被关闭。

```kotlin
val request = StringBuilder()
var readed: String?
while (reader.readLine().also { readed = it } != null && readed!!.isNotEmpty()) {
    request.append(readed).append("\n")
}
```

接着我们解析请求的数据。

```kotlin
request.lines().forEachIndexed { index, line ->
    if (index == 0) {
        val splitWithSpace = line.split(" ")
        method = Method.parse(splitWithSpace[0])
        path = splitWithSpace[1]
        version = Version.parse(splitWithSpace[2])
    } else if (line.contains(": ")) {
        val split = line.split(": ")
        headers[split[0]] = split[1]
    } else if (line.isEmpty()) {
    } else {
        body = line
    }
}
```

首先是第一行，我们使用空格分割，然后解析请求方法、路径和版本。然后是请求头，我们使用冒号空格分割，然后解析请求头。最后是请求体，如果不为空，我们就保存请求体。

最后我们将解析的数据返回。

```kotlin
return HttpRequest(
    method,
    path,
    version,
    headers,
    body
)
```

接着我们创建一个`HttpResponse`类，用于表示 HTTP 响应。

```kotlin
data class HttpResponse(
    val version: Version = Version.HTTP_1_1,
    val statusCode: Int,
    val statusText: String,
    val headers: Map<String, String> = emptyMap(),
    val body: String = ""
)
```

然后我们为`HttpResponse`类重写`toString`方法，用于将响应转换为字符串。

```kotlin
override fun toString(): String {
    return """
        $version $statusCode $statusText
        ${headers.map { "${it.key}: ${it.value}" }.joinToString("\n")}
        
        $body
    """.trimIndent()
}
```

接着我们创建一个`Handler`类型，用于处理请求。

```kotlin
typealias Handler = (HttpRequest) -> HttpResponse
```

然后我们创建一个`Route`类，用于表示路由。

```kotlin
data class Route(val method: Method, val path: String, val handler: Handler)
```

接着创建一个`Router`类，用于管理路由。

```kotlin
class Router {
    private val routes = mutableListOf<Route>()

    fun get(path: String, handler: Handler) {
        routes.add(Route(Method.GET, path, handler))
    }

    fun handle(socket: ServerSocket) {

    }
}
```

在`Router`类中，我们定义了一个`routes`属性，用于保存所有的路由。然后我们定义了一个`get`方法，用于添加一个 GET 请求的路由。最后我们定义了一个`handle`方法，用于处理请求。

接着我们需要实现`handle`方法。

```kotlin
fun handle(socket: ServerSocket) {
    while (true) {
        val client = socket.accept()
        val reader = client.getInputStream().bufferedReader()
        val writer = client.getOutputStream().bufferedWriter()
        val httpRequest = HttpRequest.parse(reader)
        routes.findLast { it.method == httpRequest.method && it.path == httpRequest.path }?.let {
            val toString = it.handler.invoke(httpRequest).toString()
            writer.write(toString)
            writer.flush()
        } ?: let {
            writer.write(
                HttpResponse(
                    Version.HTTP_1_1,
                    404,
                    "NotFound",
                    headers = mapOf("Content-Type" to "text/html"),
                    body = "<h1>404 Not Found</h1>"
                ).toString()
            )
            writer.flush()
        }
        client.close()
    }
}
```

在`handle`方法中，我们首先创建一个`ServerSocket`，然后进入一个无限循环。在循环中，我们首先接受一个客户端连接，然后创建一个`BufferedReader`和一个`BufferedWriter`，用于读取请求和写入响应。然后我们解析请求，然后查找路由，如果找到了路由，我们就调用路由的处理函数，然后将响应写入到客户端。如果没有找到路由，我们就返回一个 404 响应。最后我们关闭客户端连接。

最后我们创建一个`Server`类，用于创建服务器。

```kotlin
class HttpServer(port: Int) {
    private val serverSocket: ServerSocket = ServerSocket(port)

    fun start() {

    }
}
```

在`start`方法中，我们创建一个`Router`，然后添加一个路由，最后调用`Router`的`handle`方法。

```kotlin
val router = Router()
router.get("/") { _ ->
    HttpResponse(Version.HTTP_1_1, 200, "OK")
}
router.get("/hello") { _ ->
    HttpResponse(
        Version.HTTP_1_1, 200, "OK",
        headers = mapOf("Content-Type" to "text/html"),
        body = "<h1>Hello World!</h1>"
    )
}
router.handle(serverSocket)
```

这样我们就完成了一个简单的 HTTP 服务器。

现在来测试一下我们的服务器。

```kotlin
fun main() {
    HttpServer(8080).start()
}
```

在终端中运行`main`函数，然后在浏览器中打开`http://localhost:8080`和`http://localhost:8080/hello`，你应该能看到一个简单的页面。

## 总结

在本文中，我们使用 Kotlin 编写了一个简单的 HTTP 服务器。我们使用 Java 的 `ServerSocket` 类来实现这个服务器。我们创建了一个简单的服务器，它监听端口 8080，并在接收到请求时返回一个简单的响应。我们还创建了一个简单的路由系统，用于处理不同的请求。

[完整源码](https://github.com/Enaium/teaching-kotlin-httpserver)