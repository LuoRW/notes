## RabbitMQ

视频教程地址：【【编程不良人】MQ消息中间件之RabbitMQ以及整合SpringBoot2.x实战教程,已完结!】 https://www.bilibili.com/video/BV1dE411K7MG/

![img](img/20210705145929233.png)

## 1 MQ引言

### 1.1 什么是MQ

MQ(Message Quene) :  翻译为消息队列，通过典型的生产者和消费者模型，生产者不断向消息队列中生产消息，消费者不断的从队列中获取消息。因为消息的生产和消费都是异步的，而且只关心消息的发送和接收，没有业务逻辑的侵入，轻松的实现系统间解耦。别名为消息中间件通过利用高效可靠的消息传递机制进行平台无关的数据交流，并基于数据通信来进行分布式系统的集成。

### 1.2 MQ有哪些

当今市面上有很多主流的消息中间件，如老牌的ActiveMQ、RabbitMQ，炙手可热的Kafka，阿里巴巴自主开发RocketMQ等。

### 1.3 不同MQ特点

1. ActiveMQ
   ActiveMQ 是Apache出品，最流行的，能力强劲的开源消息总线。它是一个完全支持JMS规范的的消息中间件。丰富的API，多种集群架构模式让ActiveMQ在业界成为老牌的消息中间件，在中小型企业颇受欢迎!

2. Kafka
   Kafka是LinkedIn开源的分布式发布-订阅消息系统，目前归属于Apache顶级项目。Kafka主要特点是基于Pull的模式来处理消息消费，追求高吞吐量，一开始的目的就是用于日志收集和传输。0.8版本开始支持复制，不支持事务，对消息的重复、丢失、错误没有严格要求，适合产生大量数据的互联网服务的数据收集业务。

3. RocketMQ
   RocketMQ是阿里开源的消息中间件，它是纯Java开发，具有高吞吐量、高可用性、适合大规模分布式系统应用的特点。RocketMQ思路起源于Kafka，但并不是Kafka的一个Copy，它对消息的可靠传输及事务性做了优化，目前在阿里集团被广泛应用于交易、充值、流计算、消息推送、日志流式处理、binglog分发等场景。

4. RabbitMQ
   RabbitMQ是使用Erlang语言开发的开源消息队列系统，基于AMQP协议来实现。AMQP的主要特征是面向消息、队列、路由（包括点对点和发布/订阅）、可靠性、安全。AMQP协议更多用在企业系统内对数据一致性、稳定性和可靠性要求很高的场景，对性能和吞吐量的要求还在其次。

> RabbitMQ比Kafka可靠，Kafka更适合IO高吞吐的处理，一般应用在大数据日志处理或对实时性（少量延迟），可靠性（少量丢数据）要求稍低的场景使用，比如ELK日志收集。

## 2 RabbitMQ 的引言

### 2.1 RabbitMQ

> 基于AMQP协议，erlang语言开发，是部署最广泛的开源消息中间件,是最受欢迎的开源消息中间件之一。

![img](img/20201030175407725.png)

AMQP 协议，AMQP（advanced message queuing protocol）在2003年时被提出，最早用于解决金融领不同平台之间的消息传递交互问题。顾名思义，AMQP是一种协议，更准确的说是一种binary wire-level protocol（链接协议）。这是其和JMS的本质差别，AMQP不从API层进行限定，而是直接定义网络交换的数据格式。这使得实现了AMQP的provider天然性就是跨平台的。以下是AMQP协议模型:

![img](img/20201030175435428.png)

### 2.2 RabbitMQ 的安装

最新版的3.8版本的可以看我这篇文章https://blog.csdn.net/unique_perfect/article/details/108643804

#### 2.2.1 下载

![img](img/20201030175453979.png)

#### 2.2.2 下载的安装包

![img](img/20201030175542524.png)

> 注意：这里的安装包是centos7安装的包

#### 2.2.3 安装步骤

1. 将rabbitmq安装包上传到linux系统中

   ```bash
   erlang-22.0.7-1.el7.x86_64.rpm  #l7表示是Centosl7,Centosl8表示Centos8
   rabbitmq-server-3.7.18-1.el7.noarch.rpm
   ```

2. 安装Erlang依赖包

   ```bash
   rpm -ivh erlang-22.0.7-1.el7.x86_64.rpm
   ```

3. 安装RabbitMQ安装包(需要联网)

   ```bash
   yum install -y rabbitmq-server-3.7.18-1.el7.noarch.rpm
   注意:默认安装完成后配置文件模板在:/usr/share/doc/rabbitmq-server-3.7.18/rabbitmq.config.example目录中,需要将配置文件复制到/etc/rabbitmq/目录中,并修改名称为rabbitmq.config
   ```
   
4. 复制配置文件

   ```bash
   cp /usr/share/doc/rabbitmq-server-3.7.18/rabbitmq.config.example /etc/rabbitmq/rabbitmq.config
   ```

5. 查看配置文件位置

   ```bash
   ls /etc/rabbitmq/rabbitmq.config
   ```

6. 修改配置文件(参见下图:)

   ```bash
   vim /etc/rabbitmq/rabbitmq.config
   ```

   ![img](img/2020103017560848.png)
   
   ```bash
   将上图中配置文件中红色部分去掉`%%`,以及最后的`,`逗号 
   修改为下图:
   ```
   
   ![img](img/20201030175704618.png)

7. 执行如下命令,启动rabbitmq中的插件管理

   ```bash
   rabbitmq-plugins enable rabbitmq_management
   ```
   
   出现如下说明:
   
   ```bash
   Enabling plugins on node rabbit@localhost:
     rabbitmq_management
     The following plugins have been configured:
       rabbitmq_management
       rabbitmq_management_agent
       rabbitmq_web_dispatch
     Applying plugin configuration to rabbit@localhost...
     The following plugins have been enabled:
       rabbitmq_management
       rabbitmq_management_agent
       rabbitmq_web_dispatch
   
   set 3 plugins.
   Offline change; changes will take effect at broker restart.
   ```
   


8. 启动RabbitMQ的服务

   ```bash
   systemctl start rabbitmq-server
   systemctl restart rabbitmq-server
   systemctl stop rabbitmq-server
   ```

9. 查看服务状态(见下图:)

   ```bash
   systemctl status rabbitmq-server
   ```
   ```bash
    ● rabbitmq-server.service - RabbitMQ broker
     Loaded: loaded (/usr/lib/systemd/system/rabbitmq-server.service; disabled; vendor preset: disabled)
     Active: active (running) since 三 2019-09-25 22:26:35 CST; 7s ago
   Main PID: 2904 (beam.smp)
     Status: "Initialized"
     CGroup: /system.slice/rabbitmq-server.service
             ├─2904 /usr/lib64/erlang/erts-10.4.4/bin/beam.smp -W w -A 64 -MBas ageffcbf -MHas ageffcbf -
             MBlmbcs...
             ├─3220 erl_child_setup 32768
             ├─3243 inet_gethost 4
             └─3244 inet_gethost 4
      .........
   ```
   
   ![img](img/20201030175729252.png)

10. 关闭防火墙服务

    ```bash
    systemctl disable firewalld  # 需要关闭防火墙，否则访问不了
    Removed symlink /etc/systemd/system/multi-user.target.wants/firewalld.service.
    Removed symlink /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service.
    systemctl stop firewalld
    ```

11. 访问web管理界面

	```bash
	http://10.15.0.8:15672/
	```
	
	![img](img/20201030175753712.png)

12. 登录管理界面

    ```bash
    username:  guest
    password:  guest
    ```

    ![img](img/20201030175814170.png)

## 3 RabiitMQ 配置

### 3.1 RabbitMQ 管理命令行

1. 服务启动相关

   `systemctl start|restart|stop|status rabbitmq-server`

2. 管理命令行  用来在不使用web管理界面情况下命令操作RabbitMQ

   `rabbitmqctl  help  可以查看更多命令`

3. 插件管理命令行

   `rabbitmq-plugins enable|list|disable`

### 3.2 web管理界面介绍

#### 3.2.1 overview概览

![img](img/20201030175840800.png)

- connections：无论生产者还是消费者，都需要与RabbitMQ建立连接后才可以完成消息的生产和消费，在这里可以查看连接情况`


- channels：通道，建立连接后，会形成通道，消息的投递获取依赖通道。


- Exchanges：交换机，用来实现消息的路由


- Queues：队列，即消息队列，消息存放在队列中，等待消费，消费后被移除队列。

#### 3.2.2 Admin用户和虚拟主机管理

##### 3.2.2.1 添加用户

![img](img/20210705193528571.png)

上面的Tags选项，其实是指定用户的角色，可选的有以下几个：

- 超级管理员(administrator)

  可登陆管理控制台，可查看所有的信息，并且可以对用户，策略(policy)进行操作。

- 监控者(monitoring)

  可登陆管理控制台，同时可以查看rabbitmq节点的相关信息(进程数，内存使用情况，磁盘使用情况等)

- 策略制定者(policymaker)

  可登陆管理控制台, 同时可以对policy进行管理。但无法查看节点的相关信息(上图红框标识的部分)。

- 普通管理者(management)

  仅可登陆管理控制台，无法看到节点信息，也无法对策略进行管理。

- 其他

  无法登陆管理控制台，通常就是普通的生产者和消费者。

##### 3.2.2.2 创建虚拟主机

虚拟主机，为了让各个用户可以互不干扰的工作，RabbitMQ添加了虚拟主机（Virtual Hosts）的概念。其实就是一个独立的访问路径，不同用户使用不同路径，各自有自己的队列、交换机，互相不会影响。相当于关系型中的数据库

![img](img/202107051936520.png)

##### 3.2.2.3 绑定虚拟主机和用户

创建好虚拟主机，我们还要给用户添加访问权限：

点击添加好的虚拟主机：

![img](img/20210705193758998.png)

进入虚拟机设置界面

![img](img/20210705193911356.png)

## 4 RabbitMQ 的第一个程序

### 4.1 AMQP协议的回顾

![img](img/20210705194004304.png)

### 4.2 RabbitMQ支持的消息模型

![img](img/2020103018025840.png)

![img](img/20201030180315852.png)

### 4.3 引入依赖

```xml
<dependency>
    <groupId>com.rabbitmq</groupId>
    <artifactId>amqp-client</artifactId>
    <version>5.7.2</version>
</dependency>
```

### 4.4 第一种模型(直连)

![img](img/20201030180349461.png)

在上图的模型中，有以下概念：

- P：生产者，也就是要发送消息的程序
- C：消费者，消息的接受者，会一直等待消息到来。
- queue：消息队列，图中红色部分。类似一个邮箱，可以缓存消息；生产者向其中投递消息，消费者从其中取出消息。

#### 4.4.1 开发生产者

```java
package helloword;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.MessageProperties;
import org.junit.Test;
import utils.RabbitMQUtils;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

public class Provider {

  //生产消息
  @Test
  public void testSendMessage() throws IOException, TimeoutException {
    /*
        //创建连接mq的连接工厂对象
        ConnectionFactory connectionFactory = new ConnectionFactory();
        //设置连接rabbitmq主机
        connectionFactory.setHost("192.168.11.143");
        //设置端口号
        connectionFactory.setPort(5672);
        //设置连接那个虚拟主机
        connectionFactory.setVirtualHost("/ems");
        //设置访问虚拟主机的用户名和密码
        connectionFactory.setUsername("ems");
        connectionFactory.setPassword("123");

        //获取连接对象
        Connection connection = connectionFactory.newConnection();*/

    //通过工具类获取连接对象
    Connection connection = RabbitMQUtils.getConnection();

    //获取连接中通道
    Channel channel = connection.createChannel();

    //通道绑定对应消息队列
    //参数1:  队列名称 如果队列不存在自动创建
    //参数2:  用来定义队列特性是否要持久化 true 持久化队列   false 不持久化
    //参数3:  exclusive 是否独占队列  true 独占队列   false  不独占
    //参数4:  autoDelete: 是否在消费完成后自动删除队列  true 自动删除  false 不自动删除
    //参数5:  额外附加参数
    channel.queueDeclare("hello",true,false,false,null);

    //发布消息

    //参数1: 交换机名称 参数2:队列名称  参数3:传递消息额外设置  参数4:消息的具体内容
    channel.basicPublish("","hello", MessageProperties.PERSISTENT_TEXT_PLAIN,"hello rabbitmq".getBytes());


    /*channel.close();
        connection.close();*/

    //调用工具类
    RabbitMQUtils.closeConnectionAndChanel(channel,connection);


  }
}
```

![img](img/20210705212514383.png)

#### 4.4.2 开发消费者

```java
package helloword;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.MessageProperties;
import org.junit.Test;
import utils.RabbitMQUtils;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

public class Customer {

  //生产消息
  @Test
  public void testGetMessage() throws IOException, TimeoutException {
    /*
        //创建连接mq的连接工厂对象
        ConnectionFactory connectionFactory = new ConnectionFactory();
        //设置连接rabbitmq主机
        connectionFactory.setHost("192.168.11.134");
        //设置端口号
        connectionFactory.setPort(5672);
        //设置连接那个虚拟主机
        connectionFactory.setVirtualHost("/ems");
        //设置访问虚拟主机的用户名和密码
        connectionFactory.setUsername("admin");
        connectionFactory.setPassword("admin");

        //获取连接对象
        Connection connection = connectionFactory.newConnection();*/

    //通过工具类获取连接对象
    Connection connection = RabbitMQUtils.getConnection();

    //获取连接中通道
    Channel channel = connection.createChannel();

    //通道绑定对应消息队列
    //参数1:  队列名称 如果队列不存在自动创建
    //参数2:  用来定义队列特性是否要持久化 true 持久化队列   false 不持久化
    //参数3:  exclusive 是否独占队列  true 独占队列   false  不独占
    //参数4:  autoDelete: 是否在消费完成后自动删除队列  true 自动删除  false 不自动删除
    //参数5:  额外附加参数
    channel.queueDeclare("hello",true,false,false,null);

    //消费消息
    channel.basicConsume("hello", true, new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) {
                System.out.println("消费者: " + new String(body));
            }
        });

	}
}
```

#### 4.4.3 参数的说明

```java
channel.queueDeclare("hello", true, false, false, null);
'参数1':用来声明通道对应的队列
'参数2':用来指定是否持久化队列
'参数3':用来指定是否独占队列
'参数4':用来指定是否自动删除队列
'参数5':对队列的额外配置
```

![img](img/20210705212734830.png)

![img](img/20210705212756452.png)

#### 4.4.4 工具类的包装

```java
package utils;


import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.util.Properties;

public class RabbitMQUtils {

  private static ConnectionFactory connectionFactory;
  
  static {
    //重量级资源  类加载执行之执行一次
    //设置连接rabbitmq主机
    factory.setHost("192.168.62.100");
    //设置端口号
    factory.setPort(5672);
    //设置连接那个虚拟主机
    factory.setVirtualHost("/ems");
    //设置访问虚拟主机的用户名和密码
    factory.setUsername("admin");
    factory.setPassword("123");
  }

  //定义提供连接对象的方法
  public static Connection getConnection() {
    try {
      return connectionFactory.newConnection();
    } catch (Exception e) {
      e.printStackTrace();
    }
    return null;
  }

  //关闭通道和关闭连接工具方法
  public static void closeConnectionAndChanel(Channel channel, Connection conn) {
    try {
      if(channel!=null) channel.close();
      if(conn!=null) conn.close();
    } catch (Exception e) {
      e.printStackTrace();
    }
  }

}
```

### 4.5 第二种模型(work queue)

Work queues，也被称为（Task queues），任务模型。当消息处理比较耗时的时候，可能生产消息的速度会远远大于消息的消费速度。长此以往，消息就会堆积越来越多，无法及时处理。此时就可以使用work 模型：让多个消费者绑定到一个队列，共同消费队列中的消息。队列中的消息一旦消费，就会消失，因此任务是不会被重复执行的。

![img](img/20201030180428828.png)

角色：

- P：生产者：任务的发布者
- C1：消费者-1，领取任务并且完成任务，假设完成速度较快
- C2：消费者-2：领取任务并且完成任务，假设完成速度慢

#### 4.5.1 开发生产者

```java
public class Provider {

    public static void main(String[] args) throws IOException {
        Connection connection = RabbitMQUtils.getConnection();
        Channel channel = connection.createChannel();
        channel.queueDeclare("work", true, false, false, null);
        for (int i = 0; i < 10; i++) {
            channel.basicPublish("", "work", null, (i + "====>:我是消息").getBytes());
        }
        RabbitMQUtils.closeConnectionAndChanel(channel, connection);
    }

}
```

#### 4.5.2 开发消费者-1

```java
public class Customer1 {

    public static void main(String[] args) throws IOException {
        Connection connection = RabbitMQUtils.getConnection();
        Channel channel = connection.createChannel();
        //channel.basicQos(1); //每一次只能消费一个消息
        channel.queueDeclare("work", true, false, false, null);
        channel.basicConsume("work", true, new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) {
                System.out.println("消费者1：" + new String(body));
            }
        });
    }

}
```

#### 4.5.3 开发消费者-2

```java
public class Customer2 {

    public static void main(String[] args) throws IOException {
        Connection connection = RabbitMQUtils.getConnection();
        Channel channel = connection.createChannel();
        //channel.basicQos(1); //每一次只能消费一个消息
        channel.queueDeclare("work", true, false, false, null);
        channel.basicConsume("work", true, new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) {
                try {
                    TimeUnit.SECONDS.sleep(3); //处理消息比较慢 3秒处理一个消息
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("消费者2：" + new String(body));
            }
        });
    }

}
```

#### 4.5.4 测试结果

![img](img/20201030180452976.png)

![img](img/20201030180521397.png)

> 总结：默认情况下，RabbitMQ将按顺序将每个消息发送给下一个使用者。平均而言，每个消费者都会收到相同数量的消息。这种分发消息的方式称为循环。

#### 4.5.5 消息自动确认机制

```
Doing a task can take a few seconds. You may wonder what happens if one of the consumers 
starts a long task and dies with it only partly done.With our current code,once RabbitMQ 
delivers a message to the consumer it immediately marks it for deletion. In this case, if you kill a worker we will lose the message it was just processing. We'llalso lose all the messages that were dispatched to this particular worker but were not yet handled.

But we don't want to lose any tasks. If a worker dies,we'd like the task to be delivered to 
another worker.

完成一项任务可能需要几秒钟。您可能想知道，如果其中一个消费者开始了一项长期任务，但只完成了一部分就死了，会发生什么情况。在我们当前的代码中，一旦RabbitMQ将消息传递给使用者，它就会立即将其标记为删除。在这种情况下，如果您杀死一个worker，我们将丢失它刚刚处理的消息。我们还将丢失发送给该特定工作进程但尚未处理的所有消息。

但我们不想失去任何任务。如果一个worker死了，我们希望把任务交给另一个工人。
```

##### 4.5.5.1 开发生产者

```java
public class Provider {

    public static void main(String[] args) throws IOException {
        //获取连接对象
        Connection connection = RabbitMQUtils.getConnection();
        //获取通道对象
        Channel channel = connection.createChannel();
        //通过通道声明队列
        channel.queueDeclare("work", true, false, false, null);
        for (int i = 0; i < 10; i++) {
            //生产消息
            channel.basicPublish("", "work", null, (i + "====>:我是消息").getBytes());
        }
        //关闭资源
        RabbitMQUtils.closeConnectionAndChanel(channel, connection);
    }

}
```

##### 4.5.5.2 开发消费者-1

```java
public class Customer1 {

    public static void main(String[] args) throws IOException {
        Connection connection = RabbitMQUtils.getConnection();
        Channel channel = connection.createChannel();
        channel.basicQos(1); //每一次只能消费一个消息
        channel.queueDeclare("work", true, false, false, null);
        //参数1:队列名称  参数2:消息自动确认 true  消费者自动向rabbitmq确认消息消费  false 不会自动确认消息
        channel.basicConsume("work", false, new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("消费者1：" + new String(body));
                // 参数1:确认队列中那个具体消息 参数2:是否开启多个消息同时确实
                channel.basicAck(envelope.getDeliveryTag(), false);
            }
        });
    }

}
```

##### 4.5.5.3 开发消费者-2

```java
public class Customer2 {

    public static void main(String[] args) throws IOException {
        Connection connection = RabbitMQUtils.getConnection();
        Channel channel = connection.createChannel();
        channel.basicQos(1); //每一次只能消费一个消息
        channel.queueDeclare("work", true, false, false, null);
        channel.basicConsume("work", false, new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                try {
                    TimeUnit.SECONDS.sleep(3); //处理消息比较慢 3秒处理一个消息
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("消费者2：" + new String(body));
                // 参数1:确认队列中那个具体消息 参数2:是否开启多个消息同时确实
                channel.basicAck(envelope.getDeliveryTag(), false);
            }
        });
    }

}
```

##### 4.5.5.4 测试结果

> 设置通道一次只能消费一个消息
> 关闭消息的自动确认，开启手动确认消息

![image-20231218161955515](img/image-20231218161955515.png)

![image-20231218162012549](img/image-20231218162012549.png)

### 4.6 第三种模型(fanout)

fanout 扇出 也称为广播

![img](img/20201030183457220.png)

在广播模式下，消息发送流程是这样的：

可以有多个消费者
每个消费者有自己的queue（队列）
每个队列都要绑定到Exchange（交换机）
生产者发送的消息，只能发送到交换机，交换机来决定要发给哪个队列，生产者无法决定。
交换机把消息发送给绑定过的所有队列，队列的消费者都能拿到消息，实现一条消息被多个消费者消费。

#### 4.6.1 开发生产者

```java
public class Provider {

    public static void main(String[] args) throws IOException {
        // 获取连接对象
        Connection connection = RabbitMQUtils.getConnection();
        Channel channel = connection.createChannel();
        //将通道声明指定交换机   //参数1: 交换机名称    参数2: 交换机类型  fanout 广播类型
        channel.exchangeDeclare("logs", BuiltinExchangeType.FANOUT);
        //发送消息
        channel.basicPublish("logs", "", null, "fanout type message".getBytes());
        // 释放资源
        RabbitMQUtils.closeConnectionAndChanel(channel, connection);
    }

}
```

#### 4.6.2 开发消费者-1

```java
public class Customer1 {

    public static void main(String[] args) throws IOException {
        // 获取连接对象
        Connection connection = RabbitMQUtils.getConnection();
        Channel channel = connection.createChannel();
        // 将通道声明指定交换机
        channel.exchangeDeclare("logs", BuiltinExchangeType.FANOUT);
        // 临时队列
        String queueName = channel.queueDeclare().getQueue();
        // 绑定队列到交换机
        channel.queueBind(queueName, "logs", "");
        // 消费消息
        channel.basicConsume(queueName, true, new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) {
                System.out.println("消费者1：" + new String(body));
            }
        });
    }

}
```

#### 4.6.3 开发消费者-2

```java
public class Customer2 {

    public static void main(String[] args) throws IOException {
        // 获取连接对象
        Connection connection = RabbitMQUtils.getConnection();
        Channel channel = connection.createChannel();
        // 将通道声明指定交换机
        channel.exchangeDeclare("logs", BuiltinExchangeType.FANOUT);
        // 临时队列
        String queueName = channel.queueDeclare().getQueue();
        // 绑定队列到交换机
        channel.queueBind(queueName, "logs", "");
        // 消费消息
        channel.basicConsume(queueName, true, new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) {
                System.out.println("消费者2：" + new String(body));
            }
        });
    }

}
```

#### 4.6.4 开发消费者-3

```java
public class Customer3 {

    public static void main(String[] args) throws IOException {
        // 获取连接对象
        Connection connection = RabbitMQUtils.getConnection();
        Channel channel = connection.createChannel();
        // 将通道声明指定交换机
        channel.exchangeDeclare("logs", BuiltinExchangeType.FANOUT);
        // 临时队列
        String queueName = channel.queueDeclare().getQueue();
        // 绑定队列到交换机
        channel.queueBind(queueName, "logs", "");
        // 消费消息
        channel.basicConsume(queueName, true, new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) {
                System.out.println("消费者3：" + new String(body));
            }
        });
    }

}
```

#### 4.6.5 测试结果

![image-20231218170202049](img/image-20231218170202049.png)

![image-20231218170219971](img/image-20231218170219971.png)

![image-20231218170238074](img/image-20231218170238074.png)

### 4.7 第四种模型(Routing)

#### 4.7.1 Routing 之订阅模型-Direct(直连)

在Fanout模式中，一条消息，会被所有订阅的队列都消费。但是，在某些场景下，我们希望不同的消息被不同的队列消费。这时就要用到Direct类型的Exchange。

在Direct模型下：队列与交换机的绑定，不能是任意绑定了，而是要指定一个RoutingKey（路由key），消息的发送方在 向 Exchange发送消息时，也必须指定消息的 RoutingKey。

Exchange不再把消息交给每一个绑定的队列，而是根据消息的Routing Key进行判断，只有队列的Routingkey与消息的 Routing key完全一致，才会接收到消息。

![img](img/20201030234745232.png)

- P：生产者，向Exchange发送消息，发送消息时，会指定一个routing key。
- X：Exchange（交换机），接收生产者的消息，然后把消息递交给 与routing key完全匹配的队列
- C1：消费者，其所在队列指定了需要routing key 为 error 的消息
- C2：消费者，其所在队列指定了需要routing key 为 info、error、warning 的消息

##### 4.7.1.1 开发生产者

```java
public class Provider {

    public static void main(String[] args) throws IOException {
        //获取连接对象
        Connection connection = RabbitMQUtils.getConnection();
        //获取连接通道对象
        Channel channel = connection.createChannel();
        //通过通道声明交换机  参数1:交换机名称  参数2:direct  路由模式
        String exchangeName = "logs_direct";
        channel.exchangeDeclare(exchangeName, BuiltinExchangeType.DIRECT);
        //发送消息
        String routingkey = "info";
        channel.basicPublish(exchangeName, routingkey, null, ("这是direct模型发布的基于route key: [" + routingkey + "] 发送的消息").getBytes());
        //关闭资源
        RabbitMQUtils.closeConnectionAndChanel(channel, connection);
    }

}
```

##### 4.7.1.2 开发消费者-1

```java
public class Customer1 {

    public static void main(String[] args) throws IOException {
        Connection connection = RabbitMQUtils.getConnection();
        Channel channel = connection.createChannel();
        String exchangeName = "logs_direct";
        //通道声明交换机以及交换的类型
        channel.exchangeDeclare(exchangeName, BuiltinExchangeType.DIRECT);
        //创建一个临时队列
        String queueName = channel.queueDeclare().getQueue();
        //基于route key绑定队列和交换机
        channel.queueBind(queueName, exchangeName, "error");
        //获取消费的消息
        channel.basicConsume(queueName, true, new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) {
                System.out.println("消费者1: " + new String(body));
            }
        });
    }

}
```

##### 4.7.1.3 开发消费者-2

```java
public class Customer2 {

    public static void main(String[] args) throws IOException {
        Connection connection = RabbitMQUtils.getConnection();
        Channel channel = connection.createChannel();
        String exchangeName = "logs_direct";
        //通道声明交换机以及交换的类型
        channel.exchangeDeclare(exchangeName, BuiltinExchangeType.DIRECT);
        //创建一个临时队列
        String queueName = channel.queueDeclare().getQueue();
        //基于route key绑定队列和交换机
        channel.queueBind(queueName, exchangeName, "info");
        channel.queueBind(queueName, exchangeName, "warning");
        channel.queueBind(queueName, exchangeName, "error");
        //获取消费的消息
        channel.basicConsume(queueName, true, new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) {
                System.out.println("消费者2: " + new String(body));
            }
        });
    }

}
```

##### 4.7.1.4 测试生产者发送Route key为error的消息时

![img](img/20210706085347295.png)

![img](img/20210706085405375.png)

##### 4.7.1.5 测试生产者发送Route key为info的消息时

![img](img/20210706085227831.png)

![img](img/20210706085253271.png)

#### 4.7.2 Routing 之订阅模型-Topic

Topic类型的Exchange与Direct相比，都是可以根据RoutingKey把消息路由到不同的队列。只不过Topic类型Exchange可以让队列在绑定Routing key的时候使用通配符！这种模型Routingkey一般都是由一个或多个单词组成，多个单词之间以”.”分割，例如： item.insert

![img](img/20201030234821717.png)

通配符
		`* (star) can substitute for exactly one word.    匹配不多不少恰好1个词`
		`# (hash) can substitute for zero or more words.  匹配零个、一个或多个词`
如:
		`audit.#    匹配audit、audit.irs 、或者audit.irs.corporate等`
	   `audit.*   只能匹配 audit.irs`

##### 4.7.2.1 开发生产者

```java
public class Provider {

    public static void main(String[] args) throws IOException {
        //获取连接对象
        Connection connection = RabbitMQUtils.getConnection();
        Channel channel = connection.createChannel();
        //声明交换机以及交换机类型 topic
        String exchangeName = "topics";
        channel.exchangeDeclare(exchangeName, BuiltinExchangeType.TOPIC);
        //发布消息
        String routekey = "save.user.delete";
        channel.basicPublish(exchangeName, routekey, null, ("这里是topic动态路由模型,routekey: ["+routekey+"]").getBytes());
        //关闭资源
        RabbitMQUtils.closeConnectionAndChanel(channel, connection);
    }

}
```

##### 4.7.2.2 开发消费者-1

Routing Key中使用`*`通配符方式

```java
public class Customer1 {

    public static void main(String[] args) throws IOException {
        //获取连接对象
        Connection connection = RabbitMQUtils.getConnection();
        Channel channel = connection.createChannel();
        //声明交换机以及交换机类型 topic
        String exchangeName = "topics";
        channel.exchangeDeclare(exchangeName, BuiltinExchangeType.TOPIC);
        //创建一个临时队列
        String queueName = channel.queueDeclare().getQueue();
        //绑定队列和交换机  动态统配符形式route key
        channel.queueBind(queueName, exchangeName, "*.user.*");
        //消费消息
        channel.basicConsume(queueName, true, new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) {
                System.out.println("消费者1: "+ new String(body));
            }
        });
    }

}
```

##### 4.7.2.3 开发消费者-2

Routing Key中使用`#`通配符方式

```java
public class Customer2 {

    public static void main(String[] args) throws IOException {
        //获取连接对象
        Connection connection = RabbitMQUtils.getConnection();
        Channel channel = connection.createChannel();
        //声明交换机以及交换机类型 topic
        String exchangeName = "topics";
        channel.exchangeDeclare(exchangeName, BuiltinExchangeType.TOPIC);
        //创建一个临时队列
        String queueName = channel.queueDeclare().getQueue();
        //绑定队列和交换机  动态统配符形式route key
        channel.queueBind(queueName, exchangeName, "*.user.#");
        //消费消息
        channel.basicConsume(queueName, true, new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) {
                System.out.println("消费者2: "+ new String(body));
            }
        });
    }

}
```

##### 4.7.2.4 测试结果

![img](img/20210706091718910.png)

![img](img/20210706091736341.png)

## 5 SpringBoot中使用RabbitMQ

### 5.1 搭建初始环境

#### 5.1.1 引入依赖

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

#### 5.1.2 配置配置文件

```yaml
spring:
  application:
    name: springboot_rabbitmq
  rabbitmq:
    host: 10.15.0.9
    port: 5672
    username: ems
    password: 123
    virtual-host: /ems
```

> RabbitTemplate用来简化操作    使用时候直接在项目中注入即可使用

### 5.2 第一种hello world模型使用

开发生产者

```java
package com.lrw.rabbitmqblrboot.hello;

import org.junit.jupiter.api.Test;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
public class HelloProvider {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Test
    public void provide() {
        rabbitTemplate.convertAndSend("hello", "hello world");
        // 生产端没有指定交换机只有routingKey和Object。
        //消费方产生hello队列，放在默认的交换机(AMQP default)上。
        //而默认的交换机有一个特点，只要你的routerKey的名字与这个
        //交换机的队列有相同的名字，他就会自动路由上。
        //生产端routingKey 叫hello ，消费端生产hello队列。
        //他们就路由上了
    }

}

```

开发消费者

```java
package com.lrw.rabbitmqblrboot.hello;

import org.springframework.amqp.rabbit.annotation.Queue;
import org.springframework.amqp.rabbit.annotation.RabbitHandler;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Component
// 生产端没有指定交换机只有routingKey和Object。
//消费方产生hello队列，放在默认的交换机(AMQP default)上。
//而默认的交换机有一个特点，只要你的routerKey的名字与这个
//交换机的队列有相同的名字，他就会自动路由上。
//生产端routingKey 叫hello ，消费端生产hello队列。
//他们就路由上了
@RabbitListener(queuesToDeclare = @Queue(value = "hello"))
public class HelloCustomer {

    @RabbitHandler
    public void receive1(String message){
        System.out.println("message = " + message);
    }

}

```

![img](img/2021070610294993.png)

### 5.3 第二种work模型使用

开发生产者

```java
package com.lrw.rabbitmqblrboot.work;

import org.junit.jupiter.api.Test;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
public class WorkProvider {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Test
    public void provide() {
        for (int i = 0; i < 10; i++) {
            rabbitTemplate.convertAndSend("work", "hello work" + i);
        }
        // 生产端没有指定交换机只有routingKey和Object。
        //消费方产生work队列，放在默认的交换机(AMQP default)上。
        //而默认的交换机有一个特点，只要你的routerKey的名字与这个
        //交换机的队列有相同的名字，他就会自动路由上。
        //生产端routingKey 叫work ，消费端生产work队列。
        //他们就路由上了
    }

}

```

开发消费者

```java
package com.lrw.rabbitmqblrboot.work;

import org.springframework.amqp.rabbit.annotation.Queue;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Component
public class WorkeCustomer {

    // 生产端没有指定交换机只有routingKey和Object。
    //消费方产生work队列，放在默认的交换机(AMQP default)上。
    //而默认的交换机有一个特点，只要你的routerKey的名字与这个
    //交换机的队列有相同的名字，他就会自动路由上。
    //生产端routingKey 叫work ，消费端生产work队列。
    //他们就路由上了

    @RabbitListener(queuesToDeclare = @Queue("work"))
    public void customer1(String message) {
        System.out.println("work message1 = " + message);
    }

    @RabbitListener(queuesToDeclare = @Queue("work"))
    public void customer2(String message) {
        System.out.println("work message2 = " + message) ;
    }

}

```

![img](img/20210706103510129.png)

> 说明：默认在Spring AMQP实现中Work这种方式就是公平调度,如果需要实现能者多劳需要外配置

### 5.4 Fanout广播模型

开发生产者

```java
package com.lrw.rabbitmqblrboot.fanout;

import org.junit.jupiter.api.Test;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.util.concurrent.TimeUnit;

@SpringBootTest
public class FanoutProvider {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Test
    public void provider() {
        for (int i = 0; i < 10; i++) {
            rabbitTemplate.convertAndSend("fanout", "", "hello fanout" + i);
            // 参数1为交换机，参数2为路由key，“”表示为任意路由，参数3为消息内容
        }
        // 防止消息未处理完，程序就已经结束
        try {
            TimeUnit.SECONDS.sleep(3);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

}

```

开发消费者

```java
package com.lrw.rabbitmqblrboot.fanout;

import org.springframework.amqp.core.ExchangeTypes;
import org.springframework.amqp.rabbit.annotation.Exchange;
import org.springframework.amqp.rabbit.annotation.Queue;
import org.springframework.amqp.rabbit.annotation.QueueBinding;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Component
public class FanoutCustomer {

    @RabbitListener(bindings = @QueueBinding(
            value = @Queue, // 创建临时队列
            exchange = @Exchange(name = "fanout", type = ExchangeTypes.FANOUT) //绑定交换机类型
    ))
    public void customer1(String message) {
        System.out.println("message1 = " + message);
    }

    @RabbitListener(bindings = @QueueBinding(
            value = @Queue,
            exchange = @Exchange(value = "fanout", type = ExchangeTypes.FANOUT)
    ))
    public void customer2(String message) {
        System.out.println("message2 = " + message);
    }

}

```

![img](img/20210706104830274.png)

### 5.5 Route路由模型

开发生产者

```java
package com.lrw.rabbitmqblrboot.route;

import org.junit.jupiter.api.Test;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
public class RouteProvider {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Test
    public void provider() {
        rabbitTemplate.convertAndSend("logs", "error", "info message");
    }

}

```

开发消费者

```java
package com.lrw.rabbitmqblrboot.route;

import org.springframework.amqp.core.ExchangeTypes;
import org.springframework.amqp.rabbit.annotation.Exchange;
import org.springframework.amqp.rabbit.annotation.Queue;
import org.springframework.amqp.rabbit.annotation.QueueBinding;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Component
public class RouteCustomer {

    @RabbitListener(bindings = @QueueBinding(
            value = @Queue,
            exchange = @Exchange(value = "logs", type = ExchangeTypes.DIRECT),
            key = {"info", "error"} // 路由key
    ))
    public void customer1(String message) {
        System.out.println("message1 = " + message);
    }

    @RabbitListener(bindings = @QueueBinding(
            value = @Queue,
            exchange = @Exchange(value = "logs", type = ExchangeTypes.DIRECT),
            key = {"error"} // 路由key
    ))
    public void customer2(String message) {
        System.out.println("message2 = " + message);
    }

}

```

![img](img/20210706105459598.png)

### 5.6 Topic订阅模型（动态路由模型）

开发生产者

```java
package com.lrw.rabbitmqblrboot.topic;

import org.junit.jupiter.api.Test;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
public class TopicProvider {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Test
    public void provider() {
        rabbitTemplate.convertAndSend("topics","user.save.findAll","user.save.findAll 的消息");
    }


}

```

开发消费者

```java
package com.lrw.rabbitmqblrboot.topic;

import org.springframework.amqp.core.ExchangeTypes;
import org.springframework.amqp.rabbit.annotation.Exchange;
import org.springframework.amqp.rabbit.annotation.Queue;
import org.springframework.amqp.rabbit.annotation.QueueBinding;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Component
public class TopicCustomer {

    @RabbitListener(bindings = @QueueBinding(
            value = @Queue,
            exchange = @Exchange(value = "topics", type = ExchangeTypes.TOPIC),
            key = {"user.*"}
    ))
    public void customer1(String message) {
        System.out.println("message1 = " + message);
    }

    @RabbitListener(bindings = @QueueBinding(
            value = @Queue,
            exchange = @Exchange(value = "topics", type = ExchangeTypes.TOPIC),
            key = {"user.#"}
    ))
    public void customer2(String message) {
        System.out.println("message2 = " + message);
    }

}

```

![img](img/20210706110027465.png)

## 6 MQ的应用场景

### 6.1 异步处理

场景说明：用户注册后，需要发注册邮件和注册短信,传统的做法有两种 1.串行的方式 2.并行的方式

串行方式：将注册信息写入数据库后，发送注册邮件，再发送注册短信，以上三个任务全部完成后才返回给客户端。这有一个问题是，邮件，短信并不是必须的，它只是一个通知，而这种做法让客户端等待没有必要等待的东西。

![img](img/20201030225628901.png)

并行方式：将注册信息写入数据库后，发送邮件的同时，发送短信，以上三个任务完成后，返回给客户端，并行的方式能提高处理的时间。 

![img](img/20201030225657273.png)

消息队列：假设三个业务节点分别使用50ms，串行方式使用时间150ms，并行使用时间100ms。虽然并行已经提高的处理时间，但是，前面说过，邮件和短信对我正常的使用网站没有任何影响，客户端没有必要等着其发送完成才显示注册成功，应该是写入数据库后就返回。

消息队列：引入消息队列后，把发送邮件，短信不是必须的业务逻辑异步处理。

![img](img/20201030225723665.jpg)

由此可以看出，引入消息队列后，用户的响应时间就等于写入数据库的时间 + 写入消息队列的时间(可以忽略不计)，引入消息队列后处理后，响应时间是串行的3倍，是并行的2倍。

### 6.2 应用解耦

场景：双11是购物狂节，用户下单后，订单系统需要通知库存系统，传统的做法就是订单系统调用库存系统的接口。

![img](img/20201030225742845.png)

这种做法有一个缺点：当库存系统出现故障时，订单就会失败。 订单系统和库存系统高耦合。

引入消息队列 

![img](img/20201030225800254.png)

订单系统：用户下单后，订单系统完成持久化处理，将消息写入消息队列，返回用户订单下单成功。

库存系统：订阅下单的消息，获取下单消息,进行库操作。就算库存系统出现故障，消息队列也能保证消息的可靠投递，不会导致消息丢失。

### 6.3 流量削峰

场景：秒杀活动，一般会因为流量过大，导致应用挂掉，为了解决这个问题，一般在应用前端加入消息队列。  

作用：

1. 可以控制活动人数，超过此一定阀值的订单直接丢弃(我为什么秒杀一次都没有成功过呢^^) 
2. 可以缓解短时间的高流量压垮应用(应用程序按自己的最大处理能力获取订单) 

![img](img/20201030225837502.png)

用户的请求，服务器收到之后，首先写入消息队列，加入消息队列长度超过最大值，则直接抛弃用户请求或跳转到错误页面。

秒杀业务根据消息队列中的请求信息，再做后续处理。
