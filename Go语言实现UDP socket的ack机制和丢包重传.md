# Go语言实现UDP socket的ack机制和丢包重传



参考连接： https://juejin.cn/post/7263378772040122429



UDP 在通讯之前不需要建立连接 可以直接发送数据包 是一种无连接协议（常用于音视频传输）

但是在有些场景 即需要UDP传输也需要向TCP一样（TCP的可靠传输）



解决丢包问题：

1.添加 seq/ack机制 确保数据发送到对端



什么是 seq/ack机制

```tex


在TCP（传输控制协议）中，序号（Sequence Number，简称seq）和确认应答号（Acknowledgment Number，简称ack）是协议头部非常关键的字段，它们共同确保了TCP的可靠性和数据按顺序传输的特性。

** Sequence Number **
含义：序号是指一个TCP报文段中第一个字节的数据序列标识。它表示在一个TCP连接中，该报文段所携带的数据的开始位置。序号是用来保证数据传输的顺序性和完整性的。

作用：在TCP连接建立时，双方各自随机选择一个初始序列号（ISN）。随后传输的每个报文段的序号将基于这个初始值递增，其增量为该报文段所携带的数据量（字节数）。通过这种方式，接收方可以根据序号重组乱序到达的数据片段，确保数据的正确顺序和完整性。如果接收到的报文段不连续，接收方可以通过TCP的重传机制请求发送方重新发送缺失的数据。



**Acknowledgment Number**
含义：确认应答号是接收方期望从发送方接收到的下一个报文段的序号。它实质上是接收方告诉发送方：“我已经成功接收到了哪个序号之前的所有数据，请从这个序号开始发送后续的数据。”

作用：确认应答号用于实现可靠性传输。当一个报文段被接收方正确接收时，接收方会发送一个ACK报文，其中包含的确认应答号是接收到的数据加上1（即接收方期望接收的下一个数据的序号）。通过检查这个确认应答号，发送方能够知道其发送的数据是否已被接收方正确接收，并据此决定是否需要重传某些数据段。

```

ack和seq 保证了：

- 确保数据的顺序性：即使数据片段在网络中的传输过程中顺序被打乱，接收方也能根据序号正确地重组这些数据。

- 检测丢包：如果发送方发送的数据长时间未被确认（即没有收到对应的ACK报文），它会判断这些数据可能已丢失，并将其重新发送。
- 实现流量控制和拥塞控制：通过调整发送未被确认数据的量（即控制窗口大小），TCP可以根据网络条件动态调整数据发送的速率，避免网络拥塞。



#### Golang的socket编程：



Go语言通过标准库中的`net`包来实现UDP和TCP的socket编程。`net`包提供了用于创建和管理网络连接的函数，以及用于进行数据传输的相关类型和方法，不同于C++需要手动设置和管理socket API，不论实现UDP还是TCP都可以直接使用封装好的方法进行操作，大大简化了socket编程：



##### 使用net包实现UDP通信



###### 1.client.go

~~~
package main

import (
	"bufio"
	"fmt"
	"net"
	"os"
)

func main() {
	// 创建UDP连接到服务器的地址和端口号
	c, err := net.DialUDP("udp", nil, &net.UDPAddr{
		IP:   net.IPv4(127, 0, 0, 1),
		Port: 8282,
	})
	if err != nil {
		fmt.Println("dial err: %v\n", err)
		return
	}
	defer c.Close() // 将 defer 放在 if 语句外面

	// 从标准输入读取用户输入的数据
	input := bufio.NewReader(os.Stdin)
	for {
		// 读取用户输入知道遇见换行符
		s, err := input.ReadString('\n')
		if err != nil {
			fmt.Printf("read from stdin failed, err: %v\n", err)
			return
		}

		// 将用户输入的数据转换为字节数组并通过UDP连接发送给服务器
		_, err = c.Write([]byte(s))
		if err != nil {
			fmt.Printf("send to server failed, err: %v\n", err)
			return
		}

		// 接收来自服务器的数据
		var buf [1024]byte
		n, addr, err := c.ReadFromUDP(buf[:])
		if err != nil {
			fmt.Printf("recv from udp failed, err: %v\n", err)
			return
		}

		// 打印来自服务器的数据
		fmt.Printf("服务器 %v, 响应数据: %v\n", addr, string(buf[:n]))
	}
}

~~~



###### 2.server.go

首先创建UDP监听器监听指定IP和端口，等待连接客户端，连接后会读取客户端发来的数据并打印收到的数据，并将接收的响应信息返回发送给客户端，使用死循环使其能够持续获取客户端数据，同样实现了UDP的数据接收和发送，实现了简单的UDP服务器；



~~~
package main

import (
	"fmt"
	"net"
)

// udp server
func main() {
	// 创建一个UDP监听器，监听本地IP地址的端口
	listen, err := net.ListenUDP("udp", &net.UDPAddr{
		IP:   net.IPv4(127, 0, 0, 1),
		Port: 8282,
	})
	if err != nil {
		fmt.Printf("listen failed,err:%v\n", err)
		return
	}
	defer listen.Close()

	for {
		var buf [1024]byte
		// 从UDP连接中读取数据到buf中，n为读取到的字节数，addr为数据发送者的地址
		n, addr, err := listen.ReadFromUDP(buf[:])
		if err != nil {
			fmt.Printf("read from udp failed,err:%v\n", err)
			return
		}

		// 打印接收到的数据
		fmt.Println("接收到的数据：", string(buf[:n]))

		// 将接收到的数据原样发送回给数据发送者
		_, err = listen.WriteToUDP(buf[:n], addr)
		if err != nil {
			fmt.Printf("write to %v failed,err:%v\n", addr, err)
			return
		}
	}
}

~~~



效果：（好有趣）



~~~
> go run client.go
hello
服务器 127.0.0.1:8282, 响应数据: hello

world
服务器 127.0.0.1:8282, 响应数据: world


~~~

~~~
 go run server.go
接收到的数据： hello

接收到的数据： world


~~~



seq_apk_client.go

~~~
package main

import (
	"fmt"
	"net"
	"strconv"
	"strings"
	"time"
)

type Message struct {
	Seq int
	Msg string
}

func main() {
	c, err := net.DialUDP("udp", nil, &net.UDPAddr{
		IP:   net.IPv4(127, 0, 0, 1),
		Port: 8282,
	})

	if err != nil {
		fmt.Printf("dail err:%v\n", err)
	}
	defer c.Close()

	input := []string{"Message1", "Message2", "Message3", "Message4", "Message5"}
	seq := 0

	for _, msg := range input {
		seq++
		message := Message{Seq: seq, Msg: msg}
		fmt.Printf("Sending seq=%d: %s\n", message.Seq, message.Msg)

		// 发送带有序列号的数据包
		_, err = c.Write(encodeMessage(message))
		if err != nil {
			fmt.Printf("send to server failed,err:%v\n", err)
			return
		}

	}
	// 等待ACK，设置超时时间
	buf := make([]byte, 1024)
	c.SetReadDeadline(time.Now().Add(5 * time.Second))
	n, _, err := c.ReadFromUDP(buf)
	if err != nil {
		fmt.Println("ACK not received. Timeout or Error.")
		return
	} else {
		ack := decodeMessage(buf[:n])
		if ack.Seq == seq+1 {
			fmt.Printf("ACK = %d\n", ack.Seq)
		} else {
			fmt.Println("Invalid ACK received. Retry.")
			return
		}
	}

}

func encodeMessage(msg Message) []byte {
	// 将序列号和消息文本编码成字节数据
	return []byte(fmt.Sprintf("%d;%s", msg.Seq, msg.Msg))
}

func decodeMessage(data []byte) Message {
	// 解码收到的数据，提取序列号和消息文本
	parts := strings.Split(string(data), ";")
	seq, _ := strconv.Atoi(parts[0])
	msg := parts[1]
	return Message{Seq: seq, Msg: msg}
}

~~~



seq_apk_server.go

~~~
package main

import (
	"fmt"
	"net"
	"strconv"
	"strings"
)

type Message2 struct {
	Seq int
	Msg string
}

func main() {
	listen, err := net.ListenUDP("udp", &net.UDPAddr{
		IP:   net.IPv4(127, 0, 0, 1),
		Port: 8282,
	})
	if err != nil {
		fmt.Printf("listen failed,err:%v\n", err)
		return
	}
	defer listen.Close()

	for {
		var buf [1024]byte
		n, addr, err := listen.ReadFromUDP(buf[:])
		if err != nil {
			fmt.Printf("read from udp failed,err:%v\n", err)
			return
		}

		// 处理接收到的数据，提取序列号和消息文本
		message := decodeMessage1(buf[:n])
		fmt.Printf("Received seq=%d from %v: %s\n", message.Seq, addr, message.Msg)

		// 发送ACK回复给客户端，ACK=Seq+1
		ack := Message2{Seq: message.Seq + 1, Msg: "ACK"}
		_, err = listen.WriteToUDP(encodeMessage1(ack), addr)
		if err != nil {
			fmt.Printf("write to %v failed,err:%v\n", addr, err)
			return
		}
	}
}

func encodeMessage1(msg Message2) []byte {
	// 将序列号和消息文本编码成字节数据
	return []byte(fmt.Sprintf("%d;%s", msg.Seq, msg.Msg))
}

func decodeMessage1(data []byte) Message2 {
	// 解码收到的数据，提取序列号和消息文本
	parts := strings.Split(string(data), ";")
	seq, _ := strconv.Atoi(parts[0])
	msg := parts[1]
	return Message2{Seq: seq, Msg: msg}
}

~~~

