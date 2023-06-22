Navidrome
===

现代音乐服务器和流媒体兼容 Subsonic/Airsonic

## 使用 `docker-compose`

创建包含以下内容的 `docker-compose.yml` 文件（或将下面的 navidrome 服务添加到现有文件中）：

```yml
version: "3"
services:
  navidrome:
    image: deluan/navidrome:latest
    user: 1000:1000 # should be owner of volumes
    ports:
      - "4533:4533"
    restart: unless-stopped
    environment:
      # Optional: put your config options customization here. Examples:
      ND_SCANSCHEDULE: 1h
      ND_LOGLEVEL: info  
      ND_SESSIONTIMEOUT: 24h
      ND_BASEURL: ""
    volumes:
      - "/path/to/data:/data"
      - "/path/to/your/music/folder:/music:ro"
```

使用 `docker-compose up -d` 启动它。 请注意，上面的环境变量只是一个示例，不是必需的。 示例中的值已经是默认值

## 使用docker命令行工具

```bash
$ docker run -d \
   --name navidrome \
   --restart=unless-stopped \
   --user $(id -u):$(id -g) \
   -v /path/to/music:/music \
   -v /path/to/data:/data \
   -p 4533:4533 \
   -e ND_LOGLEVEL=info \
   deluan/navidrome:latest
```