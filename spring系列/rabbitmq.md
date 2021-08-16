# RabbitMq基本组成

主要功能：应用解耦、异步提速、削峰填谷

适用条件：①生产者不需要从消费者处获得反馈  ②不要求强一致性

消息代理broker：MQ本身     vhost:虚拟mq，有自己的隔离体系    Channel：虚拟通道，高效利用TCP

目的地destination:  队列（点对点）  publish/subscribe      投递消息给交换机

  消息：  消息头(路由键routing-key等)  +  消息体：不透明

交换器exchange   只负责转发消息，不具备存储消息的能力，失败会丢失     Direct一对一	  Fanout忽略路由键广播   Topic多播(.分隔单词  *一个单词   # 0或任意个单词)

exchange和queue绑定时，指定binding-key,一般队列名与消息头的routing-key一致

queue：可设置持久化到硬盘durable，最大长度x-max-length  ，Auto delete无被使用是否自动删除以及死信队列



日志文件 /var/log/rabbitmq/rabbit@主机名.log

![image-20210117215254805](E:\学习总结\spring系列\rabbit架构图.png)

![在这里插入图片描述](E:\学习总结\spring系列\rabbit结构图)

# springboot配置(特别注意包名要设计amqp)

## 引入和创建组件

### 引入

1. spring-boot-starter-amqp，与yml配置四个基本选项

2. 已自动配置ConnectionFactory，引入它配置RabbitTemplate。开启@EnableRabbit

3. 设置json序列化

   ```java
   @Bean   amqp包下
       public MessageConverter jackson2JsonMessageConverter() {
           return new Jackson2JsonMessageConverter(jacksonObjectMapper);
       }
   ```

4. 设置exchange、queue、Binding

   ```java
       @Bean
       public Queue myq(){
   Queue(队列名name, 持久化 durable, 临时队列程序关闭删除 exclusive, 不使用就删除autoDelete,
   			队列参数Map<String, Object> arguments)
           HashMap<String, Object> map = new HashMap<>();
           map.put("x-message-ttl",3000);//可设队列消息整体存活时间
           return new Queue("haha",false,true,false,map);
       }
   Builder简化编码
   BindingBuilder.BindingBuilder.bind(队列).to(交换机).with("binding-key");
   ExchangeBuilder.topicExchange("exchange名") .durable(true).bui1ld();
   QueueBui1der.durable("队列名".....).bui1d()
   
   @Bean
    Binding bindingDirect() {
    return BindingBuilder.bind(队列).to(交换机).with("binding-key").noargs();
       }
   
   ```

5. 发送消息：

   ```java
   convertAndSend("exchange名字", "队列名", 消息体对象, 后置处理器message -> {
       message.getMessageProperties().setExpiration("300");
       return message;//封装成消息后处理
   });
   ```

6. 可设置RabbitListenerContainerFactory，默认SimpleXXX，可设置各种复杂功能，yml可配置@RabbitListener(containerFactory=XXX)配合使用

7. 接收消息：receiveAndConvert(String queueName, long timeoutMillis)和设置监听

8. ```java
   @RabbitListener(bindings = @QueueBinding(
           value = @Queue(value = "test1", durable = "false", exclusive = "false", autoDelete = "false"),
           exchange = @Exchange(name = "test", durable = Exchange.FALSE),
           key = "test1"))
   
   @RabbitListener(queues = {"biang"})//可设置复杂绑定，但不建议。一般于类上监听队列名
   @RabbitHandler 注于方法 与@RabbitListener注解与类配合使用 根据消息体类型自动匹配对应处理方法
   @Payload标注消息体对应类型于形参   @Headers标注消息头于形参(map所有参数) @Heade rString类型的单个属性值
       handleMessage(@Payload String body, @Headers Map<String,Object> headers)
       handleMessage(@Payload String body,@Header String token)
   ```

#可靠投递      confirm确认   producer-必然执行->exchange    return确认  exchange-失败执行->queue

1. 开启confirm   无论成败==必然触发==

   1. yml配置spring.rabbitmq.listener.simple.publisher-confirm-type=correlated 

   2. RabbitTemplate配置回调函数

      ```java
      rabbitTemplate.setConfirmCallback((correlationData, ack, cause) -> {
          correlationData相关配置信息, ack是否发送成功, cause失败原因成功为null
      });
      ```

      

2. 开启return    失败才触发

   默认exchange发送失败则丢失，无回调

   - spring.rabbitmq.listener.simple.publisher-returns=true

   - rabbitTemplate.setMandatory(true);

   - 设置回调函数

     ```java
     rabbitTemplate.setReturnCallback((message, replyCode, replyText, exchange, routingKey) -> {
         //消息本身  回应码  回应消息   交换机名字   路由键
     });
     ```

#可靠消费(手动确认)

1. yml配置spring.rabbitmq.listener.simple.acknowledge-mode= manual  默认none不论消费端是否正确处理

2. @RabbitHandler/@RabbitListener方法形参设Channel.basicAck/Nack/reject

3. ```java
   @RabbitListener(queues = {"ljh"})
   public void fq(@Payload String s, Message message , Channel channel) throws IOException {              //投递号
       channel.basicAck(message.getMessageProperties().getDeliveryTag(),false);
   }
   basicNack(long deliveryTag, boolean multiple, boolean requeue)
   basicReject(long deliveryTag, boolean requeue)//requeue是否重返队列   只能回退单个消息
   basicAck(long deliveryTag, boolean multiple)
   ```

#消费端限流Qos

1. 开启手动确认模式，如上
2. 配置spring.rabbitmq.listener.simple.prefetch=1,一次拉去消息条数，处理完才能再拉下一条

#TTL存活时间

- 队列过期(针对queue所有消息，==只有顶端消息才会判断==，中间消息即时过期也会存在占空间)创建队列设置参数x-message-ttl=毫秒数
- 针对单个消息。设置消息头参数   message.getMessageProperties().setExpiration("毫秒数");可才后置处理器设置
- 两者冲突，则以最短时间为准

#死信队列DLX   死信消息被原队列发送至死信交换机专门处理

![image-20210117221858041](E:\学习总结\spring系列\死信交换机原理.png)

- 死信消息 ① 超出队列长度 ②消费端拒收且不重回队列 basicNack/basicReject,requeue=false⑶消息过期
- 死信交换机、对应queue和binging无差别，主要是==原队列作为producer==将死信消息如何作为发送至DLX
- 原队列 x-dead-letter-exchange=死信交换机名    x-dead-letter-routing-key=死信消息路由键

#延时队列(TTL+死信组合)

![image-20210117222158709](E:\学习总结\spring系列\延迟队列.png)

![image-20210117222301561](E:\学习总结\spring系列\延迟队列实现.png)

# 重试机制

- spring.rabbitmq.listener.simple.acknowledge-mode ===auto==  不能手动manual
- spring.rabbitmq.template.enable=true默认关闭 但是 `basic.nack` + `requeue=true` ，重新投递
- spring.rabbitmq.listener.simple.default-requeue-rejected是否重新入队
- spring.rabbitmq.listener.simple.retry.max-attempts最大重试，默认3  包括首次的正常消费
- spring.rabbitmq.listener.simple.retry.max-interval最大重试间隔，默认100.ms    multiplier**递乘**
- 抛出 AmqpRejectAndDontRequeueException可结束重试机制
- 重试是发生异常后，mq再次利用这条消息。而不是设置为重入，然后消费重试，再重入，消费重试...然后打到设置的 重试阈值

#补偿机制

生产者与消费者之间应该约定一个超时时间，比如5 分钟，对于超出这个时间没有得到响应的消息，可以设置一个定时重发的机制，但要发送间隔和控制次数，比如每隔2分钟发送一次，最多重发3 次，否则会造成消息堆积。

重发可以通过消息落库+定时任务来实现。

#消息幂等性

乐观锁version+1