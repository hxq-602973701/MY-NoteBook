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
