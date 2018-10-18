

## ElasticSearch 6.4.2

```bash
# 最新版本
docker pull docker.elastic.co/elasticsearch/elasticsearch:6.4.2
# 运行容器
docker run \
  -p 9200:9200 -p 9300:9300 \
  -e "discovery.type=single-node" \
  docker.elastic.co/elasticsearch/elasticsearch:6.4.2
```

## ElasticSearch 5.3.3

ElasticSearch 5.3.3 docker 运行

```bash
# 下载老版本
docker pull docker.elastic.co/elasticsearch/elasticsearch:5.3.3
# 运行容器
docker run -p 9200:9200 \
  -e "http.host=0.0.0.0" \
  -e "transport.host=127.0.0.1" \
  -d docker.elastic.co/elasticsearch/elasticsearch:5.3.3
```

**Linux**

⚠️注意：vm_max_map_count 内核设置需要设置为至少262144以供生产使用。

应在 `/etc/sysctl.conf` 中永久设置 `vm_map_max_count` 设置：

```bash
$ grep vm.max_map_count /etc/sysctl.conf
vm.max_map_count=262144
```

## 通过 docker-compose 安装使用

新建 `docker-compose.yml` 文件

```yaml
version: '2'
services:
  elasticsearch1:
    image: docker.elastic.co/elasticsearch/elasticsearch:5.3.3
    container_name: elasticsearch1
    environment:
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
      nofile:
        soft: 65536
        hard: 65536
    mem_limit: 1g
    cap_add:
      - IPC_LOCK
    volumes:
      - esdata1:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
    networks:
      - esnet
  elasticsearch2:
    image: docker.elastic.co/elasticsearch/elasticsearch:5.3.3
    environment:
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - "discovery.zen.ping.unicast.hosts=elasticsearch1"
    ulimits:
      memlock:
        soft: -1
        hard: -1
      nofile:
        soft: 65536
        hard: 65536
    mem_limit: 1g
    cap_add:
      - IPC_LOCK
    volumes:
      - esdata2:/usr/share/elasticsearch/data
    networks:
      - esnet

volumes:
  esdata1:
    driver: local
  esdata2:
    driver: local

networks:
  esnet:
    driver: bridge
```

示例显示包含两个 Elasticsearch 节点的集群。 要打开群集，请使用 [docker-compose.yml](docker-compose.yml) 并输入：

```bash
docker-compose up -d 
docker-compose down -d # 停止集群
docker-compose down -v # 销毁集群和数据卷
docker logs elasticsearch1 # 查看日志
```

`elasticsearch1` 监听 `localhost:9200`，而 `elasticsearch2` 通过 `Docker` 网络与 `elasticsearch1` 进行通信。

此示例还使用名为 `esdata1` 和 `esdata2` 的 [Docker named volumes](https://docs.docker.com/engine/tutorials/dockervolumes)，如果尚未存在，将创建它们。

## 检查集群的状态

```bash
curl -u elastic http://127.0.0.1:9200/_cat/health
Enter host password for user 'elastic':
1472225929 15:38:49 docker-cluster green 2 2 4 2 0 0 0 0 - 100.0%
```


## 用户名密码

默认用户名密码 `elastic/changeme`

```bash
curl -XPUT -u elastic 'http://localhost:9200/_xpack/security/user/kibana/_password' -d '{
  "password" : "yourpasswd"
}'
```

## 挂载配置

创建自定义配置文件并将其挂载到映像的相应文件上。 例如，可以使用以下参数来完成使用 `docker run` 绑定安装custom_elasticsearch.yml：

```bash
-v full_path_to/custom_elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
```

其它配置修改项

```bash
# 避免出现跨域问题
http.cors.enabled: true
http.cors.allow-origin: "*"
# 在chorem中 当elasticsearch安装x-pack后还可以访问
http.cors.allow-headers: Authorization
# 启用审核以跟踪与您的Elasticsearch群集进行的尝试和成功的交互
xpack.security.audit.enabled: true
```

## 定义镜像

```dockerfile
FROM docker.elastic.co/elasticsearch/elasticsearch:5.3.3
ADD elasticsearch.yml /usr/share/elasticsearch/config/
USER root
RUN chown elasticsearch:elasticsearch config/elasticsearch.yml
USER elasticsearch
```

然后，您可以使用以下内容构建和尝试运行镜像：

```bash
docker build --tag=elasticsearch-custom .
docker run -ti -v /usr/share/elasticsearch/data elasticsearch-custom
# 覆盖默认的 CMD 
docker run <各种参数> bin/elasticsearch -Ecluster.name=mynewclustername
```

## 生产的一些经验

- 镜像公开 `TCP` 端口 `9200` 和 `9300`。对于群集，建议使用 `--publish-all` 随机化已发布的端口，除非您为每个主机固定一个容器。
- 使用 `ES_JAVA_OPTS` 环境变量来设置堆大小，例如使用 `16GB` 通过使用 `-e ES_JAVA_OPTS=-Xms16g -Xms16g"` 和 `dcker run` 来运行。 还建议为容器设置内存限制。

## 其它

- [ElasticSearch 5.3 官方 Docker 安装教程](https://www.elastic.co/guide/en/elasticsearch/reference/5.3/docker.html)
- [ElasticSearch 官方 Docker 安装教程](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html)
- [Docker 镜像仓库](https://hub.docker.com/r/library/elasticsearch/)