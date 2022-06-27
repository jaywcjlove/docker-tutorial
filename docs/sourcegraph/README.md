Sourcegraph
===

<img alt="Sourcegraph" src="https://camo.githubusercontent.com/fda96ddd0983e308db3ce7dec5cf879bcbdfe0a2/68747470733a2f2f73746f726167652e676f6f676c65617069732e636f6d2f736f7572636567726170682d6173736574732f736f7572636567726170682d6c6f676f2e706e67" height="32px" />

[Sourcegraph](https://about.sourcegraph.com/) 是一个快速，开源，功能齐全的代码搜索和导航引擎。

![Screenshot](https://user-images.githubusercontent.com/1646931/46309383-09ba9800-c571-11e8-8ee4-1a2ec32072f2.png)

```bash
docker run \
  --publish 7080:7080 --rm \
  --volume ~/_docker/sourcegraph/config:/etc/sourcegraph \
  --volume ~/_docker/sourcegraph/data:/var/opt/sourcegraph \
  --volume /var/run/docker.sock:/var/run/docker.sock \
  -d sourcegraph/server:2.13.5
```