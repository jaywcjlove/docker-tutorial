FROM --platform=linux/amd64 openjdk:11.0.16

# 安装 Maven 3.8.7
ENV MAVEN_VERSION=3.8.7
RUN apt-get update && apt-get install -y wget tar \
    && wget https://archive.apache.org/dist/maven/maven-3/${MAVEN_VERSION}/binaries/apache-maven-${MAVEN_VERSION}-bin.tar.gz \
    && tar xzf apache-maven-${MAVEN_VERSION}-bin.tar.gz -C /opt \
    && ln -s /opt/apache-maven-${MAVEN_VERSION} /opt/maven \
    && rm apache-maven-${MAVEN_VERSION}-bin.tar.gz

# 设置 Maven 环境变量
ENV MAVEN_HOME=/opt/maven
ENV PATH="${MAVEN_HOME}/bin:${PATH}"

# 创建非 root 用户并设置权限
RUN useradd -u 1000 -ms /bin/bash jenkins \
    && mkdir -p /home/jenkins/.m2/repository \
    && chown -R jenkins:jenkins /home/jenkins/.m2

# 切换到非 root 用户
USER jenkins

# 验证 Maven 版本
RUN mvn -version