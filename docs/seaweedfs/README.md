SeaweedFS
===

[SeaweedFS](https://github.com/chrislusf/seaweedfs) 是一个简单且高度可扩展的分布式文件系统。 有两个目标：存储数十亿个文件！ 快速提供文件！ SeaweedFS实现了一个带有O（1）磁盘搜索的对象存储，以及一个带有POSIX接口的可选Filer。

```bash
docker build -t wcjiang/seaweedfs . # 编译docker镜像
docker push wcjiang/seaweedfs:latest
docker-compose up # 使用docker-compose 执行新建容器组
docker-compose start # 启动容器组
docker-compose stop # 停止容器组
docker-compose ps   # 查询容器组所有容器状态
docker-compose down # 删除容器组
```

⚠️： `volume` 节点不可以被负载均衡，不然会出现上传错误。

## 名词

- `master`: 主节点，即集群管理，同时存储文件和fid映射关系
- `volume`: 1、文件卷节点，实际存储文件；2、卷，一个存储级别
- `client`: 客户端，该FS使用RESTful交互，所以客户端都归纳为一类
- `dataCenter`: 数据中心，简称DC
- `rack`: 机架。一个机架属于特定的数据中心，一个数据中心可以包含多个机架。

```bash
                                    ┌┈┈┈┈┈┈┈┈┐  ┌┈┈┈┈┈┈┈┈┐
                                    ┆        ┆  ┆        ┆
┌┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┐              ┆M-master├┈┈┤S-master┆
┆                    ├┈┈┈┈┈┈┈┈┈┈┈┈›▷▶        ┆  ┆        ┆
┆ client/web browser ┆              └┈┈┈┬┈┈┈┈┘  └┈┈┈┬┈┈┈┈┘
┆                    ├┈┈┐               ┆           ┆
└┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┘  ┆         ┌┈┈┈┈┈┴┈┈┈┈┈┬┈┈┈┈┈┴┈┈┈┈┈┐
                        ┆         ┆           ┆           ┆
                        ┆         ┆           ┆           ┆
                        ┆     ┌┈┈┈┴┈┈┈┈┐  ┌┈┈┈┴┈┈┈┈┐  ┌┈┈┈┴┈┈┈┈┐
                        ┆     ┆        ┆  ┆        ┆  ┆        ┆
                        └┈┈┈›▷▶ volume ┆  ┆ volume ┆  ┆ volume ┆
                              ┆        ┆  ┆        ┆  ┆        ┆
                              └┈┈┈┈┈┈┈┈┘  └┈┈┈┈┈┈┈┈┘  └┈┈┈┈┈┈┈┈┘
```

## 写文件

**上传文件**：首先，将HTTP `POST`，`PUT`或`GET`请求发送到 `/dir/assign`以获取fid和卷服务器URL：

```bash
curl http://localhost:9333/dir/assign
# {"count":1,"fid":"3,01637037d6","url":"127.0.0.1:8080","publicUrl":"localhost:8080"}
```

其次，要存储文件内容，请从响应中向 `URL +'/'+ fid` 发送`HTTP` `POST`请求：

```bash
curl -F file=@/home/chris/myphoto.jpg http://127.0.0.1:8080/3,01637037d6
# {"size": 43234}
```

⚠️：要更新，请发送包含更新文件内容的其他POST请求。

## 删除文件

将HTTP DELETE请求发送到相同的`URL +'/'+ fid` URL：

```bash
curl -X DELETE http://127.0.0.1:8083/3,01637037d6
```

## 保存文件ID

现在，您可以将fid（本例中为3,01637037d6）保存到数据库字段中。

开头的数字 `3` 表示卷 `ID`。 在逗号之后，它是一个文件密钥 `01` 和一个文件 cookie `637037d6`

卷`id`是无符号的32位整数。 文件密钥是无符号的64位整数。 文件cookie是无符号的32位整数，用于防止URL猜测。

文件密钥和文件cookie都以十六进制编码。 您可以以自己的格式存储`<volume id，file key，file cookie>`元组，或者只是将fid存储为字符串。

如果存储为字符串，理论上，您需要`8+1+16+8=33`字节。 如果不是绰绰有余，char(33) 就足够了，因为大多数用法不需要 `2^32`卷。

如果空间确实存在问题，您可以使用自己的格式存储文件ID。 对于卷id，您需要一个4字节整数，对于文件密钥，需要8字节长整数，对于文件cookie，需要4字节整数。 所以16个字节绰绰有余。

## 读取文件

以下是如何呈现URL的示例。

首先按文件的 `volumeId` 查找卷服务器的URL：

```bash
curl http://localhost:9333/dir/lookup?volumeId=3
# {"locations":[{"publicUrl":"localhost:8080","url":"localhost:8080"}]}
```

由于（通常）卷服务器不是太多，并且卷不经常移动，因此您可以在大多数时间缓存结果。 根据复制类型，一个卷可以具有多个副本位置。 只需随机选择一个位置即可阅读。

现在您可以通过url获取公共URL，呈现URL或直接从卷服务器读取：

```bash
http://localhost:8083/3,01637037d6.jpg
```

⚠️：请注意，我们在这里添加文件扩展名 `.jpg`。 它是可选的，只是客户端指定文件内容类型的一种方式。

如果您想要更好的URL，可以使用以下其中一种替代URL格式：

```bash
http://localhost:8083/3/01637037d6/my_preferred_name.jpg
http://localhost:8083/3/01637037d6.jpg
http://localhost:8083/3,01637037d6.jpg
http://localhost:8083/3/01637037d6
http://localhost:8083/3,01637037d6
```

如果您想获得图像的缩放版本，可以添加一些参数：

```bash
http://localhost:8083/3/01637037d6.jpg?height=200&width=200
http://localhost:8083/3/01637037d6.jpg?height=200&width=200&mode=fit
http://localhost:8083/3/01637037d6.jpg?height=200&width=200&mode=fill
```