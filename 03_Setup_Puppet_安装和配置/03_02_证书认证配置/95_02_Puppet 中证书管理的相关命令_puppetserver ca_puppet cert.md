

# 1 puppetserver ca
https://puppet.com/docs/puppet/7/puppet_server_ca_cli.html
https://puppet.com/docs/puppet/7/server/subcommands.html


Puppet Server has a puppetserver ca command that performs certificate authority (CA) tasks like signing and revoking certificates. Most of its actions are performed by making HTTP requests to Puppet Server’s CA API, specifically the certificate_status endpoint. You must have Puppet Server running in order to sign or revoke certificates.

CA subcommands: 
If you have yet to review the actions for the puppetserver ca command, visit Subcommands. Some actions have additional options. Run puppetserver ca help for details.
The puppetserver-ca CLI tool is shipped as a gem alongside Puppet Server. You can update the gem between releases for bug fixes and improvements. To update the gem, run:
/opt/puppetlabs/puppet/bin/gem install -i /opt/puppetlabs/puppet/lib/ruby/vendor_gems puppetserver-ca


Syntax
`puppetserver ca <action> [options]`

## 1.1 Action

- clean: clean files from the CA for certificates
- generate: create a new certificate signed by the CA
- setup: generate a root and intermediate signing CA for Puppet Server
- import: import the CA's key, certs, and CRLs
- list: list all certificate requests
- migrate: migrate the contents of the CA directory from its current location to /etc/puppetlabs/puppetserver/ca. Adds a symlink at the old location for backwards compatibility.
- revoke: revoke a given certificate
- sign: sign a given certificate

- Use the --ttl flag with sign subcommand to send the ttl to the CA. The signed certificate's notAfter value is the current time plus the ttl. The values are valid puppet.conf ttl values, for example, 1y = 1 year, 31d = 31 days.

- purge: remove duplicate entries from the CA CRL

Important: Most of these actions only work if the puppetserver service is running. Exceptions to this requirement are:
- migrate and purge, which require you to stop the puppetserver service.
- setup and import, which require you to run the actions only once before you start your puppetserver service, for the very first time.


## 1.2 Option 


 --certname	Most commands require a target to be specified with the --certname flag. 
	For example:
	        puppetserver ca sign --certname cert.example.com
	The target is a comma separated list of names that act on multiple certificates at one time.

--config	You can supply a custom configuration file to all subcommands using the --config option. This allows you to point the command at a custom puppet.conf, instead of the default one.

## 1.3 例子

puppetserver ca list
puppetserver ca sign --all

# 2 puppet cert (只存在于 puppet 5 的命令， 67就没有这个了)

https://puppet.com/docs/puppet/5.5/man/cert.html

puppet cert list --all	查看正在申请 审批的证书 , 有可能为空 
puppet cert clean `<hostname>`	puppet cert clean ip-172-31-27-12.us-west-2.compute.internal

Puppet cert -s: 给 agent 端/ 某个客户端 颁发证书
puppet cert --clean 主机名: 删除 对某个 主机 颁发的证书 

![[03_Setup_Puppet_安装和配置/03_02_证书认证配置/images/Pasted image 20240412190632.png]]


