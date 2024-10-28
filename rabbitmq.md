# rabbitmq

docker部署

```bash
docker run -d --name rabbitmq \
  -p 5672:5672 \
  -p 15672:15672 \
  -e RABBITMQ_DEFAULT_USER=admin \
  -e RABBITMQ_DEFAULT_PASS=admin \
  -v rabbitmq_data:/var/lib/rabbitmq \
  rabbitmq:management
```

端口：ip:15672

## 快速入门

1.添加队列

<img src="C:\Users\jqdsg\AppData\Local\Packages\MicrosoftWindows.Client.CBS_cw5n1h2txyewy\TempState\ScreenClip\{E632B024-D064-478A-9485-B42FB282C67A}.png" alt="{E632B024-D064-478A-9485-B42FB282C67A}" style="zoom:25%;" />

2.在交换机中进行bind

<img src="C:\Users\jqdsg\AppData\Local\Packages\MicrosoftWindows.Client.CBS_cw5n1h2txyewy\TempState\ScreenClip\{7903EA6F-05BF-4761-B48B-6AC5225FC2B1}.png" alt="{7903EA6F-05BF-4761-B48B-6AC5225FC2B1}" style="zoom:25%;" />

3.发送消息

<img src="C:\Users\jqdsg\AppData\Local\Packages\MicrosoftWindows.Client.CBS_cw5n1h2txyewy\TempState\ScreenClip\{91BF0CD8-B63F-4E87-8E34-31F2CC4FD21A}.png" alt="{91BF0CD8-B63F-4E87-8E34-31F2CC4FD21A}" style="zoom:25%;" />

4.队列中查看消息

<img src="C:\Users\jqdsg\AppData\Local\Packages\MicrosoftWindows.Client.CBS_cw5n1h2txyewy\TempState\ScreenClip\{B6EEBD38-73EA-43C4-B119-BAE1B3FB0A78}.png" alt="{B6EEBD38-73EA-43C4-B119-BAE1B3FB0A78}" style="zoom:25%;" />

### 数据隔离



## java客户端

### 快速入门

<img src="C:\Users\jqdsg\AppData\Local\Packages\MicrosoftWindows.Client.CBS_cw5n1h2txyewy\TempState\ScreenClip\{23EC0125-FC20-4BFC-9BDC-A3E4B8BD8B6B}.png" alt="{23EC0125-FC20-4BFC-9BDC-A3E4B8BD8B6B}" style="zoom:25%;" />

1.创建队列

<img src="C:\Users\jqdsg\AppData\Local\Packages\MicrosoftWindows.Client.CBS_cw5n1h2txyewy\TempState\ScreenClip\{0454DB58-0D93-4C37-8F89-B1C735089E9E}.png" alt="{0454DB58-0D93-4C37-8F89-B1C735089E9E}" style="zoom:25%;" />

2.配置rabbitmq服务段信息

```yaml
spring:
  rabbitmq:
    host: 192.168.88.120 # rabbitMQ的ip地址
    port: 5672 # 端口
    username: hmall
    password: 123
    virtual-host: /hmall
```

3.发送消息

SpringAMQP提供了RabbitTemplate工具类，方便我们发送消息。

```java
 @Autowired
    private RabbitTemplate rabbitTemplate;
 
    @Test
    public void testSimpleQueue() {
        // 队列名称
        String queueName = "simple.queue";
        // 消息
        String message = "hello, spring amqp!";
        // 发送消息
        rabbitTemplate.convertAndSend(queueName, message);
    }
```

4.接受消息

SpringAMQP提供声明式的消息监听，我们只需要通过注解在方法上的声明要监听的队列名称，将来SpringAMQP就会把消息传递给当前方法：

<img src="C:\Users\jqdsg\AppData\Local\Packages\MicrosoftWindows.Client.CBS_cw5n1h2txyewy\TempState\ScreenClip\{7BB9F405-87BB-4FA3-800F-1101F9AAFA37}.png" alt="{7BB9F405-87BB-4FA3-800F-1101F9AAFA37}" style="zoom:50%;" />

### workQueues模型

Work queues，任务模型。简单来说就是**让多个消费者绑定到一个队列，共同消费队列中的消息**。

![img](https://i-blog.csdnimg.cn/blog_migrate/26f19b1ef8dc30fbac9aed770faed436.png)

当消息处理比较耗时的时候，可能生产消息的速度会远远大于消息的消费速度。长此以往，消息就会堆积越来越多，无法及时处理。 此时就可以使用work 模型，**多个消费者共同处理消息处理，消息处理的速度就能大大提高**了。

**模拟场景**：创建一个新的队列，命名为work.queue

![image.png](https://i-blog.csdnimg.cn/blog_migrate/2cc6fc4e7b7fac62b1ad879af67b6cee.png)

1.消息发送

这次我们往队列中循环发送，模拟出一个**大量消息堆积**的队列。

```java
   @Test
    public void testSendMessage2WorkQueue() throws InterruptedException {
        String queueName = "work.queue";
        String message = "hello, work,message_";
        for (int i = 1; i <= 50; i++) {
            rabbitTemplate.convertAndSend(queueName, message + i);
            Thread.sleep(20);
        }
    }
```

2.消息接受

要模拟多个消费者绑定同一个队列，我们在consumer服务的SpringRabbitListener中添加2个新的方法：

```java
    @RabbitListener(queues = "work.queue")
    public void listenWorkQueue1(String msg) throws InterruptedException {
        System.out.println("消费者1接收到消息：【" + msg + "】" + LocalTime.now());
        Thread.sleep(20);
    }

    @RabbitListener(queues = "work.queue")
    public void listenWorkQueue2(String msg) throws InterruptedException {
        System.err.println("消费者2........接收到消息：【" + msg + "】" + LocalTime.now());
        Thread.sleep(200);
    }
```

注意到这两消费者，都设置了`Thead.sleep`，模拟任务耗时：

- **消费者1 sleep了20毫秒，相当于每秒钟处理50个消息**
- **消费者2 sleep了200毫秒，相当于每秒处理5个消息**

3.**测试**

启动ConsumerApplication后，在执行publisher服务中刚刚编写的发送测试方法testWorkQueue。 最终结果如下：

消费者1和消费者2竟然**每人消费了25条消息**：

- 消费者1很快完成了自己的25条消息
- 消费者2却在缓慢的处理自己的25条消息。

也就是说消息是**平均分配**给每个消费者，并没有考虑到消费者的处理能力。导致1个消费者空闲，另一个消费者忙的不可开交。**没有充分利用每一个消费者的能力**，最终消息处理的耗时远远超过了1秒。这样显然是有问题的。

4.能者多劳

在spring中有一个简单的配置，可以解决这个问题。我们修改**consumer服务**的application.yml文件，添加配置：

```yaml
spring:
  rabbitmq:
    listener:
      simple:
        prefetch: 1 # 每次只能获取一条消息，处理完成才能获取下一个消息
```

在配置中，`prefetch: 1`表示每个消费者每次只能从队列中预取1个消息，消费完就能拿下一次，不需要等轮询。它可以帮助保证每个消息在被消费者处理时都能得到较为均匀的分配，避免某个消费者处理速度慢而导致其他消费者空闲的情况。如果不配置着东东的话，那么rabbitmq采用的就是一个公平轮询的方式，将消息依次发给一个消费，等他消费完了再发下一个给另外的消费者
**再次测试**

![{413B1DF9-1ECA-4AB1-8119-8CF855F3B2B0}](C:\Users\jqdsg\AppData\Local\Packages\MicrosoftWindows.Client.CBS_cw5n1h2txyewy\TempState\ScreenClip\{413B1DF9-1ECA-4AB1-8119-8CF855F3B2B0}.png)

#### **Work模型的使用**

- 多个消费者绑定到一个队列，可以加快消息处理速度
- 同一条消息只会被一个消费者处理
- 通过设置prefetch来控制消费者预取的消息数量，处理完一条再处理下一条，实现能者多劳

### 交换机类型

在之前的两个测试案例中，都没有交换机，生产者直接发送消息到队列。而一旦引入交换机，消息发送的模式会有很大变化：

![img](https://i-blog.csdnimg.cn/blog_migrate/f67219aaeca8078d4c8d6bd1dab6aca5.png)

可以看到，在订阅模型中，多了一个exchange角色，而且过程略有变化：

- **Publisher**：生产者，不再发送消息到队列中，而是发给交换机
- **Exchange**：交换机，一方面，接收生产者发送的消息。另一方面，知道如何处理消息，例如递交给某个特别队列、递交给所有队列、或是将消息丢弃。到底如何操作，取决于Exchange的类型。
- **Queue**：消息队列也与以前一样，接收消息、缓存消息。不过队列一定要与交换机绑定。
- **Consumer**：消费者，与以前一样，订阅队列，没有变化

**Exchange（交换机）只负责转发消息，不具备存储消息的能力，因此如果没有任何队列与Exchange绑定，或者没有符合路由规则的队列，那么消息会丢失！**

**交换机的类型有四种：**

Fanout：广播，将消息交给所有绑定到交换机的队列。我们最早在控制台使用的正是Fanout交换机

Direct：订阅，基于RoutingKey（路由key）发送给订阅了消息的队列

Topic：通配符订阅，与Direct类似，只不过RoutingKey可以使用通配符

Headers：头匹配，基于MQ的消息头匹配，用的较少。

### Fanout交换机

Fanout，英文翻译是扇出，我觉得在MQ中叫广播更合适。 在广播模式下，消息发送流程是这样的：

**![img](https://i-blog.csdnimg.cn/blog_migrate/8aca1936f6a747ef1adcd6d565065013.png)**

- 1）  可以有多个队列
- 2）  每个队列都要绑定到Exchange（交换机）
- 3）  生产者发送的消息，只能发送到交换机
- 4）  交换机把消息发送给绑定过的***\*所有队列\****
- 5）  订阅队列的消费者**都能拿到消息**

![{350508C1-8708-40CB-A35F-0DB7FF33342B}](C:\Users\jqdsg\AppData\Local\Packages\MicrosoftWindows.Client.CBS_cw5n1h2txyewy\TempState\ScreenClip\{350508C1-8708-40CB-A35F-0DB7FF33342B}.png)

1.创建两个队列

![{74C85768-3087-4B3F-97EE-39CBEE66E160}](C:\Users\jqdsg\AppData\Local\Packages\MicrosoftWindows.Client.CBS_cw5n1h2txyewy\TempState\ScreenClip\{74C85768-3087-4B3F-97EE-39CBEE66E160}.png)

2.创建一个fanout类型的交换机，命名为hmall.fanout,并bind两个队列

![{CE419636-6EC7-40A4-AA54-FDAD45B85C57}](C:\Users\jqdsg\AppData\Local\Packages\MicrosoftWindows.Client.CBS_cw5n1h2txyewy\TempState\ScreenClip\{CE419636-6EC7-40A4-AA54-FDAD45B85C57}.png)

3.写两个消费者方法

```java

    @RabbitListener(queues = "fanout.queue1")
    public void listenFanoutQueue1(String msg) {
        System.out.println("消费者接收到fanout.queue1的消息：【" + msg + "】");
    }
    @RabbitListener(queues = "fanout.queue2")
    public void listenFanoutQueue2(String msg) {
        System.out.println("消费者接收到fanout.queue2的消息：【" + msg + "】");
    }
```

4.测试方法

```java
@Test
public void testSendFanoutExchange() {
    // 交换机名称
    String exchangeName = "hmall.fanout";
    // 消息
    String message = "hello, every one!";
    // 发送消息
    rabbitTemplate.convertAndSend(exchangeName, "", message);
}
```

#### 总结

交换机的作用是什么？

- 接收publisher发送的消息
- 将消息按照规则路由到与之绑定的队列
- 不能缓存消息，路由失败，消息丢失
- FanoutExchange的会将消息路由到每个绑定的队列

### Direct交换机

在Fanout模式中，一条消息，会被所有订阅的队列都消费。但是，在某些场景下，我们希望不同的消息被不同的队列消费。这时就要用到Direct类型的Exchange。

![image.png](https://i-blog.csdnimg.cn/blog_migrate/46c9526dce1633e037d50e2a546c2e19.png)

在Direct模型下：

- 队列与交换机的绑定，不能是任意绑定了，而是要指定一个RoutingKey（路由key）

- 消息的发送方在 向 Exchange发送消息时，也必须指定消息的 RoutingKey。

- Exchange不再把消息交给每一个绑定的队列，而是根据消息的Routing Key进行判断，只有队列的Routingkey与消息的 Routing key完全一致，才会接收到消息

![{B7F7E148-BED8-4F02-ABB0-8CE60BC4416A}](C:\Users\jqdsg\AppData\Local\Packages\MicrosoftWindows.Client.CBS_cw5n1h2txyewy\TempState\ScreenClip\{B7F7E148-BED8-4F02-ABB0-8CE60BC4416A}.png)



创建队列，已经交换机，进行bind

![{96B1772D-F464-4C8C-932B-8B89B6915CF1}](C:\Users\jqdsg\AppData\Local\Packages\MicrosoftWindows.Client.CBS_cw5n1h2txyewy\TempState\ScreenClip\{96B1772D-F464-4C8C-932B-8B89B6915CF1}.png)

3.消息接收

```java
@RabbitListener(bindings = @QueueBinding(
        value = @Queue(name = "direct.queue1"),
        exchange = @Exchange(name = "hmall.direct", type = ExchangeTypes.DIRECT),
        key = {"red", "blue"}
))
public void listenDirectQueue1(String msg){
    System.out.println("消费者接收到direct.queue1的消息：【" + msg + "】");
}

@RabbitListener(bindings = @QueueBinding(
        value = @Queue(name = "direct.queue2"),
        exchange = @Exchange(name = "hmall.direct", type = ExchangeTypes.DIRECT),
        key = {"red", "yellow"}
))
```

4.消息发送

```java
@Test
public void testSendDirectExchange() {
    // 交换机名称
    String exchangeName = "hmall.direct";
    // 消息
    String message = "hello, red!";
    // 发送消息
    rabbitTemplate.convertAndSend(exchangeName, "blue", message);
}
```

#### 总结

描述下Direct交换机与Fanout交换机的差异？

- Fanout交换机将消息路由给每一个与之绑定的队列
- Direct交换机根据RoutingKey判断路由给哪个队列
- 如果多个队列具有相同的RoutingKey，则与Fanout功能类似

### Topic交换机

尽管使用 direct 交换机改进了我们的系统，但是它仍然存在局限性——比方说我们的交换机绑定了多个不同的routingKey，在direct模式中虽然能做到有选择性地接收日志，但是它的选择性是单一的，就是说我的一条消息，只能被一个相同routingKey的绑定缩消费，但是如果我们想要在让它的选择性变得多元，比如划分一个子组，一个消息可以根据一个组别的队列进行投递，就需要用到Topics模式 

------

Topic和Direct类似，区别在于routingkey可以是多个单词的列表，并且以.分割。

Queue与Exchange指定Bindingkey时可以使用通配符：

- **#**:代指0个或多个单词
- *****：代指一个单词

举例：

- `item.#`：能够匹配`item.spu.insert` 或者 `item.spu`
- `item.*`：只能匹配`item.`

![img](https://i-blog.csdnimg.cn/blog_migrate/66d8fe447658fe8f1937e65705f1954d.png)

![{6A0E1742-6D1D-4313-855D-CDDE8C40926B}](C:\Users\jqdsg\AppData\Local\Packages\MicrosoftWindows.Client.CBS_cw5n1h2txyewy\TempState\ScreenClip\{6A0E1742-6D1D-4313-855D-CDDE8C40926B}.png)

consumer

```java

    @RabbitListener(bindings = @QueueBinding(
            value = @Queue(name = "topic.queue1"),
            exchange = @Exchange(name = "hmall.topic", type = ExchangeTypes.TOPIC),
            key = "china.#"
    ))
    public void listenTopicQueue1(String msg){
        System.out.println("消费者接收到topic.queue1的消息：【" + msg + "】");
    }

    @RabbitListener(bindings = @QueueBinding(
            value = @Queue(name = "topic.queue2"),
            exchange = @Exchange(name = "hmall.topic", type = ExchangeTypes.TOPIC),
            key = "#.news"
    ))
    public void listenTopicQueue2(String msg){
        System.out.println("消费者接收到topic.queue2的消息：【" + msg + "】");
    }
```

**测试方法**

```java
 @Test
    public void testSendTopicExchange() {
        // 交换机名称
        String exchangeName = "hmall.topic";
        // 消息
        String message = "今天天气不错，我的心情好极了!";
        // 发送消息
        rabbitTemplate.convertAndSend(exchangeName, "china.weather", message);
    }
```

#### 总结

描述下Direct交换机与Topic交换机的差异？

- Topic交换机接收的消息RoutingKey必须是多个单词，以 `**.**` 分割
- Topic交换机与队列绑定时的bindingKey可以指定通配符
- `#`：代表0个或多个词
- `*`：代表1个词

### 声明队列交换机

在之前我们都是基于RabbitMQ控制台来创建队列、交换机。但是在实际开发时，队列和交换机是程序员定义的，将来项目上线，又要交给运维去创建。那么程序员就需要把程序中运行的所有队列和交换机都写下来，交给运维。在这个过程中是很容易出现错误的。 因此推荐的做法是由程序启动时检查队列和交换机是否存在，如果不存在自动创建。

#### 基本API

SpringAMQP提供了几个类，用来声明队列、交换机及其绑定关系：

- Queue:用于声明队列，可以用工厂类QueueBuilder构建
- Exchange:用于声明交换机，可以用工厂类ExchangeBuilder构建
- Binding:用于声明队列和交换机的绑定关系，可以用BindingBuilder构建

SpringAMQP提供了一个Queue类，用来创建队列：

![image.png](https://i-blog.csdnimg.cn/blog_migrate/ae15eb6fc6e3e5a75b06a5abff3b5bc6.png)

SpringAMQP还提供了一个Exchange接口，来表示所有不同类型的交换机：

![image.png](https://i-blog.csdnimg.cn/blog_migrate/0594519178f9f6876955d830d2c6bb9c.png)

我们可以自己创建队列和交换机，不过SpringAMQP还提供了**ExchangeBuilder**来简化这个过程：

![image.png](https://i-blog.csdnimg.cn/blog_migrate/fd7f86bbcf340b22de8b5bcd5c3182df.png)

而在绑定队列和交换机时，则需要使用BindingBuilder来创建Binding对象：

![image.png](https://i-blog.csdnimg.cn/blog_migrate/05a74510595e3e40d273189d29ee59fb.png)

例如，声明一个Fanout类型的交换机，并且创建队列与绑定：

```java
@Configuration
public class FanoutConfig {
    /**
     * 声明交换机
     * @return Fanout类型交换机
     */
    @Bean
    public FanoutExchange fanoutExchange(){
        return new FanoutExchange("hmall.fanout");
    }
 
    /**
     * 第1个队列
     */
    @Bean
    public Queue fanoutQueue1(){
        return new Queue("fanout.queue1");
    }
 
    /**
     * 绑定队列和交换机
     */
    @Bean
    public Binding bindingQueue1(Queue fanoutQueue1, FanoutExchange fanoutExchange){
        return BindingBuilder.bind(fanoutQueue1).to(fanoutExchange);
    }
}
```

```java
 // hmall.fanout
    @Bean
    public FanoutExchange fanoutExchange(){
//        ExchangeBuilder.fanoutExchange("").build();
        return new FanoutExchange("hmall.fanout2");
    }

    // fanout.queue1
    @Bean
    public Queue fanoutQueue3(){
//        QueueBuilder.durable("fanout.queue3");//持久化
        return new Queue("fanout.queue3");
    }

    // 绑定队列3到交换机
    @Bean
    public Binding fanoutBinding3(Queue fanoutQueue3, FanoutExchange fanoutExchange){
        return BindingBuilder
                .bind(fanoutQueue3)
                .to(fanoutExchange);
    }

    // fanout.queue2
    @Bean
    public Queue fanoutQueue4(){
        return new Queue("fanout.queue4");
    }

    // 绑定队列2到交换机
    @Bean
    public Binding fanoutBinding4(){
        return BindingBuilder
                .bind(fanoutQueue4())
                .to(fanoutExchange());
    }
```

#### 基于注解声明

基于@Bean的方式声明队列和交换机比较麻烦，Spring还提供了基于注解方式来声明。

![{7E1A0418-B847-4D9D-8C07-7C8077590AF9}](C:\Users\jqdsg\AppData\Local\Packages\MicrosoftWindows.Client.CBS_cw5n1h2txyewy\TempState\ScreenClip\{7E1A0418-B847-4D9D-8C07-7C8077590AF9}.png)

例如，我们同样声明Direct模式的交换机和队列：

```java
  @RabbitListener(bindings = @QueueBinding(
            value = @Queue(name = "direct.queue1",durable = "true"),
            exchange = @Exchange(name = "hmall.direct", type = ExchangeTypes.DIRECT),
            key = {"red", "blue"}
    ))
    public void listenDirectQueue1(String msg){
        System.out.println("消费者接收到direct.queue1的消息：【" + msg + "】");
    }

    @RabbitListener(bindings = @QueueBinding(
            value = @Queue(name = "direct.queue2",durable = "true"),
            exchange = @Exchange(name = "hmall.direct", type = ExchangeTypes.DIRECT),
            key = {"red", "yellow"}
    ))
    public void listenDirectQueue2(String msg){
        System.out.println("消费者接收到direct.queue2的消息：【" + msg + "】");
    }

```

#### 消息转换器

![{96F72BA7-C464-4D9D-A8B5-B6E1EC0EBDF1}](C:\Users\jqdsg\AppData\Local\Packages\MicrosoftWindows.Client.CBS_cw5n1h2txyewy\TempState\ScreenClip\{96F72BA7-C464-4D9D-A8B5-B6E1EC0EBDF1}.png)

而在数据传输时，它会把你发送的消息序列化为字节发送给MQ，接收消息的时候，还会把字节反序列化为Java对象。 只不过，默认情况下Spring采用的序列化方式是**JDK序列化**。众所周知，JDK序列化存在下列问题：

- 数据体积过大
- 有安全漏洞
- 可读性差

建议采用JSON序列化代替默认的JDK序列化，要做两件事情：

在`publisher`和`consumer`两个服务中都引入jackson依赖：

***\*注意，如果项目中引入了`spring-boot-starter-web`依赖，则无需再次引入`Jackson`依赖。\****

```
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
        </dependency>
```

在`publisher`和`consumer`中都要配置MessageConverter:

配置消息转换器，在`publisher`和`consumer`两个服务的启动类中添加一个Bean即可：

```java
  @Bean
    public MessageConverter messageConverter(){
        return new Jackson2JsonMessageConverter();
    }
```



**收消息**

```java
  @RabbitListener(queues = "object.queue")
    public void listenObjectQueue(Map<String,Object> msg){
        System.out.println("接收到object.queue的消息：" + msg);
    }
```

# MQ高级

## 消息可靠性问题

### 发送者的可靠性

#### 生产者重连

有的时候由于网络波动，可能会出现客户端连接MQ失败的情况。通过配置我们可以开启连接失败后的重连机制：

![{11DFBE56-BE25-42EE-8D46-5F4EF1DB5B5A}](C:\Users\jqdsg\AppData\Local\Packages\MicrosoftWindows.Client.CBS_cw5n1h2txyewy\TempState\ScreenClip\{11DFBE56-BE25-42EE-8D46-5F4EF1DB5B5A}.png)

```
spring:
  rabbitmq:
    host: 192.168.88.120 # rabbitMQ的ip地址
    port: 5672 # 端口
    username: hmall
    password: 123
    virtual-host: /hmall
    connection-timeout: 1s
    template:
      retry:
        enabled: true
        multiplier: 1
        max-attempts: 3
        initial-interval: 1000ms
```



#### 生产者确认

RabbitMQ有Publisher Confirm和Publisher Return两种确认机制。开启确认机制后，在MQ成功收到消息后会返回确认消息给生产者。返回的结果有以下几种情况：

- 消息投递到了MQ，但是路由失败。此时会通过pubulisherReturn返回路由异常，然后返回ACK，告知投递成功
- **临时**消息投递到了MQ，并且入队成功，返回ACK，告知投递成功
- 持久消息投递了MQ，并且入队完成持久化，返回ACK，告知投递成功
- 其它情况都会返回**NACK**，告知投递失败

![{B7D1742C-6898-4A0E-9E66-69604B3A78DD}](C:\Users\jqdsg\AppData\Local\Packages\MicrosoftWindows.Client.CBS_cw5n1h2txyewy\TempState\ScreenClip\{B7D1742C-6898-4A0E-9E66-69604B3A78DD}.png)

SpringAMQP实现生产者确认

1.在publisher这个微服务的application.yml中添加配置

![{ABD9EFA3-B85C-404B-A6A1-E6240ED7A828}](C:\Users\jqdsg\AppData\Local\Packages\MicrosoftWindows.Client.CBS_cw5n1h2txyewy\TempState\ScreenClip\{ABD9EFA3-B85C-404B-A6A1-E6240ED7A828}.png)

配置说明：

- 这里publisher-confirm-type有三种模式可选：
  - none：关闭confirm机制
  - simple：同步阻塞等待MQ的回执消息
  - correlated：MQ异步回调方式放回回执消息

2.每个RabbitTemplate只能配置一个ReturnCallback,因此需要在项目启动过程中配置：

![{86DEF507-02C5-442C-8718-0C20BF11BB1B}](C:\Users\jqdsg\AppData\Local\Packages\MicrosoftWindows.Client.CBS_cw5n1h2txyewy\TempState\ScreenClip\{86DEF507-02C5-442C-8718-0C20BF11BB1B}.png)

3.发送消息，指定消息id，消息ConfirmCallback

![{F00CC60C-59AE-4B21-9B0E-EB511061B05D}](C:\Users\jqdsg\AppData\Local\Packages\MicrosoftWindows.Client.CBS_cw5n1h2txyewy\TempState\ScreenClip\{F00CC60C-59AE-4B21-9B0E-EB511061B05D}.png)

##### 总结：

**SpringAMQP中生产者消息确认的几种返回值情况：**

- 消息投递到了MQ，但是路由失败。此时会通过pubulisherReturn返回路由异常，然后返回ACK，告知投递成功
- **临时**消息投递到了MQ，并且入队成功，返回ACK，告知投递成功
- 持久消息投递了MQ，并且入队完成持久化，返回ACK，告知投递成功
- 其它情况都会返回**NACK**，告知投递失败

**如何处理生产者的确认消息？**

- 生产者确认需要额外的网络和系统资源开销，尽量不要使用
- 如果一定要使用，无需开启publisher-Retrun机制，因为一般路由失败是自己业务问题
- 对于nack消息可以有限次数重试，依然失败则记录异常消息

## MQ的可靠性

在默认情况下，RabbitMQ会将接收到的信息保存在内存中以降低消息收发的延迟。这样会导致两个问题：

- 一旦MQ宕机，内存中的消息会丢失
- 内存空间有限，当消费者故障或处理过慢时，会导致消息积压，引发MQ阻塞

### 数据持久化

为了提升性能，默认情况下MQ的数据都是在内存存储的临时数据，重启后就会消失。为了保证数据的可靠性，必须配置数据持久化，包括：

- 交换机持久化
- 队列持久化
- 消息持久化

我们以控制台界面为例来说明。

在控制台发送消息的时候，可以添加很多参数，而消息的持久化是要配置一个`properties`：

![img](https://i-blog.csdnimg.cn/blog_migrate/0657679f9854b9ea265d4b652112353e.png)

说明：在开启持久化机制以后，如果同时还开启了生产者确认，那么MQ会在消息持久化以后才发送ACK回执，进一步确保消息的可靠性。不过出于性能考虑，为了减少IO次数，发送到MQ的消息并不是逐条持久化到数据库的，而是每隔一段时间批量持久化。一般间隔在100毫秒左右，这就会导致ACK有一定的延迟，因此建议生产者确认全部采用异步方式。



### Lazy Queue

在默认情况下，RabbitMQ会将接收到的信息保存在内存中以降低消息收发的延迟。但在某些特殊情况下，这会导致消息积压，比如：

- 消费者宕机或出现网络故障

- 消息发送量激增，超过了消费者处理速度

- 消费者处理业务发生阻塞


一旦出现消息堆积问题，RabbitMQ的内存占用就会越来越高，直到触发内存预警上限。此时RabbitMQ会将内存消息刷到磁盘上，这个行为成为PageOut. PageOut会耗费一段时间，并且会阻塞队列进程。因此在这个过程中RabbitMQ不会再处理新的消息，生产者的所有请求都会被阻塞。


从RabbitMQ的3.6.0版本开始，就增加了Lazy Queues的模式，也就是惰性队列。惰性队列的特征如下：

接收到消息后直接存入磁盘而非内存

消费者要消费消息时才会从磁盘中读取并加载到内存（也就是懒加载）

支持数百万条的消息存储

而在3.12版本之后，LazyQueue已经成为所有队列的默认格式。因此官方推荐升级MQ为3.12版本或者所有队列都设置为LazyQueue模式。

#### 配置Lazy模式

##### 控制台配置Lazy模式

在添加队列的时候，添加`x-queue-mod=lazy`参数即可设置队列为Lazy模式：

![img](https://i-blog.csdnimg.cn/blog_migrate/065b9783ed9214e9f2cd53762cecd12c.png)

##### 代码配置Lazy模式

在利用SpringAMQP声明队列的时候，添加`x-queue-mod=lazy`参数也可设置队列为Lazy模式：

```java
@Bean
public Queue lazyQueue(){
    return QueueBuilder
            .durable("lazy.queue")
            .lazy() // 开启Lazy模式
            .build();
}
```

当然，我们也可以基于注解来声明队列并设置为Lazy模式：

```java
@RabbitListener(queuesToDeclare = @Queue(
        name = "lazy.queue",
        durable = "true",
        arguments = @Argument(name = "x-queue-mode", value = "lazy")
))
public void listenLazyQueue(String msg){
    log.info("接收到 lazy.queue的消息：{}", msg);
}
```

##### 性能测试

```java
    @Test
    public void testPageOut(){
        Message message = MessageBuilder.withBody("hello".getBytes(StandardCharsets.UTF_8)).setDeliveryMode(MessageDeliveryMode.NON_PERSISTENT).build();
        for (int i = 0; i < 1000000; i++) {
            rabbitTemplate.convertAndSend("lazy.queue",message);
        }
    }
```

### 总结

RabbitMQ如何保证消息的可靠性

- 首先通过配置可以让交换机、队列、以及发送的消息都持久化。这样队列中的消息会持久化到磁盘，MQ重启消息依然存在
- RabbitMQ在3.6引入了Lazy Queue，并且在3.12版本后会成为队列的默认模式。LazyQueue会将所有的消息都持久化。
- 开启持久化和生产者确认时，RabbitMQ只有在消息持久化完成后才会给生产者返回ACK回执

## 消费者的可靠性

当RabbitMQ向消费者投递消息以后，需要知道消费者的处理状态如何。因为消息投递给消费者并不代表就一定被正确消费了，可能出现的故障有很多，比如：

消息投递的过程中出现了网络故障

消费者接收到消息后突然宕机

消费者接收到消息后，因处理不当导致异常

...

一旦发生上述情况，消息也会丢失。因此，RabbitMQ必须知道消费者的处理状态，一旦消息处理失败才能重新投递消息。但问题来了：RabbitMQ如何得知消费者的处理状态呢？

### 消费者确认机制

为了确认消费者是否成功处理消息，RabbitMQ提供了消费者确认机制（Consumer Acknowledgement）。即：当消费者处理消息结束后，应该向RabbitMQ发送一个回执，告知RabbitMQ自己消息处理状态。回执有三种可选值：

ack：成功处理消息，RabbitMQ从队列中删除该消息

nack：消息处理失败，RabbitMQ需要再次投递消息

reject：消息处理失败并拒绝该消息，RabbitMQ从队列中删除该消息

一般reject方式用的较少，除非是消息格式有问题，那就是开发问题了。因此大多数情况下我们需要将消息处理的代码通过try catch机制捕获，消息处理成功时返回ack，处理失败时返回nack.
![{CF05F169-0A70-4944-98F2-3CECC46A7206}](C:\Users\jqdsg\AppData\Local\Packages\MicrosoftWindows.Client.CBS_cw5n1h2txyewy\TempState\ScreenClip\{CF05F169-0A70-4944-98F2-3CECC46A7206}.png)



SpringAMQP帮我们实现了消息确认。并允许我们通过配置文件设置ACK处理方式，有三种模式：

- none：不处理。即消息投递给消费者后立刻ack，消息会立刻从MQ删除。非常不安全，不建议使用

- manual：手动模式。需要自己在业务代码中调用api，发送ack或reject，存在业务入侵，但更灵活

- auto：自动模式。SpringAMQP利用AOP对我们的消息处理逻辑做了环绕增强，当业务正常执行时则自动返回ack. 

当业务出现异常时，根据异常判断返回不同结果：

- 如果是业务异常，会自动返回nack；

- 如果是消息处理或校验异常，自动返回reject;

```
spring:
  rabbitmq:
    listener:
      simple:
        acknowledge-mode: auto # 自动ack
```

在异常位置打断点，再次发送消息，程序卡在断点时，可以发现此时消息状态为`unacked`（未确定状态）：

![img](https://i-blog.csdnimg.cn/blog_migrate/67cde063e7aef4d0c50e596c7a345d20.png)

放行以后，由于抛出的是**消息转换异常**，因此Spring会自动返回`reject`，所以消息依然会被删除：

![img](https://i-blog.csdnimg.cn/blog_migrate/47b2df5b634080378eecb178dcd00066.png)

我们将异常改为RuntimeException类型：

```java
@RabbitListener(queues = "simple.queue")
public void listenSimpleQueueMessage(String msg) throws InterruptedException {
    log.info("spring 消费者接收到消息：【" + msg + "】");
    if (true) {
        throw new RuntimeException("故意的");
    }
    log.info("消息处理完成");
}
```

在异常位置打断点，然后再次发送消息测试，程序卡在断点时，可以发现此时消息状态为`unacked`（未确定状态）：

![img](https://i-blog.csdnimg.cn/blog_migrate/d632cb30769007cfbdfc8ec93dba06a5.png)

放行以后，由于抛出的是业务异常，所以Spring返回`ack`，最终消息恢复至`Ready`状态，并且没有被RabbitMQ删除：

![img](https://i-blog.csdnimg.cn/blog_migrate/bbe3d49803216b51040917ac13fa712f.png)

当我们把配置改为`auto`时，消息处理失败后，会回到RabbitMQ，并重新投递到消费者。

### 消费失败处理

当消费者出现异常后，消息会不断requeue（重入队）到队列，再重新发送给消费者。如果消费者再次执行依然出错，消息会再次requeue到队列，再次投递，直到消息处理成功为止。极端情况就是消费者一直无法执行成功，那么消息requeue就会无限循环，导致mq的消息处理飙升，带来不必要的压力：
我们可以利用Spring的retry机制，在消费者出现异常时利用本地重试，而不是无限制的requeue到mq队列

```
spring:
  rabbitmq:
    listener:
      simple:
        retry:
          enabled: true # 开启消费者失败重试
          initial-interval: 1000ms # 初识的失败等待时长为1秒
          multiplier: 1 # 失败的等待时长倍数，下次等待时长 = multiplier * last-interval
          max-attempts: 3 # 最大重试次数
          stateless: true # true无状态；false有状态。如果业务中包含事务，这里改为false
```

重启consumer服务，重复之前的测试。可以发现：

- 消费者在失败后消息没有重新回到MQ无限重新投递，而是在本地重试了3次

- 本地重试3次以后，抛出了AmqpRejectAndDontRequeueException异常。查看RabbitMQ控制台，发现消息被删除了，说明最后SpringAMQP返回的是reject


结论：

- 开启本地重试时，消息处理过程中抛出异常，不会requeue到队列，而是在消费者本地重试

- 重试达到最大次数后，Spring会返回reject，消息会被丢弃

#### 失败处理策略

在之前的测试中，本地测试达到最大重试次数后，消息会被丢弃。这在某些对于消息可靠性要求较高的业务场景下，显然不太合适了。因此Spring允许我们自定义重试次数耗尽后的消息处理策略，这个策略是由MessageRecovery接口来定义的，它有3个不同实现：

- RejectAndDontRequeueRecoverer：重试耗尽后，直接reject，丢弃消息。默认就是这种方式

- ImmediateRequeueMessageRecoverer：重试耗尽后，返回nack，消息重新入队

- RepublishMessageRecoverer：重试耗尽后，将失败消息投递到指定的交换机


比较优雅的一种处理方案是RepublishMessageRecoverer，失败后将消息投递到一个指定的，专门存放异常消息的队列，后续由人工集中处理。

![{4ACBE29C-C9CC-43E7-BC01-76B2C7B34B08}](C:\Users\jqdsg\AppData\Local\Packages\MicrosoftWindows.Client.CBS_cw5n1h2txyewy\TempState\ScreenClip\{4ACBE29C-C9CC-43E7-BC01-76B2C7B34B08}.png)

1）在consumer服务中定义处理失败消息的交换机和队列

```java
@Bean
public DirectExchange errorMessageExchange(){
    return new DirectExchange("error.direct");
}
@Bean
public Queue errorQueue(){
    return new Queue("error.queue", true);
}
@Bean
public Binding errorBinding(Queue errorQueue, DirectExchange errorMessageExchange){
    return BindingBuilder.bind(errorQueue).to(errorMessageExchange).with("error");
}
```

2）定义一个RepublishMessageRecoverer，关联队列和交换机

```java
@Bean
public MessageRecoverer republishMessageRecoverer(RabbitTemplate rabbitTemplate){
    return new RepublishMessageRecoverer(rabbitTemplate, "error.direct", "error");
}
```

完整代码如下：

```java
package com.itheima.consumer.config;
 
import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.BindingBuilder;
import org.springframework.amqp.core.DirectExchange;
import org.springframework.amqp.core.Queue;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.amqp.rabbit.retry.MessageRecoverer;
import org.springframework.amqp.rabbit.retry.RepublishMessageRecoverer;
import org.springframework.context.annotation.Bean;
 
@Configuration
@ConditionalOnProperty(name = "spring.rabbitmq.listener.simple.retry.enabled", havingValue = "true")
public class ErrorMessageConfig {
    @Bean
    public DirectExchange errorMessageExchange(){
        return new DirectExchange("error.direct");
    }
    @Bean
    public Queue errorQueue(){
        return new Queue("error.queue", true);
    }
    @Bean
    public Binding errorBinding(Queue errorQueue, DirectExchange errorMessageExchange){
        return BindingBuilder.bind(errorQueue).to(errorMessageExchange).with("error");
    }
 
    @Bean
    public MessageRecoverer republishMessageRecoverer(RabbitTemplate rabbitTemplate){
        return new RepublishMessageRecoverer(rabbitTemplate, "error.direct", "error");
    }
}

```



### 业务幂等性

**幂等**是一个数学概念，用函数表达式来描述是这样的：`f(x) = f(f(x))`，例如求绝对值函数。在程序开发中，则是指同一个业务，执行一次或多次对业务状态的影响是一致的。例如：

- 根据id删除数据
- 查询数据
- 新增数据

但数据的更新往往不是幂等的，如果重复执行可能造成不一样的后果。比如：

- 取消订单，恢复库存的业务。如果多次恢复就会出现库存重复增加的情况
- 退款业务。重复退款对商家而言会有经济损失

所以，我们要尽可能避免业务被重复执行。然而在实际业务场景中，由于意外经常会出现业务被重复执行的情况，例如：

- 页面卡顿时频繁刷新导致表单重复提交
- 服务间调用的重试
- MQ消息的重复投递

我们在用户支付成功后会发送MQ消息到交易服务，修改订单状态为已支付，就可能出现消息重复投递的情况。如果消费者不做判断，很有可能导致消息被消费多次，出现业务故障。举例：

1. 假如用户刚刚支付完成，并且投递消息到交易服务，交易服务更改订单为已支付状态。

2. 由于某种原因，例如网络故障导致生产者没有得到确认，隔了一段时间后重新投递给交易服务。

3. 但是，在新投递的消息被消费之前，用户选择了退款，将订单状态改为了已退款状态。

4. 退款完成后，新投递的消息才被消费，那么订单状态会被再次改为已支付。业务异常。


因此，我们必须想办法保证消息处理的幂等性。这里给出两种方案：

- 唯一消息ID

- 业务状态判断

#### 唯一消息ID

这个思路非常简单：

1. 每一条消息都生成一个唯一的id，与消息一起投递给消费者。

2. 消费者接收到消息后处理自己的业务，业务处理成功后将消息ID保存到数据库

3. 如果下次又收到相同消息，去数据库查询判断是否存在，存在则为重复消息放弃处理。


我们该如何给消息添加唯一ID呢？其实很简单，SpringAMQP的MessageConverter自带了MessageID的功能，我们只要开启这个功能即可。以Jackson的消息转换器为例：

```java
@Bean
public MessageConverter messageConverter(){
    // 1.定义消息转换器
    Jackson2JsonMessageConverter jjmc = new Jackson2JsonMessageConverter();
    // 2.配置自动创建消息id，用于识别不同消息，也可以在业务中基于ID判断是否是重复消息
    jjmc.setCreateMessageIds(true);
    return jjmc;
}
```

#### 业务判断

业务判断就是基于业务本身的逻辑或状态来判断是否是重复的请求或消息，不同的业务场景判断的思路也不一样。例如我们当前案例中，处理消息的业务逻辑是把订单状态从未支付修改为已支付。因此我们就可以在执行业务时判断订单状态是否是未支付，如果不是则证明订单已经被处理过，无需重复处理。

![{1EAA79F1-F8FA-42C7-BB4B-F792C5FFFBF2}](C:\Users\jqdsg\AppData\Local\Packages\MicrosoftWindows.Client.CBS_cw5n1h2txyewy\TempState\ScreenClip\{1EAA79F1-F8FA-42C7-BB4B-F792C5FFFBF2}.png)

相比较而言，消息ID的方案需要改造原有的数据库，所以我更推荐使用业务判断的方案。

以支付修改订单的业务为例，我们需要修改`OrderServiceImpl`中的`markOrderPaySuccess`方法：

```java
@Override
public void markOrderPaySuccess(Long orderId) {
	// 1.查询订单
	Order old = getById(orderId);
	// 2.判断订单状态
	if (old == null || old.getStatus() != 1) {
		// 订单不存在或者订单状态不是1，放弃处理
		return;
	}
	// 3.尝试更新订单
	Order order = new Order();
	order.setId(orderId);
	order.setStatus(2);
	order.setPayTime(LocalDateTime.now());
	updateById(order);
}
```

上述代码逻辑上符合了幂等判断的需求，但是由于判断和更新是两步动作，因此在极小概率下可能存在线程安全问题。

我们可以合并上述操作为这样：

```java
@Override
public void markOrderPaySuccess(Long orderId) {
    // UPDATE `order` SET status = ? , pay_time = ? WHERE id = ? AND status = 1
    lambdaUpdate()
            .set(Order::getStatus, 2)
            .set(Order::getPayTime, LocalDateTime.now())
            .eq(Order::getId, orderId)
            .eq(Order::getStatus, 1)
            .update();
    //UPDATE `order` SET status = ? , pay_time = ? WHERE id = ? AND status = 1
}
```

我们在where条件中除了判断id以外，还加上了status必须为1的条件。如果条件不符（说明订单已支付），则SQL匹配不到数据，根本不会执行。

#### 兜底方案



### 总结

**支付服务与交易服务之间的订单状态一致性是如何保证的？**

- 首先，支付服务会正在用户支付成功以后利用MQ消息通知交易服务，完成订单状态同步。

- 其次，为了保证MQ消息的可靠性，我们采用了生产者确认机制、消费者确认、消费者失败重试等策略，确保消息投递的可靠性

- 最后，我们还在交易服务设置了定时任务，定期查询订单支付状态。这样即便MQ通知失败，还可以利用定时任务作为兜底方案，确保订单支付状态的最终一致性。

**如果交易服务消息处理失败，有没有什么兜底方案？**

我们可以在交易服务设置定时任务，定期查询订单支付状态。这样即使MQ通知失败，还可以利用定时任务作为兜底方案，确保订单支付状态的最终一致性。

## 延迟消息

在电商的支付业务中，对于一些库存有限的商品，为了更好的用户体验，通常都会在用户下单时立刻扣减商品库存。例如电影院购票、高铁购票，下单后就会锁定座位资源，其他人无法重复购买。

但是这样就存在一个问题，假如用户下单后一直不付款，就会一直占有库存资源，导致其他客户无法正常交易，最终导致商户利益受损！

因此，电商中通常的做法就是：**对于超过一定时间未支付的订单，应该立刻取消订单并释放占用的库存。**

例如，订单支付超时时间为30分钟，则我们应该在用户下单后的第30分钟检查订单支付状态，如果发现未支付，应该立刻取消订单，释放库存。

但问题来了：如何才能准确的实现在下单后第30分钟去检查支付状态呢？

像这种在一段时间以后才执行的任务，我们称之为延迟任务，而要实现**延迟任务**，最简单的方案就是利用MQ的延迟消息了。

在RabbitMQ中实现延迟消息也有两种方案：

- 死信交换机+TTL

- 延迟消息插件


这一章我们就一起研究下这两种方案的实现方式，以及优缺点。


### 死信交换机和延迟消息



#### 死信交换机

什么是死信？

当一个队列中的消息满足下列情况之一时，可以成为死信（dead letter）：

- 消费者使用basic.reject或 basic.nack声明消费失败，并且消息的requeue参数设置为false

- 消息是一个过期消息，超时无人消费

- 要投递的队列消息满了，无法投递


如果一个队列中的消息已经成为死信，并且这个队列通过dead-letter-exchange属性指定了一个交换机，那么队列中的死信就会投递到这个交换机中，而这个交换机就称为**死信交换机**（Dead Letter Exchange）。而此时加入有队列与死信交换机绑定，则最终死信就会被投递到这个队列中。

死信交换机有什么作用呢？

1. 收集那些因处理失败而被拒绝的消息

2. 收集那些因队列满了而被拒绝的消息

3. 收集因TTL（有效期）到期的消息
   ![{6938A2C4-FD56-40C3-B9F8-56DFB65DE415}](C:\Users\jqdsg\AppData\Local\Packages\MicrosoftWindows.Client.CBS_cw5n1h2txyewy\TempState\ScreenClip\{6938A2C4-FD56-40C3-B9F8-56DFB65DE415}.png)

### DelayExchange插件

基于死信队列虽然可以实现延迟消息，但是太麻烦了。因此RabbitMQ社区提供了一个延迟消息插件来实现相同的效果。

```java
@RabbitListener(bindings = @QueueBinding(
        value = @Queue(name = "delay.queue", durable = "true"),
        exchange = @Exchange(name = "delay.direct", delayed = "true"),
        key = "delay"
))
public void listenDelayMessage(String msg){
    log.info("接收到delay.queue的延迟消息：{}", msg);
}
```



```java
package com.itheima.consumer.config;
 
import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.core.*;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
 
@Slf4j
@Configuration
public class DelayExchangeConfig {
 
    @Bean
    public DirectExchange delayExchange(){
        return ExchangeBuilder
                .directExchange("delay.direct") // 指定交换机类型和名称
                .delayed() // 设置delay的属性为true
                .durable(true) // 持久化
                .build();
    }
 
    @Bean
    public Queue delayedQueue(){
        return new Queue("delay.queue");
    }
    
    @Bean
    public Binding delayQueueBinding(){
        return BindingBuilder.bind(delayedQueue()).to(delayExchange()).with("delay");
    }
}
```

发送消息时，必须通过x-delay属性设定延迟时间

```java
@Test
void testPublisherDelayMessage() {
    // 1.创建消息
    String message = "hello, delayed message";
    // 2.发送消息，利用消息后置处理器添加消息头
    rabbitTemplate.convertAndSend("delay.direct", "delay", message, new MessagePostProcessor() {
        @Override
        public Message postProcessMessage(Message message) throws AmqpException {
            // 添加延迟消息属性
            message.getMessageProperties().setDelay(5000);
            return message;
        }
    });
}
```

**注意：**延迟消息插件内部会维护一个本地数据库表，同时使用Elang Timers功能实现计时。如果消息的延迟时间设置较长，可能会导致堆积的延迟消息非常多，会带来较大的CPU开销，同时延迟消息的时间会存在误差。因此，**不建议设置延迟时间过长的延迟消息**。

### 订单状态同步问题

![img](https://i-blog.csdnimg.cn/blog_migrate/51f1fa164c71ba51b95ad3829dd01402.png)

假如订单超时支付时间为30分钟，理论上说我们应该在下单时发送一条延迟消息，延迟时间为30分钟。这样就可以在接收到消息时检验订单支付状态，关闭未支付订单。但是大多数情况下用户支付都会在1分钟内完成，我们发送的消息却要在MQ中停留30分钟，额外消耗了MQ的资源。因此，我们最好多检测几次订单支付状态，而不是在最后第30分钟才检测。例如：我们在用户下单后的第10秒、20秒、30秒、45秒、60秒、1分30秒、2分、...30分分别设置延迟消息，如果提前发现订单已经支付，则后续的检测取消即可。这样就可以有效避免对MQ资源的浪费了。

优化后的实现思路如下
![img](https://i-blog.csdnimg.cn/blog_migrate/81aa555836f9314b6dc23090eb3a6cf9.png)
