# efk

以下服务单节点配置,启一个elasticsearch

我们会创建四个容器：

    httpd              # 测试用,已经关闭了(发送日志给EFK) ,需要可以在docker-compose.yml中打开注释
    Fluentd            # 日志收集
    Elasticsearch      # 数据库
    Kibana             # 图形管理系统

参考:
https://blog.csdn.net/shykevin/article/details/104568715/

## author wanghaima

## 目录结构
```sh
.
├── docker-compose.yml
├── elasticsearch
│   ├── config
│   │   └── elasticsearch.yml
│   └── es
│       ├── data
│       │   └── empty.keep
│       ├── logs
│       │   └── empty.keep
│       └── plugins
│           └── ik
│               ├── commons-codec-1.9.jar
│               ├── commons-logging-1.2.jar
│               ├── config
│               │   ├── extra_main.dic
│               │   ├── extra_single_word.dic
│               │   ├── extra_single_word_full.dic
│               │   ├── extra_single_word_low_freq.dic
│               │   ├── extra_stopword.dic
│               │   ├── IKAnalyzer.cfg.xml
│               │   ├── main.dic
│               │   ├── preposition.dic
│               │   ├── quantifier.dic
│               │   ├── stopword.dic
│               │   ├── suffix.dic
│               │   └── surname.dic
│               ├── elasticsearch-analysis-ik-7.3.1.jar
│               ├── httpclient-4.5.2.jar
│               ├── httpcore-4.4.4.jar
│               ├── plugin-descriptor.properties
│               └── plugin-security.policy
├── fluentd
│   ├── conf
│   │   └── fluent.conf
│   └── Dockerfile
├── kibana
│   ├── config
│   │   └── kibana.yml
│   └── Dockerfile
└── README.md
```


## es 配置/data和log 已经挂载出来

## kibana 配置和log 已经挂载出来

## 环境
linux系统
安装好docker 和 docker-compose


## 系统调优，linux系统修改配置文件

 1. 修改 `/etc/sysctl.conf` 文件 

    此参数一定要改，否则Elasticsearch 无法启动

    ```sh
    vm.max_map_count = 2621440
    fs.file-max = 655350
    ```

 2. 加载配置

    `sysctl -p`


## 克隆项目
```sh
git clone https://github.com/haimait/docker_compose_efk.git
```

## 启动的容器

```sh
[root@HmEduCentos01 efk]# cd docker_compose_efk
[root@HmEduCentos01 efk]# chmod -R 777 ./*
[root@HmEduCentos01 efk]# docker-compose build
[root@HmEduCentos01 efk]# docker-compose up -d #启动
[root@HmEduCentos01 efk]# docker ps -a
CONTAINER ID   IMAGE                                                 COMMAND                  CREATED         STATUS                    PORTS                                                          NAMES
aa1052cc09e9   efk_fluentd                                           "/bin/entrypoint.sh …"   2 seconds ago   Up 2 seconds              5140/tcp, 0.0.0.0:24224->24224/tcp, 0.0.0.0:24224->24224/udp   efk_fluentd_1
2628c7476d8f   efk_kibana                                            "/usr/local/bin/dumb…"   2 seconds ago   Up 2 seconds              0.0.0.0:5601->5601/tcp                                         efk_kibana_1
f3db4922f745   docker.elastic.co/elasticsearch/elasticsearch:7.3.1   "/usr/local/bin/dock…"   3 seconds ago   Up 2 seconds              0.0.0.0:9200->9200/tcp, 9300/tcp                               efk_elasticsearch_1
```

## 常用命令

```sh
docker-compose up -d   #启动
docker-compose down    #停用
docker-compose restart #重启
```



## httpd服务产生日志( 选择操作, 默认关闭,需要在docker-compose.yml)

使用curl执行3遍

```sh
curl http://localhost:1080/
curl http://localhost:1080/
curl http://localhost:1080/
```


## 访问kibana管理页面
http://localhost:5601/ #内网ip:5601

或者

http://10.10.10.254:5601 #公网ip:5601

### Create index pattern 创建索引

输入fluentd-*

选择时间戳

刷新即可

详细操作参考下面的网址:

https://www.cnblogs.com/edisonchou/p/docker_logs_study_summary_part2.html



## 收集其它的服务

启动服务时主要加入面这两行参数

```sh

 --log-driver=fluentd \ #定其log-dirver为fluentd
 --log-opt fluentd-address=localhost:24224 \  #efk_fluentd_1服务的ip地址,这里写localhost即可
 --log-opt tag="hello-world" \  #设立了tag，方便我们后面验证查看日志
```

例子:

```sh

docker run \
    --log-driver=fluentd \
    --log-opt fluentd-address=localhost:24224 \
    --log-opt tag=httpd.access \
    -d hello-world

[root@HmEduCentos01 efk]# docker ps -a
CONTAINER ID   IMAGE                           COMMAND                  CREATED          STATUS                    PORTS           NAMES
6d8c17fdb672   hello-world                  "./hello-world"             4 seconds ago    Up 3 seconds                              hello-world
```

## 添加登录及权限

**下以为单节点配置**

内置用户为elastic,apm_system,kibana,logstash_system,beats_system,remote_monitoring_user，一定要记住所有密码，目前用到了elastic 别的以后肯定用，只是目前没有涉及。

**操作步骤**

1. 打开 `elasticsearch\config\elasticsearch.yml` 中的 `xpack.security.enabled: true ` 的注释

2. 打开 `kibana\config\kibana.yml` 中的 `连接 elasticsearch的账号/密码` 的注释

3. 设置ES服务容器里内置账号的密码

```sh
[root@HmEduCentos01 efk]# docker exec -it efk_elasticsearch_1 bash
[root@fe0c639f1188 bin]# ./bin/elasticsearch-setup-passwords interactive
Initiating the setup of passwords for reserved users elastic,apm_system,kibana,logstash_system,beats_system,remote_monitoring_user.
You will be prompted to enter passwords as the process progresses.
Please confirm that you would like to continue [y/N]y

下面输入密码要和kibana里写的密码一致

Enter password for [elastic]: 
Reenter password for [elastic]: 
Enter password for [apm_system]: 
Reenter password for [apm_system]: 
Enter password for [kibana]: 
Reenter password for [kibana]: 
Enter password for [logstash_system]: 
Reenter password for [logstash_system]: 
Enter password for [beats_system]: 
Reenter password for [beats_system]: 
Enter password for [remote_monitoring_user]: 
Reenter password for [remote_monitoring_user]: 
Changed password for user [apm_system]
Changed password for user [kibana]
Changed password for user [logstash_system]
Changed password for user [beats_system]
Changed password for user [remote_monitoring_user]
Changed password for user [elastic]
```

4. web访问kibana已经有登陆页面了

`http://10.10.10.254:5601/`

5. 如果没有登陆页面,重启服务,再刷新页面

`docker-compose restart`


更多参考下面的地址:
https://blog.csdn.net/qiaorui_/article/details/97375237


## Kibana 创建用户和权限

参考下面的地址:
https://blog.csdn.net/cui884658/article/details/106805325/


## elasticsearch安装ik分词器

这里默认已经安装好了，7.3.1版本的ik分词器

如果想要换成其它的版本的es，就需要下载对应版本的ik分词器

到下面的地址里找到对应的版本

https://github.com/medcl/elasticsearch-analysis-ik

以7.3.1为例

```sh
cd elasticsearch/es/plugins 
rm -rf ik/ #删除原来之前的ik分词器
mkdir ik #新建ik目录
wget https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.3.1/elasticsearch-analysis-ik-7.3.1.zip
unzip elasticsearch-analysis-ik-7.3.1.zip -d ./ik/ #解压ik插件到ik目录
chmod -R 777 elasticsearch/es/ #添加权限
#要删除或移走压缩包，elasticsearch/es/plugins目录里不能放压缩包不然会报错
rm elasticsearch-analysis-ik-7.3.1.zip 
docker-compose restart #重启服务
root@haima-PC:/usr/local/docker/efk/docker_compose_efk# docker exec -it docker_compose_efk_elasticsearch_1 bash ./bin/elasticsearch-plugin list #查看插件列表已经生效
ik
```

参考下面的地址:
https://www.cnblogs.com/szwdun/p/10664348.html
