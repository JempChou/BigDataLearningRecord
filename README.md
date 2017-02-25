#Hadoop集群搭建步骤
##一、环境搭建
> 1.	配置本机hostname 
> 2.	配置java运行环境jdk
> 3.	配置ssh免密码登录
> 4.	安装、配置hadoop
> 5.	测试hadoop

###以上过程可以分为2个大的步骤

> 1.	配置系统环境以使hadoop能运行
> 2.	配置hadoop配置文件让hadoop能正确运行

##二、为什么需要配置相应的软件
1. 本次集群搭建我们希望使用2台机器作为集群一台机器作为主节点，共运行namenode、SecondaryNameNode、ResourceManager 3个java守护进程另外一台作为slave节点，运行DataNode、NodeManager进程其中，master是一个资源调度的管理者，假如集群中用于成千上万的机器，master就是这些机器中的管理者，它负责怎个集群的任务调度。Slave作为计算节点，将会执行master指定的相应任务，并通过心跳包报告任务进度及结果等等数据。
2. 在环境搭建的第一个大步骤中，配置jdk是因为hadoop是一个以java 语言实现的项目，它需要运行在java环境下，比如namenode、DataNode等等进程都是一个个java进程。
在系统中如果需要查看可以使用jps命令查看。
配置ssh免密码登录时因为，hadoop的每个进程间会相互调用、并发送指令，那么你就能想到在集群工作方式中有两台独立的计算机需要通信的情况，计算机A直接给计算机B发送指令，那么计算机B需要验证计算机A发送过来的身份信息，如果只是发送一两次命令还好我们可以人为的输入身份信息，但是在hadoop集群调度的时候需要频繁的执行这样的操作.
所以配置ssh免密码登录时为了通过配置让计算机自动完成身份验证这个步骤。

##三、搭建集群

1.	Jdk
首先我们在一台机器（master）上配置jdk然后通过vm的克隆公共克隆机器为slave节点。
 拷贝你下载的jdk，通过tar –zxf 命令解压（可添加 –C 参数指定解压后文件保存路径，如果不指定默认存在压缩包路径），如下图：

然后再通过sudo vim /etc/profil配置环境变量，如下图所示
 
然后通过source /etc/profile将profile写入系统，写入后需要重启计算机，否则关闭计算机后打开其他终端环境变量不会生效，可以通过java –version和javac命令测试是否配置成功。
2.	克隆机器
  由于我们需要两台机器作为集群，在vm中有克隆机器的功能，所以我们克隆机器作为slave节点，克隆完成后需要重新配置机器的ip地址为slave01的地址master同一局域网的ip即可，可以在master机器上通过ifconfig查看master的ip地址。
完成后通过ifconfig查看两台机器的ip。如下图

   

3.	ssh免密码
  centos系统自带了ssh，并且自带了秘钥生成工具ssh-keygen，在终端中配置ssh免密码登录步骤如下
1)	生成秘钥
2)	将秘钥配置在本机，并且将需要将slave01的ssh公钥发送给master节点
Ssh-keygen –t rsa 生成密匙 
ssh-copy-id localhost 将密匙拷贝到authorized_keys
如图，另外一台机器同样。
 

这样就可以通过ssh localhost免密码登录本机了，但是还不能从其他机器登录本机，我们就可以通过scp 命令将公钥发送给master机器将公钥写入authorized_keys再次发送给其他slave机器。如下图。
 
这条命名中hadoop是你们master机器的用户名，@后面是你master机器的ip地址，：后面是你想将该文件发送到的位置，这样就将文件发送到master上面了，然后再master机器上通过cat命令将公钥追加到authorized_keys,然后在将master的authorized_keys重新发回给slave，如下图
 
这样两台机器就可以相互通过ssh+ip地址的方式访问彼此了。
4.	hadoop配置文件
在官网下载hadoop后将其解压，就可以正式配置hadoop了，需要配置解压文件中的etc/hadoop目录下的6个文件分别是：
hadoop-env.sh 这里面修改{JAVA_HOME}为你的jdk安装路径
core-site.xml 
hdfs-site.xml
mapred-site.xml
yarn-site.xml
slaves
附上全部配置：
	<!--core-site.xml-->
	<property>
        <name>fs.defaultFS</name>
        <value>hdfs://master:8020</value>
    </property>
	    <property>
        <name>hadoop.tmp.dir</name>
        <value>你想存放的临时文件路径</value>
    </property>
	
	<!--hdfs-site.xml-->
	<property>
        <name>dfs.replication</name>
        <value>3</value>
    </property>
	
	<!--mapred-site.xml-->
	<property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
	
	<!--yarn-site.xml-->
	<property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
	</property>

最后将slaves中的localhost删除，并写入你的slave节点的ip地址，这样就完成了全部配置工作。然后格式化一下hadf的文件系统（多次格式化会出现集群id不一样，可以通过删除在core-site.xlm中配置的临时文件目录下的文件来解决）

bin/hdfs namenode –format 命令用来格式化hdfs文件系统

完成以上步骤后就可以在master上面通过运行sbin目录下的start-all.sh就可以运行hadoop了，运行成功后可以通过jps命令查看master和slave机器的进程，如图
 
 
