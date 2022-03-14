---
layout: post
title: "Netty自定义协议"
data: 2022-3-14 14:26
categories: java
---

协议就用上篇文章的协议

```java
public class Message implements Serializable {
    private final long order;

    public Message(long order) {
        this.order = order;
    }

    public long getOrder() {
        return order;
    }
}
```

只不过`Message`加了个`Order`熟悉,

创建`Request`类,继承`Message`,`klass`是调用的`Class`目标,`name`,`parameterType`,`argument`分别是方法名称,参数类型,参数

```java
public class Request extends Message {
    private final String klass;
    private final String name;
    private final Class<?>[] parameterType;
    private final Object[] argument;

    public Request(long order, String klass, String name, Class<?>[] parameterType, Object[] argument) {
        super(order);
        this.klass = klass;
        this.name = name;
        this.parameterType = parameterType;
        this.argument = argument;
    }

    public String getKlass() {
        return klass;
    }

    public String getName() {
        return name;
    }

    public Class<?>[] getParameterType() {
        return parameterType;
    }

    public Object[] getArgument() {
        return argument;
    }
}
```

创建`Response`类继承`Message`,`result`调用的结果,`throwable`调用的异常

```java
public class Response extends Message {
    private final Object result;
    private final Throwable throwable;

    public Response(long order, Object result, Throwable throwable) {
        super(order);
        this.result = result;
        this.throwable = throwable;
    }

    public Object getResult() {
        return result;
    }

    public Throwable getThrowable() {
        return throwable;
    }
}
```

创建一个`PRCHandler`类,来处理请求,用反射调用即可

```java
public class PRCHandler extends SimpleChannelInboundHandler<Request> {
    @Override
    public void channelRead0(ChannelHandlerContext channelHandlerContext, Request request) {
        try {
            Class<?> aClass = Class.forName(request.getKlass());
            Object o = aClass.getConstructor().newInstance();
            Object invoke = aClass.getMethod(request.getName(), request.getParameterType()).invoke(o, request.getArgument());
            channelHandlerContext.channel().writeAndFlush(new Response(request.getOrder(), invoke, null));
        } catch (ClassNotFoundException | InvocationTargetException | InstantiationException | IllegalAccessException | NoSuchMethodException e) {
            e.printStackTrace();
            channelHandlerContext.channel().writeAndFlush(new Response(request.getOrder(), null, e.getCause()));
        }
    }
}
```

接着启动服务器,服务器就这样写好了

```java
public class RPCServer {

    private static final LoggingHandler LOGGING_HANDLER = new LoggingHandler(LogLevel.INFO);
    private static final MessageCodec MESSAGE_CODEC = new MessageCodec();
    private static final PRCHandler PRC_HANDLER = new PRCHandler();

    public static void main(String[] args) {
        NioEventLoopGroup boss = new NioEventLoopGroup();
        NioEventLoopGroup worker = new NioEventLoopGroup();
        try {
            Channel localhost = new ServerBootstrap().group(boss, worker).channel(NioServerSocketChannel.class).childHandler(new ChannelInitializer<SocketChannel>() {
                @Override
                public void initChannel(SocketChannel channel) {
                    channel.pipeline().addLast(LOGGING_HANDLER);
                    channel.pipeline().addLast(MESSAGE_CODEC);
                    channel.pipeline().addLast(PRC_HANDLER);
                }
            }).bind("localhost", 3828).sync().channel();
            System.out.println("Runnable...");
            localhost.closeFuture().sync();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            boss.shutdownGracefully();
            worker.shutdownGracefully();
        }
    }
}
```

现在在`test`里测试一下,写好客户端连接,`Hanlder`先不用太关注

```java
public class Main {

    private static final LoggingHandler LOGGING_HANDLER = new LoggingHandler(LogLevel.INFO);
    private static final MessageCodec MESSAGE_CODEC = new MessageCodec();
    private static final Handler HANDLER = new Handler();

    private static Channel channel;

    public static void main(String[] args) {
        NioEventLoopGroup nioEventLoopGroup = new NioEventLoopGroup();
        try {
            channel = new Bootstrap().group(nioEventLoopGroup).channel(NioSocketChannel.class).handler(new ChannelInitializer<SocketChannel>() {
                @Override
                public void initChannel(SocketChannel channel) {
                    channel.pipeline().addLast(LOGGING_HANDLER);
                    channel.pipeline().addLast(MESSAGE_CODEC);
                    channel.pipeline().addLast(HANDLER);
                }
            }).connect("localhost", 3828).sync().channel();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        Runtime.getRuntime().addShutdownHook(new Thread(nioEventLoopGroup::shutdownGracefully));
    }
}
```

创建一个`Call`注解,`klass`是目标类,`name`是目标类的方法

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Call {
    String klass();

    String name();
}
```

现在创建一个目标类

```java
public class Target {
    public String render() {
        return "RENDER HELLO WORLD!";
    }
}
```

创建个一个`Service`接口

```java
public interface Service {
    @Call(klass = "cn.enaium.Target", name = "render")
    String render();
}
```

接着使用动态代理

```java
@SuppressWarnings("unchecked")
private static <T> T getService(Class<T> klass) {

}
```

没有`Call`注解的返回`null`

```java
Object o = Proxy.newProxyInstance(klass.getClassLoader(), new Class<?>[]{klass}, (proxy, method, args) -> {
    if (!method.isAnnotationPresent(Call.class)) {
        return null;
    }
}
```

使用`Promise`来获取结果

```java
Object o = Proxy.newProxyInstance(klass.getClassLoader(), new Class<?>[]{klass}, (proxy, method, args) -> {
    if (!method.isAnnotationPresent(Call.class)) {
        return null;
    }
    Promise<Object> promise = new DefaultPromise<>(channel.eventLoop());
    Call annotation = method.getAnnotation(Call.class);
    long increment = Util.increment();
    channel.writeAndFlush(new Request(increment, annotation.klass(), annotation.name(), method.getParameterTypes(), args));
    Main.HANDLER.getPromiseMap().put(increment, promise);
    promise.await();
    if (promise.cause() != null) {
        return new RuntimeException(promise.cause());
    }
    return promise.getNow();
});
```

```java
@SuppressWarnings("unchecked")
private static <T> T getService(Class<T> klass) {
    Object o = Proxy.newProxyInstance(klass.getClassLoader(), new Class<?>[]{klass}, (proxy, method, args) -> {
        if (!method.isAnnotationPresent(Call.class)) {
            return null;
        }
        Promise<Object> promise = new DefaultPromise<>(channel.eventLoop());
        Call annotation = method.getAnnotation(Call.class);
        long increment = Util.increment();
        channel.writeAndFlush(new Request(increment, annotation.klass(), annotation.name(), method.getParameterTypes(), args));
        Main.HANDLER.getPromiseMap().put(increment, promise);
        promise.await();
        if (promise.cause() != null) {
            return new RuntimeException(promise.cause());
        }
        return promise.getNow();
    });
    return (T) o;
}
```

序号自增

```java
public class Util {
    private static final AtomicLong atomicLong = new AtomicLong();

    public static long increment() {
        return atomicLong.incrementAndGet();
    }
}
```

`Handler`来处理响应,根据请求的`order`获取返回值

```java
public class Handler extends SimpleChannelInboundHandler<Response> {

    private final Map<Long, Promise<Object>> promiseMap = new HashMap<>();

    @Override
    public void channelRead0(ChannelHandlerContext channelHandlerContext, Response response) throws Exception {

        if (null == promiseMap.get(response.getOrder())) {
            return;
        }

        Promise<Object> promise = promiseMap.remove(response.getOrder());

        if (response.getResult() != null) {
            promise.setSuccess(response.getResult());
        } else {
            promise.setFailure(response.getThrowable());
        }
    }

    public Map<Long, Promise<Object>> getPromiseMap() {
        return promiseMap;
    }
}
```

现在来运行服务器和客户端测试一下