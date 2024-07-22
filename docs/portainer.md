Portainer
===

[Portainer](https://github.com/portainer/portainer) 是一个开源、轻量级Docker管理用户界面，基于Docker API，提供状态显示面板、应用模板快速部署、容器镜像网络数据卷的基本操作（包括上传下载镜像，创建容器等操作）、事件日志显示、容器控制台操作、Swarm集群和服务等集中管理和操作、登录用户管理和控制等功能。功能十分全面，基本能满足中小型单位对容器管理的全部需求。

## v2 安装

```bash
docker volume create portainer_data
```

Docker Standalone

```bash
docker run -d \
  -p 8000:8000 \
  -p 9000:9000 \
  --name=portainer \
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce
```

Docker Swarm

```bash
curl -L https://downloads.portainer.io/ee2-18/portainer-agent-stack.yml -o portainer-agent-stack.yml
# 下载 https://downloads.portainer.io/portainer-agent-stack.yml
docker stack deploy -c portainer-agent-stack.yml portainer
```

## v2.20

新建 `portainer-agent-stack.yml` 文件， 将下面内容复制到配置文件中，你可以从官方仓库拷贝 [`portainer/portainer-compose`](https://github.com/portainer/portainer-compose) 配置。

```yml
version: '3.2'
services:
  agent:
    image: portainer/agent:2.20.3-alpine
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      #- ${HOME}/.orbstack/run/docker.sock:/var/run/docker.sock
      - /var/lib/docker/volumes:/var/lib/docker/volumes
    networks:
      - agent_network
    deploy:
      mode: global
      placement:
        constraints: [node.platform.os == linux]

  portainer:
    image: portainer/portainer-ce:2.20.3-alpine
    command: -H tcp://tasks.agent:9001 --tlsskipverify
    ports:
      - "9443:9443"
      - "9000:9000"
      - "8000:8000"
    volumes:
      - portainer_data:/data
      - /etc/localtime:/etc/localtime:ro
    networks:
      - agent_network
    deploy:
      mode: replicated
      replicas: 1
      placement:
        constraints: [node.role == manager]

networks:
  agent_network:
    driver: overlay
    attachable: true

volumes:
  portainer_data:
```

## 启动容器

```bash
docker stack deploy --compose-file=portainer-agent-stack.yml portainer
```

## 运行容器

```bash
docker run -d -p 8000:8000 -p 9000:9000 -p 9443:9443 \
  --name=portainer --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v ~/_docker/portainer/data:/data \
  portainer/portainer-ce:latest
```