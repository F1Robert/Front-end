# 前端使用Websocket

如果您已经启用了Netty服务，并希望前端界面连接到后端服务，通常您可以使用WebSocket来建立实时的双向通信通道。以下是一些步骤来连接前端界面和后端Netty服务：

1.**前端WebSocket客户端**：在前端界面中使用WebSocket客户端库（例如JavaScript的WebSocket API）来建立WebSocket连接。您需要提供WebSocket服务器的URL以连接到后端服务。

```
const socket = new WebSocket("ws://localhost:8080/your-websocket-endpoint");
```

替换上面的URL为您的Netty服务器的WebSocket端点。

2.**处理WebSocket事件**：在前端中，您可以监听WebSocket的事件，如`onopen`、`onmessage`、`onclose`和`onerror`。这样，您可以处理连接建立、消息接收、连接关闭和错误等事件。

```
socket.onopen = (event) => {
    console.log("WebSocket连接已建立.");
    // 在此处可以发送初始消息或执行其他操作
};

socket.onmessage = (event) => {
    const message = event.data;
    console.log("收到消息: " + message);
    // 在此处处理接收到的消息
};

socket.onclose = (event) => {
    if (event.wasClean) {
        console.log("WebSocket连接已关闭.");
    } else {
        console.error("WebSocket连接意外关闭.");
    }
    console.log("关闭码: " + event.code);
    console.log("关闭原因: " + event.reason);
};

socket.onerror = (error) => {
    console.error("WebSocket连接错误: " + error.message);
};
```

3.**与Netty服务通信**：通过WebSocket连接，前端界面可以向后端Netty服务发送消息，并接收来自服务器的消息。在前端界面中，您可以使用`socket.send(message)`来发送消息给后端服务。

```
// 向服务器发送消息
function sendMessage(message) {
    if (socket.readyState === WebSocket.OPEN) {
        socket.send(message);
    } else {
        console.error("WebSocket连接尚未建立.");
    }
}
```

4.**Netty服务器处理**：在后端，您需要在Netty服务器中编写WebSocket处理程序（Handler）来处理前端发送的消息，执行业务逻辑，并将消息发送回前端。

```
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;
import io.netty.handler.codec.http.websocketx.TextWebSocketFrame;
import io.netty.handler.codec.http.websocketx.WebSocketFrame;

public class WebSocketHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) {
        if (msg instanceof WebSocketFrame) {
            WebSocketFrame frame = (WebSocketFrame) msg;
            if (frame instanceof TextWebSocketFrame) {
                String message = ((TextWebSocketFrame) frame).text();
                // 处理收到的消息，执行业务逻辑
                // 发送消息给客户端
                ctx.writeAndFlush(new TextWebSocketFrame("服务器回复: " + message));
            }
        }
    }

@Override
public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
    cause.printStackTrace();
    ctx.close();
}
}
```

