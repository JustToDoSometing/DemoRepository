## 《分布式任务调度平台XXL-JOB》


## 一、快速入门

### 1.1 初始化“调度数据库”
请下载项目源码并解压，获取 "调度数据库初始化SQL脚本" 并执行即可，正常情况下应该生成16张表。

"调度数据库初始化SQL脚本" 位置为:

    /xxl-job/doc/db/tables_xxl_job.sql

调度中心支持集群部署，集群情况下各节点务必连接同一个mysql实例;

如果mysql做主从,调度中心集群节点务必强制走主库;

### 1.2 编译源码
解压源码,按照maven格式将源码导入IDE, 使用maven进行编译即可，源码结构如下：

    xxl-job-admin：调度中心
    xxl-job-core：公共依赖
    xxl-job-executor：执行器Sample示例（选择合适的版本执行器，可直接使用，也可以参考其并将现有项目改造成执行器）
        ：xxl-job-executor-sample-spring：Spring版本，通过Spring容器管理执行器，比较通用，推荐这种方式；
        ：xxl-job-executor-sample-springboot：Springboot版本，通过Springboot管理执行器；
        ：xxl-job-executor-sample-jfinal：JFinal版本，通过JFinal管理执行器；
        ：xxl-job-executor-sample-nutz：Nutz版本，通过Nutz管理执行器；
        

### 1.3 配置部署“调度中心”

    调度中心项目：xxl-job-admin
    作用：统一管理任务调度平台上调度任务，负责触发调度执行，并且提供任务管理平台。

#### 步骤一：调度中心配置：
调度中心配置文件地址：

    /xxl-job/xxl-job-admin/src/main/resources/xxl-job-admin.properties


调度中心配置内容说明：

    ### 调度中心JDBC链接：链接地址请保持和 2.1章节 所创建的调度数据库的地址一致
    xxl.job.db.driverClass=com.mysql.jdbc.Driver
    xxl.job.db.url=jdbc:mysql://localhost:3306/xxl-job?useUnicode=true&characterEncoding=UTF-8
    xxl.job.db.user=root
    xxl.job.db.password=root_pwd
    
    ### 报警邮箱
    xxl.job.mail.host=smtp.163.com
    xxl.job.mail.port=25
    xxl.job.mail.username=ovono802302@163.com
    xxl.job.mail.password=asdfzxcv
    xxl.job.mail.sendFrom=ovono802302@163.com
    xxl.job.mail.sendNick=《任务调度平台XXL-JOB》
    
    ### 登录账号
    xxl.job.login.username=admin
    xxl.job.login.password=123456
    
    ### 调度中心通讯TOKEN，非空时启用
    xxl.job.accessToken=
    
    ### 调度中心国际化设置，默认为中文版本，值设置为“en”时切换为英文版本
    xxl.job.i18n=

#### 步骤二：部署项目：
如果已经正确进行上述配置，可将项目编译打war包并部署到tomcat中。
调度中心访问地址：http://localhost:8080/xxl-job-admin (该地址执行器将会使用到，作为回调地址)，登录后运行界面如下图所示

![输入图片说明](https://static.oschina.net/uploads/img/201705/08194505_6yC0.png "在这里输入图片标题")

至此“调度中心”项目已经部署成功。

#### 步骤三：调度中心集群（可选）：
调度中心支持集群部署，提升调度系统容灾和可用性。

调度中心集群部署时，几点要求和建议：
- DB配置保持一致；
- 登陆账号配置保持一致；
- 集群机器时钟保持一致（单机集群忽视）；
- 建议：推荐通过nginx为调度中心集群做负载均衡，分配域名。调度中心访问、执行器回调配置、调用API服务等操作均通过该域名进行。

### 1.4 配置部署“执行器项目”

    “执行器”项目：xxl-job-executor-sample-spring (提供多种版本执行器供选择，现以Spring版本为例，可直接使用，也可以参考其并将现有项目改造成执行器)
    作用：负责接收“调度中心”的调度并执行；可直接部署执行器，也可以将执行器集成到现有业务项目中。
    
#### 步骤一：maven依赖
确认pom文件中引入了 "xxl-job-core" 的maven依赖；
    
#### 步骤二：执行器配置
执行器配置，配置文件地址：

    /xxl-job/xxl-job-executor-samples/xxl-job-executor-sample-spring/src/main/resources/xxl-job-executor.properties

执行器配置，配置内容说明：

    ### xxl-job admin address list：调度中心部署跟地址：如调度中心集群部署存在多个地址则用逗号分隔。执行器将会使用该地址进行"执行器心跳注册"和"任务结果回调"。
    xxl.job.admin.addresses=http://127.0.0.1:8080/xxl-job-admin
    
    ### xxl-job executor address：执行器"AppName"和地址信息配置：AppName执行器心跳注册分组依据；地址信息用于"调度中心请求并触发任务"和"执行器注册"。执行器默认端口为9999，执行器IP默认为空表示自动获取IP，多网卡时可手动设置指定IP，手动设置IP时将会绑定Host。单机部署多个执行器时，注意要配置不同执行器端口；
    xxl.job.executor.appname=xxl-job-executor-sample
    xxl.job.executor.ip=
    xxl.job.executor.port=9999
    
    ### xxl-job, access token：执行器通讯TOKEN，非空时启用
    xxl.job.accessToken=
        
    ### xxl-job log path：执行器运行日志文件存储的磁盘位置，需要对该路径拥有读写权限
    xxl.job.executor.logpath=/data/applogs/xxl-job/jobhandler/
    
    ### xxl-job log retention days：执行器Log文件定期清理功能，指定日志保存天数，日志文件过期自动删除。限制至少保持3天，否则功能不生效；
    xxl.job.executor.logretentiondays=-1
    

#### 步骤三：执行器组件配置

执行器组件，配置文件地址：

    /xxl-job/xxl-job-executor-samples/xxl-job-executor-sample-spring/src/main/resources/applicationcontext-xxl-job.xml

执行器组件，配置内容说明：

```
<!-- 配置01、JobHandler 扫描路径：自动扫描容器中JobHandler -->
<context:component-scan base-package="com.xxl.job.executor.service.jobhandler" />

<!-- 配置02、执行器 -->
<bean id="xxlJobExecutor" class="com.xxl.job.core.executor.XxlJobExecutor" init-method="start" destroy-method="destroy" >
    <!-- 执行器注册中心地址[选填]，为空则关闭自动注册 -->
    <property name="adminAddresses" value="${xxl.job.admin.addresses}" />
    <!-- 执行器AppName[选填]，为空则关闭自动注册 -->
    <property name="appName" value="${xxl.job.executor.appname}" />
    <!-- 执行器IP[选填]，为空则自动获取 -->
    <property name="ip" value="${xxl.job.executor.ip}" />
    <!-- 执行器端口号[选填]，为空则自动获取 -->
    <property name="port" value="${xxl.job.executor.port}" />
    <!-- 访问令牌[选填]，非空则进行匹配校验 -->
    <property name="accessToken" value="${xxl.job.accessToken}" />
    <!-- 执行器日志路径[选填]，为空则使用默认路径 -->
    <property name="logPath" value="${xxl.job.executor.logpath}" />
    <!-- 日志保存天数[选填]，值大于3时生效 -->
    <property name="logRetentionDays" value="${xxl.job.executor.logretentiondays}" />
</bean>
```

#### 步骤四：部署执行器项目：
如果已经正确进行上述配置，可将执行器项目编译打部署，系统提供多种执行器Sample示例项目，选择其中一个即可，各自的部署方式如下。

    xxl-job-executor-sample-springboot：项目编译打包成springboot类型的可执行JAR包，命令启动即可；
    xxl-job-executor-sample-spring：项目编译打包成WAR包，并部署到tomcat中。
    xxl-job-executor-sample-jfinal：同上
    xxl-job-executor-sample-nutz：同上
    

至此“执行器”项目已经部署结束。

#### 步骤五：执行器集群（可选）：
执行器支持集群部署，提升调度系统可用性，同时提升任务处理能力。

执行器集群部署时，几点要求和建议：
- 执行器回调地址（xxl.job.admin.addresses）需要保持一致；执行器根据该配置进行执行器自动注册等操作。 
- 同一个执行器集群内AppName（xxl.job.executor.appname）需要保持一致；调度中心根据该配置动态发现不同集群的在线执行器列表。


### 1.5 开发第一个任务“Hello World”       
本示例以新建一个 “GLUE模式(Java)” 运行模式的任务为例。更多有关任务的详细配置，请查看“章节三：任务详解”。
（ “GLUE模式(Java)”的执行代码托管到调度中心在线维护，相比“Bean模式任务”需要在执行器项目开发部署上线，更加简便轻量）

> 前提：请确认“调度中心”和“执行器”项目已经成功部署并启动；

#### 步骤一：新建任务：
登录调度中心，点击下图所示“新建任务”按钮，新建示例任务。然后，参考下面截图中任务的参数配置，点击保存。

![输入图片说明](https://static.oschina.net/uploads/img/201704/27205910_o8HQ.png "在这里输入图片标题")

![输入图片说明](https://static.oschina.net/uploads/img/201712/25183654_ZAsz.png "在这里输入图片标题")


#### 步骤二：“GLUE模式(Java)” 任务开发：
请点击任务右侧 “GLUE” 按钮，进入 “GLUE编辑器开发界面” ，见下图。“GLUE模式(Java)” 运行模式的任务默认已经初始化了示例任务代码，即打印Hello World。
（ “GLUE模式(Java)” 运行模式的任务实际上是一段继承自IJobHandler的Java类代码，它在执行器项目中运行，可使用@Resource/@Autowire注入执行器里中的其他服务，详细介绍请查看第三章节）

![输入图片说明](https://static.oschina.net/uploads/img/201704/27210307_Fgql.png "在这里输入图片标题")

![输入图片说明](https://static.oschina.net/uploads/img/201704/27210314_dNUJ.png "在这里输入图片标题")

#### 步骤三：触发执行：
请点击任务右侧 “执行” 按钮，可手动触发一次任务执行（通常情况下，通过配置Cron表达式进行任务调度出发）。

#### 步骤四：查看日志： 
请点击任务右侧 “日志” 按钮，可前往任务日志界面查看任务日志。
在任务日志界面中，可查看该任务的历史调度记录以及每一次调度的任务调度信息、执行参数和执行信息。运行中的任务点击右侧的“执行日志”按钮，可进入日志控制台查看实时执行日志。

![输入图片说明](https://static.oschina.net/uploads/img/201704/27232850_inc8.png "在这里输入图片标题")

在日志控制台，可以Rolling方式实时查看任务在执行器一侧运行输出的日志信息，实时监控任务进度；

![输入图片说明](https://static.oschina.net/uploads/img/201704/27211631_eYrv.png "在这里输入图片标题")

## 二、任务详解

### 配置属性详细说明：

    - 执行器：任务的绑定的执行器，任务触发调度时将会自动发现注册成功的执行器, 实现任务自动发现功能; 另一方面也可以方便的进行任务分组。每个任务必须绑定一个执行器, 可在 "执行器管理" 进行设置;
    - 描述：任务的描述信息，便于任务管理；
    - 路由策略：当执行器集群部署时，提供丰富的路由策略，包括；
        FIRST（第一个）：固定选择第一个机器；
        LAST（最后一个）：固定选择最后一个机器；
        ROUND（轮询）：；
        RANDOM（随机）：随机选择在线的机器；
        CONSISTENT_HASH（一致性HASH）：每个任务按照Hash算法固定选择某一台机器，且所有任务均匀散列在不同机器上。
        LEAST_FREQUENTLY_USED（最不经常使用）：使用频率最低的机器优先被选举；
        LEAST_RECENTLY_USED（最近最久未使用）：最久为使用的机器优先被选举；
        FAILOVER（故障转移）：按照顺序依次进行心跳检测，第一个心跳检测成功的机器选定为目标执行器并发起调度；
        BUSYOVER（忙碌转移）：按照顺序依次进行空闲检测，第一个空闲检测成功的机器选定为目标执行器并发起调度；
        SHARDING_BROADCAST(分片广播)：广播触发对应集群中所有机器执行一次任务，同时传递分片参数；可根据分片参数开发分片任务；
        
    - Cron：触发任务执行的Cron表达式；
    - 运行模式：
        BEAN模式：任务以JobHandler方式维护在执行器端；需要结合 "JobHandler" 属性匹配执行器中任务；
        GLUE模式(Java)：任务以源码方式维护在调度中心；该模式的任务实际上是一段继承自IJobHandler的Java类代码并 "groovy" 源码方式维护，它在执行器项目中运行，可使用@Resource/@Autowire注入执行器里中的其他服务；
        GLUE模式(Shell)：任务以源码方式维护在调度中心；该模式的任务实际上是一段 "shell" 脚本；
        GLUE模式(Python)：任务以源码方式维护在调度中心；该模式的任务实际上是一段 "python" 脚本；
        GLUE模式(NodeJS)：任务以源码方式维护在调度中心；该模式的任务实际上是一段 "nodejs" 脚本；
    - JobHandler：运行模式为 "BEAN模式" 时生效，对应执行器中新开发的JobHandler类“@JobHandler”注解自定义的value值；
    - 子任务：每个任务都拥有一个唯一的任务ID(任务ID可以从任务列表获取)，当本任务执行结束并且执行成功时，将会触发子任务ID所对应的任务的一次主动调度。
    - 阻塞处理策略：调度过于密集执行器来不及处理时的处理策略；
        单机串行（默认）：调度请求进入单机执行器后，调度请求进入FIFO队列并以串行方式运行；
        丢弃后续调度：调度请求进入单机执行器后，发现执行器存在运行的调度任务，本次请求将会被丢弃并标记为失败；
        覆盖之前调度：调度请求进入单机执行器后，发现执行器存在运行的调度任务，将会终止运行中的调度任务并清空队列，然后运行本地调度任务；
    - 失败处理策略；调度失败时的处理策略；
        失败告警（默认）：调度失败和执行失败时，都将会触发失败报警，默认会发送报警邮件；
        失败重试：调度失败时，除了进行失败告警之外，将会自动重试一次；注意在执行失败时不会重试，而是根据回调返回值判断是否重试；
    - 执行参数：任务执行所需的参数，多个参数时用逗号分隔，任务执行时将会把多个参数转换成数组传入；
    - 报警邮件：任务调度失败时邮件通知的邮箱地址，支持配置多邮箱地址，配置多个邮箱地址时用逗号分隔；
    - 负责人：任务的负责人；
    
### 2.1 BEAN模式
任务逻辑以JobHandler的形式存在于“执行器”所在项目中，开发流程如下：

#### 步骤一：执行器项目中，开发JobHandler：

     - 1、继承"IJobHandler"：“com.xxl.job.core.handler.IJobHandler”；
     - 2、注册到Spring容器：添加“@Component”注解，被Spring容器扫描为Bean实例；
     - 3、注册到执行器工厂：添加“@JobHandler(value="自定义jobhandler名称")”注解，注解value值对应的是调度中心新建任务的JobHandler属性的值。
     - 4、执行日志：需要通过 "XxlJobLogger.log" 打印执行日志；
    （可参考Sample示例执行器中的DemoJobHandler，见下图）

![输入图片说明](https://static.oschina.net/uploads/img/201607/23232347_oLlM.png "在这里输入图片标题")

#### 步骤二：调度中心，新建调度任务
参考上文“配置属性详细说明”对新建的任务进行参数配置，运行模式选中 "BEAN模式"，JobHandler属性填写任务注解“@JobHandler”中定义的值；

![输入图片说明](https://static.oschina.net/uploads/img/201712/25183654_ZAsz.png "在这里输入图片标题")

### 2.2 GLUE模式(Java)
任务以源码方式维护在调度中心，支持通过Web IDE在线更新，实时编译和生效，因此不需要指定JobHandler。开发流程如下：

#### 步骤一：调度中心，新建调度任务：
参考上文“配置属性详细说明”对新建的任务进行参数配置，运行模式选中 "GLUE模式(Java)"；

![输入图片说明](https://static.oschina.net/uploads/img/201712/25183837_tJOq.png "在这里输入图片标题")

#### 步骤二：开发任务代码：
选中指定任务，点击该任务右侧“GLUE”按钮，将会前往GLUE任务的Web IDE界面，在该界面支持对任务代码进行开发（也可以在IDE中开发完成后，复制粘贴到编辑中）。

版本回溯功能（支持30个版本的版本回溯）：在GLUE任务的Web IDE界面，选择右上角下拉框“版本回溯”，会列出该GLUE的更新历史，选择相应版本即可显示该版本代码，保存后GLUE代码即回退到对应的历史版本；

![输入图片说明](https://static.oschina.net/uploads/img/201704/27210314_dNUJ.png "在这里输入图片标题")

### 2.3 GLUE模式(Shell)

#### 步骤一：调度中心，新建调度任务   
参考上文“配置属性详细说明”对新建的任务进行参数配置，运行模式选中 "GLUE模式(Shell)"；

#### 步骤二：开发任务代码：
选中指定任务，点击该任务右侧“GLUE”按钮，将会前往GLUE任务的Web IDE界面，在该界面支持对任务代码进行开发（也可以在IDE中开发完成后，复制粘贴到编辑中）。

该模式的任务实际上是一段 "shell" 脚本；

![输入图片说明](https://static.oschina.net/uploads/img/201704/27232259_iUw0.png "在这里输入图片标题")

### 2.4 GLUE模式(Python)

#### 步骤一：调度中心，新建调度任务   
参考上文“配置属性详细说明”对新建的任务进行参数配置，运行模式选中 "GLUE模式(Python)"；

#### 步骤二：开发任务代码：
选中指定任务，点击该任务右侧“GLUE”按钮，将会前往GLUE任务的Web IDE界面，在该界面支持对任务代码进行开发（也可以在IDE中开发完成后，复制粘贴到编辑中）。

该模式的任务实际上是一段 "python" 脚本；

![输入图片说明](https://static.oschina.net/uploads/img/201704/27232305_BPLG.png "在这里输入图片标题")

### 2.5 GLUE模式(NodeJS)

#### 步骤一：调度中心，新建调度任务   
参考上文“配置属性详细说明”对新建的任务进行参数配置，运行模式选中 "GLUE模式(NodeJS)"；

#### 步骤二：开发任务代码：
选中指定任务，点击该任务右侧“GLUE”按钮，将会前往GLUE任务的Web IDE界面，在该界面支持对任务代码进行开发（也可以在IDE中开发完成后，复制粘贴到编辑中）。

该模式的任务实际上是一段 "nodejS" 脚本；


## 三、任务管理

### 3.0 配置执行器  
点击进入"执行器管理"界面, 如下图:
![输入图片说明](https://static.oschina.net/uploads/img/201703/12223509_Hr2T.png "在这里输入图片标题")

    1、"调度中心OnLine:"右侧显示在线的"调度中心"列表, 任务执行结束后, 将会以failover的模式进行回调调度中心通知执行结果, 避免回调的单点风险;
    2、"执行器列表" 中显示在线的执行器列表, 可通过"OnLine 机器"查看对应执行器的集群机器。

点击按钮 "+新增执行器" 弹框如下图, 可新增执行器配置:

![输入图片说明](https://static.oschina.net/uploads/img/201712/25183958_V3vF.png "在这里输入图片标题")

### 执行器属性说明

    AppName: 是每个执行器集群的唯一标示AppName, 执行器会周期性以AppName为对象进行自动注册。可通过该配置自动发现注册成功的执行器, 供任务调度时使用;
    名称: 执行器的名称, 因为AppName限制字母数字等组成,可读性不强, 名称为了提高执行器的可读性;
    排序: 执行器的排序, 系统中需要执行器的地方,如任务新增, 将会按照该排序读取可用的执行器列表;
    注册方式：调度中心获取执行器地址的方式；
        自动注册：执行器自动进行执行器注册，调度中心通过底层注册表可以动态发现执行器机器地址；
        手动录入：人工手动录入执行器的地址信息，多地址逗号分隔，供调度中心使用；
    机器地址："注册方式"为"手动录入"时有效，支持人工维护执行器的地址信息；

### 3.1 新建任务
进入任务管理界面，点击“新增任务”按钮，在弹出的“新增任务”界面配置任务属性后保存即可。详情页参考章节 "三、任务详解"。

### 3.2 编辑任务
进入任务管理界面，选中指定任务。点击该任务右侧“编辑”按钮，在弹出的“编辑任务”界面更新任务属性后保存即可，可以修改设置的任务属性信息：

### 3.3 编辑GLUE代码

该操作仅针对GLUE任务。

选中指定任务，点击该任务右侧“GLUE”按钮，将会前往GLUE任务的Web IDE界面，在该界面支持对任务代码进行开发。可参考章节 "3.2 GLUE模式(Java)"。

### 3.4 暂停/恢复任务
可对任务进行“暂停”和“恢复”操作。
需要注意的是，此处的暂停/恢复仅针对任务的后续调度触发行为，不会影响到已经触发的调度任务，如需终止已经触发的调度任务，可查看“4.8 终止运行中的任务”

![输入图片说明](https://static.oschina.net/uploads/img/201607/24130337_ZAhX.png "在这里输入图片标题")

### 3.5 手动触发一次调度
点击“执行”按钮，可手动触发一次任务调度，不影响原有调度规则。

![输入图片说明](https://static.oschina.net/uploads/img/201607/24133348_Z5wp.png "在这里输入图片标题")

### 3.6 查看调度日志
点击“日志”按钮，可以查看任务历史调度日志。在历史调入日志界面可查看每次任务调度的调度结果、执行结果等，点击“执行日志”按钮可查看执行器完整日志。

![输入图片说明](https://static.oschina.net/uploads/img/201607/24133500_9235.png "在这里输入图片标题")

![输入图片说明](https://static.oschina.net/uploads/img/201712/25184206_UDSo.png "在这里输入图片标题")

    调度时间："调度中心"触发本次调度并向"执行器"发送任务执行信号的时间；
    调度结果："调度中心"触发本次调度的结果，200表示成功，500或其他表示失败；
    调度备注："调度中心"触发本次调度的日志信息；
    执行器地址：本次任务执行的机器地址
    运行模式：触发调度时任务的运行模式，运行模式可参考章节 "三、任务详解"；
    任务参数：本地任务执行的入参
    执行时间："执行器"中本次任务执行结束后回调的时间；
    执行结果："执行器"中本次任务执行的结果，200表示成功，500或其他表示失败；
    执行备注："执行器"中本次任务执行的日志信息；
    操作：
        "执行日志"按钮：点击可查看本地任务执行的详细日志信息；详见“4.7 查看执行日志”；
        "终止任务"按钮：点击可终止本地调度对应执行器上本任务的执行线程，包括未执行的阻塞任务一并被终止；

### 3.7 查看执行日志
点击执行日志右侧的 “执行日志” 按钮，可跳转至执行日志界面，可以查看业务代码中打印的完整日志，如下图；

![输入图片说明](https://static.oschina.net/uploads/img/201703/25124816_tvGI.png "在这里输入图片标题")

### 3.8 终止运行中的任务
仅针对执行中的任务。
在任务日志界面，点击右侧的“终止任务”按钮，将会向本次任务对应的执行器发送任务终止请求，将会终止掉本次任务，同时会清空掉整个任务执行队列。

![输入图片说明](https://static.oschina.net/uploads/img/201607/24140048_hIci.png "在这里输入图片标题")

任务终止时通过 "interrupt" 执行线程的方式实现, 将会触发 "InterruptedException" 异常。因此如果JobHandler内部catch到了该异常并消化掉的话, 任务终止功能将不可用。

因此, 如果遇到上述任务终止不可用的情况, 需要在JobHandler中应该针对 "InterruptedException" 异常进行特殊处理 (向上抛出) , 正确逻辑如下:
```
try{
    // TODO
} catch (Exception e) {
    if (e instanceof InterruptedException) {
        throw e;
    }
    logger.warn("{}", e);
}
```

而且，在JobHandler中开启子线程时，子线程也不可catch处理"InterruptedException"，应该主动向上抛出。


### 3.9 删除执行日志
在任务日志界面，选中执行器和任务之后，点击右侧的"删除"按钮将会出现"日志清理"弹框，弹框中支持选择不同类型的日志清理策略，选中后点击"确定"按钮即可进行日志清理操作；
![输入图片说明](https://static.oschina.net/uploads/img/201705/08210711_Ypik.png "在这里输入图片标题")

![输入图片说明](https://static.oschina.net/uploads/img/201705/08211152_EB65.png "在这里输入图片标题")

### 3.10 删除任务
点击删除按钮，可以删除对应任务。

![输入图片说明](https://static.oschina.net/uploads/img/201607/24140641_Z9Qr.png "在这里输入图片标题")


## 四、总体设计
### 4.1 源码目录介绍
    - /doc :文档资料
    - /db :“调度数据库”建表脚本
    - /xxl-job-admin :调度中心，项目源码
    - /xxl-job-core :公共Jar依赖
    - /xxl-job-executor-samples :执行器，Sample示例项目（大家可以在该项目上进行开发，也可以将现有项目改造生成执行器项目）

### 4.2 “调度数据库”配置
XXL-JOB调度模块基于Quartz集群实现，其“调度数据库”是在Quartz的11张集群mysql表基础上扩展而成。

XXL-JOB首先定制了Quartz原生表结构前缀（XXL_JOB_QRTZ_）。

![输入图片说明](https://static.oschina.net/uploads/img/201607/24143957_bNwm.png "在这里输入图片标题")

然后，在此基础上新增了几张张扩展表，如下：
    - XXL_JOB_QRTZ_TRIGGER_GROUP：执行器信息表，维护任务执行器信息；
    - XXL_JOB_QRTZ_TRIGGER_REGISTRY：执行器注册表，维护在线的执行器和调度中心机器地址信息；
    - XXL_JOB_QRTZ_TRIGGER_INFO：调度扩展信息表： 用于保存XXL-JOB调度任务的扩展信息，如任务分组、任务名、机器地址、执行器、执行入参和报警邮件等等；
    - XXL_JOB_QRTZ_TRIGGER_LOG：调度日志表： 用于保存XXL-JOB任务调度的历史信息，如调度结果、执行结果、调度入参、调度机器和执行器等等；
    - XXL_JOB_QRTZ_TRIGGER_LOGGLUE：任务GLUE日志：用于保存GLUE更新历史，用于支持GLUE的版本回溯功能；

因此，XXL-JOB调度数据库共计用于16张数据库表。

### 4.3 架构设计
#### 4.3.1 设计思想
将调度行为抽象形成“调度中心”公共平台，而平台自身并不承担业务逻辑，“调度中心”负责发起调度请求。

将任务抽象成分散的JobHandler，交由“执行器”统一管理，“执行器”负责接收调度请求并执行对应的JobHandler中业务逻辑。

因此，“调度”和“任务”两部分可以相互解耦，提高系统整体稳定性和扩展性；

#### 4.3.2 系统组成
- **调度模块（调度中心）**：
    负责管理调度信息，按照调度配置发出调度请求，自身不承担业务代码。调度系统与任务解耦，提高了系统可用性和稳定性，同时调度系统性能不再受限于任务模块；
    支持可视化、简单且动态的管理调度信息，包括任务新建，更新，删除，GLUE开发和任务报警等，所有上述操作都会实时生效，同时支持监控调度结果以及执行日志，支持执行器Failover。
- **执行模块（执行器）**：
    负责接收调度请求并执行任务逻辑。任务模块专注于任务的执行等操作，开发和维护更加简单和高效；
    接收“调度中心”的执行请求、终止请求和日志请求等。

#### 4.3.3 架构图

![输入图片说明](https://static.oschina.net/uploads/img/201801/03103007_Qohm.png "在这里输入图片标题")

### 4.4 调度模块剖析
#### 4.4.1 quartz的不足
Quartz作为开源作业调度中的佼佼者，是作业调度的首选。但是集群环境中Quartz采用API的方式对任务进行管理，从而可以避免上述问题，但是同样存在以下问题：
    - 问题一：调用API的的方式操作任务，不人性化；
    - 问题二：需要持久化业务QuartzJobBean到底层数据表中，系统侵入性相当严重。
    - 问题三：调度逻辑和QuartzJobBean耦合在同一个项目中，这将导致一个问题，在调度任务数量逐渐增多，同时调度任务逻辑逐渐加重的情况加，此时调度系统的性能将大大受限于业务；
XXL-JOB弥补了quartz的上述不足之处。

#### 4.4.2 RemoteHttpJobBean
常规Quartz的开发，任务逻辑一般维护在QuartzJobBean中，耦合很严重。XXL-JOB中“调度模块”和“任务模块”完全解耦，调度模块中的所有调度任务使用同一个QuartzJobBean，即RemoteHttpJobBean。不同的调度任务将各自参数维护在各自扩展表数据中，当触发RemoteHttpJobBean执行时，将会解析不同的任务参数发起远程调用，调用各自的远程执行器服务。

这种调用模型类似RPC调用，RemoteHttpJobBean提供调用代理的功能，而执行器提供远程服务的功能。

#### 4.4.3 调度中心HA（集群）
基于Quartz的集群方案，数据库选用Mysql；集群分布式并发环境中使用QUARTZ定时任务调度，会在各个节点会上报任务，存到数据库中，执行时会从数据库中取出触发器来执行，如果触发器的名称和执行时间相同，则只有一个节点去执行此任务。

```
# for cluster
org.quartz.jobStore.tablePrefix = XXL_JOB_QRTZ_
org.quartz.scheduler.instanceId: AUTO
org.quartz.jobStore.class: org.quartz.impl.jdbcjobstore.JobStoreTX
org.quartz.jobStore.isClustered: true
org.quartz.jobStore.clusterCheckinInterval: 1000
```

#### 4.4.4 调度线程池
调度采用线程池方式实现，避免单线程因阻塞而引起任务调度延迟。

```
org.quartz.threadPool.class: org.quartz.simpl.SimpleThreadPool
org.quartz.threadPool.threadCount: 15
org.quartz.threadPool.threadPriority: 5
org.quartz.threadPool.threadsInheritContextClassLoaderOfInitializingThread: true
```

XXL-JOB系统中业务逻辑在远程执行器执行，全异步化设计，调度中心每次触发调度时仅发送一次调度请求，执行器会将请求存入执行队列并且立即响应调度中心，异步运行；相比直接在quartz的QuartzJobBean中执行业务逻辑，极大的降低了调度线程占用时间；

XXL-JOB调度中心中每个JOB逻辑非常 “轻”，单个JOB一次运行平均耗时基本在 "10ms" 之内（基本为一次请求的网络开销）；因此，可以保证使用有限的线程支撑大量的JOB并发运行；

理论支撑任务量公式如下：

    理论支撑任务量 = 线程数配置 / 平均调度频率（每秒） * 平均触发耗时（单位s）

理论上采用推荐机器配置 "4核4G内存" + "配置1s运行1次密集任务" + "调度中心与执行器ping延迟10ms（0.01s）" 的情况下，

    - 单线程支撑任务量  ：1 / 1 * 0.01 = 100个任务
    - 15个线程支撑任务量：15 / 1 * 0.01 = 1500个任务

实际场景中，由于调度中心与执行器ping延迟不同、DB读写耗时不同、任务调度密集程度不同，会导致任务量上限会上下波动。

如若需要支撑更多的任务量，可以通过 "调大调度线程数" 、"降低调度中心与执行器ping延迟" 和 "提升机器配置" 几种方式实现。

#### 4.4.5 @DisallowConcurrentExecution
XXL-JOB调度模块的“调度中心”默认不使用该注解，即默认开启并行机制，因为RemoteHttpJobBean为公共QuartzJobBean，这样在多线程调度的情况下，调度模块被阻塞的几率很低，大大提高了调度系统的承载量。

XXL-JOB的每个调度任务虽然在调度模块是并行调度执行的，但是任务调度传递到任务模块的“执行器”确实串行执行的，同时支持任务终止。

#### 4.4.6 misfire
错过了触发时间，处理规则。
可能原因：服务重启；调度线程被QuartzJobBean阻塞，线程被耗尽；某个任务启用了@DisallowConcurrentExecution，上次调度持续阻塞，下次调度被错过；

quartz.properties中关于misfire的阀值配置如下，单位毫秒：
```
org.quartz.jobStore.misfireThreshold: 60000
```

Misfire规则：
    withMisfireHandlingInstructionDoNothing：不触发立即执行，等待下次调度；
    withMisfireHandlingInstructionIgnoreMisfires：以错过的第一个频率时间立刻开始执行；
    withMisfireHandlingInstructionFireAndProceed：以当前时间为触发频率立刻触发一次执行；

XXL-JOB默认misfire规则为：withMisfireHandlingInstructionDoNothing

```
CronScheduleBuilder cronScheduleBuilder = CronScheduleBuilder.cronSchedule(jobInfo.getJobCron()).withMisfireHandlingInstructionDoNothing();
CronTrigger cronTrigger = TriggerBuilder.newTrigger().withIdentity(triggerKey).withSchedule(cronScheduleBuilder).build();
```

#### 4.4.7 日志回调服务
调度模块的“调度中心”作为Web服务部署时，一方面承担调度中心功能，另一方面也为执行器提供API服务。

调度中心提供的"日志回调服务API服务"代码位置如下：
```
xxl-job-admin#com.xxl.job.admin.controller.JobApiController.callback
```

“执行器”在接收到任务执行请求后，执行任务，在执行结束之后会将执行结果回调通知“调度中心”：

#### 4.4.8 任务HA（Failover）
执行器如若集群部署，调度中心将会感知到在线的所有执行器，如“127.0.0.1:9997, 127.0.0.1:9998, 127.0.0.1:9999”。

当任务"路由策略"选择"故障转移(FAILOVER)"时，当调度中心每次发起调度请求时，会按照顺序对执行器发出心跳检测请求，第一个检测为存活状态的执行器将会被选定并发送调度请求。

调度成功后，可在日志监控界面查看“调度备注”，如下；
![输入图片说明](https://static.oschina.net/uploads/img/201703/12230733_jrdI.png "在这里输入图片标题")

“调度备注”可以看出本地调度运行轨迹，执行器的"注册方式"、"地址列表"和任务的"路由策略"。"故障转移(FAILOVER)"路由策略下，调度中心首先对第一个地址进行心跳检测，心跳失败因此自动跳过，第二个依然心跳检测失败……
直至心跳检测第三个地址“127.0.0.1:9999”成功，选定为“目标执行器”；然后对“目标执行器”发送调度请求，调度流程结束，等待执行器回调执行结果。

#### 4.4.9 调度日志
调度中心每次进行任务调度，都会记录一条任务日志，任务日志主要包括以下三部分内容：

- 任务信息：包括“执行器地址”、“JobHandler”和“执行参数”等属性，点击任务ID按钮可查看，根据这些参数，可以精确的定位任务执行的具体机器和任务代码；
- 调度信息：包括“调度时间”、“调度结果”和“调度日志”等，根据这些参数，可以了解“调度中心”发起调度请求时具体情况。
- 执行信息：包括“执行时间”、“执行结果”和“执行日志”等，根据这些参数，可以了解在“执行器”端任务执行的具体情况；

调度日志，针对单次调度，属性说明如下：
- 执行器地址：任务执行的机器地址；
- JobHandler：Bean模式表示任务执行的JobHandler名称；
- 任务参数：任务执行的入参；
- 调度时间：调度中心，发起调度的时间；
- 调度结果：调度中心，发起调度的结果，SUCCESS或FAIL；
- 调度备注：调度中心，发起调度的备注信息，如地址心跳检测日志等；
- 执行时间：执行器，任务执行结束后回调的时间；
- 执行结果：执行器，任务执行的结果，SUCCESS或FAIL；
- 执行备注：执行器，任务执行的备注信息，如异常日志等；
- 执行日志：任务执行过程中，业务代码中打印的完整执行日志，见“4.7 查看执行日志”；

#### 4.4.10 任务依赖
原理：XXL-JOB中每个任务都对应有一个任务ID，同时，每个任务支持设置属性“子任务ID”，因此，通过“任务ID”可以匹配任务依赖关系。

当父任务执行结束并且执行成功时，将会根据“子任务ID”匹配子任务依赖，如果匹配到子任务，将会主动触发一次子任务的执行。

在任务日志界面，点击任务的“执行备注”的“查看”按钮，可以看到匹配子任务以及触发子任务执行的日志信息，如无信息则表示未触发子任务执行，可参考下图。

![输入图片说明](https://static.oschina.net/uploads/img/201607/24194134_Wb2o.png "在这里输入图片标题")

![输入图片说明](https://static.oschina.net/uploads/img/201607/24194212_jOAU.png "在这里输入图片标题")

### 4.5 任务 "运行模式" 剖析
#### 4.5.1 "Bean模式" 任务
开发步骤：可参考 "章节三" ；
原理：每个Bean模式任务都是一个Spring的Bean类实例，它被维护在“执行器”项目的Spring容器中。任务类需要加“@JobHandler(value="名称")”注解，因为“执行器”会根据该注解识别Spring容器中的任务。任务类需要继承统一接口“IJobHandler”，任务逻辑在execute方法中开发，因为“执行器”在接收到调度中心的调度请求时，将会调用“IJobHandler”的execute方法，执行任务逻辑。

#### 4.5.2 "GLUE模式(Java)" 任务
开发步骤：可参考 "章节三" ；
原理：每个 "GLUE模式(Java)" 任务的代码，实际上是“一个继承自“IJobHandler”的实现类的类代码”，“执行器”接收到“调度中心”的调度请求时，会通过Groovy类加载器加载此代码，实例化成Java对象，同时注入此代码中声明的Spring服务（请确保Glue代码中的服务和类引用在“执行器”项目中存在），然后调用该对象的execute方法，执行任务逻辑。

#### 4.5.3 GLUE模式(Shell) + GLUE模式(Python) + GLUE模式(NodeJS)
开发步骤：可参考 "章节三" ；
原理：脚本任务的源码托管在调度中心，脚本逻辑在执行器运行。当触发脚本任务时，执行器会加载脚本源码在执行器机器上生成一份脚本文件，然后通过Java代码调用该脚本；并且实时将脚本输出日志写到任务日志文件中，从而在调度中心可以实时监控脚本运行情况；脚本返回码为0时表示执行成功，其他标示执行失败。

目前支持的脚本类型如下：

    - shell脚本：任务运行模式选择为 "GLUE模式(Shell)"时支持 "shell" 脚本任务；
    - python脚本：任务运行模式选择为 "GLUE模式(Python)"时支持 "python" 脚本任务；
    - nodejs脚本：务运行模式选择为 "GLUE模式(NodeJS)"时支持 "nodejs" 脚本任务；
    

#### 4.5.4 执行器
执行器实际上是一个内嵌的Jetty服务器，默认端口9999（配置项：xxl.job.executor.port）。

在项目启动时，执行器会通过“@JobHandler”识别Spring容器中“Bean模式任务”，以注解的value属性为key管理起来。

“执行器”接收到“调度中心”的调度请求时，如果任务类型为“Bean模式”，将会匹配Spring容器中的“Bean模式任务”，然后调用其execute方法，执行任务逻辑。如果任务类型为“GLUE模式”，将会加载GLue代码，实例化Java对象，注入依赖的Spring服务（注意：Glue代码中注入的Spring服务，必须存在与该“执行器”项目的Spring容器中），然后调用execute方法，执行任务逻辑。

#### 4.5.5 任务日志
XXL-JOB会为每次调度请求生成一个单独的日志文件，需要通过 "XxlJobLogger.log" 打印执行日志，“调度中心”查看执行日志时将会加载对应的日志文件。

(历史版本通过重写LOG4J的Appender实现，存在依赖限制，该方式在新版本已经被抛弃)

日志文件存放的位置可在“执行器”配置文件进行自定义，默认目录格式为：/data/applogs/xxl-job/jobhandler/“格式化日期”/“数据库调度日志记录的主键ID.log”。

在JobHandler中开启子线程时，子线程将会将会把日志打印在父线程即JobHandler的执行日志中，方便日志追踪。

### 4.6 通讯模块剖析

#### 4.6.1 一次完整的任务调度通讯流程 
    - 1、“调度中心”向“执行器”发送http调度请求: “执行器”中接收请求的服务，实际上是一台内嵌jetty服务器，默认端口9999;
    - 2、“执行器”执行任务逻辑；
    - 3、“执行器”http回调“调度中心”调度结果: “调度中心”中接收回调的服务，是针对执行器开放一套API服务;

#### 4.6.2 通讯数据加密
调度中心向执行器发送的调度请求时使用RequestModel和ResponseModel两个对象封装调度请求参数和响应数据, 在进行通讯之前底层会将上述两个对象对象序列化，并进行数据协议以及时间戳检验,从而达到数据加密的功能;

### 4.7 任务注册, 任务自动发现   
自v1.5版本之后, 任务取消了"任务执行机器"属性, 改为通过任务注册和自动发现的方式, 动态获取远程执行器地址并执行。

    AppName: 每个执行器机器集群的唯一标示, 任务注册以 "执行器" 为最小粒度进行注册; 每个任务通过其绑定的执行器可感知对应的执行器机器列表;
    注册表: 见"XXL_JOB_QRTZ_TRIGGER_REGISTRY"表, "执行器" 在进行任务注册时将会周期性维护一条注册记录，即机器地址和AppName的绑定关系; "调度中心" 从而可以动态感知每个AppName在线的机器列表;
    执行器注册: 任务注册Beat周期默认30s; 执行器以一倍Beat进行执行器注册, 调度中心以一倍Beat进行动态任务发现; 注册信息的失效时间被三倍Beat; 
    执行器注册摘除：执行器销毁时，将会主动上报调度中心并摘除对应的执行器机器信息，提高心跳注册的实时性；
    

为保证系统"轻量级"并且降低学习部署成本，没有采用Zookeeper作为注册中心，采用DB方式进行任务注册发现；

### 4.8 任务执行结果
自v1.6.2之后，任务执行结果通过 "IJobHandler" 的返回值 "ReturnT" 进行判断；
当返回值符合 "ReturnT.code == ReturnT.SUCCESS_CODE" 时表示任务执行成功，否则表示任务执行失败，而且可以通过 "ReturnT.msg" 回调错误信息给调度中心；
从而，在任务逻辑中可以方便的控制任务执行结果；

### 4.9 分片广播 & 动态分片   
执行器集群部署时，任务路由策略选择"分片广播"情况下，一次任务调度将会广播触发对应集群中所有执行器执行一次任务，同时传递分片参数；可根据分片参数开发分片任务；

"分片广播" 以执行器为维度进行分片，支持动态扩容执行器集群从而动态增加分片数量，协同进行业务处理；在进行大数据量业务操作时可显著提升任务处理能力和速度。

"分片广播" 和普通任务开发流程一致，不同之处在于可以可以获取分片参数，获取分片参数进行分片业务处理。

- Java语言任务获取分片参数方式：BEAN、GLUE模式(Java)
```
// 可参考Sample示例执行器中的示例任务"ShardingJobHandler"了解试用 
ShardingUtil.ShardingVO shardingVO = ShardingUtil.getShardingVo();
```
- 脚本语言任务获取分片参数方式：GLUE模式(Shell)、GLUE模式(Python)、GLUE模式(Nodejs)
```
// 脚本任务入参固定为三个，依次为：任务传参、分片序号、分片总数。以Shell模式任务为例，获取分片参数代码如下
echo "分片序号 index = $2"
echo "分片总数 total = $3"
```  
    
分片参数属性说明：

    index：当前分片序号(从0开始)，执行器集群列表中当前执行器的序号；
    total：总分片数，执行器集群的总机器数量；

该特性适用场景如：
- 1、分片任务场景：10个执行器的集群来处理10w条数据，每台机器只需要处理1w条数据，耗时降低10倍；
- 2、广播任务场景：广播执行器机器运行shell脚本、广播集群节点进行缓存更新等

### 4.10 访问令牌（AccessToken）
为提升系统安全性，调度中心和执行器进行安全性校验，双方AccessToken匹配才允许通讯；

调度中心和执行器，可通过配置项 "xxl.job.accessToken" 进行AccessToken的设置。

调度中心和执行器，如果需要正常通讯，只有两种设置；

- 设置一：调度中心和执行器，均不设置AccessToken；关闭安全性校验；
- 设置二：调度中心和执行器，设置了相同的AccessToken；

### 4.11 调度中心API服务
调度中心提供了API服务，供执行器和业务方选择使用，目前提供的API服务有：

    1、任务结果回调服务；
    2、执行器注册服务；
    3、执行器注册摘除服务；
    4、触发任务单次执行服务，支持任务根据业务事件触发；
    
调度中心API服务位置：com.xxl.job.core.biz.AdminBiz.java

调度中心API服务请求参考代码：com.xxl.job.adminbiz.AdminBizTest.java

### 4.12 执行器API服务
执行器提供了API服务，供调度中心选择使用，目前提供的API服务有：

    1、心跳检测
    2、忙碌检测
    3、触发任务执行
    4、获取Rolling Log
    5、终止任务
    
执行器API服务位置：com.xxl.job.core.biz.ExecutorBiz

执行器API服务请求参考代码：com.xxl.executor.test.DemoJobHandlerTest

### 4.13 故障转移 & 失败重试
一次完整任务流程包括"调度（调度中心） + 执行（执行器）"两个阶段。
    
- "故障转移"发生在调度阶段，在执行器集群部署时，如果某一台执行器发生故障，该策略支持自动进行Failover切换到一台正常的执行器机器并且完成调度请求流程。
- "失败重试"发生在"调度 + 执行"两个阶段，如下：
    - 调度中心调度失败时，任务失败处理策略选择"失败重试"，将会自动重试一次；
    - 执行器运行失败时，任务执行结果返回"失败重试（IJobHandler.FAIL_RETRY）"回调，将会自动重试一次；


