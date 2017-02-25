本文档为过程中遇到的坑，故无顺序可言，遇到问题记录解决方案

##1.	【集群】运行任务卡在job进度，8088端口的Applications中显示任务FinalStatus为未定义
###解决方案：查看hadoop配置中的yarn-site.xml是否配置了
	<property>
        <name>yarn.resourcemanager.hostname</name>
        <value>master</value>
	</property>
  其中master为namenode节点的地址，可用ip代替，也可在/etc/hosts中配置，ip address +空格+你想给该ip定义的别名，这样该别名在该机器上就相当于该ip地址了

###2.	可以连接到8088端口 ，但是连接不上50070端口，并且jps查看不到namenode进程
###原因分析：
> 1）	未格式化文件系统，使用bin/hdfs namenode –format格式化文件系统

> 2）	未配置临时文件路径导致namenode 默认在/tmp下建立临时文件，但关机后，/tmp下文档自动删除，再次启动Master造成文件不匹配，所以namenode启动失败。解决方案在core-site.xml中配置
> 
	<property>
		<name>Hadoop.tmp.dir</name>
		<value>你想要的路径</value>
	<property>
> 然后，重新格式化文件系统。
###3.	集群数据节点NodeManager和DataNode成功启动，但是web上看不到节点数据，查看节点日志文件显示Datanode denied communication with namenode because hostname cannot be resolved
####原因分析：(待查明)

####解决方案：
> Add datanode ip/hostname into /etc/hosts. Or enable reverse DNS(Make sure you could get hostname via command: host ip). Or add following property in hdfs-site.xml:
> 
        <property>
                  <name>dfs.namenode.datanode.registration.ip-hostname-check</name>                   
                  <value>false</value>
        </property>

###4.	SSH配置，将公钥写入authorized_keys，但是每次打开终端还是需要输入密码，但是一个终端内只需一输入一次密码即可。
####检测错误：
> 1.	检查/etc/ssh/sshd_config文件中的第54、55行的RSAAuthentication、PubkeyAuthentication是否删掉注释，并且设置为yes,59行的AuthorizedKeysFile是否设置为.ssh/authorized_keys，如果不是设置正确（如下图）后，使用systemctl restart sshd.service重启ssh服务。
 
> 2.	检查/etc/selinux/config文件中第7行SELINUX是否等于disable，不修改后保存退出，使用systemctl restart sshd.service重启ssh服务即可！


