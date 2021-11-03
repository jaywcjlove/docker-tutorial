postgres
===

![postgres logo](https://raw.githubusercontent.com/docker-library/docs/01c12653951b2fe592c1f93a13b4e289ada0e3a1/postgres/logo.png)

快速启动

```bash
docker run \
  -p 5432:5432 \
  -e POSTGRES_PASSWORD=wcjiang \
  --name postgres \
  --restart always \
  -d \
  postgres:latest
```

使用 stack 部署示例

```yml
# Use postgres/example user/password credentials
version: '3.1'

services:

  db:
    image: postgres:latest
    restart: always
    environment:
      POSTGRES_PASSWORD: example

  adminer:
    image: adminer:latest
    restart: always
    ports:
      - 8081:8080
```

```bash
docker stack deploy -c stack.yml postgres
```

