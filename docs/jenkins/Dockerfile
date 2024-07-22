FROM --platform=linux/amd64 jenkins/jenkins:2.468-jdk21

# 切换到 root 用户以安装 Docker CLI
USER root

# 安装必要的软件包
RUN apt-get update && apt-get install -y \
    curl \
    gnupg2 \
    lsb-release \
    software-properties-common

# 安装特定版本的 Docker CLI
RUN curl -fsSL https://download.docker.com/linux/static/stable/x86_64/docker-26.1.4.tgz | tar xzvf - --strip-components=1 -C /usr/local/bin

# 创建 docker 组并将 jenkins 用户添加到该组
RUN groupadd docker && usermod -aG docker jenkins

# 切换回 jenkins 用户
USER jenkins