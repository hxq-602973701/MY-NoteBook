* 新linux系统上rz 与sz命令不可用?
  * 使用命令进行安装：yum install lrzsz  即可<br><br>
  
* 查看端口情况
  * netstat   -nultp(所有已使用端口)
  * netstat  -anp  |grep   端口号(查看某个端口是否被占用)
  
* [linux上安装java](http://www.cnblogs.com/xuliangxing/p/7066913.html)
* [linux上安装es](https://www.jianshu.com/p/975326e65f65)
  * 查看es是否启动：ps -aux| grep elasticsearch
  * 停止es：kill -9 进程号
  * 后台启动es：./elasticsearch -d
  * 安装Kibana 后启动方式 nohup  ./kibana > /nohub.out &  （可关闭终端，在nohup.out中查看log）
