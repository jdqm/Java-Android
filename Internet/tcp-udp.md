###1.TCP与UDP的区别
①基于连接与无连接（是指数据传输之前）
①TCP对资源的需求比UDP高
③UDP的程序结构简单，首部开销只有8个字节，而TCP有16个字节
④流模式与数据报模式，UDP没有网络拥堵控制策略，当出现网络拥堵时不会降低主机的发送速率，而TCP会有相应的网络拥堵策略。UDP通常应用在实时应用中，如IP电话，视频直播等
⑤TCP保证数据的安全，有序到达，而UDP不会，可能会丢包
⑥TCP是点对点工作的，而UDP可以是一对一，一对多，多对一，多对多的交互通信
⑦TCP的逻辑通信信道是全双工的可靠信道，而UDP则是不可靠信道
⑧TCP多用于传递少量数据，而UDP多用于传递大量数据

###2.使用TCP进行Socket通信
步骤：
（1）服务端创建ServerSocket，需要传入一个端口号；
（2）调用serverSocket.accept()监听（1）中设置的端口；
（3）客户端创建一个Socket对象，传入主机地址、端口号；
（4）客户端和服务端之间可以通过InputStream和OutputStream来进行交互数据。

Socket服务端
```
public class TCPServer {
    public static void main(String[] args) {
        try {
            ServerSocket serverSocket = new ServerSocket(8888);
            while (true) {
                Socket socket = serverSocket.accept();
                new ServerThread(socket).start();
                InetAddress address = socket.getInetAddress();
                System.out.println("当前客户端IP为：" + address.getHostAddress());
            }

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

```

public class ServerThread extends Thread {
    private Socket socket;

    public ServerThread(Socket socket) {
        this.socket = socket;
    }

    @Override
    public void run() {
        BufferedReader bufferedReader = null;
        OutputStreamWriter writer = null;
        try {
            InputStream inputStream = socket.getInputStream();
            bufferedReader = new BufferedReader(new InputStreamReader(inputStream));
            String data;
            while ((data = bufferedReader.readLine()) != null) {
                System.out.println(data);
            }
            socket.shutdownInput();

            writer = new OutputStreamWriter(socket.getOutputStream());
            writer.write("我是TCP服务器");
            writer.flush();

            socket.close();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                bufferedReader.close();
            } catch (IOException e) {
                e.printStackTrace();
            } finally {
                try {
                    writer.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }

        }
    }
}
```


Socket客户端
```
public class TCPClient {
    public static void main(String[] args) {
        PrintWriter writer = null;
        BufferedReader reader = null;
        try {
            Socket socket = new Socket("localhost", 8888);
            OutputStream os = socket.getOutputStream();
            writer = new PrintWriter(os);
            writer.write("我是TCP客户端");
            writer.flush();
            socket.shutdownOutput();

            InputStream inputStream = socket.getInputStream();
            reader = new BufferedReader(new InputStreamReader(inputStream));
            String reply;
            while ((reply = reader.readLine()) != null) {
                System.out.println(reply);
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            writer.close();
            try {
                reader.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```