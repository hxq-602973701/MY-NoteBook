```sh
#!/bin/bash

#################################### 安装前配置 #########################
#系统名称
export serviceName="xxx平台"

#服务器用途(应用服务器/数据库服务器/或其他)
export serviceType="应用服务器"

#要添加的服务器用户
export addUser=admin

#服务器IP,子网掩码,网关
#是否设置服务器IP(1表示设置IP,0表示不设置IP)
export isSetIP=1

export ipAddr=10.141.21.82
export netMask=255.255.255.0
export gateWay=10.141.21.249

#是否设置yum源地址(1表示设置yum源地址,0表示不设置)
export isSetYUM=1

#yum源地址(内网源地址:http://xx.xx.xx.xx:81/centos7)
export yumPath=http://xx.xx.xx.xx:81/centos7

#docker的yum源地址(内网docker源地址:http://xx.xx.xx.xx:81/docker-yum/yum/repo/centos7)
export dockerYumPath=http://xx.xx.xx.xx:81/docker-yum/yum/repo/centos7

#docker的镜像下载地址(内网镜像地址是10.118.224.102:50)
export dockerRegistryPath=https://registry.docker-cn.com

#dockerTag(内网是:10.118.224.102:50)
export dockerTag=docker.io

#docker资源目录
export dockerHome=/home/docker

#docker-compose的yml文件目录
export dockerComposeDir=/home/docker-compose

#docker-compose.yml的路径
export dockerComposeYml=$dockerComposeDir/docker-compose.yml

#主程序tomcat端口
export tomcatPort=8080

#nginx端口
export nginxPort=80

#oracle端口
export oraclePort=15219

#mysql端口
export mysqlPort=33069

#ssh远程端口
export sshPort=22315

#要开放的防火墙端口(括弧中填要开放的端口,用空格隔开)
export openPortArr=($sshPort $oraclePort $mysqlPort $tomcatPort $nginxPort)


#######################################################################

source ./setup.sh
