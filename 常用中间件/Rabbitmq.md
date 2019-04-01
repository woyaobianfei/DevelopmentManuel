# Rabbitmq

## 什么是MQ

消息队列（Message Queue，简称 MQ），**本质是个队列**。

其主要用途：不同进程 Process /线程 Thread 之间通信。


### 为什么使用消息队列

1. **混乱的调用关系**
系统中存在好几个应用，应用2需要使用应用1的输出作为输入，应用3需要使用应用2的输出作为输入···，当某一天由于业务调整或其他原因，应用调用的先后顺序发生变化，这会使得整个系统将会非常难以维护。

2. **上游应用不关心下游执行结果**，当下游宕机时，上游应用受到严重牵连。
当应用1执行完后，要分别通知应用2,3,4去执行某些相应的操作，当调用某一个应用时发现宕机了，这种情况的出现上游应用还要添加相应的保护机制明显已经受到了牵连，而且还增加了上游应用的执行时间。

3. **上游关心执行结果，但是执行时间较长**
应用1调应用2，需要拿到应用2的执行结果，如果是用户交互强的业务，会大大影响用户体验。


**总结：**

什么时候不适用MQ：
- 上游实时关注执行结果

什么时候适用MQ：
- 涉及应用调用链的业务
- 上游不关心下游执行结果
- 异步返回执行时间长

### MQ的优缺点

缺点：
- 系统复杂，需要维护MQ中间件
- 消息传递，肯定会涉及延时，重复，丢失的问题

优点：
- 项目间的解耦


## RabbitMQ的简介
![mark](http://pic-cloud.ice-leaf.top/pic-cloud/20190318/wHJB1KEgVzs6.png?imageslim)

Rabbitmq 中生产者将消息发送到 Exchange（交换器，下图中的 Exchange1 和 Exchange2），再由 Binding 将 Exchange 与 Queue 通过 RoutingKey（黄色部分）关联起来，再由监听队列的客户端从 Queue 中获得消息。

Rabbitmq的协议学习请阅读：[AMQP协议详解](https://wiki.tvflnet.com/pages/viewpage.action?pageId=9765858)

### 编码教程

#### 1. POM 引入`spring-boot-starter-amqp`依赖
```xml
<!-- 引入 Rabbitmq 依赖-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

#### 2. `application-${env}.yml`配置 Rabbitmq 服务器地址
```yaml
spring:
    rabbitmq:
        username: xiaoming
        password: xiaoming123
        addresses: localhost:5672
        virtual-host: /local
        publisher-confirms: true
        publisher-returns: true
```
`virtual-host`: 虚拟主机命名空间，可用于资源隔离，我们一般用于环境隔离，开发环境为`/dev`，测试环境为`test`。

*配置中有关生产者的 confirm 机制和消费者的 ack 机制，下文详细说明*。

#### 3. 公共`rabbitTemplate`定义
配置路径：`util.configurer.RabbitmqConfigurer`
```java
/**
 * RabbitMQ 公共配置
 *
 * @author : ACE
 * @date : 2019/3/18
 */
@Slf4j
@Configuration
public class RabbitmqConfigurer {

    @Value("${spring.rabbitmq.username:guest}")
    private String username;

    @Value("${spring.rabbitmq.password:guest}")
    private String password;

    @Value("${spring.rabbitmq.addresses:localhost:5672}")
    private String addresses;

    @Value("${spring.rabbitmq.publisher-confirms:true}")
    private Boolean publisherConfirms;

    @Value("${spring.rabbitmq.publisher-returns:true}")
    private Boolean publisherReturns;

    @Value("${spring.rabbitmq.virtual-host:/}")
    private String virtualHost;

    @Bean
    public ConnectionFactory connectionFactory() {
        CachingConnectionFactory connectionFactory = new CachingConnectionFactory();
        connectionFactory.setAddresses(addresses);
        connectionFactory.setUsername(username);
        connectionFactory.setPassword(password);
        connectionFactory.setVirtualHost(virtualHost);
        // 设置 生产者开启 confirms
        connectionFactory.setPublisherConfirms(publisherConfirms);
        // 设置 生产者开启 returns
        // 经测试，此处配置无用，启用 ReturnCallback 需手动 setMandatory(true)
        connectionFactory.setPublisherReturns(publisherReturns);
        return connectionFactory;
    }

    /**
     * 采用多例模式
     * 如果未开启生产者回调，可使用单例模式，否则多队列回调，单例模式会导致确认异常
     */
    @Bean
    @Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
    public RabbitTemplate rabbitTemplate() {
        return new RabbitTemplate(connectionFactory());
    }
}
```
#### 4. 业务生产者
```java
/**
 * 消息生产者
 *
 * @author : ACE
 * @date : 2019/3/18
 */
@Slf4j
@Component
public class StudentMqSender {

    @Value("${rabbitmq-exchange-student:studentExchange}")
    private String studentExchange;

    @Value("${rabbitmq-routingKey-student:studentRoutingKey}")
    private String studentRoutingKey;

    @Autowired
    private RabbitTemplate rabbitTemplate;

    /**
     * MQ消息发送
     *
     * @param studentDTO 需要发送的对象
     */
    public void sendMsg(StudentDTO studentDTO) {
        CorrelationData correlationId = new CorrelationData(studentDTO.toString());
        Message message = new Message(JSONObject.toJSONBytes(studentDTO), new MessageProperties());
        log.debug(">>>>>>> 消息内容:{}", studentDTO.toString());
        rabbitTemplate.convertAndSend(studentExchange, studentRoutingKey, message, correlationId);
    }
}
```







未完待续。。。



