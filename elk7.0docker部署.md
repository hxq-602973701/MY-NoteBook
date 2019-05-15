
#elasticsearch 部署：
# 需要挂载的目录：elasticsearch.yml、logs、log4j2.properties
# 注意:要将elasticsearch.yml中的elasticsearch.hosts 节点中的localhost改为服务器ip

docker run --detach \
           --name es \
           --restart always \
           --env TZ='Asia/Shanghai' \
           -e discovery.type=single-node \
           --publish 9200:9200 \
           --network docker-node-02 \
           --network-alias es \
           --volume /opt/volume/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
           --volume /opt/volume/elasticsearch/config/log4j2.properties:/usr/share/elasticsearch/config/log4j2.properties \
           --volume /opt/volume/elasticsearch/logs:/usr/share/elasticsearch/logs \
           docker.elastic.co/elasticsearch/elasticsearch:7.0.1 \


# logstash 部署：
# 需要挂载的目录：pipelines.yml 、log4j2.properties、config-orcal 、logs
# 1、复制DockerFile到任意目录
# 2、运行docker build -t my-logstash .
# 3、建立/opt/volume/logstash/config/pipelines.yml   /opt/volume/logstash/config-orcal   /opt/volume/logstash/logs
# 4、在config-oracl下放入ojdbc-6.jar  orcal.conf文件(需要将这个文件中的数据库链接地址换掉)

           docker run --detach \
           --name logstash \
           --restart always \
           --env TZ='Asia/Shanghai' \
           --network docker-node-02 \
           --network-alias logstash \
           --volume /opt/volume/logstash/config/pipelines.yml:/usr/share/logstash/config/pipelines.yml \
           --volume /opt/volume/logstash/config/log4j2.properties:/usr/share/logstash/config/log4j2.properties \
           --volume /opt/volume/logstash/logs:/usr/share/logstash/logs \
           --volume /opt/volume/logstash/config-orcal:/usr/share/logstash/bin/config-orcal \
           my-logstash:latest \


# kibana 部署：
# 需要挂载的目录：kibana.yml 、logs(在 kibana.yml中配置logs.dest)
# 注意：要将kibana.yml 中的elasticsearch.hosts 节点改为服务器ip
            docker run --detach \
           --name kibana \
           --restart always \
           --env TZ='Asia/Shanghai' \
           --publish 5601:5601 \
           --network docker-node-02 \
           --network-alias kibana \
           --volume /opt/volume/kibana/config/kibana.yml:/usr/share/kibana/config/kibana.yml \
           --volume /opt/volume/kibana/logs:/usr/share/kibana/logs \
           docker.elastic.co/kibana/kibana:7.0.1 \
