## 一、7.x版本安装(docker)

### 1.  elasticsearch安装

#### 1.1 pull image

```shell
docker pull docker.elastic.co/elasticsearch/elasticsearch:7.7.0
```

#### 1.2 build subnetwork

simply

```
docker network create esnet
```

custom ip

```
docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 esnet
```

#### 1.3 create and run

```
docker run --name es  -p 9200:9200 -p 9300:9300  --network esnet -e "discovery.type=single-node" 7ec4f35ab452
```

--name 名称                         给容器起个名字

-p 外部访问端口:容器端口              9200是供htpp访问端口，9300是供tcp访问的端口，如果不做端口映射，浏览器就不能访问elasticsearch的服务

--network 网络名                     用于多个服务通信与隔离，例如用kibana连接elasticsearch就需要他们在同一个网络下

bdaab402b220                      通过docker images命令查看到需要创建的容器id，此处用镜像名也可以

#### 1.4 ik install

```
docker exec -it es /bin/bash
./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.7.0/elasticsearch-analysis-ik-7.7.0.zip
```



### 2. kibana安装

#### 2.1 pull image

```
docker pull docker.elastic.co/kibana/kibana:7.7.0
```

#### 2.2 create container

```
docker run -it -d -e ELASTICSEARCH_HOSTS=http://ip:9200 -e ES_JAVA_OPTS="-Xms512m -Xmx512m" --name kibana -p 5601:5601  --network esnet docker.elastic.co/kibana/kibana:7.7.0
```

es ip可以通过下面命令查询再手动指定

```
docker network inspect mynet
```

也可以直接配置成容器名称:port

#### 2.3 modify es IPadress

```
docker exec -it 容器ID /bin/bash
vi /cofnfig/kibana.yml
```

## 二 .6.x版本安装(docker)

### 1. 安装elasticsearch

#### 1.1 拉取镜像

```bash
$ docker pull elasticsearch:6.8.0
1
```

#### 1.2 运行容器（内存不够用，设置小点）

```bash
$ sysctl -w vm.max_map_count=262144
$ sysctl -a|grep vm.max_map_count
$ docker run --name elasticsearch -p 9200:9200 -p 9300:9300 -e "ES_JAVA_OPTS=-Xms256m -Xmx256m" -d elasticsearch:6.8.0
```

### 1.3 进入容器，更改文件elasticsearch.ym，并安装ik分词器

```bash
$ docker exec -it elasticsearch /bin/bash
```

在elasticsearch.ym最后面添加

```bash
http.cors.enabled: true
http.cors.allow-origin: "*"
```

安装ik分词器

```bash
$ cd /usr/share/elasticsearch/plugins/
$ elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.8.0/elasticsearch-analysis-ik-6.8.0.zip
```

然后退出容器，重启elasticsearch

```bash
$ exit
$ docker restart elasticsearch
```

#### 1.4 kibana安装同7.x版本上

#### 1.5 cerebro安装

```
docker search cerebro
docker pull yannart/cerebro
docker run -d --name cerebro -p 9000:9000 --network mynet yannart/cerebro
```





### 三、elasticsearch docker compose

