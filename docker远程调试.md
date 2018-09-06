```sh
docker run --detach \
           --name yhym_jx_sr_sr_xcx \
           --restart always \
           --env TZ='Asia/Shanghai' \
           --env JPDA_ADDRESS=5005 \
           --env JPDA_TRANSPORT=dt_socket \
           --publish 5005:5005 \
           --network docker-node-02 \
           --network-alias yhym_xcx \
           --volume /opt/volume/yhym_xcx/ROOT:/usr/local/tomcat/webapps/ROOT \
           --volume /opt/volume/yhym_xcx/logs:/usr/local/tomcat/logs \
           --volume /opt/volume/statics:/home/data/statics \
           hub.youlove.com.cn/u-infrastructure/tomcat:9-jre8-alpine \
           /usr/local/tomcat/bin/catalina.sh jpda run
