为了让大家更快的上手PMQ消息系统，我们这里准备了一个Quick Start，能够帮助大家快速的在本地环境部署、启动PMQ。

[TOC]

# 一、准备工作

## 1.1 Java

*   PMQ服务端：1.8+

由于Quick Start会在本地同时启动服务端、管理端portal，所以需要在本地安装Java 1.8+。 在配置好后，可以通过如下命令检查：

~~~shell
java -version
~~~

样例输出：

~~~shell
java version "1.8.0_74"
Java(TM) SE Runtime Environment (build 1.8.0_74-b02)
Java HotSpot(TM) 64-Bit Server VM (build 25.74-b02, mixed mode)
~~~

## 1.2 MySQL

*   版本要求：5.7+

连接上MySQL后，可以通过如下命令检查：

~~~sql
SHOW VARIABLES WHERE Variable_name = 'version';
~~~

| Variable\_name | Value |
| --- | --- |
| version | 5.7.11 |

## 1.3 maven

需要安装maven3.0，可以去官网下载最新版本的[maven](http://maven.apache.org/download.cgi), maven 环境变量设置请参考[百度](https://jingyan.baidu.com/article/acf728fd68b4bef8e510a31c.html)

配置完成后，打开cmd命令，输入mvn -version,显示下图，表示配置完成。![mvn配置](https://github.com/ppdaicorp/pmq/wiki/img/mvn.png)

# 二、安装步骤

## 2.1 下载代码

下载代码到本地。如d:\\temp,以下所有操作默认代码文件夹为d:\\mq-open

## 2.2 创建数据库

PMQ需要三种类型的数据库，分别为元数据库、正常消息库、和失败消息库。

### 2.2.1元数据库

元数据库用于存储PMQ的topic、consumerGroup、订阅关系等基础信息。元数据库的结构，请参考项目中d:\\mq-open\\doc下的mq\_basic.sql文件。

### 2.2.2正常消息库

正常消息库用于存储普通消息，正常消息库需要多份（建议分布到不同的物理机上）。正常消息库的结构，请参考项目中d:\\mq-open\\doc下的mq\_message\_node\_01.sql文件。

### 2.2.3失败消息库

失败消息库用于存储处理失败的消息，失败消息库也需要多份（建议分布到不同的物理机上）。失败消息库的结构，请参考d:\\mq-open\\doc下的mq\_fail\_message\_node\_01.sql文件。

### 2.2.4创建数据库

根据mq\_basic.sql创建元数据库，元数据库只有一个（可以设置主备）。根据mq\_message\_node\_01.sql创建正常消息库，根据mq\_fail\_message\_node\_01.sql文件创建失败消息库。正常消息库和失败消息库都需要多个（可以根据自己的消息量扩容）。下图是创建的数据库样例，包括：元数据库、两个正常消息库、两个失败消息库。

![mvn 打包](https://github.com/ppdaicorp/pmq/wiki/img/createDBModel.png)

## 2.3 配置数据库连接信息

注意PMQ默认提供了三个环境的配置信息，fat,uat,pro 用户可以自行添加多环境配置信息，当前演示默认发布fat环境。 用文本编辑工具打开d:\\mq\_open\\mq-ui\\src\\main\\resources\\application-fat.properties 和 d:\\mq\_open\\mq-rest\\src\\main\\resources\\application-fat.properties

~~~
spring.datasource.url = jdbc:mysql://localhost:3306/mq_basic?useUnicode=true&characterEncoding=utf8&useSSL=false
spring.datasource.username = root
spring.datasource.password = root

~~~

## 2.4 编译运行

*   打开cmd，进入d:\\mq-open\\mq-client 目录，输入mvn clean install -DskipTests命令,如果显示如下，表示install成功。

![mvn install](https://github.com/ppdaicorp/pmq/wiki/img/install.png)

*   打开cmd，进入d:\\mq-open 目录，输入mvn clean package -DskipTests命令,如果显示如下，表示编译打包成功。

![mvn 打包](https://github.com/ppdaicorp/pmq/wiki/img/build.png)

注意默认情况portal占用8089端口，服务端占用8080端口，测试demo占用8087端口。

*   然后通过cmd 分别进入

> d:\\mq-open\\mq-rest\\target 目录下执行`java -jar mq-rest.jar --spring.profiles.active=fat`启动服务端。

> d:\\mq-open\\mq-ui\\target 目录下执行`java -jar mq-ui.jar --spring.profiles.active=fat`启动管理端portal。

注意：测试demo暂时还不能启动（需要等到消息库初始化完成，并且在portal上手动创建测试程序所需的topic、consumerGroup、订阅关系以后可以启动）

# 三、启动验证消息系统

## 3.1 验证服务端

在chrome浏览器访问[http://localhost:8080/](http://localhost:8080/)如果出现下图表示PMQ服务端启动正常。

![mq-rest-home](https://github.com/ppdaicorp/pmq/wiki/img/mq-rest-home.png)

## 3.1 验证管理端portal

在chrome浏览器访问[http://localhost:8089/](http://localhost:8089/)如果出现登录页面则表示启动正常。

通过用户名`mqadmin`和密码是`mqadmin`,进入portal内容界面。

# 四、消息库初始化

## 4.1元数据库初始化

元数据初始化是为了创建消息库与元数据库中db\_node表、queue表的对应关系。 初始化步骤如下：

*   启动PMQ的管理界面（即mq-ui项目），通过账号mqadmin/mqadmin登录，然后点击左侧的“数据节点管理”进入如下界面：

![数据节点管理](https://github.com/ppdaicorp/pmq/wiki/img/dbNodeManage.png)

*   点击“创建”弹出“创建数据节点”弹框：

![初始化数据库节点](https://github.com/ppdaicorp/pmq/wiki/img/initDBNode.png)

*   数据库的连接信息分为主库和从库，如果没有从库可以不填。数据库名为我们上面创建的消息库的名字。创建样例如下图所示：

![初始化正常消息库](https://github.com/ppdaicorp/pmq/wiki/img/initNormalMessageDb.png)注意：如果“”数据库名”填写的是正常消息库的名字，则“消息类型”需选择正常队列消息（如上图标红所示）。如果“”数据库名”填写的是失败消息库的名字，则“消息类型”需选择失败队列消息。

*   点击“提交”，如果看到如下记录，则说明ppdai\_mq\_message\_node\_01这一个节点初始化成功。

![初始化记录01](https://github.com/ppdaicorp/pmq/wiki/img/initRecord01.png)

*   重复之前的创建步骤，去初始化之前创建的ppdai\_mq\_message\_node\_01、ppdai\_mq\_fail\_2\_3\_message\_node\_01、ppdai\_mq\_fail\_2\_3\_message\_node\_02数据库节点。

# 五、客户端demo演示

完成以上步骤之后，我们就可以进行PMQ的demo：mq-client-test-001的演示了。

*   客户端需要添加PMQ的订阅配置文件。demo：mq-client-test-001已经添加了该配置，如下图所示：

![demoXml](https://github.com/ppdaicorp/pmq/wiki/img/demoXml.png)

messageQueue.xml文件中定义了comsumerGroup、topic、以及它们之间的订阅关系（含义：test1sub订阅了topic：test1和test4）。receiverType指定的是每一个topic的消息处理类。

demo：mq-client-test-001中我们已经添加好了messageQueue.xml配置，此处您无需做任何添加。

*   把messageQueue.xml中配置信息添加到管理界面中

(1)第一步 新建topic

登录管理界面portal，点击左侧的“消息主题管理”进入如下界面：

![消息主题管理](https://github.com/ppdaicorp/pmq/wiki/img/topicManage.png)

点击“创建”进入topic的创建页面,然后在“topic名称”处输入test1，如下所示：

![newTopic](https://github.com/ppdaicorp/pmq/wiki/img/new_topic.png)

点击“立即提交”，完成topic：test1的创建。按照同样的步骤创建topic：test4。

(2)第二步 新建consumerGroup

登录管理界面portal，点击左侧的“消费者组管理”，然后点击“创建”按钮，进入consumerGroup的创建页面：

![newGroup](https://github.com/ppdaicorp/pmq/wiki/img/new_consumerGroup.png)

点击“提交”按钮，重新回到“消费组管理”页面：

![消费者组管理](https://github.com/ppdaicorp/pmq/wiki/img/consumerGroupManage.png)

(3)第三步 创建订阅关系

上图中可以看到上一步创建的消费者组：test1sub。点击右侧的“订阅管理”按钮,进入“test1sub订阅管理”页面：

![subscribe](https://github.com/ppdaicorp/pmq/wiki/img/subscribe.png)

如上图所示：在“主题名称”输入我们要订阅的topic：test1,点击“添加订阅”即可完成test1sub对test1的订阅。 然后在“主题名称”输入我们要订阅的topic：test4,点击“添加订阅”即可完成test1sub对test4的订阅，结果如下图所示：

![sub_result](https://github.com/ppdaicorp/pmq/wiki/img/sub_result.png)

(4)第四步 启动客户端demo

然后通过cmd 进入

> d:\\mq-open\\mq-client-test-001\\target 目录下执行`java -jar mq-client-test-001-1.0.0.jar --spring.profiles.active=fat`启动客户端demo。

进入“队列消费管理”页面，出现下图表示客户端启动正常。![queue_offset](https://github.com/ppdaicorp/pmq/wiki/img/queue_offset.png)

如上图所示，“消费者”列，可以看到客户端实例ip，则说明客户端订阅成功。

(5)消息发送和消费测试

在chrome浏览器访问[http://localhost:8087/test1?topicName=test1&count=1000](http://localhost:8087/test1?topicName=test1&count=1000)可以给topic：test1发送1000条消息。

进入“队列消费管理页面”,如果看到下图数据，则说明test1的消息发送和消费正常.

![publish_consumer](https://github.com/ppdaicorp/pmq/wiki/img/publish_consumer.png)

同理，也可以访问[http://localhost:8087/test1?topicName=test4&count=1000](http://localhost:8087/test1?topicName=test4&count=1000)可以给topic：test4发送1000条消息。

# 六、常见问题

如果启动应用无法启动，请检查 8080，8089,8087端口是否被占用。
