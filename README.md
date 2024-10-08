DBdoctor支持**安装包解压一键安装**与**Docker镜像手动导入**安装两种模式。

其中，安装包解压一键安装模式目前仅支持Linux系统，Docker镜像手动导入安装模式支持Linux和Mac系统。

# 安装包解压一键安装
下载地址：

- x86安装包：https://github.com/juhaokan/DBdoctor/tags

- ARM安装包：https://github.com/juhaokan/DBdoctor/tags

预估时间：**1分钟**

将下载的tar.gz安装包解压缩，进入解压后的根目录，执行./dbd -I进行DBdoctor零依赖快速安装。例如：

- x86环境

```shell
tar -zxvf DBdoctorV3.2.3_20240820_x86.tar.gz -C ${INSTALL_PATH}
cd${INSTALL_PATH}
./dbd -I
```
- arm环境

```shell
tar -zxvf DBdoctorV3.2.3_20240820_arm.tar.gz -C ${INSTALL_PATH}
cd${INSTALL_PATH}
./dbd -I
```
注：使用./dbd -I参数执行脚本会进行单机版的一键安装。如果需要安装集群版，请参考集群安装。

关于dbd命令的更多参数说明，请参考dbd命令参数说明。

`./dbd --install`或`-I`执行安装时，可以添加选项`--unlimited`忽略4c8g的限制。

安装成功后会打印访问地址：

    WebSite:   http://xxx.xxx.xxx.xxx:13000/#/login

    Default User:     tester

    Default Password: Root2023!

其中，xxx.xxx.xxx.xxx为用户安装的主机IP地址。

脚本会自动创建一个初始测试账号tester（初始密码Root2023!）和一个初始管理员账号admin（初始密码123456）。

##### 日志路径

mysql日志：${INSTALL_PATH}/service/mysql/mysql_error.log

agent日志：${INSTALL_PATH}/usr/local/dra-agent/logs/*.log

kafka日志：${INSTALL_PATH}/service/kafka/logs/server.log

dra-pi日志：${INSTALL_PATH}/service/dra-pi/logs/*.log

dra-data-sample日志：${INSTALL_PATH}/service/dra-data-sample/logs/*.log


# docker镜像手动导入安装
由于Docker Hub国内无法访问，目前DBdoctor已将镜像仓库迁移到阿里云的ACR，同样支持docker pull直接下载安装。同时DBdoctor也支持官网直接下载docker镜像压缩包，解压导入即可进行部署安装。上述两种方式任选一种即可。

## 下载地址（任选一种）
### a）Docker压缩包下载导入
压缩包下载地址：

- x86安装包

    - DBdoctor服务端X86安装包：https://jhktob.oss-cn-beijing.aliyuncs.com/dbdoctor-3.2.3_x86.zip

    - DBdoctor Agent采集器X86安装包：https://jhktob.oss-cn-beijing.aliyuncs.com/dbdoctor-agent-3.2.3_x86.zip

- ARM安装包

    - DBdoctor服务端ARM安装包：https://jhktob.oss-cn-beijing.aliyuncs.com/dbdoctor-3.2.3_arm.zip

    - DBdoctor Agent采集器ARM安装包：https://jhktob.oss-cn-beijing.aliyuncs.com/dbdoctor-agent-3.2.3_arm.zip

- 安装包导入步骤：

    - 需要提前安装并启动Docker服务，需要检查docker所在磁盘空间充足

    - 下载镜像文件并解压，得到dbdoctor-3.2.3_x86.tar

    - 导入镜像docker load -i ./dbdoctor-3.2.3_x86.tar

### b）阿里云ACR Docker镜像仓库下载地址：
- X86 Server/Agent下载地址

docker pull registry.cn-zhangjiakou.aliyuncs.com/dbdoctor/dbdoctor:3.2.3_x86

docker pull registry.cn-zhangjiakou.aliyuncs.com/dbdoctor/dbdoctor-agent:3.2.3_x86

- ARM Server/Agent下载地址

docker pull registry.cn-zhangjiakou.aliyuncs.com/dbdoctor/dbdoctor:3.2.3_arm

docker pull registry.cn-zhangjiakou.aliyuncs.com/dbdoctor/dbdoctor-agent:3.2.3_arm

## 执行`docker run启动命令，需要注意Mac版与Linux版启动差异
- Linux版本：demo实例支持锁分析功能，需要挂载系统目录。

```shell
docker run -d \
    -e HOST_IP=<host ip> \
    -e HOST_PORT=<web port> \
    -e KAFKA_HOST_PORT=<kafka port> \
    -e OS=linux \
    -p <web port>:13000 \
    -p <kafka port>:9092 \
    -v /proc:/host/proc:ro \
    -v /lib/modules:/lib/modules \
    -v /usr/src:/usr/src:ro \
    -v /sys/kernel/debug:/sys/kernel/debug \
    -v /<datadir>:/data \
    --privileged \
    --name dbdoctor \
    dbdoctor-server:3.2.3
```
- Mac版本：demo实例不支持锁分析功能，不需要挂载系统目录。

```shell
docker run -d \
   -e HOST_IP=<host ip> \
   -e HOST_PORT=<web port> \
   -e KAFKA_HOST_PORT=<kafka port> \
   -e OS=mac \
   -p <web port>:13000 \
   -p <kafka port>:9092 \
   --name dbdoctor \
   dbdoctor-server:3.2.3
```
检查启动日志docker logs -f dbdoctor，启动成功后会打印WEB地址与账号密码。

默认DBA用户tester密码Root2023!，默认管理员账号admin密码123456。

Server启动后自带一套纳管的本地Demo实例，通过Web页面添加的纳管实例所在机器有如下要求：

- 要在官方支持的Linux版本列表范围内

- 需要与server端使用相同CPU架构

## 其他问题
##### 既有ARM又有X86的数据库如何部署安装？

DBdoctor的Server用户根据主机CPU架构自行选择安装包一键拉起即可。而针对Agent，自动安装的是需要数据库节点和Server节点的CPU架构一致的。当数据库节点CPU架构不一致时，可以选择单独下载适合的Agent，并进行手动安装Agent。

##### K8S部署方式是否可以支持？

当然支持K8S方式部署Sever和Daemonset安装Agent，我们内部均采用此种部署架构，如果有需要可以联系我们。
