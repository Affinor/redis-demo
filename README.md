**redis集群安装**
 - 1、准备一台sentos7虚拟机
192.168.80.129
 - 2、去官网下载最新redis5安装包
redis-5.0.8.tar.gz
 - 3、创建redis文件夹并上传redis安装包
mkdir -p /home/db/redis
 - 4、解压
tar -zxvf redis-5.0.8.tar.gz
 - 5、安装
cd /redis-5.0.8/src
make
 - 6、复制配置文件
cd /home/db/redis
cp /redis-5.0.8/redis.conf redis-7000.conf
 - 7、修改配置文件
vim redis-7000.conf
注释掉bind 127.0.0.1
protected-mode no
port 7000
daemonize yes
pidfile /var/run/redis_7000.pid
cluster-enabled yes
cluster-config-file nodes-7000.conf
 - 8、复制配置文件从redis-7000.conf到redis-7007.conf，总共8个，并修改对应端口
 - 9、创建启动脚本
vim redis-cluster-start.sh
./redis-5.0.8/src/redis-server redis-7000.conf
./redis-5.0.8/src/redis-server redis-7001.conf
./redis-5.0.8/src/redis-server redis-7002.conf
./redis-5.0.8/src/redis-server redis-7003.conf
./redis-5.0.8/src/redis-server redis-7004.conf
./redis-5.0.8/src/redis-server redis-7005.conf
./redis-5.0.8/src/redis-server redis-7006.conf
./redis-5.0.8/src/redis-server redis-7007.conf
 - 10、启动所有节点
sh redis-cluster-start.sh
 - 11、创建集群命（此时只创建7000到7005）
./redis-5.0.8/src/redis-cli --cluster create 192.168.80.129:7000 192.168.80.129:7001 192.168.80.129:7002 192.168.80.129:7003 192.168.80.129:7004 192.168.80.129:7005 --cluster-replicas 1
 - 12、验证集群创建成功
./redis-5.0.8/src/redis-cli -p 7000 -c
cluster nodes
 - 13、添加主节点（扩容）
./redis-5.0.8/src/redis-cli --cluster add-node 192.168.80.129:7006 192.168.80.129:7000
暂不执行删除主节点（缩容）
./redis-5.0.8/src/redis-cli --cluster del-node 192.168.80.129:7006 33ce551fde006370da238a6470a6eb529d667d45
 - 14、重新分配哈希槽
./redis-5.0.8/src/redis-cli --cluster reshard 192.168.80.129:7000 --cluster-from 374a5fc11ba7a5d9f88fd20b4ca75bbd65433290 --cluster-to bf674349992c3b9b8d0ac42dfa788f2230597fcc --cluster-slots 1024
 - 15、添加从节点命令
./redis-5.0.8/src/redis-cli --cluster add-node 192.168.80.129:7007 192.168.80.129:7000 --cluster-slave --cluster-master-id 374a5fc11ba7a5d9f88fd20b4ca75bbd65433290
 - 16、验证集群扩容成功
cluster nodes