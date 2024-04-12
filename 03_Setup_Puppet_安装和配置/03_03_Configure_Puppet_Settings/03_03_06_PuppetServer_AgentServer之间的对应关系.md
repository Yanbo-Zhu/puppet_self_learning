

# 1 PuppetServer_AgentServer之间的对应关系

## 1.1 从 puppet master server 上 查看, 那些 aws umgebung 是他早的, 那些 server 和他有关联

1 /etc/puppetlabs/puppet/puppet.conf  中填入一行

server=e31-madpp-s01.ivu-cloud.local

不填入这行 不行 ,  puppetserver ca list --all 运行不了

![[03_Setup_Puppet_安装和配置/03_01_安装/images/Pasted image 20240412165618.png]]

2 运行 puppet master server 上面 运行 puppetserver ca list --all

![[03_Setup_Puppet_安装和配置/03_01_安装/images/Pasted image 20240412165627.png]]

## 1.2 从 装有 puppet agent 的 server 上常看,  他对应的 puppet master server 

1 linux 机器中 
进入 /etc/puppetlabs/puppet/puppet.conf 

![[03_Setup_Puppet_安装和配置/03_01_安装/images/Pasted image 20240412165647.png]]

2 windows server 中
`C:\ProgramData\PuppetLabs\puppet\etc\puppet.conf`


## 1.3 Puppet agent server 上面 切换 主 puppet master server 

To connect IVU.plan - systems to a monitoring / puppet server, do the following on each of its hosts:
	• Stop the puppet agent
		○ Windows: sc stop puppet
		○ Linux: systemctl stop puppet
	• Delete the SSL directory on the puppet agent
		○ Windows: Delete the C:\ProgramData\puppetlabs\puppet\etc\ssl
		○ Linux: rm -rf /etc/puppetlabs/puppet/ssl
	• Edit the puppet.conf file to connect to the Monitoring / Puppet Server. You just need to change the entry for 'server' to point to the FQDN of the Monitoring / Puppet Test server. Comment out all other lines. 
		○ On Windows systems, the file is here: C:\ProgramData\puppetlabs\puppet\etc\puppet.conf
		○ On Linux systems, the file is here: /etc/puppetlabs/puppet/puppet.conf
	• Start the puppet agent
		○ Windows: sc start puppet
Linux: systemctl start puppet




