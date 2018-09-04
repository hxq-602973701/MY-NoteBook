## docker 简易部署
```sh
docker run --detach \
           --name yhym_jx_sr_sr_xcx \
           --restart always \
           --publish 8080:8080 \
           --network docker-node-02 \
           --network-alias yhym_xcx \
           --volume /opt/volume/yhym_xcx/ROOT:/usr/local/tomcat/webapps/ROOT \
           --volume /opt/volume/yhym_xcx/logs:/usr/local/tomcat/logs \
           --volume /opt/volume/statics:/home/data/statics \
           hub.youlove.com.cn/u-infrastructure/tomcat:9-jre8-alpine

docker简单步骤
1、docker search tomcat -- 寻找镜像
2、docker pull tomcat  -- 下载tomcat
3、doker images  -- 查看镜像
4、cd /home  /xcx/webapps、logs/ROOT  配置挂载目录
5、docker -d -p 8888:8080 -V /home/xcx/webapps/:/usr/local/tomcat/webapps -V /home/xcx/logs/:/usr/local/tomcat/logs -e TZ=Asia/Shanghai  --restart= always --name=xcx tomcat   启动

