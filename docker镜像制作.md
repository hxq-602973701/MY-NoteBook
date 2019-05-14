#docker 镜像制作  

https://blog.csdn.net/keepd/article/details/80569797    镜像制作地址

```ssh
FROM openjdk:8-jre-alpine

ADD intelligence-excel4j-0.0.1-SNAPSHOT.jar app.jar

EXPOSE 8086

ENTRYPOINT ["java","-jar","/app.jar"]


sudo docker build -t="task" .

docker save > /home/admin/ticket.tar ticket:latest  //将做好的镜像打包到本地
docker load < ./ticket.tar //将镜像添加到docker images中




#使用指定配置文件启动docker

以下为DockerFile文件
FROM docker.elastic.co/logstash/logstash:7.0.1

#安装input插件
RUN logstash-plugin install logstash-input-jdbc
#安装output插件
RUN logstash-plugin install logstash-output-elasticsearch
#容器启动时执行的命令.(CMD 能够被 docker run 后面跟的命令行参数替换)
CMD ["-f", "/usr/share/logstash/bin/config-orcal/orcal.conf"]
