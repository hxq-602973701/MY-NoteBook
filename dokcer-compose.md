```sh
#!/bin/bash
cat > $dockerComposeYml << EOF
#version: '2'
#services:
#   intelligence_tomcat:
#       image: $dockerTag/tomcat:latest
#       ports: 
#           - $tomcatPort:8080
#       container_name: intelligence
#       restart: always
#       volumes: 
#           - /home/tomcat/intelligence/webapps:/usr/local/tomcat/webapps
#           - /home/tomcat/intelligence/logs:/usr/local/tomcat/logs
#           - /home/tomcat/intelligence/g_logs:/logs
#           - /home/tomcat/intelligence/bin_logs:/usr/local/tomcat/bin/logs
#           - /home/statics:/home/statics
#       environment:
#           - TZ=Asia/Shanghai

#   solr_tomcat:
#       image: $dockerTag/tomcat:latest
#       ports: 
#           - 7080:8080#
#       container_name: solr
#       restart: always
#       volumes: 
#           - /home/tomcat/solr/webapps:/usr/local/tomcat/webapps
#           - /home/tomcat/solr/logs:/usr/local/tomcat/logs
#           - /home/tomcat/solr/g_logs:/logs
#           - /home/tomcat/solr/bin_logs:/usr/local/tomcat/bin/logs
#        environment:
#            - TZ=Asia/Shanghai

#  nginx:
#      image: $dockerTag/nginx:latest
#      ports: 
#          - $nginxPort:80
#      container_name: nginx
#      restart: always
#      volumes:
#          - /home/nginx/conf.d:/etc/nginx/conf.d
#          - /home/nginx/logs:/var/log/nginx
#          - /home/statics:/home/statics
#      environment:
#          - TZ=Asia/Shanghai
#
#  #oracle u:system p:oracle sid:xe
#  oracle:
#      image: $dockerTag/sath89/oracle-xe-11g
#      container_name: oracle
#      restart: always
#      ports:
#          - 15219:1521
#      volumes:
#          - /home/oracle/data:/u01/app/oracle
#      environment:
#          - TZ=Asia/Shanghai

#    mysql:
#        image: $dockerTag/mysql:latest
#        environment:
#            MYSQL_ROOT_PASSWORD: 'Ulove'
#        container_name: mysql
#        ports:
#            - 33069:3306
#        restart: always
#        volumes:
#            - /home/mysql/db:/var/lib/mysql
#            - /home/mysql/conf:/etc/mysql/conf.d
#        environment:
#            - TZ=Asia/Shanghai

#    mongo-rs1:
#        image: mongo
#        container_name: mongo-rs1
#        ports:
#            - 2777:27017
#        restart: always
#        volumes:
#            - /home/mongo/mongo-rs1-db:/data/db
#        command: --smallfiles --replSet "rs0"
#        environment:
#            - TZ=Asia/Shanghai

#    mongo-rs2:
#        image: mongo
#        container_name: mongo-rs2
#        ports:
#            - 3777:27017
#        restart: always
#        volumes:
#            - /home/mongo/mongo-rs2-db:/data/db
#        command: --smallfiles --replSet "rs0"
#        environment:
#            - TZ=Asia/Shanghai

#    mongo-rs3:
#        image: mongo
#        container_name: mongo-rs3
#        ports:
#            - 4777:27017
#        restart: always
#        volumes:
#            - /home/mongo/mongo-rs3-db:/data/db
#        command: --smallfiles --replSet "rs0"
#        environment:
#            - TZ=Asia/Shanghai
EOF
