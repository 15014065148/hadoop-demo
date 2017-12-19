#### hadoop mapreduce
####下载hadoop包
[下载页面](http://www.apache.org/dyn/closer.cgi/hadoop/common/)

wget https://mirrors.tuna.tsinghua.edu.cn/apache/hadoop/common/hadoop-3.0.0/

####单机环境搭建
```text
Required Software
Required software for Linux include:
Java™ must be installed. Recommended Java versions are described at HadoopJavaVersions.
ssh must be installed and sshd must be running to use the Hadoop scripts that manage remote Hadoop daemons if the optional start and stop scripts are to be used. Additionally, it is recommmended that pdsh also be installed for better ssh resource management.
```
  $ sudo apt-get install ssh
  
  $ sudo apt-get install pdsh
  
  1. Unpack the downloaded Hadoop distribution. In the distribution, edit the file etc/hadoop/hadoop-env.sh to define some parameters as follows:
```text
  # set to the root of your Java installation
  export JAVA_HOME=/usr/java/latest
```
  
  2. 试着执行下面的命令
```text
 $ bin/hadoop
 $ mkdir input
 $ cp etc/hadoop/*.xml input
 $ bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.0.0.jar grep input output 'dfs[a-z.]+'
 $ cat output/*
```
####伪分布式环境搭建
  1. 编辑 etc/hadoop/core-site.xml:
  ```xml
  <configuration>
      <property>
          <name>fs.defaultFS</name>
          <value>hdfs://localhost:9000</value>
      </property>
  </configuration>

```
  2. 编辑 etc/hadoop/hdfs-site.xml:
  ```xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>
```
  3. 免密登录 
  
      如果不能登录执行下面的命令：
```text
  $ ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
  $ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
  $ chmod 0600 ~/.ssh/authorized_keys
```

  4. 初始化hadoop 环境
   * 格式化文件系统
        ```text
        $ bin/hdfs namenode -format
        ```
   * 启动名字节点和数据节点
        ```text
         $ sbin/start-dfs.sh
       ```
   * 通过web 接口访问名字节点 ``` http://localhost:9870/```
   
   * 创建HDFS文件目录
        
           $ bin/hdfs dfs -mkdir /user
           $ bin/hdfs dfs -mkdir /user/<username>
   
   * 将输入文件复制到HDFS分布式文件系统
      
      ```  
       $ bin/hdfs dfs -mkdir input
       $ bin/hdfs dfs -put etc/hadoop/*.xml input
     ```


   * 运行例子
   
   ```text
      $ bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.0.0.jar grep input output 'dfs[a-z.]+'
   ```
   
   * 查看输出文件: 将输出文件从分布式文件系统复制到本地系统
   
   ```text
      $ bin/hdfs dfs -get output output
      $ cat output/*
   ```
    
  * 如果不用，关闭系统
  
  ```text
     $ sbin/stop-dfs.sh
  ```
    
  * 在YARN 上运行
  
   You can run a MapReduce job on YARN in a pseudo-distributed mode by setting a few parameters and running ResourceManager daemon and NodeManager daemon in addition.
   
   编辑: etc/hadoop/mapred-site.xml:
   ```xml
    <configuration>
        <property>
            <name>mapreduce.framework.name</name>
            <value>yarn</value>
        </property>
    </configuration>
   ```
   
   编辑: etc/hadoop/yarn-site.xml:
  ```xml
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.env-whitelist</name>
        <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
    </property>
</configuration>
  ```
   启动：资源管理和节点管理
   ```text
    $ sbin/start-yarn.sh
   ```
   通过浏览器查看: ResourceManager - http://localhost:8088/
   
   运行一个job
   
   关闭：
   ```text
   $ sbin/stop-yarn.sh
   ```
####下载源码 

1. mvn clean package 打包生成 mapreduce-1.0-SNAPSHOT.jar

2. 上传文件到hadoop目录

3. hadoop 操作

$ vi file01
```text
Hello World, Bye World!
```
$vi file02 
```text
Hello Hadoop, Goodbye to hadoop.
```
$ bin/hadoop fs -mkdir -p /user/root/wordcount/input 

$ bin/hadoop fs -put file01 /user/root/wordcount/input

$ bin/hadoop fs -put file02 /user/root/wordcount/input

$ bin/hadoop fs -ls /user/root/wordcount/input/
```text
/user/root/wordcount/input/file01
/user/root/wordcount/input/file02
```


$ bin/hadoop fs -cat /user/root/wordcount/input/file01
Hello World, Bye World!

$ bin/hadoop fs -cat /user/root/wordcount/input/file02
Hello Hadoop, Goodbye to hadoop.

$ bin/hadoop fs -cat /user/joe/wordcount/patterns.txt
```text
\.
\,
\!
to
```

bin/hadoop jar mapreduce-1.0-SNAPSHOT.jar hadoop.demo.WordCount2 -Dwordcount.case.sensitive=false /user/root/wordcount/input /user/root/wordcount/output3 -skip /user/root/wordcount/input/patterns.txt

[参考官方文档1](http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/SingleCluster.html)

[参考官方文档2](http://hadoop.apache.org/docs/current/hadoop-mapreduce-client/hadoop-mapreduce-client-core/MapReduceTutorial.html)
