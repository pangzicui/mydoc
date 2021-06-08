# Docker创建jdk基础镜像
arm-64
```
FROM cyphernode/alpine-glibc-base:v3.12.4_2.31-0
COPY ./jdk1.8.0_291 /usr/local/jdk1.8.0_291
ENV JAVA_HOME /usr/local/jdk1.8.0_291
ENV JRE_HOME /usr/local/jdk1.8.0_291/jre
ENV PATH $JAVA_HOME/bin:$PATH
```