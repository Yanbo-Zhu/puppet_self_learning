
单机模型中 只需要安装 puppet
yum install -y puppet

安装完成过后，我们可以通过rpm -ql puppet | less来查看一下包中都有一些什么文件。
　　其中主配置文件为/etc/puppet/puppet.conf，使用的主程序为/usr/bin/puppet。