
# hadoop wordcount，第一个hadoop程序

完成hadoop 基础配置之后写入第一个Wordcount程序，
关于hadoop基础配置，参考
-[hadoop配置](https://github.com/chpLion/hadoopeb/blob/master/README.md)

## 命令行运行Wordcount
Wordcount程序是hadoop的应官方实例，在hadoop安装环境下的/share/hadoop/mapreduce中的hadoop-mapreduce-examples-2.7.1.jar。

## 运行方式
1. 配置hdfs路径
  - 在伪分布式hadoop中，Wordcount的执行，需要一个input文件夹来存放words文件，也就是统计的文件，所以我们需要制定HDFS的input路径
  - 创建目录
     在hadoop安装目录下执行创建路径操作
     
     ```
     root@lionschen:/opt/hadoop-2.7.1# ./bin/hdfs dfs -mkdir -p /user/hadoop
     root@lionschen:/opt/hadoop-2.7.1# ./bin/hdfs dfs -mkdir -p input
     ```
     这两句的命令是使用dfs创建一个新的路径/user/hadoop,并在这个文件夹下，再创建一个input文件夹作为输入的文件路径
     
     然后使用put命令将需要输入的文件，上传到这个input路径中
     
     ```
     root@lionschen:/opt/hadoop-2.7.1# ./bin/hdfs dfs -put input/word input

     ```
2. 执行jar程序
    在hadoop安装目录下执行命令
    
    ```
    root@lionschen:/opt/hadoop-2.7.1# ./bin/hadoop jar ./share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.1.jar wordcount input output
    ```
    之后就可以看到程序运行成功。
3. 查看程序运行结果
  我们要查看程序的运行结果，需要执行命令 
  首先看output文件夹的详细信息
  ```
  root@lionschen:/# hadoop fs -ls output
  
  Found 2 items
-rw-r--r--   1 root supergroup          0 2016-08-08 14:32 output/_SUCCESS
-rw-r--r--   1 root supergroup       1436 2016-08-08 14:32 output/part-r-00000

  ```
     我们可以看到两条，其中计算结果存放在part-r-00000
     查看这个文件
     
  ```
  root@lionschen:/# hadoop fs -cat output/part-r-00000
  
  ```
  这样就可以看到了Wordcount的执行结果了。
  
## 使用eclipse远程调试
我的hadoop是安装在阿里云服务器上的，Ubuntu的系统，如果要使用eclipse调试的话是比较麻烦的，所以可以在本地使用eclipse编写代码，然后提交到服务器的hadoop集群上执行。

首先需要安装对应的hadoop版本的eclipse的pluging，放在/Applications/eclipse/Eclipse.app/Contents/Eclipse/dropins目录下，然后重启之后就可以看到插件。之后导入相应的jar包，具体配置可以在代码中看到。之后需要在mapreduce配置中制定下载下来的hadoop路径。一般选择bin版本就可以了。
  
有几个需要注意的问题，首先是hadoop的core.site.xml配置文件中，制定的fs.defaultFS的value应该制定为公网ip，这样才能让别的主机访问到，端口号就是eclipse新建项目时需要的端口号。

之后可以run on reduce，就可以看到结果了。
## 一些错误
远程调试的时候会遇到permission denied的问题,那么这样的话，可以将hadoop的hdfs.site.xml中新增一条属性，将dfs.permissions 设置为false
```
  <property>
                <name>dfs.permissions</name>
                <value>False</value>
  </property>
```
或者将hadoop的权限修改
```
hadoop fs -chmod 777 /user/root
```
然后在eclipse中重新连接即可
