---
layout:     post
title:      "hadoop笔记: 在ubuntu上安装hadoop及其组件"
subtitle:   "含碰到过的各种问题坑"
date:       2018-03-10 03:00:00
author:     "Yuna"
header-img: "img/home-bg.jpg"
tags:
    - linux
    - hadoop相关
    - 安装笔记
---

*（该笔记创建于16年4月，后续在本地不断修改完善。仅仅是为了记录安装流程，并非博文形式）*

# JDK安装
### 准备
* 官网下载jdk安装包：jdk-7u79-linux-x64.tar.gz

### 安装步骤
1.	Ubuntu切换到root账号：`su root`
3.	新建jdk安装目录：`mkdir /usr/local/java`
4.	将压缩包复制到该目录下：`cp jdk-7u79-linux-x64.tar.gz /usr/local/java`
5.	解压压缩包：`tar xvf jdk-7u79-linux-x64.tar.gz`
6.	编辑配置文件：`vi /etc/profile`
7.	文件中插入命令：

    ``` 
	export JAVA_HOME=/usr/local/java/jdk1.7.0_79
	export JRE_HOME=/usr/local/java/jdk1.7.0_79/jre
	export PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
	export CLASSPATH=$CLASSPATH:$JAVA_HOME/lib:$JRE_HOME/lib
	```

8.	输入`:wq`保存
9.	使环境变量生效：`source /etc/profile`

### 检验
输入`java –version`，可以看到java版本信息
### 注意事项
* 执行`source /etc/profile`后只是临时让环境变量生效，一旦退出终端变会失效。想让环境变量永久生效，需要重启系统

# hadoop安装
### 准备
* 官网下载安装包：hadoop-2.7.2.tar.gz
* 创建hadoop用户`useradd hadoop`,并添加到管理员用户组`adduser hadoop sudo`
* 安装ssh`sudo apt-get install openssh-server`，并登陆到本机`ssh localhost`。
可以将ssh设置为无密码登陆：

	```
	exit
	cd ~/.ssh/
	ssh-keygen –t rsa
	cat ./id_rsa.pub >> ./authorized_keys
	```

### 安装hadoop
1. 用hadoop用户登陆：`su hadoop`
2. ssh登陆到本机：`ssh localhost`
10.	将hadoop压缩包解压到/usr/local/中：`sudo tar –zxf ~/Downloads/hadoop-2.7.2.tar.gz -C /usr/local`
11.	将解压文件夹重命名为hadoop：

	```
	cd /usr/local/
	sudo mv ./hadoop-2.7.2/ ./hadoop
	```
12.	修改文件权限：`sudo chown –R hadoop:hadoop ./hadoop`

### 配置单机模式
13.	修改配置文件：`vi etc/hadoop/hadoop-env.sh`
14.	添加命令：

	```
	export JAVA_HOME=/usr/local/java/jdk1.7.0_79
	export HADOOP_COMMON_HOME=/usr/local/hadoop
	export HADOOP_INSTALL=/usr/local/hadoop
	export PATH=$PATH:/usr/local/hadoop/bin
	```

15.	修改环境变量：`vi /etc/profile`
16.	将命令修改如下：

	```
	export JAVA_HOME=/usr/local/java/jdk1.7.0_79
	export JRE_HOME=/usr/local/java/jdk1.7.0_79/jre
	export HADOOP_HOME=/usr/local/hadoop
	export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
	export PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
	export CLASSPATH=$CLASSPATH:$JAVA_HOME/lib:$JRE_HOME/lib
	```

17.	使环境变量生效：`source /etc/profile`

### 验证单机模式

	mkdir input
	cp etc/hadoop/*.xml input
	bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar grep input output 'dfs[a-z.]+'
	cat output/*

### 配置伪分布模式
19.	手动创建临时目录：`mkdir tmp`
20.	修改文件：`vi etc/hadoop/core-site.xml`
21.	添加内容：

	```
	<configuration>
	  <property>
	    <name>fs.defaultFS</name>
	    <value>hdfs://localhost:9000</value>
	  </property>
	  <property>
	    <name>hadoop.tmp.dir</name>
	    <value>/usr/local/hadoop/tmp</value>
	  </property>
	</configuration>
	```

22.	修改文件：`vi etc/hadoop/hdfs-site.xml`
23.	添加内容：

	```
	<configuration>
	  <property>
	    <name>dfs.replication</name>
	    <value>1</value>
	  </property>
	  <property>
	    <name>dfs.name.dir</name>
	    <value>/home/hadoop/dfs/name1</value>
	  </property>
	  <property>
	    <name>dfs.data.dir</name>
	    <value>/home/hadoop/dfs/data1</value>
	  </property>
	  <property>
	    <name>dfs.namenode.name.dir</name>
	    <value>/home/hadoop/dfs/name1</value>
	  </property>
	  <property>
	    <name>dfs.datanode.data.dir</name>
	    <value>/home/hadoop/dfs/data1</value>
	  </property>
	</configuration>
	```

24.	修改文件：`vi etc/hadoop/mapred-site.xml`
25.	添加内容：

	```
	<configuration>
	  <property>
	    <name>mapreduce.framework.name</name>
	    <value>yarn</value>
	  </property>
	</configuration>
	```

26.	修改文件：`vi etc/hadoop/yarn-site.xml`
27.	添加内容：

	```
	<configuration>
	  <property>
	    <name>yarn.nodemanager.aux-services</name>
	    <value>mapreduce_shuffle</value>
	  </property>
	</configuration>
	```

28.	格式化namenode：`bin/hdfs namenode –format`
29.	启动dfs、yarn：

	```
	sbin/start-dfs.sh
	sbin/start-yarn.sh
	```

### 验证伪分布模式
* 用`jps`检查进程（6个进程）
* web管理界面查看：
  http://localhost:50070/
  http://localhost:8088/

### 故障排查
* 格式化namenode后出现找不到namenode的情况：检查环境变量是否生效
* datanode无法启动，具体日志如下：
> 2014-06-18 20:34:59,622 FATAL org.apache.hadoop.hdfs.server.datanode.DataNode: Initialization failed for block pool Block pool <registering> (Datanode Uuid unassigned) service to localhost/127.0.0.1:9000
java.io.IOException: Incompatible clusterIDs in /usr/local/hadoop/hdfs/data: namenode clusterID = CID-af6f15aa-efdd-479b-bf55-77270058e4f7; datanode clusterID = CID-736d1968-8fd1-4bc4-afef-5c72354c39ce
at org.apache.hadoop.hdfs.server.datanode.DataStorage.doTransition(DataStorage.java:472)
at org.apache.hadoop.hdfs.server.datanode.DataStorage.recoverTransitionRead(DataStorage.java:225)
at org.apache.hadoop.hdfs.server.datanode.DataStorage.recoverTransitionRead(DataStorage.java:249)
at org.apache.hadoop.hdfs.server.datanode.DataNode.initStorage(DataNode.java:929)
at org.apache.hadoop.hdfs.server.datanode.DataNode.initBlockPool(DataNode.java:900)
at org.apache.hadoop.hdfs.server.datanode.BPOfferService.verifyAndSetNamespaceInfo(BPOfferService.java:274)
at org.apache.hadoop.hdfs.server.datanode.BPServiceActor.connectToNNAndHandshake(BPServiceActor.java:220)
at org.apache.hadoop.hdfs.server.datanode.BPServiceActor.run(BPServiceActor.java:815)
at java.lang.Thread.run(Thread.java:744)

	问题原因：在第一次格式化dfs后，启动并使用了hadoop，后来又重新执行了格式化命令（hdfs namenode -format)，这时namenode的clusterID会重新生成，而datanode的clusterID 保持不变。

	解决方法：打开hdfs-site.xml里配置的datanode和namenode对应的目录，分别打开current文件夹里的VERSION，修改datanode里VERSION文件的clusterID 与namenode里的一致，再重新启动dfs。

# 安装eclipse
### 准备
* 下载eclipse安装包：eclipse-jee-luna-SR2-linux-gtk-x86_64.tar.gz
* 下载hadoop对应版本插件：hadoop-eclipse-plugin-2.7.2.jar

### 开始
1. 将eclipse解压到安装目录： `sudo tar –zxf ~/Downloads/eclipse-jee-luna-SR2-linux-gtk-x86_64.tar.gz -C /usr/local`
2. 将hadoop插件放到plugins文件夹中： `sudo cp ~/Downloads/hadoop-eclipse-plugin-2.7.2.jar /usr/local/eclipse/plugins/hadoop-eclipse-plugin-2.7.2.jar`
3. 重启eclipse
4. 打开windows->preferences->hadoop map/reduce，填写hadoop安装地址
5. 点击open perspective->map/reduce，添加map/reduce界面视图
6. 点击map/reduce locations,添加新location，填写相关信息。
   其中map/reduce master对应mapred-site.xml中mapred-job-tracker的相关配置（默认端口50020），DFS Master对应core-site.xml中fs.defaultFS的配置。
7. 查看左边DFS locations能否正常访问hdfs目录。

### 性能改进
*（虚拟机跑eclipse被坑得死死的，先是被迫弃了eclipse改用命令行，受不了后又去费心思做各种配置优化，最后依旧被迫改装双系统...此处纪念我因此逝去的时间T^T）*

1. 修改eclipse.ini:

	```
	-Xms512m 
	-Xmx512m 
	-XX:MaxNewSize=256m 
	-XX:MaxPermSize=256m
	```

2. 打开windows->preferences->validation，build那列除了"classpath dependency validator"其他全部消勾。
2. startup and shutdown：启动项全部取消
4. install/update：取消自动更新
5. editor->spelling：ignore全勾

# HBASE安装

### 安装准备
* 版本对应的hadoop
* 官网下载：hbase-1.1.3-bin.tar（注意它所支持的hadoop版本）

### 开始
1. 解压到适当位置：`tar -zxvf hbase-1.1.3-bin.tar -C /usr/local`
1. 改名：`sudo mv hbase-1.1.3-bin hbase`
1. 修改属主和属组：`sudo chown -R hadoop:hadoop ./hbase`
2. 设置环境变量：`sudo vim /etc/profile`

	```
	export HBASE_HOME=/usr/local/hbase
	...
	export PATH=$HBASE_HOME/bin:$PATH...
	```

3. 修改配置：`vim $HBASE_HOME/conf/hbase-env.sh`

	```
	export JAVA_HOME=/usr/local/java/jdk1.7.0_79
	expott HBASE_MANAGES_ZK=true
	```

4. 修改文件：`vim $HBASE_HOME/conf/hbase-site.xml`

	```
	<configuration>
		<property>
			<name>hbase.rootdir</name>
			<value>hdfs://localhost:9000/hbase</value>
		</property>
		<property>
			<name>hbase.cluster.distributed</name>
			<value>true</value>
		</property>
		<property>
			<name>hbase.zookeeper.quorum</name>
			<value>localhost</value>
		</property>
	</configuration>
	```

### 启动
1. 先启动hadoop
2. 再启动hbase：`$HBASE_HOME/bin/start-hbase.sh`
3. 运行jps，应该能看到多出HMaster、HRegionServer、HQuorumPeer三个进程。

# HIVE安装

### 准备
* 版本对应的hadoop
* hive安装包：apache-hive-2.0.0-bin.tar.gz
* mysql驱动包：mysql-connector-java-5.1.38-bin.jar

### 安装mysql
1. 采用apt-get安装：`
sudo apt-get install mysql-server` `sudo apt-get install mysql-client`
2. 登陆管理员用户：`mysql -u root -p`
3. 建立hive帐号并授予权限：

	```
	create user 'hive' identified by 'hive';
	grant all privileges on *.* to 'hive'@'%' with grant option;
	```

4. 登陆hive帐号：`mysql -u hive -p`
5. 建立hive元数据库：`create database hive`

### 安装hive
1. 解压到适当位置：`tar -zxvf apache-hive-2.0.0-bin.tar -C /usr/local`
1. 修改属主和属组：`sudo chown -R hadoop:hadoop ./hive`
2. 添加环境变量：

	```
	export HIVE_HOME=/usr/local/hive
	export PATH=...:$HIVE_HOME/bin:$PATH
	```

3. 进入hive/conf目录，复制hive-env.sh:`cp hive-env.sh.template hive-env.sh`
4. 修改hive-env.sh：

	```
	HADOOP_HOME=/usr/local/hadoop
	export HIVE_CONF_DIR=/usr/local/hive/conf
	export HIVE_AUX_JARS_PATH=/usr/local/hive/lib
	```

5. 编辑hive-site.xml（复制自hive-default.xml):

	```
	<property>
	 	<name>hive.metastore.local</name>
	   	<value>false</value> 
	</property>
	<property>   
	   <name>javax.jdo.option.ConnectionURL </name>   
	   <value>jdbc:mysql://localhost:3306/hive?createDatabaseIfNotExist=true</value>   
	</property>   
	<property>   
	   <name>javax.jdo.option.ConnectionDriverName </name>   
	   <value>com.mysql.jdbc.Driver </value>   
	</property>  
	<property>
		<name>javax.jdo.option.ConnectionUserName</name>
		<value>hive</value>
	</property>
	<property>   
	   <name>javax.jdo.option.ConnectionPassword </name>   
	   <value>hive</value>   
	</property>
	<property>
		<name>hive.metastore.warehouse.dir</name>
		<value>/hive/warehouse</value>
	</property>
	<property>
	    <name>system:java.io.tmpdir</name>
	    <value>/usr/local/hive/tmpdir</value>
	    <description>java.net.URISyntaxException: Relative path in absolute URI</description>
	</property>
	<property>
		<name>system:user.name</name>
	    <value>hadoop</value>
	    <description>java.net.URISyntaxException: Relative path in absolute URI</description>
	</property>
	```

6. 创建hive仓库目录：`hadoop fs -mkdir -p /hive/warehouse`
7. 将mysql-connector-java-5.1.38-bin.jar复制到目录$HIVE_HOME/lib中
8. 初始化：`schematool -dbType mysql -initSchema`

### 检验
1. 检查mysql的socket监听状态：`sudo netstat -tap | grep mysql`
1. 检查初始化情况：`schematool -dbType mysql -info`
2. 登陆mysql后检查表hive：`use hive` `show tables`
3. 启动hive（先启动hadoop）：`hive`

### 常见错误：
* > Exception in thread "main" java.lang.RuntimeException: java.lang.RuntimeException: The root scratch dir: /tmp/hive on HDFS should be writable. Current permissions are: rwx--x--x

	解决方法：`hadoop fs -chmod -R 777 /tmp/hive`
* > Exception in thread "main" java.lang.RuntimeException: java.lang.IllegalArgumentException: java.net.URISyntaxException: Relative path in absolute URI: ${system:java.io.tmpdir%7D/$%7Bsystem:user.name%7D

	解决办法：如安装步骤中设置system:java.io.tmpdir项，注意路径需要有读写权限

* 使用java api访问hiveserver2时编译报错：
> org.apache.hadoop.ipc.RemoteException: User: hadoop is not allowed to impersonate hive

	解决办法：修改hadoop配置文件core-site.xml，加入如下配置：
	```
	<property>
    	<name>hadoop.proxyuser.hadoop.hosts</name>
    	<value>*</value>
	</property>
	<property>
   		<name>hadoop.proxyuser.hadoop.groups</name>
   		<value>*</value>
	</property>
	```
	配置项中的“hadoop”是你的{username}

# SPARK安装 #
### 准备 ###
+ 官网下载**对应hadoop版本**的spark安装包如：spark-2.1.0-bin-hadoop2.7.tgz
+ 按SPARK官网要求在SCALA官网上下载符合版本的安装包：scala-2.11.8.tgz

### 安装SCALA ###
1. 新建scala文件夹：`sudo mkdir /usr/local/scala`
2. 将安装包解压到文件夹：`sudo tar -zxf scala-2.11.8.tgz -C /usr/local/scala`
3. 添加环境变量：`sudo vi /etc/profile`
    
        export SCALA_HOME=/usr/local/scala/scala-2.11.8
        export PATH=$PATH:$SCALA_HOME/bin

4. 使环境变量临时生效：`source /etc/profile`

### 检验SCALA ###
1. 输入：`scala -version`，出现版本信息则说明安装成功

### 安装SPARK ###
1. 解压安装包：`sudo tar -zxf spark-2.1.0-bin-hadoop2.7.tgz -C /usr/local/`
2. 切换到安装目录：`cd /usr/local`
3. 重命名：`sudo mv spark-2.1.0-bin-hadoop2.7 spark`
4. 设置访问权限：`sudo chown -R hadoop:hadoop ./spark`
5. 配置环境变量：`sudo vi /etc/profile`
        
        export SPARK_HOME=/usr/local/spark
        export PATH=$PATH:$SPARK_HOME/bin

6. 使环境变量临时生效：`source /etc/profile`
7. 进入spark目录：`cd /usr/local/spark`
8. 备份slaves文件：`cp conf/slaves.template slaves`
9. slaves文件末尾添加worker结点列表（一行一个结点名）：`vi conf/slaves`

        ubuntu  # 此处是你的主机名

8. 在spark-env.sh文件末尾添加配置：`vi conf/spark-env.sh`

        export JAVA_HOME=/usr/local/java/jdk1.8.0_111
        export SCALA_HOME=/usr/local/scala/scala-2.11.8
        export HADOOP_HOME=/usr/local/hadoop
        export HADOOP_CONF_DIR=/usr/local/hadoop/etc/hadoop
        export SPARK_MASTER_IP=ubuntu   # 可填IP可填主机名
        export SPARK_WORKER_MEMORY=1G

### 检验SPARK ###
1. 先确保伪分布式hadoop集群已启动（jps命令查看）
2. 启动SPARK：`sbin/start-all.sh`
3. 启动后查看jps可看到有新进程“master”和“worker”
4. 访问spark控制台页面：[http://localhost:8080/](http://localhost:8080) 页面如下<br>
  ![](http://i.imgur.com/2LTUAKz.png)
5. 启动spark-shell：`bin/spark-shell`
6. 成功后可进入scala控制台，也可访问控制台页面：[http://localhost:4040/](http://localhost:4040) 出现界面信息：
  ![](http://i.imgur.com/yvCPyP1.png)

### 常见错误 ###
+ 启动spark-shell时警告：spark unable to load native-hadoop library
<br>解决：
    1. 进入HADOOP_HOME下的文件夹：`cd /usr/local/hadoop/lib/native`
    2. 将libhadoop.so复制到JAVA_HOME下：`sudo cp libhadoop.so $JAVA_HOME/jre/lib/amd64/libhadoop.so`

+ 将任务部署到spark集群运行时出错：Initial job has not accepted any resources
  + spark-submit提交任务时默认使用1024M的内存空间，可能是由于内存不足引起。解决方式有两种：
      1. spark-env.sh配置中将SPARK_WORKER_MEMORY改为1G以上，即`export SPARK_WORKER_MEMORY=1G`
      2. 在执行spark-submit时设置--executor-memory参数，即`$ spark-submit --executor-memory 512m app_jar`
      3. 在conf/spark-defaults.conf中设置spark-submit启动时的默认参数：
            
                spark.executor.memory 512M

# IDEA开发环境配置技巧 #
### 关于SBT与maven ###
两个我都试过了，理论上SBT和maven在管理包上应该是很方便的，然而由于它要把包管理在自己的仓库里，而包又要从指定的远程公开库下载，所以会很慢，即便是换成了国内的阿里云公开库依旧需要近半个钟。而且更重要的。。。我发现它的spark包加载不全= =||，所以只好暂时放弃用他们管理包的想法。改成自行导入

### 新建工程 ###
1. 新建工程需要自己在file->project structure->modules->sources下添加/src/main/scala目录，并将scala目录设为Sources目录（本来如果是用SBT或maven的话这步直接就建好了）
2. 添加库：在file->project structure->modules->dependencies，要有jdk的library、scala-sdk的library，还要添加$SPARK_HOME/jars下的所有包（直接导入jars文件夹就可以，新版spark已经没有spark-assembly的包）

### idea内调试运行 ###
1. 设置运行选项：点击run->edit configurations，填写main class。
2. vm options填入`-Dspark.master=local`表示在本地运行，也可填入spark集群地址`spark://hostname:7077`提交到集群运行，作用相当于SparkConf.setMaster的缺省值。
3. program arguments是运行时主函数的传入参数（这个好像不用说了吧= =）
4. 如果vm options设置成在集群上运行，则在idea中使用run或debug时需要先生成项目的jar包，并在代码中为sparkconf添加jar包地址，即`sc.addJar("/workspace/projectname/out/artifacts/.../jarname.jar")`。否则会报错“Lost task 1.0 in stage 0.0”的WARN
5. idea内提供了terminal工具，可以在idea的终端中直接将生成的jar包用spark-submit提交到spark集群上运行。在终端使用spark-submit提交到spark运行时不需要添加上述sc.addJar的代码
6. 断点调试工具可以研究下

### 生成jar包 ###
1. 配置打包选项：点击file->project structure->artifacts->add->jar->from modules...，填写主函数所在类，点击ok
2. output layout下会列出打包进的所有依赖包，这些在spark集群已经存在了，因此都可以删除减小jar包大小，但要保留最后一项compile output。点击ok。<br>
  ![](http://i.imgur.com/35EFKXu.png)
3. 项目打包：点击build->build artifacts->build，生成的jar包默认在项目的/out/artifacts下

### spark-submit提交运行 ###
1. 常用的选项指令：

        --master MASTER_URL                   选择运行模式，spark://host:port, mesos://host:port, yarn, or local.(default:local)
        --deploy-mode DEPLOY_MODE    将driver运行在本地(client)或其他worker节点上(cluster) (Default: client).
        --class CLASS_NAME                     程序主类名
        --name NAME                                    应用名
        --jars JARS                                         driver和executor都需要的包，多个包之间用逗号(,)分割
        --properties-file FILE                         读取的环境变量文件位置，默认读取的位置为conf/spark-defaults.conf
        --driver-memory MEM                      driver使用的内存(e.g. 1000M, 2G) (Default: 512M).
        --driver-class-path                             driver所依赖的包，多个包之间用冒号(:)分割
        --executor-memory MEM                 每个executor使用的内存 (e.g. 1000M, 2G) (Default: 1G).

2. 可在$SPARK_HOME/conf/spark-defaults.conf中定义这些指令的默认值（有template模板）。如：`spark.master spark://hostname:7077`，这样就不用每次用spark-submit提交时都定义这些参数了。
3. 可以在localhost:8080查看spark集群的任务状态