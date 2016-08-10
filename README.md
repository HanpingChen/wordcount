# hadoop伪分布式环境搭建

##搭建环境前准备

- 系统

  - 本机使用Mac os
  - 搭建Hadoop的机器使用Ubuntu系统，租用阿里云服务器 
- 需要的软件
  - jdk
  - Hadoop2.7
  
## 连接云服务器
Mac终端自带ssh协议，只需要打开终端，在shell中选择连接的主机IP就可以，或者直接在命令行中输入ssh 服务器登录用户名@服务器公网地址
如：

```
chendeMacBook-Air:~ chen$ ssh root\@112.74.78.239 
```

之后会提示输入密码，之后就可以成功连接，并直接使用命令行操作云主机

##在Ubuntu中更新apt
```
root@chp:/usr# apt-get update

```

## 修改网络配置

修改/etc/hosts,将配置修改如下

```
127.0.0.1 localhost
#127.0.1.1      localhost.localdomain   localhost

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
10.163.229.8 lionschen
```
其中第二行注释掉是为了让127.0.0.1能正确映射localhost
最后一行，lionschen是主机名，是为了让主机名能够映射本地ip
修改之后重启系统。
更新成功之后就可以利用apt来下载所需要的软件了。

##下载jdk

在Ubuntu终端中输入
```
root@chp:apt-get install openjdk-1.7-jdk 
```
## 配置环境变量

在使用上面的命令安装的过程中，可以看到有很多语句跳出来，里面会有jdk被安装的目录，一般来说是 /usr/lib/jvm/java-7-openjdk-amd64

找到路径之后，进入/etc/profile 中修改

```
root@chp:/# vi etc/profile
```
将配置文件修改成如下
``` sh
# /etc/profile: system-wide .profile file for the Bourne shell (sh(1))
# and Bourne compatible shells (bash(1), ksh(1), ash(1), ...).

export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64
export JRE_HOME=$JAVA_HOME/jre
export CLASSPATH=$JAVA_HOME/lib:$JRE_HOME/lib:$CLASSPATH
export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH
if [ "$PS1" ]; then
  if [ "$BASH" ] && [ "$BASH" != "/bin/sh" ]; then
    # The file bash.bashrc already sets the default PS1.
    # PS1='\h:\w\$ '
    if [ -f /etc/bash.bashrc ]; then
      . /etc/bash.bashrc
    fi
  else
    if [ "`id -u`" -eq 0 ]; then
      PS1='# '
    else
      PS1='$ '
    fi
  fi
fi
```
修改之后:wq 保存退出
之后让修改的配置文件生效
```
root@chp:/# source /etc/profile
```
之后再输入javac就可以看到出现的javac的用法的提示了。

## 下载Hadoop 
在终端输入
```
root@chp:wget http://mirror.bit.edu.cn/apache/hadoop/common/hadoop-2.7.1/hadoop-2.7.1.tar.gz
```
由于Hadoop不在apt的源中 ，所以使用wget的方式，下载Hadoop的压缩包

下载之后ls，就可以查看到下载下来的压缩包，将压缩包移动到/opt目录下，或者别的你喜欢的目录,之后解压并查看
```
root@chp:mv hadoop-2.7.1.tar.gz /opt
root@chp:tar -zxvf hadoop-2.7.1.tar.gz 
root@chp:ls 
```
就可以看到如下信息
```
hadoop-2.7.1  hadoop-2.7.1.tar.gz
```
这样Hadoop就解压好了，之后我们需要修改配置文件让Hadoop正常运行

## Hadoop配置
1. Hadoop2.7的配置文件都放在 hadoop-2.7./etc/hadoop 中，下面需要修改几个配置文件 
  - hadoop-env.sh  
    进入修改配置文件
    将其中的export JAVA_HOME的注释取消，并修改成java安装路径
    
    ``` sh
    export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64
    
    ```
    存盘退出
    
  - hdfs-site.xml
    需要新建目录，作为namenode和datanode的路径
    
    ```
    mkdir dfs
    mkdir dfs/data
    mkdir dfs/name
    ```
   
    将文件修改如下
    
    ``` sh
    <configuration>
          <property>
               <name>dfs.replication</name>
               <value>1</value>
          </property>
          <property>
               <name>dfs.namenode.name.dir</name>
               <value>file:/opt/hadoop-2.7.1/dfs/name</value>
          </property>
          <property>
               <name>dfs.datanode.data.dir</name>
               <value>file:/opt/hadoop-2.7.1/dfs/data</value>
          </property>
  </configuration>
    ```
  - core.site.xml 

    ```
    <configuration>
          <property>
               <name>hadoop.tmp.dir</name>
               <value>file:/usr/local/hadoop/tmp</value>
               <description>Abase for other temporary directories.</description>
          </property>
          <property>
               <name>fs.defaultFS</name>
               <value>hdfs://localhost:9000</value>
          </property>
  </configuration>
    ```
    这里是设置namenode和datanode两个节点的路径，来模拟分布式
  为了能让终端更好的使用hadoop的命令，我们需要设置一下hadoop的环境变量
  同样的 进入/etc/profile
  修改如下
  
  ``` sh
  export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64
  export HADOOP_HOME=/opt/hadoop-2.7.1
  export JRE_HOME=$JAVA_HOME/jre
  export CLASSPATH=$JAVA_HOME/lib:$JRE_HOME/lib:$CLASSPATH
  export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$HADOOP_HOME/bin:$PATH
  ```
  
  让配置生效
  
  ```
  root@chp:source /etc/profile
  ```
2. 启动hadoop 
  - 格式化namenode
  在hadoop安装目录下执行
  
  ```
  root@chp:/opt/hadoop-2.7.1# ./bin/hdfs namenode -format
  ```
  - 完成之后，可以看到exit 0 就表示成功格式化，如果格式化不成功，可以先尝试将core.site.xml文件中的localhost改成你的主机的名字
  ，比如我的是chp。
  - 启动
  输入命令
  
  ```
  root@chp:/opt/hadoop-2.7.1:cd sbin
  root@chp:/opt/hadoop-2.7.1/sbin#:./start-all.sh
  ```
  之后会多次让你输入root密码
  
  最后完成启动之后输入jps查看是否启动成功
  
  ```
  root@chp:/opt/hadoop-2.7.1/sbin# jps
  7791 Jps
  1328 NameNode
  1465 DataNode
  2089 NodeManager
  1790 ResourceManager
  1639 SecondaryNameNode
  ```
  如果看到这些就表示成功的启动了。
