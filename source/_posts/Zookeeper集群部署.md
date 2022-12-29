---
title: Zookeeper集群部署
date: 2022-12-29 10:17:48
tags: Zookeeper
---

# Zookeeper集群部署

## 1 云硬盘挂载

<!--more-->

```
# 查看挂盘名称
fdisk -l

# 创建物理卷
pvcreate /dev/vdb

# 创建卷组
vgcreate data_vg /dev/vdb

# 创建逻辑卷
lvcreate -l +100%free -n data data_vg

# 格式化磁盘
mkfs.xfs /dev/data_vg/data

# 创建挂载目录
mkdir -p /data1

# 挂载
mount /dev/data_vg/data /data1

# 添加挂载信息
echo "/dev/mapper/data_vg-data /data1 xfs     defaults        0 0" >> /etc/fstab

```



## 2 安装Java

### 2.1 检查系统是否安装了JAVA

如果有以下输出，则表示已安装

```
rpm -qa | grep java

java-1.8.0-openjdk-headless-1.8.0.272.b10-1.el7_9.x86_64
tzdata-java-2020d-2.el7.noarch
python-javapackages-3.4.1-11.el7.noarch
java-1.8.0-openjdk-1.8.0.272.b10-1.el7_9.x86_64
javapackages-tools-3.4.1-11.el7.noarch
```

需要进行卸载操作

```
rpm -e java-1.8.0-openjdk-1.8.0.272.b10-1.el7_9.x86_64
rpm -e java-1.8.0-openjdk-headless-1.8.0.272.b10-1.el7_9.x86_64
rpm -e javapackages-tools-3.4.1-11.el7.noarch
```

### 2.2 解压JAVA安装包

```
tar -zxf /usr/local/java-1.8.0.tar.gz -C /usr/local
```

### 2.3 配置JAVA环境变量

```
# 在/etc/profile，追加JAVA_HOME和PATH两个变量。

vi /etc/profile
export JAVA_HOME=/usr/local/java-1.8.0
export PATH=$JAVA_HOME/bin:$PATH
```

### 2.4 检查JAVA安装

```
# 使环境变量生效，然后检查Java版本

source /etc/profile
java -version
```

## 3 安装zookeeper

```
# 解压安装好的包
tar -zxf /usr/local/apache-zookeeper-3.5.8-bin.tar.gz -C /usr/local/
mv /usr/local/apache-zookeeper-3.5.8-bin /usr/local/apache-zookeeper-3.5.8

# 新建数据目录
mkdir -p /data1/zookeeper/{data,logs}

# 修改配置文件
# 10.8.132.22配置为1
# 10.8.132.23配置为2
# 10.8.132.24配置为3
echo "1">/data1/zookeeper/data/myid

vi /usr/local/apache-zookeeper-3.5.8/conf/zoo.cfg

clientPort=2181
dataDir=/data1/zookeeper/data
dataLogDir=/data1/zookeeper/logs
tickTime=2000
initLimit=10
syncLimit=5
maxClientCnxns=10000
minSessionTimeout=4000
maxSessionTimeout=40000
autopurge.snapRetainCount=3
autopurge.purgeInterval=12
server.1=10.8.132.22:2888:3888
server.2=10.8.132.23:2888:3888
server.3=10.8.132.24:2888:3888

# 编辑启动脚本
vi /usr/local/apache-zookeeper-3.5.8/monitor.sh

#!/bin/bash

source /etc/profile

cur_path=$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)

function log_info(){
	echo "[INFO] [$(date "+%Y-%m-%d %T")] $1"
}

function log_error(){
	echo "[ERROR] [$(date "+%Y-%m-%d %T")] $1"
	exit 1
}

function check_status(){
	pids=$(ps -ef|grep java|grep apache-zookeeper-3.5.8|awk '{print $2}')

	if [[ -n ${pids} ]];then
		echo "True"
	else 
		echo "False"
	fi
}

function stop(){
	log_info "${FUNCNAME[0]} begin."

	pids=$(ps -ef|grep java|grep apache-zookeeper-3.5.8|awk '{print $2}')
	
	log_info "kill pid ${pids} begin."
	if [[ -n ${pids} ]];then
		log_info "process exist,kill it ${pids}!"
		for i in ${pids}
		do
			log_info "kill process $i!"
			kill -9 $i
		done
	else
		log_info "process not exist!"

	fi
	log_info "kill pid ${pids} end."

	log_info "${FUNCNAME[0]} end."
}

function start(){
	log_info "${FUNCTION[0]} begin."

	if [[ $(check_status $node) == "True" ]];then
		log_info "process is exist, not need to start!"
	else
		log_info "start zookeeper begin."

		/usr/local/apache-zookeeper-3.5.8/bin/zkServer.sh start

		log_info "start zookeeper success."
	fi

	log_info "${FUNCTION[0]} complete."
}

function restart(){
	log_info "${FUNCTION[0]} begin."

	stop

	sleep 5

	start

	log_info "${FUNCTION[0]} end."
}

function main(){
	log_info "${FUNCTION[0]} begin."

	ACTION="$1"

	case "${ACTION}" in
	    start)
            start
	    ;;
            stop)
            stop
            ;;
	    restart)
            restart
            ;;
	    *)
            usage
            ;;
            esac

	log_info "${FUNCTION[0]} complete."
}

main "$@"

# 修改权限
chmod a+x /usr/local/apache-zookeeper-3.5.8/monitor.sh

# 启动
sh /usr/local/apache-zookeeper-3.5.8/monitor.sh start

# 查看进程
jps

# 将健康检查加入crontab
vi /etc/crontab

*/1 * * * * root /usr/local/apache-zookeeper-3.5.8/monitor.sh start >/dev/null 2>&1

# 重启crontab
systemctl restart crond
```

