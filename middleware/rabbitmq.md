# RabbitMQ简介

RabbitMQ 是一个由 Erlang 语言开发的 AMQP 的开源实现。

AMQP ：Advanced Message Queue，高级消息队列协议。它是应用层协议的一个开放标准，为面向消息的中间件设计，基于此协议的客户端与消息中间件可传递消息，并不受产品、开发语言等条件的限制。

RabbitMQ 最初起源于金融系统，用于在分布式系统中存储转发消息，在易用性、扩展性、高可用性等方面表现不俗。

具体特点包括：

1. 可靠性（Reliability）
   RabbitMQ 使用一些机制来保证可靠性，如持久化、传输确认、发布确认。
2. 灵活的路由（Flexible Routing）
   在消息进入队列之前，通过 Exchange 来路由消息的。对于典型的路由功能，RabbitMQ 已经提供了一些内置的 Exchange 来实现。针对更复杂的路由功能，可以将多个 Exchange 绑定在一起，也通过插件机制实现自己的 Exchange 。
3. 消息集群（Clustering）
   多个 RabbitMQ 服务器可以组成一个集群，形成一个逻辑 Broker 。
4. 高可用（Highly Available Queues）
   队列可以在集群中的机器上进行镜像，使得在部分节点出问题的情况下队列仍然可用。
5. 多种协议（Multi-protocol）
   RabbitMQ 支持多种消息队列协议，比如 STOMP、MQTT 等等。
6. 多语言客户端（Many Clients）
   RabbitMQ 几乎支持所有常用语言，比如 Java、.NET、Ruby 等等。
7. 管理界面（Management UI）
   RabbitMQ 提供了一个易用的用户界面，使得用户可以监控和管理消息 Broker 的许多方面。
8. 跟踪机制（Tracing）
   如果消息异常，RabbitMQ 提供了消息跟踪机制，使用者可以找出发生了什么。
9. 插件机制（Plugin System）
   RabbitMQ 提供了许多插件，来从多方面进行扩展，也可以编写自己的插件。

## RabbitMQ的优势

现在的市面上有很多MQ可以选择，比如ActiveMQ、ZeroMQ、Appche Qpid，那问题来了为什么要选择RabbitMQ？

1. 除了Qpid，RabbitMQ是唯一一个实现了AMQP标准的消息服务器。
2. 可靠性，RabbitMQ的持久化支持，保证了消息的稳定性。
3. 高并发，RabbitMQ使用了Erlang开发语言，Erlang是为电话交换机开发的语言，天生自带高并发光环，和高可用特性。
4. 集群部署简单，正是应为Erlang使得RabbitMQ集群部署变的超级简单。
5. 社区活跃度高，根据网上资料来看，RabbitMQ也是首选。

# RabbitMQ工作原理

## RabbitMQ 基本概念

上面介绍过 RabbitMQ 是 AMQP 协议的一个开源实现，所以其内部实际上也是 AMQP 中的基本概念：

![rabbitmq_1](../assets/rabbitmq_1.png)

- Message
  消息，消息是不具名的，它由消息头和消息体组成。消息体是不透明的，而消息头则由一系列的可选属性组成，这些属性包括routing-key（路由键）、priority（相对于其他消息的优先权）、delivery-mode（指出该消息可能需要持久性存储）等。
- Publisher
  消息的生产者，也是一个向交换器发布消息的客户端应用程序。
- Exchange
  交换器，用来接收生产者发送的消息并将这些消息路由给服务器中的队列。
- Binding
  绑定，用于消息队列和交换器之间的关联。一个绑定就是基于路由键将交换器和消息队列连接起来的路由规则，所以可以将交换器理解成一个由绑定构成的路由表。
- Queue
  消息队列，用来保存消息直到发送给消费者。它是消息的容器，也是消息的终点。一个消息可投入一个或多个队列。消息一直在队列里面，等待消费者连接到这个队列将其取走。
- Connection
  网络连接，比如一个TCP连接。
- Channel
  信道，多路复用连接中的一条独立的双向数据流通道。信道是建立在真实的TCP连接内地虚拟连接，AMQP 命令都是通过信道发出去的，不管是发布消息、订阅队列还是接收消息，这些动作都是通过信道完成。因为对于操作系统来说建立和销毁 TCP 都是非常昂贵的开销，所以引入了信道的概念，以复用一条 TCP 连接。
- Consumer
  消息的消费者，表示一个从消息队列中取得消息的客户端应用程序。
- Virtual Host
  虚拟主机，表示一批交换器、消息队列和相关对象。虚拟主机是共享相同的身份认证和加密环境的独立服务器域。每个 vhost 本质上就是一个 mini 版的 RabbitMQ 服务器，拥有自己的队列、交换器、绑定和权限机制。vhost 是 AMQP 概念的基础，必须在连接时指定，RabbitMQ 默认的 vhost 是 / 。
- Broker
  表示消息队列服务器实体。

## AMQP 消息路由

AMQP 中消息的路由过程和 Java 开发者熟悉的 JMS 存在一些差别，AMQP 中增加了 Exchange 和 Binding 的角色。生产者把消息发布到 Exchange 上，消息最终到达队列并被消费者接收，而 Binding 决定交换器的消息应该发送到那个队列。

![rabbitmq_2](../assets/rabbitmq_2.png)

## RabbitMQ 工作流程

![rabbitmq_3](../assets/rabbitmq_3.png)

生产者发送消息流程：

1. 生产者和Broker建立TCP连接。
2. 生产者和Broker建立通道。
3. 生产者通过通道消息发送给Broker，由Exchange将消息进行转发。
4. Exchange将消息转发到指定的Queue（队列）。



消费者接收消息流程：

1. 消费者和Broker建立TCP连接。
2. 消费者和Broker建立通道。
3. 消费者监听指定的Queue（队列）。
4. 当有消息到达Queue时Broker默认将消息推送给消费者。
5. 消费者接收到消息。
6. ack回复。

# 消息模型

## 基本消息模型

![rabbitmq_4](../assets/rabbitmq_4.png)

在上图的模型中，有以下概念：

- P：生产者，也就是要发送消息的程序。
- C：消费者：消息的接受者，会一直等待消息到来。
- queue：消息队列，图中红色部分。可以缓存消息；生产者向其中投递消息，消费者从其中取出消息。 

公共部分：

```go
// open a connection
conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
failOnError(err, "Failed to connect to RabbitMQ")
defer conn.Close()

// open a channel
ch, err := conn.Channel()
failOnError(err, "Failed to open a channel")
defer ch.Close()

// handling error
func failOnError(err error, msg string) {
  if err != nil {
    log.Fatalf("%s: %s", msg, err)
  }
}
```

send.go

```shell
q, err := ch.QueueDeclare(
		"hello", // name
		false,   // durable
		false,   // delete when unused
		false,   // exclusive
		false,   // no-wait
		nil,     // arguments
	)
	failOnError(err, "Failed to declare a queue")

	body := "Hello World!"
	err = ch.Publish(
		"",     // exchange
		q.Name, // routing key
		false,  // mandatory
		false,  // immediate
		amqp.Publishing{
			ContentType: "text/plain",
			Body:        []byte(body),
		})
	failOnError(err, "Failed to publish a message")
```

receive.go

```shell
q, err := ch.QueueDeclare(
		"hello", // name
		false,   // durable
		false,   // delete when unused
		false,   // exclusive
		false,   // no-wait
		nil,     // arguments
	)
	failOnError(err, "Failed to declare a queue")

	msgs, err := ch.Consume(
		q.Name, // queue
		"",     // consumer
		true,   // auto-ack
		false,  // exclusive
		false,  // no-local
		false,  // no-wait
		nil,    // args
	)
	failOnError(err, "Failed to register a consumer")

	forever := make(chan bool)

	go func() {
		for d := range msgs {
			log.Printf("Received a message: %s", d.Body)
		}
	}()

	log.Printf(" [*] Waiting for messages. To exit press CTRL+C")
	<-forever
```

## 工作队列消息模型

![rabbitmq_5](../assets/rabbitmq_5.png)

work queues与入门程序相比，多了一个消费端，两个消费端共同消费同一个队列中的消息，但是一个消息只能被一个消费者获取。

这个消息模型在Web应用程序中特别有用，可以处理短的HTTP请求窗口中无法处理复杂的任务。



接下来我们来模拟这个流程：

- P：生产者：任务的发布者。
- C1：消费者1：领取任务并且完成任务，假设完成速度较慢（模拟耗时）。
- C2：消费者2：领取任务并且完成任务，假设完成速度较快。

send.go

```go
q, err := ch.QueueDeclare(
		"hello", // name
		false,   // durable
		false,   // delete when unused
		false,   // exclusive
		false,   // no-wait
		nil,     // arguments
	)
	failOnError(err, "Failed to declare a queue")

	body := "Hello World!"
	err = ch.Publish(
		"",     // exchange
		q.Name, // routing key
		false,  // mandatory
		false,  // immediate
		amqp.Publishing{
			ContentType: "text/plain",
			Body:        []byte(body),
		})
	failOnError(err, "Failed to publish a message")
```

receive1.go

```shell
q, err := ch.QueueDeclare(
		"hello", // name
		false,   // durable
		false,   // delete when unused
		false,   // exclusive
		false,   // no-wait
		nil,     // arguments
	)
	failOnError(err, "Failed to declare a queue")

	msgs, err := ch.Consume(
		q.Name, // queue
		"",     // consumer
		true,   // auto-ack
		false,  // exclusive
		false,  // no-local
		false,  // no-wait
		nil,    // args
	)
	failOnError(err, "Failed to register a consumer")

	forever := make(chan bool)

	go func() {
		for d := range msgs {
			log.Printf("Received a message: %s", d.Body)
			time.Sleep(5 * time.Second)
		}
	}()

	log.Printf(" [*] Waiting for messages. To exit press CTRL+C")
	<-forever
```

receive2.go

```shell
q, err := ch.QueueDeclare(
		"hello", // name
		false,   // durable
		false,   // delete when unused
		false,   // exclusive
		false,   // no-wait
		nil,     // arguments
	)
	failOnError(err, "Failed to declare a queue")

	msgs, err := ch.Consume(
		q.Name, // queue
		"",     // consumer
		true,   // auto-ack
		false,  // exclusive
		false,  // no-local
		false,  // no-wait
		nil,    // args
	)
	failOnError(err, "Failed to register a consumer")

	forever := make(chan bool)

	go func() {
		for d := range msgs {
			log.Printf("Received a message: %s", d.Body)
			time.Sleep(1 * time.Second)
		}
	}()

	log.Printf(" [*] Waiting for messages. To exit press CTRL+C")
	<-forever
```

刚才的实现有问题吗？

消费者1比消费者2的效率要低，一次任务的耗时较长，然而两人最终消费的消息数量是一样的，消费者2大量时间处于空闲状态，消费者1一直忙碌，现在的状态属于是把任务平均分配，正确的做法应该是消费越快的人，消费的越多。



怎么实现呢？

通过 BasicQos 方法设置prefetchCount = 1。这样RabbitMQ就会使得每个Consumer在同一个时间点最多处理1个Message。换句话说，在接收到该Consumer的ack前，他它不会将新的Message分发给它。相反，它会将其分派给不是仍然忙碌的下一个Consumer。

**值得注意的是：prefetchCount在手动ack的情况下才生效，自动ack不生效。**



```go
err = ch.Qos(
		1,     // prefetch count
		0,     // prefetch size
		false, // global
	)
	failOnError(err, "Failed to set QoS")
```

## 订阅消息模型

### 订阅消息模型特点

- 一个生产者多个消费者。
- 每个消费者都有一个自己的队列。
- 生产者没有将消息直接发送给队列，而是发送给exchange(交换机、转发器)。
- 每个队列都需要绑定到交换机上。
- 生产者发送的消息，经过交换机到达队列，实现一个消息被多个消费者消费。



交换机一方面：接收生产者发送的消息。另一方面：知道如何处理消息，例如递交给某个特别队列、递交给所有队列、或是将消息丢弃。到底如何操作，取决于Exchange的类型。



**Exchange（交换机）只负责转发消息，不具备存储消息的能力，因此如果没有任何队列与Exchange绑定，或者没有符合路由规则的队列，那么消息会丢失！**



### Exchange类型

- Fanout：广播，将消息交给所有绑定到交换机的队列。
- Direct：定向，把消息交给符合指定routing key 的队列。
- Topics：通配符，把消息交给符合routing pattern（路由模式） 的队列。
- Header：header模式与routing不同的地方在于，header模式取消routingkey，使用header中的 key/value（键值对）匹配队列。Header和routing很像，但性能很差，用的很少，此处不做介绍。

#### Publish/Subscribe模式（交换机类型：Fanout，也称为广播 ）

![rabbitmq_6](../assets/rabbitmq_6.png)

send.go

```go
err = ch.ExchangeDeclare(
		"logs",   // name
		"fanout", // type
		true,     // durable
		false,    // auto-deleted
		false,    // internal
		false,    // no-wait
		nil,      // arguments
	)
	failOnError(err, "Failed to declare an exchange")
	body := "Hello World"
	err = ch.Publish(
		"logs", // exchange
		"",     // routing key
		false,  // mandatory
		false,  // immediate
		amqp.Publishing{
			ContentType: "text/plain",
			Body:        []byte(body),
		})
	failOnError(err, "Failed to publish a message")
```

receive.go

```shell
err = ch.ExchangeDeclare(
		"logs",   // name
		"fanout", // type
		true,     // durable
		false,    // auto-deleted
		false,    // internal
		false,    // no-wait
		nil,      // arguments
	)
	failOnError(err, "Failed to declare an exchange")

	q, err := ch.QueueDeclare(
		"",    // name
		false, // durable
		false, // delete when unused
		true,  // exclusive
		false, // no-wait
		nil,   // arguments
	)
	failOnError(err, "Failed to declare a queue")

	err = ch.QueueBind(
		q.Name, // queue name
		"",     // routing key
		"logs", // exchange
		false,
		nil,
	)
	failOnError(err, "Failed to bind a queue")

	msgs, err := ch.Consume(
		q.Name, // queue
		"",     // consumer
		true,   // auto-ack
		false,  // exclusive
		false,  // no-local
		false,  // no-wait
		nil,    // args
	)
	failOnError(err, "Failed to register a consumer")

	forever := make(chan bool)

	go func() {
		for d := range msgs {
			log.Printf(" [x] %s", d.Body)
		}
	}()

	log.Printf(" [*] Waiting for logs. To exit press CTRL+C")
	<-forever
```

##### 思考

1. publish/subscribe与work queues有什么区别。

不同点：

- - work queues不用定义交换机，而publish/subscribe需要定义交换机。
  - publish/subscribe的生产方是面向交换机发送消息，work queues的生产方是面向队列发送消息(底层使用默认交换机)。
  - publish/subscribe需要设置队列和交换机的绑定，work queues不需要设置，实际上work queues会将队列绑定到默认的交换机 。

相同点：

- - 所以两者实现的发布/订阅的效果是一样的，多个消费端监听同一个队列不会重复消费消息。



1. 实际工作用 publish/subscribe还是work queues。

建议使用 publish/subscribe，发布订阅模式比工作队列模式更强大（也可以做到同一队列竞争），并且发布订阅模式可以指定自己专用的交换机。
 

#### Routing模式（交换机类型：direct）

![rabbitmq_7](../assets/rabbitmq_7.png)

- P：生产者，向Exchange发送消息，发送消息时，会指定一个routing key。
- X：Exchange（交换机），接收生产者的消息，然后把消息递交给 与routing key完全匹配的队列。
- C1：消费者，其所在队列指定了需要routing key 为 error 的消息。
- C2：消费者，其所在队列指定了需要routing key 为 info、error、warning 的消息。

send.go

```go
err = ch.ExchangeDeclare(
                "logs_direct", // name
                "direct",      // type
                true,          // durable
                false,         // auto-deleted
                false,         // internal
                false,         // no-wait
                nil,           // arguments
        )
        failOnError(err, "Failed to declare an exchange")

        body := "Hello World"
        err = ch.Publish(
                "logs_direct",         // exchange
                "black", // routing key
                false, // mandatory
                false, // immediate
                amqp.Publishing{
                        ContentType: "text/plain",
                        Body:        []byte(body),
                })
        failOnError(err, "Failed to publish a message")

        log.Printf(" [x] Sent %s", body)
```

receive.go

```shell
err = ch.ExchangeDeclare(
		"logs_direct", // name
		"direct",      // type
		true,          // durable
		false,         // auto-deleted
		false,         // internal
		false,         // no-wait
		nil,           // arguments
	)
	failOnError(err, "Failed to declare an exchange")

	q, err := ch.QueueDeclare(
		"",    // name
		false, // durable
		false, // delete when unused
		true,  // exclusive
		false, // no-wait
		nil,   // arguments
	)
	failOnError(err, "Failed to declare a queue")

	err = ch.QueueBind(
		q.Name,        // queue name
		"black",       // routing key
		"logs_direct", // exchange
		false,
		nil)
	failOnError(err, "Failed to bind a queue")

	msgs, err := ch.Consume(
		q.Name, // queue
		"",     // consumer
		true,   // auto ack
		false,  // exclusive
		false,  // no local
		false,  // no wait
		nil,    // args
	)
	failOnError(err, "Failed to register a consumer")

	forever := make(chan bool)

	go func() {
		for d := range msgs {
			log.Printf(" [x] %s", d.Body)
		}
	}()

	log.Printf(" [*] Waiting for logs. To exit press CTRL+C")
	<-forever
```

#### Topics模式（交换机类型：topic）

![rabbitmq_8](../assets/rabbitmq_8.png)

每个消费者监听自己的队列，并且设置带统配符的routingkey,生产者将消息发给broker，由交换机根据routingkey来转发消息到指定的队列。

Routingkey一般都是有一个或者多个单词组成，多个单词之间以“.”分割，例如：inform.sms。



通配符规则：

- \#：匹配一个或多个词。
- *：匹配不多不少恰好1个词。



举例：

- audit.#：能够匹配audit.irs.corporate 或者 audit.irs。
- audit.*：只能匹配audit.irs。



从示意图可知，我们将发送所有描述动物的消息。消息将使用由三个字（两个点）组成的Routing key发送。路由关键字中的第一个单词将描述速度，第二个颜色和第三个种类：“<speed>.<color>.<species>”。

我们创建了三个绑定：Q1绑定了“*.orange.*”，Q2绑定了“.*.*.rabbit”和“lazy.＃”。Q1匹配所有的橙色动物。Q2匹配关于兔子以及懒惰动物的消息。



下面做个小练习，假如生产者发送如下消息，会进入哪个队列：

- quick.orange.rabbit       Q1 Q2   routingKey="quick.orange.rabbit"的消息会同时路由到Q1与Q2。
- lazy.orange.elephant    Q1 Q2。
- quick.orange.fox           Q1。
- lazy.pink.rabbit              Q2  (值得注意的是，虽然这个routingKey与Q2的两个bindingKey都匹配，但是只会投递Q2一次)。
- quick.brown.fox            不匹配任意队列，被丢弃。
- quick.orange.male.rabbit   不匹配任意队列，被丢弃。
- orange         不匹配任意队列，被丢弃。

send.go

```go
err = ch.ExchangeDeclare(
		"logs_topic", // name
		"topic",      // type
		true,         // durable
		false,        // auto-deleted
		false,        // internal
		false,        // no-wait
		nil,          // arguments
	)
	failOnError(err, "Failed to declare an exchange")

	body := bodyFrom(os.Args)
	err = ch.Publish(
		"logs_topic",          // exchange
		"black.white", // routing key
		false,                 // mandatory
		false,                 // immediate
		amqp.Publishing{
			ContentType: "text/plain",
			Body:        []byte(body),
		})
	failOnError(err, "Failed to publish a message")
```

receive.go

```shell
err = ch.ExchangeDeclare(
		"logs_topic", // name
		"topic",      // type
		true,         // durable
		false,        // auto-deleted
		false,        // internal
		false,        // no-wait
		nil,          // arguments
	)
	failOnError(err, "Failed to declare an exchange")

	q, err := ch.QueueDeclare(
		"",    // name
		false, // durable
		false, // delete when unused
		true,  // exclusive
		false, // no-wait
		nil,   // arguments
	)
	failOnError(err, "Failed to declare a queue")

	err = ch.QueueBind(
		q.Name,       // queue name
		"black.*",            // routing key
		"logs_topic", // exchange
		false,
		nil)
	failOnError(err, "Failed to bind a queue")
	
	msgs, err := ch.Consume(
		q.Name, // queue
		"",     // consumer
		true,   // auto ack
		false,  // exclusive
		false,  // no local
		false,  // no wait
		nil,    // args
	)
	failOnError(err, "Failed to register a consumer")

	forever := make(chan bool)

	go func() {
		for d := range msgs {
			log.Printf(" [x] %s", d.Body)
		}
	}()

	log.Printf(" [*] Waiting for logs. To exit press CTRL+C")
	<-forever
```

#### RPC模式

![rabbitmq_9](../assets/rabbitmq_9.png)

##### 基本概念

- Callback queue 回调队列，客户端向服务器发送请求，服务器端处理请求后，将其处理结果保存在一个存储体中。而客户端为了获得处理结果，那么客户在向服务器发送请求时，同时发送一个回调队列地址reply_to。
- Correlation id 关联标识，客户端可能会发送多个请求给服务器，当服务器处理完后，客户端无法辨别在回调队列中的响应具体和那个请求时对应的。为了处理这种情况，客户端在发送每个请求时，同时会附带一个独有correlation_id属性，这样客户端在回调队列中根据correlation_id字段的值就可以分辨此响应属于哪个请求。

##### 流程说明

1. 当客户端启动的时候，它创建一个匿名独享的回调队列。
2. 在 RPC 请求中，客户端发送带有两个属性的消息：一个是设置回调队列的 reply_to 属性，另一个是设置唯一值的 correlation_id 属性。
3. 将请求发送到一个 rpc_queue 队列中。
4. 服务器等待请求发送到这个队列中来。当请求出现的时候，它执行他的工作并且将带有执行结果的消息发送给 reply_to 字段指定的队列。
5. 客户端等待回调队列里的数据。当有消息出现的时候，它会检查 correlation_id 属性。如果此属性的值与请求匹配，将它返回给应用。

server.go

```shell
package main

import (
        "log"
        "strconv"

        "github.com/streadway/amqp"
)

func failOnError(err error, msg string) {
        if err != nil {
                log.Fatalf("%s: %s", msg, err)
        }
}

func fib(n int) int {
        if n == 0 {
                return 0
        } else if n == 1 {
                return 1
        } else {
                return fib(n-1) + fib(n-2)
        }
}

func main() {
        conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
        failOnError(err, "Failed to connect to RabbitMQ")
        defer conn.Close()

        ch, err := conn.Channel()
        failOnError(err, "Failed to open a channel")
        defer ch.Close()

        q, err := ch.QueueDeclare(
                "rpc_queue", // name
                false,       // durable
                false,       // delete when unused
                false,       // exclusive
                false,       // no-wait
                nil,         // arguments
        )
        failOnError(err, "Failed to declare a queue")

        err = ch.Qos(
                1,     // prefetch count
                0,     // prefetch size
                false, // global
        )
        failOnError(err, "Failed to set QoS")

        msgs, err := ch.Consume(
                q.Name, // queue
                "",     // consumer
                false,  // auto-ack
                false,  // exclusive
                false,  // no-local
                false,  // no-wait
                nil,    // args
        )
        failOnError(err, "Failed to register a consumer")

        forever := make(chan bool)

        go func() {
                for d := range msgs {
                        n, err := strconv.Atoi(string(d.Body))
                        failOnError(err, "Failed to convert body to integer")

                        log.Printf(" [.] fib(%d)", n)
                        response := fib(n)

                        err = ch.Publish(
                                "",        // exchange
                                d.ReplyTo, // routing key
                                false,     // mandatory
                                false,     // immediate
                                amqp.Publishing{
                                        ContentType:   "text/plain",
                                        CorrelationId: d.CorrelationId,
                                        Body:          []byte(strconv.Itoa(response)),
                                })
                        failOnError(err, "Failed to publish a message")

                        d.Ack(false)
                }
        }()

        log.Printf(" [*] Awaiting RPC requests")
        <-forever
}
```

client.go

```shell
package main

import (
        "log"
        "math/rand"
        "os"
        "strconv"
        "strings"
        "time"

        "github.com/streadway/amqp"
)

func failOnError(err error, msg string) {
        if err != nil {
                log.Fatalf("%s: %s", msg, err)
        }
}

func randomString(l int) string {
        bytes := make([]byte, l)
        for i := 0; i < l; i++ {
                bytes[i] = byte(randInt(65, 90))
        }
        return string(bytes)
}

func randInt(min int, max int) int {
        return min + rand.Intn(max-min)
}

func fibonacciRPC(n int) (res int, err error) {
        conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
        failOnError(err, "Failed to connect to RabbitMQ")
        defer conn.Close()

        ch, err := conn.Channel()
        failOnError(err, "Failed to open a channel")
        defer ch.Close()

        q, err := ch.QueueDeclare(
                "",    // name
                false, // durable
                false, // delete when unused
                true,  // exclusive
                false, // noWait
                nil,   // arguments
        )
        failOnError(err, "Failed to declare a queue")

        msgs, err := ch.Consume(
                q.Name, // queue
                "",     // consumer
                true,   // auto-ack
                false,  // exclusive
                false,  // no-local
                false,  // no-wait
                nil,    // args
        )
        failOnError(err, "Failed to register a consumer")

        corrId := randomString(32)

        err = ch.Publish(
                "",          // exchange
                "rpc_queue", // routing key
                false,       // mandatory
                false,       // immediate
                amqp.Publishing{
                        ContentType:   "text/plain",
                        CorrelationId: corrId,
                        ReplyTo:       q.Name,
                        Body:          []byte(strconv.Itoa(n)),
                })
        failOnError(err, "Failed to publish a message")

        for d := range msgs {
                if corrId == d.CorrelationId {
                        res, err = strconv.Atoi(string(d.Body))
                        failOnError(err, "Failed to convert body to integer")
                        break
                }
        }

        return
}

func main() {
        rand.Seed(time.Now().UTC().UnixNano())

        n := bodyFrom(os.Args)

        log.Printf(" [x] Requesting fib(%d)", n)
        res, err := fibonacciRPC(n)
        failOnError(err, "Failed to handle RPC request")

        log.Printf(" [.] Got %d", res)
}

func bodyFrom(args []string) int {
        var s string
        if (len(args) < 2) || os.Args[1] == "" {
                s = "30"
        } else {
                s = strings.Join(args[1:], " ")
        }
        n, err := strconv.Atoi(s)
        failOnError(err, "Failed to convert arg to integer")
        return n
}
```

# 消息可靠性

消息一旦被消费者接收，队列中的消息就会被删除。那么问题来了：RabbitMQ怎么知道消息被接收了呢？

如果消费者领取消息后，还没执行操作就挂掉了呢？或者抛出了异常？消息消费失败，但是RabbitMQ无从得知，这样消息就丢失了！

## 消息确认机制（ACK）

因此，RabbitMQ有一个ACK机制。当消费者获取消息后，会向RabbitMQ发送回执ACK，告知消息已经被接收。不过这种回执ACK分两种情况：

- 自动ACK：消息一旦被接收，消费者自动发送ACK。
- 手动ACK：消息接收后，不会发送ACK，需要手动调用。

到底哪种更好呢？这需要看消息的重要性：

- 如果消息不太重要，丢失也没有影响，那么自动ACK会比较方便。
- 如果消息非常重要，不容丢失。那么最好在消费完成后手动ACK，否则接收消息后就自动ACK，RabbitMQ就会把消息从队列中删除。如果此时消费者宕机，那么消息就丢失了。



```go
msgs, err := ch.Consume(
  q.Name, // queue
  "",     // consumer
  false,  // auto-ack
  false,  // exclusive
  false,  // no-local
  false,  // no-wait
  nil,    // args
)
failOnError(err, "Failed to register a consumer")

forever := make(chan bool)

go func() {
  for d := range msgs {
    log.Printf("Received a message: %s", d.Body)
    dotCount := bytes.Count(d.Body, []byte("."))
    t := time.Duration(dotCount)
    time.Sleep(t * time.Second)
    log.Printf("Done")
    d.Ack(false) // manually ack
  }
}()

log.Printf(" [*] Waiting for messages. To exit press CTRL+C")
<-forever
```



Rabbit队列和交换器有一个不可告人的秘密，就是默认情况下重启服务器会导致消息丢失，那么怎么保证Rabbit在重启的时候不丢失呢？答案就是消息持久化。

## 持久化

当你把消息发送到Rabbit服务器的时候，你需要选择你是否要进行持久化，但这并不能保证Rabbit能从崩溃中恢复，想要Rabbit消息能恢复必须满足3个条件：

1. 投递消息的时候durable设置为true，消息持久化。
2. 设置投递模式deliveryMode设置为2（持久），参数3设置为存储纯文本到磁盘。
3. 消息已经到达持久化交换器上。
4. 消息已经到达持久化的队列。

### 持久化工作原理

Rabbit会将你的持久化消息写入磁盘上的持久化日志文件，等消息被消费之后，Rabbit会把这条消息标识为等待垃圾回收。

```shell
err = ch.ExchangeDeclare(
  "logs",   // name
  "fanout", // type
  true,     // durable
  false,    // auto-deleted
  false,    // internal
  false,    // no-wait
  nil,      // arguments
)

q, err := ch.QueueDeclare(
  "task_queue", // name
  true,         // durable
  false,        // delete when unused
  false,        // exclusive
  false,        // no-wait
  nil,          // arguments
)
failOnError(err, "Failed to declare a queue")

err = ch.Publish(
  "",           // exchange
  q.Name,       // routing key
  false,        // mandatory
  false,
  amqp.Publishing {
    DeliveryMode: amqp.Persistent,
    ContentType:  "text/plain",
    Body:         []byte(body),
})
```

### 持久化的缺点

**消息持久化的优点显而易见，但缺点也很明显，那就是性能，因为要写入硬盘要比写入内存性能较低很多，从而降低了服务器的吞吐量，尽管使用SSD硬盘可以使事情得到缓解，但他仍然吸干了Rabbit的性能，当消息成千上万条要写入磁盘的时候，性能是很低的。所以使用者要根据自己的情况，选择适合自己的方式。**



如果要保证消息的可靠性，需要对消息进行持久化处理，然而消息持久化除了需要代码的设置之外，还有一个重要步骤是至关重要的，那就是保证你的消息顺利进入Broker（代理服务器）。

正常情况下，如果消息经过交换器进入队列就可以完成消息的持久化，但如果消息在没有到达broker之前出现意外，那就造成消息丢失，有没有办法可以解决这个问题？

RabbitMQ有两种方式来解决这个问题：

1. 通过AMQP提供的事务机制实现。
2. 使用发送者确认模式实现。

## 事务

事务的实现主要是对信道（Channel）的设置，主要的方法有三个：

1. 声明启动事务模式。
2. 提交事务。
3. 回滚事务。

```go
ch.Tx()

ch.TxCommit()

ch.TxRoolback
```



从上面的代码我们可以看出，在发送消息之前的代码和之前介绍的都是一样的，只是在发送消息之前，需要声明channel为事务模式，提交或者回滚事务即可。



**那么，既然已经有事务了，为什么还要使用发送方确认模式呢，原因是因为事务的性能是非常差的。**



我们知道，消费者可以使用消息自动或手动发送来确认消费消息，那如果我们在消费者模式中使用事务（当然如果使用了手动确认消息，完全用不到事务的），会发生什么呢？

消费者模式使用事务：

假设消费者模式中使用了事务，并且在消息确认之后进行了事务回滚，那么RabbitMQ会产生什么样的变化？

结果分为两种情况：

1. autoAck=false手动应对的时候是支持事务的，也就是说即使你已经手动确认了消息已经收到了，但在确认消息会等事务的返回解决之后，在做决定是确认消息还是重新放回队列，如果你手动确认现在之后，又回滚了事务，那么已事务回滚为主，此条消息会重新放回队列。
2. autoAck=true如果自定确认为true的情况是不支持事务的，也就是说你即使在收到消息之后在回滚事务也是于事无补的，队列已经把消息移除了。

## Confirm发送方确认模式

Confirm发送方确认模式使用和事务类似，也是通过设置Channel进行发送方确认的。

### Confirm的三种实现方式

1. 普通发送方确认模式。
2. 批量确认模式。
3. 异步监听发送方确认模式。



Confirm批量确定和Confirm异步模式性能相差不大，Confirm模式要比事务快10倍左右。

# RabbitMQ安装

RabbitMQ官方镜像地址：https://registry.hub.docker.com/_/rabbitmq

```shell
#拉取镜像
docker pull rabbitmq:3.9.11-management-alpine
#启动容器
docker run -d --name rabbitmq -p 5672:5672 -p 15672:15672 -v /data/rabbitmq:/var/lib/rabbitmq --hostname rabbitmq-0 -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin rabbitmq:3.9.11-management-alpine
```

# RabbitMQ集群

RabbitMQ本身是基于Erlang编写，Erlang语言天生具备分布式特性（通过同步Erlang集群各节点的erlang.cookie来实现）。因此，RabbitMQ天然支持集群。集群是保证可靠性的一种方式，同时可以通过水平扩展以达到增加消息吞吐量能力的目的。

内建集群作为RabbitMQ最优秀的功能之一，它的作用有两个：

1. 允许消费者和生产者在Rabbit节点崩溃的情况下继续运行。
2. 通过增加节点来扩展Rabbit处理更多的消息，承载更多的业务量。

## 集群的特点

RabbitMQ的集群是由多个节点组成的，但我们发现不是每个节点都有所有队列的完全拷贝。

RabbitMQ节点不完全拷贝特性

为什么默认情况下RabbitMQ不将所有队列内容和状态复制到所有节点？

有两个原因：

1. 存储空间——如果每个节点都拥有所有队列的完全拷贝，这样新增节点不但没有新增存储空间，反而增加了更多的冗余数据。
2. 性能——如果消息的发布需安全拷贝到每一个集群节点，那么新增节点对网络和磁盘负载都会有增加，这样违背了建立集群的初衷，新增节点并没有提升处理消息的能力，最多是保持和单节点相同的性能甚至是更糟。

所以其他非所有者节点只知道队列的元数据，和指向该队列节点的指针。

## 集群模式

RabbitMQ有3种模式，但集群模式是2种。详细如下：

- 单一模式：即单机情况不做集群，就单独运行一个rabbitmq而已。
- 普通模式：默认模式，以两个节点（rabbit01、rabbit02）为例来进行说明。对于Queue来说，消息实体只存在于其中一个节点rabbit01（或者rabbit02），rabbit01和rabbit02两个节点仅有相同的元数据，即队列的结构。当消息进入rabbit01节点的Queue后，consumer从rabbit02节点消费时，RabbitMQ会临时在rabbit01、rabbit02间进行消息传输，把A中的消息实体取出并经过B发送给consumer。所以consumer应尽量连接每一个节点，从中取消息。即对于同一个逻辑队列，要在多个节点建立物理Queue。否则无论consumer连rabbit01或rabbit02，出口总在rabbit01，会产生瓶颈。当rabbit01节点故障后，rabbit02节点无法取到rabbit01节点中还未消费的消息实体。如果做了消息持久化，那么得等rabbit01节点恢复，然后才可被消费；如果没有持久化的话，就会产生消息丢失的现象。
- 镜像模式:把需要的队列做成镜像队列，存在与多个节点属于RabbitMQ的HA方案。该模式解决了普通模式中的问题，其实质和普通模式不同之处在于，消息实体会主动在镜像节点间同步，而不是在客户端取数据时临时拉取。该模式带来的副作用也很明显，除了降低系统性能外，如果镜像队列数量过多，加之大量的消息进入，集群内部的网络带宽将会被这种同步通讯大大消耗掉。所以在对可靠性要求较高的场合中适用。

## RabbitMQ默认集群

![rabbitmq_10](../assets/rabbitmq_10.png)

上面图中采用三个节点组成了一个RabbitMQ的集群，Exchange A（交换器）的元数据信息在所有节点上是一致的，而Queue（存放消息的队列）的完整数据则只会存在于它所创建的那个节点上。，其他节点只知道这个queue的metadata信息和一个指向queue的owner node的指针。

### RabbitMQ集群元数据的同步

RabbitMQ集群会始终同步四种类型的内部元数据：

1. 1. 队列元数据：队列名称和它的属性。
   2. 交换器元数据：交换器名称、类型和属性。
   3. 绑定元数据：一张简单的表格展示了如何将消息路由到队列。
   4. vhost元数据：为vhost内的队列、交换器和绑定提供命名空间和安全属性。

因此，当用户访问其中任何一个RabbitMQ节点时，通过rabbitmqctl查询到的queue/user/exchange/vhost等信息都是相同的。 

#### 为何RabbitMQ集群仅采用元数据同步的方式

第一，存储空间。如果每个集群节点都拥有所有Queue的完全数据拷贝，那么每个节点的存储空间会非常大，集群的消息积压能力会非常弱（无法通过集群节点的扩容提高消息积压能力）。

第二，性能。消息的发布者需要将消息复制到每一个集群节点，对于持久化消息，网络和磁盘同步复制的开销都会明显增加。

### RabbitMQ集群发送/订阅消息的基本原理

![rabbitmq_11](../assets/rabbitmq_11.png)

#### 客户端直接连接队列所在节点

如果有一个消息生产者或者消息消费者通过amqp-client的客户端连接至节点1进行消息的发布或者订阅，那么此时的集群中的消息收发只与节点1相关。

#### 客户端连接的是非队列数据所在节点

如果消息生产者所连接的是节点2或者节点3，此时队列1的完整数据不在该两个节点上，那么在发送消息过程中这两个节点主要起了一个路由转发作用，根据这两个节点上的元数据转发至节点1上，最终发送的消息还是会存储至节点1的队列1上。同样，如果消息消费者所连接的节点2或者节点3，那这两个节点也会作为路由节点起到转发作用，将会从节点1的队列1中拉取消息进行消费。 

### 集群节点类型

#### 磁盘节点

将配置信息和元信息存储在磁盘上（单节点系统必须是磁盘节点，否则每次重启RabbitMQ之后所有的系统配置信息都会丢失）。

#### 内存节点

将配置信息和元信息存储在内存中。性能是优于磁盘节点的。



RabbitMQ要求集群中至少有一个磁盘节点，当节点加入和离开集群时，必须通知磁盘节点（如果集群中唯一的磁盘节点崩溃了，则不能进行创建队列、创建交换器、创建绑定、添加用户、更改权限、添加和删除集群节点）。总之如果唯一磁盘的磁盘节点崩溃，集群是可以保持运行的，但不能更改任何东西。因此建议在集群中设置两个磁盘节点，只要一个可以，就能正常操作。



**普通集群模式，并不保证队列的高可用性。尽管交换机、绑定这些可以复制到集群里的任何一个节点，但是队列内容不会复制。虽然该模式解决一项目组节点压力，但队列节点宕机直接导致该队列无法应用，只能等待重启。所以要想在队列节点宕机或故障也能正常应用，就要复制队列内容到集群里的每个节点，必须要创建镜像队列。**

## 镜像队列

镜像队列是基于普通的集群模式的，然后再添加一些策略，所以还是得先配置普通集群，然后才能设置镜像队列。镜像队列存在于多个节点。要实现镜像模式，需要先搭建一个普通集群模式，在这个模式的基础上再配置镜像模式以实现高可用。

### 镜像队列的结构

![rabbitmq_12](../assets/rabbitmq_12.png)

镜像队列基本上就是一个特殊的BackingQueue，它内部包裹了一个普通的BackingQueue做本地消息持久化处理，在此基础上增加了将消息和ack复制到所有镜像的功能。所有对mirror_queue_master的操作，会通过可靠组播GM的方式同步到各slave节点。GM负责消息的广播，mirror_queue_slave负责回调处理，而master上的回调处理是由coordinator负责完成。mirror_queue_slave中包含了普通的BackingQueue进行消息的存储，master节点中BackingQueue包含在mirror_queue_master中由AMQQueue进行调用。

消息的发布（除了Basic.Publish之外）与消费都是通过master节点完成。master节点对消息进行处理的同时将消息的处理动作通过GM广播给所有的slave节点，slave节点的GM收到消息后，通过回调交由mirror_queue_slave进行实际的处理。

对于Basic.Publish，消息同时发送到master和所有slave上，如果此时master宕掉了，消息还发送slave上，这样当slave提升为master的时候消息也不会丢失。

### GM（Guarenteed Multicast） 

GM模块实现的一种可靠的组播通讯协议，该协议能够保证组播消息的原子性，即保证组中活着的节点要么都收到消息要么都收不到。

它的实现大致如下：

将所有的节点形成一个循环链表，每个节点都会监控位于自己左右两边的节点，当有节点新增时，相邻的节点保证当前广播的消息会复制到新的节点上；当有节点失效时，相邻的节点会接管保证本次广播的消息会复制到所有的节点。在master节点和slave节点上的这些gm形成一个group，group（gm_group）的信息会记录在mnesia中。不同的镜像队列形成不同的group。消息从master节点对于的gm发出后，顺着链表依次传送到所有的节点，由于所有节点组成一个循环链表，master节点对应的gm最终会收到自己发送的消息，这个时候master节点就知道消息已经复制到所有的slave节点了。

#### 新增节点

新节点的加入过程如下图所示：

![rabbitmq_13](../assets/rabbitmq_13.png)

每当一个节点加入或者重新加入（例如从网络分区中恢复过来）镜像队列，之前保存的队列内容会被清空。

#### 节点的失效

如果某个slave失效了，系统处理做些记录外几乎啥都不做。master依旧是master，客户端不需要采取任何行动，或者被通知slave失效。 如果master失效了，那么slave中的一个必须被选中为master。被选中作为新的master的slave通常是最老的那个，因为最老的slave与前任master之间的同步状态应该是最好的。然而，需要注意的是，如果存在没有任何一个slave与master完全同步的情况，那么前任master中未被同步的消息将会丢失。

#### 消息的同步

将新节点加入已存在的镜像队列是，默认情况下ha-sync-mode=manual，镜像队列中的消息不会主动同步到新节点，除非显式调用同步命令。当调用同步命令后，队列开始阻塞，无法对其进行操作，直到同步完毕。

当ha-sync-mode=automatic时，新加入节点时会默认同步已知的镜像队列。由于同步过程的限制，所以不建议在生产的消费队列中操作。 



**镜像节点在集群中的其他节点拥有从队列拷贝，一旦主节点不可用，最老的从队列将被选举为新的主队列。但镜像队列不能作为负载均衡使用，因为每个操作在所有节点都要做一遍。该模式带来的副作用也很明显，除了降低系统性能外，如果镜像队列数量过多，加之大量的消息进入，集群内部的网络带宽将会被这种同步通讯大大消耗掉。所以在对可靠性要求较高的场合中适用。**

## 集群异常处理

根据节点不无安全拷贝的特性，当集群节点崩溃时，该节点队列和关联的绑定就都丢失了，附加在该队列的消费者丢失了其订阅的信息，那么怎么处理这个问题呢？

这个问题要分为两种情况：

1. 消息已经进行了持久化，那么当节点恢复，消息也恢复了。
2. 消息未持久化，可以使用下文要介绍的双活冗余队列，镜像队列保证消息的可靠性。

### 集群重启顺序

**集群重启的顺序是固定的，并且是相反的。**如下所述：

- 启动顺序：磁盘节点 => 内存节点。
- 关闭顺序：内存节点 => 磁盘节点。

**最后关闭必须是磁盘节点**，不然可能回造成集群启动失败、数据丢失等异常情况。

# 面试题

1. 避免消息堆积？

1. 1. 采用workqueue，多个消费者监听同一队列。
   2. 接收到消息以后，而是通过线程池，异步消费。

2. 如何避免消息丢失？

1. 1. 消费者的ACK机制。可以防止消费者丢失消息。

​              但是，如果在消费者消费之前，MQ就宕机了，消息就没了？

1. 1. 可以将消息进行持久化。要将消息持久化，前提是：队列、Exchange都持久化。

3. 为什么不通过TCP直接发送命令？

   对于操作系统来说创建和销毁TCP会话是非常昂贵的开销，假设高峰期每秒有成千上万条连接，每个连接都要创建一条TCP会话，这就造成了TCP连接的巨大浪费，而且操作系统每秒能创建的TCP也是有限的，因此很快就会遇到系统瓶颈。如果我们每个请求都使用一条TCP连接，既满足了性能的需要，又能确保每个连接的私密性，这就是引入信道概念的原因。

   消息发送原理：

   首先你必须连接到Rabbit才能发布和消费消息，那怎么连接和发送消息的呢？

   你的应用程序和Rabbit Server之间会创建一个TCP连接，一旦TCP打开，并通过了认证，认证就是你试图连接Rabbit之前发送的Rabbit服务器连接信息和用户名和密码，有点像程序连接数据库，使用Java有两种连接认证的方式，后面代码会详细介绍，一旦认证通过你的应用程序和Rabbit就创建了一条AMQP信道（Channel）。

   信道是创建在“真实”TCP上的虚拟连接，AMQP命令都是通过信道发送出去的，每个信道都会有一个唯一的ID，不论是发布消息，订阅队列或者介绍消息都是通过信道完成的。

4. 正常情况下，如果消息经过交换器进入队列就可以完成消息的持久化，但如果消息在没有到达broker之前出现意外，那就造成消息丢失，有没有办法可以解决这个问题？

   RabbitMQ有两种方式来解决这个问题：

1. 1. 1. 通过AMQP提供的事务机制实现。
      2. 使用发送者确认模式实现。

2. 但是这两种方法都会影响性能，事务尤为明显。

5. 如何确保消息正确地发送至RabbitMQ？

   RabbitMQ使用发送方确认模式，确保消息正确地发送到RabbitMQ。
   发送方确认模式：将信道设置成confirm模式（发送方确认模式），则所有在信道上发布的消息都会被指派一个唯一的ID。一旦消息被投递到目的队列后，或者消息被写入磁盘后（可持久化的消息），信道会发送一个确认给生产者（包含消息唯一ID）。如果RabbitMQ发生内部错误从而导致消息丢失，会发送一条nack（not acknowledged，未确认）消息。发送方确认模式是异步的，生产者应用程序在等待确认的同时，可以继续发送消息。当确认消息到达生产者应用程序，生产者应用程序的回调方法就会被触发来处理确认消息。

6. 如何确保消息接收方消费了消息？

   接收方消息确认机制：消费者接收每一条消息后都必须进行确认（消息接收和消息确认是两个不同操作）。只有消费者确认了消息，RabbitMQ才能安全地把消息从队列中删除。这里并没有用到超时机制，RabbitMQ仅通过Consumer的连接中断来确认是否需要重新发送消息。也就是说，只要连接不中断，RabbitMQ给了Consumer足够长的时间来处理消息。

   下面罗列几种特殊情况：

- - 如果消费者接收到消息，在确认之前断开了连接或取消订阅，RabbitMQ会认为消息没有被分发，然后重新分发给下一个订阅的消费者。（可能存在消息重复消费的隐患，需要根据bizId去重）
  - 如果消费者接收到消息却没有确认消息，连接也未断开，则RabbitMQ认为该消费者繁忙，将不会给该消费者分发更多的消息。

8. 如何避免消息重复投递或重复消费？

   在消息生产时，MQ内部针对每条生产者发送的消息生成一个inner-msg-id，作为去重和幂等的依据（消息投递失败并重传），避免重复的消息进入队列；在消息消费时，要求消息体中必须要有一个bizId（对于同一业务全局唯一，如支付ID、订单ID、帖子ID等）作为去重和幂等的依据，避免同一条消息被重复消费。

   这个问题针对业务场景来答分以下几点：

1. 1. 比如，你拿到这个消息做数据库的insert操作。那就容易了，给这个消息做一个唯一主键，那么就算出现重复消费的情况，就会导致主键冲突，避免数据库出现脏数据。
   2. 再比如，你拿到这个消息做redis的set的操作，那就容易了，不用解决，因为你无论set几次结果都是一样的，set操作本来就算幂等操作。
   3. 如果上面两种情况还不行，上大招。准备一个第三方介质,来做消费记录。以redis为例，给消息分配一个全局id，只要消费过该消息，将<id,message>以K-V形式写入redis。那消费者开始消费前，先去redis中查询有没消费记录即可。

9. 如何解决丢数据的问题？

- - 生产者丢数据

    生产者的消息没有投递到MQ中怎么办？从生产者弄丢数据这个角度来看，RabbitMQ提供transaction和confirm模式来确保生产者不丢消息。

    transaction机制就是说，发送消息前，开启事物(channel.txSelect())，然后发送消息，如果发送过程中出现什么异常，事物就会回滚(channel.txRollback())，如果发送成功则提交事物(channel.txCommit())。

    然而缺点就是吞吐量下降了。因此，生产上用confirm模式的居多。一旦channel进入confirm模式，所有在该信道上面发布的消息都将会被指派一个唯一的ID(从1开始)，一旦消息被投递到所有匹配的队列之后，rabbitMQ就会发送一个Ack给生产者(包含消息的唯一ID)，这就使得生产者知道消息已经正确到达目的队列了.如果rabiitMQ没能处理该消息，则会发送一个Nack消息给你，你可以进行重试操作。

- - 消息队列丢数据

    处理消息队列丢数据的情况，一般是开启持久化磁盘的配置。这个持久化配置可以和confirm机制配合使用，你可以在消息持久化磁盘后，再给生产者发送一个Ack信号。这样，如果消息持久化磁盘之前，rabbitMQ阵亡了，那么生产者收不到Ack信号，生产者会自动重发。

    那么如何持久化呢，这里顺便说一下吧，其实也很容易，就下面两步：

    1. 将queue的持久化标识durable设置为true,则代表是一个持久的队列

    2. 发送消息的时候将deliveryMode=2

       这样设置以后，rabbitMQ就算挂了，重启后也能恢复数据。在消息还没有持久化到硬盘时，可能服务已经死掉，这种情况可以通过引入mirrored-queue即镜像队列，但也不能保证消息百分百不丢失（整个集群都挂掉）

- - 消费者丢数据

    启用手动确认模式可以解决这个问题

1. 1. 1. 自动确认模式，消费者挂掉，待ack的消息回归到队列中。消费者抛出异常，消息会不断的被重发，直到处理成功。不会丢失消息，即便服务挂掉，没有处理完成的消息会重回队列，但是异常会让消息不断重试。
      2. 手动确认模式，如果消费者来不及处理就死掉时，没有响应ack时会重复发送一条信息给其他消费者；如果监听程序处理异常了，且未对异常进行捕获，会一直重复接收消息，然后一直抛异常；如果对异常进行了捕获，但是没有在finally里ack，也会一直重复发送消息(重试机制)。
      3. 不确认模式，acknowledge="none" 不使用确认机制，只要消息发送完成会立即在队列移除，无论客户端异常还是断开，只要发送完就移除，不会重发。

10. 死信队列和延迟队列的使用?

- - 死信消息：

1. 1. 1. 消息被拒绝（Basic.Reject或Basic.Nack）并且设置 requeue 参数的值为 false
      2. 消息过期了
      3. 队列达到最大的长度

- - 过期消息：

    在 rabbitmq 中存在2种方可设置消息的过期时间，第一种通过对队列进行设置，这种设置后，该队列中所有的消息都存在相同的过期时间，第二种通过对消息本身进行设置，那么每条消息的过期时间都不一样。如果同时使用这2种方法，那么以过期时间小的那个数值为准。当消息达到过期时间还没有被消费，那么那个消息就成为了一个 死信 消息。

    队列设置：在队列申明的时候使用 x-message-ttl 参数，单位为 毫秒

    单个消息设置：是设置消息属性的 expiration 参数的值，单位为 毫秒

    延时队列：在rabbitmq中不存在延时队列，但是我们可以通过设置消息的过期时间和死信队列来模拟出延时队列。消费者监听死信交换器绑定的队列，而不要监听消息发送的队列。

    有了以上的基础知识，我们完成以下需求：

    需求：用户在系统中创建一个订单，如果超过时间用户没有进行支付，那么自动取消订单。

    分析：

    1. 上面这个情况，我们就适合使用延时队列来实现，那么延时队列如何创建
    2. 延时队列可以由 过期消息+死信队列 来时间
    3. 过期消息通过队列中设置 x-message-ttl 参数实现
    4. 死信队列通过在队列申明时，给队列设置 x-dead-letter-exchange 参数，然后另外申明一个队列绑定x-dead-letter-exchange对应的交换器。

11. 多个消费者监听一个队列时，消息如何分发?

- - 轮询: 默认的策略，消费者轮流，平均地接收消息
  - 公平分发: 根据消费者的能力来分发消息，给空闲的消费者发送更多消息

12. 消息幂等性

    生产者方面：可以对每条消息生成一个msgID，以控制消息重复投递

    消费者方面：消息体中必须携带一个业务ID，如银行流水号，消费者可以根据业务ID去重，避免重复消费

13. 如何保证消息的顺序性

    一个队列只有一个消费者的情况下才能保证顺序，否则只能通过全局ID实现（每条消息都一个msgId，关联的消息拥有一个parentMsgId。可以在消费端实现前一条消息未消费，不处理下一条消息；也可以在生产端实现前一条消息未处理完毕，不发布下一条消息）

# 参考文档

[1]: https://blog.csdn.net/kavito/article/details/91403659
[2]: https://www.cnblogs.com/vipstone/p/9368106.html
[3]: https://rdc.hundsun.com/portal/article/937.html
[4]: https://www.cnblogs.com/coder-programming/p/12465314.html
[5]: https://www.rabbitmq.com/tutorials/tutorial-one-go.html