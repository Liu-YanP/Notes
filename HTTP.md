# HTTP

## http协议的结构

HTTP中报文分为请求报文（request message） 和响应报文（response message）两种类型，

这两种类型都包括三部分：**首行**、**头 部**和**主体**。

*请求报文*的首行是请求行，包括方法（请求类型）、URL和HTTP版本三 项内容，

*响应请求*的首行是状态行，包括HTTP版本、状态码和简短原因三项内容， 其中原因可有可无。

头部保存一些键值对的属性，用冒号“：”分割。主体保存具 体内容，请求报文中主要保存POST类型的参数，响应报文中保存页面要显示的结 果。首行、头部和主体以及头部的各项内容用回车换行（\r\n）分割，另外头部和 主体之间多一个空行，也就是有两个连续的回车换行

## 实现一个http协议

HTTP协议是在应用层解析内容的，只需要按照它的报文的格式封装和解析 数据就可以了，具体的传输还是使用的Socket。

```java
import java.io.IOException; 
import java.net.InetSocketAddress; 
import java.nio.ByteBuffer; 
import java.nio.channels.SelectionKey; 
import java.nio.channels.Selector; 
import java.nio.channels.ServerSocketChannel; 
import java.nio.channels.SocketChannel; 
import java.nio.charset.Charset; 
import java.util.Iterator; 
public class HttpServer { 
    public static void main(String[] args) throws Exception{ 
        //创建 ServerSocketChannel，监听8080端口 
        ServerSocketChannel ssc=ServerSocketChannel.open(); 
        ssc.socket().bind(new InetSocketAddress(8080)); 
        //设置为非阻塞模式
        ssc.configureBlocking(false); 
        //为ssc注册选择器 
        Selector selector=Selector.open(); 
        ssc.register(selector,SelectionKey.OP_ACCEPT); 
        //创建处理器 
        while(true){ 
            // 等待请求，每次等待阻塞3s，超过3s后线程继续向下运行，如果传入0或者不传参数将一直阻塞
            if(selector.select(3000)==0){ continue; } 
            // 获取待处理的 
            SelectionKey Iterator keyIter=selector.selectedKeys().iterator(); 
            while(keyIter.hasNext()){ 
                SelectionKey key=keyIter.next(); 
                // 启动 新线程处理
                SelectionKey new Thread(new HttpHandler(key)).run();
                // 处 理完后，从待处理的SelectionKey迭代器中移除当前所使用的key 
                keyIter.remove(); 
            } 
        } 
    } 
    private static class HttpHandler implements Runnable{ 
        private int bufferSize = 1024; 
        private String localCharset = "UTF-8"; 
        private SelectionKey key; 
        public HttpHandler(SelectionKey key){ 
            this.key = key; 
        } 
        public void handleAccept() throws IOException { 
            SocketChannel clientChannel=((ServerSocketChannel)key.channel()).accept();
			clientChannel.configureBlocking(false);
			clientChannel.register(key.selector(), SelectionKey.OP_READ,
			ByteBuffer.allocate(bufferSize)); 
        } 
        public void handleRead() throws IOException { 
            // 获取channel 
            SocketChannel sc=(SocketChannel)key.channel(); 
            // 获取buffer并重置 ByteBuffer
			buffer=(ByteBuffer)key.attachment(); 
            buffer.clear(); 
            // 没有读到内容则关闭 
            if(sc.read(buffer)==-1){ 
                sc.close(); 
            } else { 
                // 接收请求数据
				buffer.flip(); 
                String receivedString =
				Charset.forName(localCharset).newDecoder().decode(buffer).toString();
				// 控制台打印请求报文头 
                String[] requestMessage =receivedString.split("\r\n"); 
                for(String s: requestMessage){
					System.out.println(s); 
                    // 遇到空行说明报文头已经打印完
					if(s.isEmpty()) break; 
                } 
                // 控制台打印首行信息 
                String[] firstLine =requestMessage[0].split(" "); System.out.println();
				System.out.println("Method:\t"+firstLine[0]);
				System.out.println("url:\t"+firstLine[1]); 
                System.out.println("HTTPVersion:\t"+firstLine[2]); System.out.println(); 				// 返回客户端
				StringBuilder sendString = new StringBuilder();
				sendString.append("HTTP/1.1 200 OK\r\n");
                //响应报文首行，200表示处理成功
                sendString.append("Content-Type:text/html;charset="+localCharset+"\r\n"); 
                sendString.append("\r\n");
                // 报文头结束后加一个空行 
                sendString.append(" 显示报文"); 
                sendString.append("接收到请求报文是："); 
                for(String s: requestMessage){ 
                    sendString.append(s + " "); 
                } 
                sendString.append(""); 
                buffer = ByteBuffer.wrap(sendString.toString().getBytes(localCharset)); 
                sc.write(buffer); sc.close(); 
            }
        } 
        @Override 
        public void run() { 
            try{ 
                // 接收到连接请求时 
                if(key.isAcceptable()){ 
                    handleAccept(); 
                } 
                // 读数据 
                if(key.isReadable()){ 
                    handleRead(); 
                } 
            } catch(IOException ex) { 
                ex.printStackTrace(); 
            }
        } 
    } 
}


```

整个过程非常简单，按照报文的格式来读取和发送就可以了，接收到数据后 按“\r\n”分割成每一行，在空行之前都是报文头（包含首行），空行下面如果有 内容就是报文的主体，因为这里是`Get`请求所以就没有主体了，首行使用空格分割后 可以得到请求的方法、`Url`和`Http`的版本，如果需要请求头的值只需要把头部的每 一行用冒号分割开就行了。