PostgreSQL
===

[PostgreSQL](https://www.postgresql.org/) (也叫 Postgres)是一个自由的对象-关系数据库服务器(数据库管理系统)，它在灵活的 BSD-风格许可证下发行。

## 快速启动容器

```bash
docker run \
  -p 5432:5432 \
  -e POSTGRES_PASSWORD=wcjiang \
  --name postgres \
  --restart always \
  -d \
  postgres:latest
```

## 使用 stack 部署示例

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

