https://blog.csdn.net/hliq5399/article/details/78193113

# 安装Hadoop

https://github.com/apache/hadoop
https://www.jianshu.com/p/0d4a365ef350?utm_campaign=maleskine&utm_content=note&utm_medium=seo_notes&utm_source=recommendation

### 准备

1. 配置ip

2. 配置hostname /etc/hosts

3. selinux

   ```bash
   setenforce 0
   sed -i 's/^SELINUX=enforcing$/SELINUX=disabled/' /etc/selinux/config
   ```

4. 防火墙

5. 安装jdk 7/8

   

### 伪集群模式

> hadoop-env.sh 
>
> core-site.xml
>
> hdfs-site.xml 
>
> yarn-site.xml
>
> mapred-site.xml

#### hdfs

##### 环境

1. 设置环境变量 HADOOP_HOME

2. ${HADOOP_HOME}/etc/hadoop/hadoop-env.sh中配置 JAVA_HOME

   ```bash
   # The java implementation to use.
   if [ x${JAVA_HOME} = x ]; then 
       echo "JAVA_HOME set"
       export JAVA_HOME=/usr/local/java
   else 
       export JAVA_HOME=${JAVA_HOME}
   fi
   ```

##### hdfs-site.xml core-site.xml   

3. 配置 hdfs-site.xml core-site.xml

   hdfs-site.xml 增加

   ```xml
   <configuration>
       <property>
         <name>dfs.replication</name>
         <value>1</value>
       </property>
       <property>
         <name>dfs.webhdfs.enabled</name>
         <value>true</value>
       </property>
       <property>
               <name>dfs.datanode.data.dir</name>
               <value>/usr/local/hadoop-2.7.7/data/data</value>
       </property>
       <property>
               <name>dfs.namenode.name.dir</name>
               <value>/usr/local/hadoop-2.7.7/data/name</value>
       </property>
   </configuration>
   ```

   core-site.xml 增加

   ```xml
   <configuration>
       <property>
           <name>fs.defaultFS</name>
           <value>hdfs://localhost:8020</value>
       </property>
       <property>
           <name>io.file.buffer.size</name>
           <value>131072</value>
       </property>
       <property>
            <name>hadoop.tmp.dir</name>
            <value>/usr/local/hadoop-2.7.7/data</value>
       </property>
   </configuration>
   ```

   > 注意hadoop.tmp.dir目录的权限问题，用户要有读写权限才行

   ```bash
   mkdir -p /usr/local/hadoop-2.7.7/data
   ```
##### 格式化namenode
4. 格式化namenode

   ```bash
   ${HADOOP_HOME}/bin/hdfs namenode –format
   ```

   > 在 /usr/local/hadoop-2.7.7/data/下生成name目录
#####  启动NameNode，DataNode
5. 启动NameNode

   ```bash
   ${HADOOP_HOME}/sbin/hadoop-daemon.sh start namenode
   ```

6. 启动DataNode

   ```bash
   ${HADOOP_HOME}/sbin/hadoop-daemon.sh start datanode
   ```

   >  无法启动 SecondaryNameNode
   >
   > ```java
   > ${HADOOP_HOME}/sbin/hadoop-daemon.sh start sencondarynamenode
   > ```
##### 检查 命令，浏览器
7. jps 检查 2个进程

8. hdfs文件操作 

   ```bash
   ${HADOOP_HOME}/bin/hdfs dfs --help
   ```

9. 检查hadoop启动情况

   浏览器访问

   > localhost:50070
   > localhost:50075

   ```bash
   # 打印
   export HADOOP_ROOT_LOGGER=DEBUG,console
   unset HADOOP_ROOT_LOGGER
   ```

#### yarn    
##### mapred-site.xml yarn-site.xml  
 11. 配置mapred-site.xml

     ```bash
      cp mapred-site.xml.template mapred-site.xml
     ```

      mapred-site.xml 
     
     ```bash
                 <!-- 指定mapreduce运行在yarn框架上 -->
                 <property>
                     <name>mapreduce.framework.name</name>
                     <value>yarn</value>
                 </property>
     ```
     
 12. 配置yarn-site.xml  

     ```xml
         <property>
           <name>yarn.nodemanager.aux-services</name>
           <value>mapreduce_shuffle</value>
         </property>
         <property>
             <name>yarn.resourcemanager.hostname</name>
             <value>master</value>
         </property>
     ```

     > yarn.nodemanager.aux-services配置了yarn的默认混洗方式，选择为mapreduce的默认混洗算法。
     > yarn.resourcemanager.hostname指定了Resourcemanager运行在哪个节点上。

##### 启动Resourcemanager，nodemanager
13.  启动Resourcemanager
     
     ```bash
     ${HADOOP_HOME}/sbin/yarn-daemon.sh start resourcemanager
     ```
     
14. 启动nodemanager

    ```bash
    ${HADOOP_HOME}/sbin/yarn-daemon.sh start nodemanager
    ```
##### 检查 命令，浏览器
15. jps 检查 2个进程 新增 resourcemanager nodemanager

16. YARN的Web页面 YARN的Web客户端端口号是8088，

    浏览器访问

    > localhost:8088

##### 运行示例
17. 运行MapReduce Job
    数据准备
    
    ```bash
    bin/hdfs dfs -mkdir /wordcount/
    bin/hdfs dfs -mkdir /wordcount/input
    bin/hdfs dfs -put etc/hadoop/*.xml /wordcount/input/
    ```
    
    mapreduce
    
    ```bash
    bin/yarn jar share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar    wordcount \
        /wordcount/input /wordcount/output
    bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar wordcount \
        /wordcountdemo/input /wordcountdemo/output
    ```
    
    查看结果和导出
    
    ```bash
    bin/hdfs dfs -cat /wordcount/output/part-r-00000
    bin/hdfs dfs -get /wordcount/output output
    ```
##### 开启关闭
18. 关闭对应启动操作

    ```bash
    # sbin/yarn-daemon.sh stop nodemanager
    # sbin/yarn-daemon.sh stop resourcemanager
    # sbin/hadoop-daemon.sh stop sencondarynamenode
    # sbin/hadoop-daemon.sh stop datanode
    # sbin/hadoop-daemon.sh stop namenode
    ```

19.  开启历史服务

     ```bash
     sbin/mr-jobhistory-daemon.sh start historyserver
     ```

20.  检查进程

     ```bash
     jps 
     ```

     开启网页

     > hdfs: localhost:50070/50075
     >
     > yarn: localhost:8088
     >
     > 历史服务 localhost:19888

#### 启动关闭方法

```bash
sbin/start-all.sh
sbin/stop-all.sh
sbin/start-hdfs.sh  && sbin/start-yarn.sh
sbin/stop-yarn.sh  && sbin/stop-hdfs.sh
```

### 集群安装

> 根伪集群分布一样
>
> hadoop-env.sh 
>
> core-site.xml
>
> hdfs-site.xml 
>
> yarn-site.xml
>
> mapred-site.xml
>
> 增加slaves

#### 规划

| hostname/ip    |                 |      |
| -------------- | --------------- | ---- |
| 192.168.122.30 | namenode        |      |
| 192.168.122.31 | datanode        |      |
| 192.168.122.32 | datanode        |      |
| 192.168.122.33 | datanode        |      |
| 192.168.122.34 | sencondNamenode |      |

#### 准备

1. 免密

   主机到其他机器的免密

   ```bash
   ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
   ssh-copy-idh
   ```

2. /etc/hosts

   ```bash
   192.168.122.30 master node0
   192.168.122.31 node1
   192.168.122.32 node2
   192.168.122.33 node3
   192.168.122.34 node4
   ```

3. selinux firewalld

   ```bash
   setenforce 0
   sed -i 's/^SELINUX=enforcing$/SELINUX=disabled/' /etc/selinux/config
   systemctl stop firewalld
   systemctl disable firewalld
   ```

#### 配置

> hadoop-env.sh 
>
> core-site.xml
>
> hdfs-site.xml 
>
> yarn-site.xml
>
> mapred-site.xml
>
> 增加slaves

1. hadoop-env.sh

   ```bash
   if [ x${JAVA_HOME} = x ]; then 
       export JAVA_HOME=/usr/local/java
   else 
       export JAVA_HOME=${JAVA_HOME}
   fi
   ```

2.  slaves

    ```
    node1
    node2
    node3
    node4
    ```

3.  hdfs-site.xml

    ```bash
    mkdir -p /usr/local/hadoop-2.7.7/data/
    ```

    ```xml
    <configuration>
        <property>
            <name>dfs.namenode.name.dir</name>
            <value>file:/usr/local/hadoop-2.7.7/data/dfs/name</value>
        </property>
        <property>
            <name>dfs.datanode.data.dir</name>
            <value>file:/usr/local/hadoop-2.7.7/data/dfs/data</value>
        </property>
        <property>
            <name>dfs.replication</name>
            <value>2</value>
        </property>
        <property>
            <name>dfs.webhdfs.enabled</name>
            <value>true</value>
        </property>
    </configuration>
    ```

4.  core-site.xml

    ```xml
    <configuration>
        <property>
            <name>fs.defaultFS</name>
            <value>hdfs://master:8020</value>
        </property>
        <property>
            <name>io.file.buffer.size</name>
            <value>131072</value>
        </property>
        <property>
            <name>hadoop.tmp.dir</name>
            <value>file:/usr/local/hadoop-2.7.7/data</value>
        </property>
    </configuration>
    ```

5.  yarn-site.xml

    ```xml
    <configuration>
        <property>
            <name>yarn.resourcemanager.hostname</name>
            <value>master</value>
        </property>
        <property>
            <name>yarn.nodemanager.aux-services</name>
            <value>mapreduce_shuffle</value>
        </property>
    </configuration>
    ```

6.  mapred-site.xml

    ```xml
    <configuration>
        <property>
            <name>mapreduce.framework.name</name>
            <value>yarn</value>
        </property>
        <property>
            <name>mapreduce.jobhistory.address</name>
            <value>master:10020</value>
        </property>
        <property>
            <name>mapreduce.jobhistory.webapp.address</name>
            <value>master:19888</value>
        </property>
    </configuration>
    ```

7.  同步目录到所有集群机器上

#### 启动检查

1.  主机上执行

    ```bash
    hdfs namenode  -format
    ```

2.  hdfs

    ```bash
    sbin/start-dfs.sh
    # 检查
    bin/hdfs dfs -ls /
    ```

3.  yarn

    ```bash
    sbin/start-yarn.sh
    ```

    数据准备

    ```bash
    bin/hdfs dfs -mkdir -p /wordcount/input
    bin/hdfs dfs -put LICENSE.txt /wordcount/input/
    ```

    mapreduce

    ```bash
    bin/yarn jar share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar    wordcount \
        /wordcount/input /wordcount/output
    bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar wordcount \
        /wordcountdemo/input /wordcountdemo/output
    ```

    查看结果和导出

    ```bash
    bin/hdfs dfs -cat /wordcount/output/part-r-00000
    bin/hdfs dfs -get /wordcount/output output
    ```

    

    >hadoop在启动时候，报
    > WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
    >增加调试信息，执行命令
    >export HADOOP_ROOT_LOGGER=DEBUG,console
    >
    >参考了
    >http://stackoverflow.com/questions/19943766/hadoop-unable-to-load-native-hadoop-library-for-your-platform-warning
    >http://www.xuebuyuan.com/2178383.html 的解决方案，在/HADOOP_HOME/etc/hadoop/中的hadoop_env.sh头部添加了如下信息
    >export HADOOP_COMMON_LIB_NATIVE_DIR="/usr/local/hadoop/lib/native/"
    >export HADOOP_OPTS="$HADOOP_OPTS -Djava.library.path=/usr/local/hadoop/lib/"

### HDFS Federation 

> + 命名空间的限制。
>
> ​		NameNode上存储着整个HDFS上的文件的元数据，NameNode是部署在一台机器上的，因为单个机器硬件的限制，必然会限制NameNode所能管理的文件个数，制约了数据量的增长。
>
> + 数据隔离问题。
>   		整个HDFS上的文件都由一个NameNode管理，所以一个程序很有可能会影响到整个HDFS上的程序，并且权限控制比较复杂。
>
> + 性能瓶颈。
>
>   ​		单个NameNode时HDFS文件系统的吞吐量受限于单个NameNode的吞吐量。因为NameNode是个JVM进程，JVM进程所占用的内存很大时，性能会下降很多。



namenode namenode namenode

datanode datanode datanode datanode datanode datanode



### hadoop-ha部署

```
                      zookeeper

ZKFC                  JournalNode(3台)              ZKFC

        namenode(active)               namenode(standby)
datanode    datanode    datanode    datanode    datanode    datanode
```

#### namenode

HDFS HA架构中有两台NameNode节点，一台是处于活动状态（Active）为客户端提供服务，另外一台处于热备份状态（Standby）。
元数据文件有两个文件：fsimage和edits，备份元数据就是备份这两个文件。JournalNode用来实时从 Active NameNode上拷贝edits文件，JournalNode有三台也是为了实现高可用。
Standby NameNode不对外提供元数据的访问，它从Active NameNode上拷贝fsimage文件，从JournalNode上拷贝edits文件，然后负责合并fsimage和edits文件，相当于SecondaryNameNode的作用。
最终目的是保证Standby NameNode上的元数据信息和Active NameNode上的元数据信息一致，以实现热备份。

#### Zookeeper

  Zookeeper来保证在Active NameNode失效时及时将Standby NameNode修改为Active状态。
  ZKFC（失效检测控制）是Hadoop里的一个Zookeeper客户端，在每一个NameNode节点上都启动一个ZKFC进程，来监控NameNode的状态，
  	并把NameNode的状态信息汇报给Zookeeper集群，其实就是在Zookeeper上创建了一个Znode节点，节点里保存了NameNode状态信息。
  	当NameNode失效后，ZKFC检测到报告给Zookeeper，Zookeeper把对应的Znode删除掉，Standby ZKFC发现没有Active状态的NameNode时，
  	就会用shell命令将自己监控的NameNode改为Active状态，并修改Znode上的数据。 
  	Znode是个临时的节点，临时节点特征是客户端的连接断了后就会把znode删除，所以当ZKFC失效时，也会导致切换NameNode。

#### Datanode

  DataNode会将心跳信息和Block汇报信息同时发给两台NameNode，DataNode只接受Active NameNode发来的文件读写操作指令。

#### zookeeper 部署 

(一般部署奇数个，用于支持 JournalNode ZKFC)

#### hdfs

```bash
配置和分发
	参看现在的配置文件 
        将配置文件分发到规划的机器上
    2.1 规划 journalnode 的各台机器上启动
        > hadoop/sbin/hadoop-daemon.sh start journalnode
        > zookeeper/bin/zkServer.sh start
    2.2 检查 jps zookeeper/bin/zkServer.sh status
    2.3 在规划的2台namenode机器上执行
	#机器1
        > hadoop/bin/hdfs namenode -format   #只有首次需要执行，注意清空配置中的存储文件
        > hadoop/sbin/hadoop-deamon.sh start namenode
	#机器2 必须机器1 启动namenode
        > hadoop/bin/hdfs namenode -bootstrapStandby   #只有首次需要执行，注意清空配置中的存储文件
        > hadoop/sbin/hadoop-deamon.sh start namenode
    2.4 设置namenode为active和standby 
        > hdfs haadmin -transitionToActive nn1 
        > hdfs haadmin –transitionToActive -forcemanual nn1  # 一般需要用forcemanual 更改为active
        > hdfs haadmin -getServiceState nn1
        > hdfs haadmin -getServiceState nn2
    2.5 dfs 启动
        > hadoop/sbin/start-dfs.sh 
    2.6 检查
        > hdfs dfs -put ~/.bashrc /bashrc
 	# 关闭namenode为active的机器 namenode服务
        > hadoop/sbin/hadoop-deamon.sh stop namenode
        > hdfs dfs -ls /
        > hdfs haadmin -getServiceState nn2
 	# 重新启动namenode，关闭另外一台namenode
        > hadoop/sbin/hadoop-deamon.sh stop namenode
```

Q:

```bash
hadoop在启动时候，报
 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
```

A:

```bash
增加调试信息，执行命令
export HADOOP_ROOT_LOGGER=DEBUG,console

参考了
http://stackoverflow.com/questions/19943766/hadoop-unable-to-load-native-hadoop-library-for-your-platform-warning
http://www.xuebuyuan.com/2178383.html 的解决方案，在/HADOOP_HOME/etc/hadoop/中的hadoop_env.sh头部添加了如下信息
export HADOOP_COMMON_LIB_NATIVE_DIR="/usr/local/hadoop/lib/native/"
export HADOOP_OPTS="$HADOOP_OPTS -Djava.library.path=/usr/local/hadoop/lib/"
再次启动没有问题
```

## 监控

Ganglia

[安装ganglia*监控**hadoop**集群*](https://blog.csdn.net/sang1203/article/details/52329159)

https://blog.csdn.net/dear_csdn/article/details/76528750

https://www.cnblogs.com/caizhenghui/p/9132414.html



prometheus 监控 hadoop + Hbase + zookeeper





指标

https://www.cnblogs.com/xinfang520/p/10136439.html