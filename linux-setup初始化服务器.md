```sh
#!/bin/bash
######### 函数定义 ########
#判断任务配置是否成功,参数传任务名称
function isSucceed (){
    if [ $? -eq 0 ]; then
        echo "配置或启动成功:$1"
        sleep 1
    else
        read -p "配置或启动失败:$1. 按回车继续脚本"
    fi
}
#重启docker
function restartDocker (){
    systemctl restart docker
    systemctl status docker | grep active\ \(running\)
    isSucceed "docker"
}
#修改docker配置文件
function editDaemonJson (){
    cat > /etc/docker/daemon.json << EOF
{
    "live-restore": true
   ,"registry-mirrors": ["$dockerRegistryPath"]
   ,"insecure-registries": ["$dockerRegistryPath"]
   ,"graph": "$dockerHome"
}
EOF
    restartDocker 
    systemctl enable docker
}

######## 设置系统环境 ########
cat > /etc/motd << EOF
         __  ____    ____ _    ________
        / / / / /   / __ \ |  / / ____/
       / / / / /   / / / / | / / __/
      / /_/ / /___/ /_/ /| |/ / /___
      \____/_____/\____/ |___/_____/

   Welcome to Youlove Elastic Compute Service!
   服务器用途:$serviceType   IP地址:$ipAddr
   系统名称:$serviceName
EOF

cat /etc/motd | grep $serviceName
isSucceed "欢迎页面"

if [ $isSetIP -eq 1 ]; then
    # 配置服务器IP地址
    fileNameTmp=`ls /etc/sysconfig/network-scripts | grep ifcfg-`
    fileNames=($(echo $fileNameTmp))
    fileNamesNum=${#fileNames[@]}
    
    if [ $fileNamesNum -gt 1 ]; then
        for (( i = 0; i < $fileNamesNum; i++ )); do
            if [ $i -eq 0 ]; then
                fileName=${fileNames[$i]}
                cat >> /etc/sysconfig/network-scripts/$fileName << EOF
IPADDR=$ipAddr
NETMASK=$netMask
GATEWAY=$gateWay
EOF
                sed -i "s/dhcp/static/g" /etc/sysconfig/network-scripts/$fileName
                sed -i "s/ONBOOT=no/ONBOOT=yes/g" /etc/sysconfig/network-scripts/$fileName
    
                systemctl restart network
                systemctl status network | grep active\ \(exited\)
                if [ $? -eq 0 ]; then
                    echo "network 重启成功"
                    sleep 1
                    ip addr | grep $ipAddr
                    isSucceed "IP地址"
                else
                    read -p "network 重启失败,可能IP配置有问题. 按回车继续脚本"
                fi
            fi
        done
    fi
fi


# 关闭SElinux
sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config
cat /etc/selinux/config | grep SELINUX=disabled
isSucceed "SElinux"

# 添加帐号,并加入sudo权限
adduser $addUser
passwd $addUser
touch /etc/sudoers.d/$addUser
echo "$addUser    ALL=(ALL)    NOPASSWD:ALL" >> /etc/sudoers.d/$addUser
cat /etc/sudoers.d/$addUser | grep $addUser
isSucceed "用户$addUser加入sudo"

# 修改ssh端口,禁止root远程登录
sed -i "s/#PermitRootLogin yes/PermitRootLogin no/g" /etc/ssh/sshd_config
cat /etc/ssh/sshd_config | grep PermitRootLogin\ no
isSucceed "禁止root远程登录"

sed -i "s/#Port 22/Port $sshPort/g" /etc/ssh/sshd_config
cat /etc/ssh/sshd_config | grep Port\ $sshPort
isSucceed "SSH远程端口修改"

if [ $isSetYUM -eq 1 ]; then
    # 设置yum源
    mkdir /etc/yum.repos.d/bak
    mv /etc/yum.repos.d/* /etc/yum.repos.d/bak
    cp /etc/yum.repos.d/bak/CentOS-Media.repo /etc/yum.repos.d/
    
    cat > /etc/yum.repos.d/CentOS-Media.repo << EOF
[c7-media]
name=CentOS
baseurl=$yumPath
gpgcheck=0
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

[docker]
name=docker
baseurl=$dockerYumPath
gpgcheck=0
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
EOF
    sleep 1
    yum clean all
    yum makecache
fi


#判断yum源配置是否成功,安装必要工具
yum -y install telnet
rpm -qa | grep telnet
isSucceed "YUM源"

yum -y install net-tools
yum -y install tree
yum -y install lrzsz

######## 安装docker ##########
yum search docker | grep ^docker-engine
if [ $? -eq 0 ]; then
	yum -y install docker-engine
	docker --version
	isSucceed "docker"
else
	yum search docker | grep ^docker-ce.x86_64
	if [ $? -eq 0 ]; then
		yum -y install docker-ce.x86_64
		docker --version
		isSucceed "docker"
    else
        yum search docker | grep ^docker.x86_64
        if [ $? -eq 0 ]; then
            yum -y install docker.x86_64
            docker --version
            isSucceed "docker"
        else
		    read -p "yum源没有找到docker,无法安装 按回车继续脚本"
        fi
	fi
fi

######## 配置docker
restartDocker
sleep 1
ls -l /etc/docker/ | grep daemon.json
if [ $? -eq 0 ]; then
	editDaemonJson 
else
	touch /etc/docker/daemon.json
	editDaemonJson
fi

######## 配置docker-compose
mv ./docker-compose /usr/local/bin/
chmod +x /usr/local/bin/docker-compose
docker-compose --version

isSucceed "docker-compose"

mkdir $dockerComposeDir
touch $dockerComposeYml

source ./docker-compose.sh

######## 配置iptables ##########

# 关闭firewalld
systemctl stop firewalld
systemctl disable firewalld

#安装iptables
yum install -y iptables-services
iptables --version
isSucceed "iptables"
systemctl enable iptables

iptables -F
iptables -X
iptables -Z
iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -p icmp --icmp-type 8 -j ACCEPT
iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
for port in ${openPortArr[@]}; do
    iptables -A INPUT -p tcp --dport $port -j ACCEPT
done
iptables -P INPUT DROP
iptables -P OUTPUT ACCEPT
iptables -P FORWARD DROP

iptables -L -n | grep $nginxPort
isSucceed "iptables规则"

pkill docker
restartDocker 

########## 运行镜像 ##########
docker-compose -f $dockerComposeYml up -d

#修改nginx配置文件
mv ./nginx.conf /home/nginx/conf.d
sed -i "s/127.0.0.1/$ipAddr/g" /home/nginx/conf.d/nginx.conf

#保存iptables规则
service iptables save

# 重启电脑
reboot
