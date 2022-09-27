---
title: "Netty Http服务器接收请求"
date: 2021-07-19T15:38:00+0800
categories: java
---

```java
public class Server {
    public static void main(String[] args) {
        EventLoopGroup eventLoopGroup = new NioEventLoopGroup();
        ServerBootstrap serverBootstrap = new ServerBootstrap();
        try {
            serverBootstrap.group(eventLoopGroup).channel(NioServerSocketChannel.class).childHandler(new ChannelInitializer<SocketChannel>() {
                @Override
                protected void initChannel(SocketChannel ch) throws Exception {
                    ch.pipeline().addLast(new HttpRequestDecoder());
                    ch.pipeline().addLast(new HttpResponseEncoder());
                    ch.pipeline().addLast(new HttpObjectAggregator(65535));
                    ch.pipeline().addLast(new SimpleChannelInboundHandler<FullHttpRequest>() {
                        @Override
                        protected void channelRead0(ChannelHandlerContext ctx, FullHttpRequest msg) throws Exception {
                            ByteBuf byteBuf = Unpooled.wrappedBuffer("Hello world!".getBytes(StandardCharsets.UTF_8));
                            FullHttpResponse fullHttpResponse = new DefaultFullHttpResponse(HttpVersion.HTTP_1_1, HttpResponseStatus.OK, byteBuf);
                            fullHttpResponse.headers().set("Content-Type", "text/plain;charset=UTF-8");
                            fullHttpResponse.headers().set("Content-Length", byteBuf.readableBytes());
                            ctx.writeAndFlush(fullHttpResponse).addListener(ChannelFutureListener.CLOSE);
                        }
                    });
                }
            }).bind(8080).sync().channel().closeFuture().sync();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            eventLoopGroup.shutdownGracefully();
        }
    }
}
```
