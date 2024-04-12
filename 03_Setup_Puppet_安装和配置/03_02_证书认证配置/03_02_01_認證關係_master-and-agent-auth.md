

# 1 总览

犹豫Puppet 客户端与服务器端之间是通过 SSL隧道通信的, 客户端安装完成后, 需要将服务器端申请证书 


Agent 通过标准的 SSL 加密认证的方式与 Master 建立连接，获取本机需要的配置信息。
puppet的agent和master之间的数据传输是通过SSL证书认证完成的，具有服务端身份验证和数据传输加密功能

在 Agent 未取得配置信息、或者已经达到配置状态时，Puppet 不会对系统进行改动，它只有在被要求的时候才修改系统，这是 Puppet 的一个关键特征，称为幂等性（Idempotency），这个修改过程称为一次配置运行（Configuration run）。

![](03_Setup_Puppet_安装和配置/03_02_证书认证配置/image/Pasted%20image%2020231215211441.png)


![](03_Setup_Puppet_安装和配置/03_02_证书认证配置/image/Pasted%20image%2020231215211445.png)


# 2 Day 7 - MASTER 和 AGENT 的認證關係

本系列文資料可參考以下：

- [Github](https://github.com/shazi7804/ops-puppet-30-days)
- [Gitbook](https://gitbook.com/book/shazi7804/puppet-manage-guide/details)
- [Mr.沙先生](https://shazi.info)

---

Day 7 ... 連假日都要被折磨 Orz .. 

CA : Certificate Authority

前面兩天把 Master 和 Agent 都安裝好之後，來講一下 Master 和 Agent 之間的認證，像 Puppet 這類型的工具管理著整個 infrastructure 這麼重責大任的事情就交付給一個工具你不擔心嗎？

先別太擔心，Puppet Master 和 Agent 是使用 CA 憑證來產生 SSL Cert 進行溝通，除此之外你還可以自定義認證的規則，來做到自動 or 非自動甚至更複雜的規則來簽署憑證，那麼就來認識一下 Puppet 的 Trust 機制。

```
               +------------------------+
               |                        |
               |  Root self-signed CA   |
               |                        |
               +------+----------+------+
                      |          |
           +----------+          +------------+
           |                                  |
           v                                  v
  +-----------------+                +----------------+
  |                 |                |                |
  | Server SSL Cert |                | Agent SSL Cert |
  |                 |                |                |
  +-----------------+                +----------------+
```

CA 憑證可以是由外部頒發，或是自簽 CA，但通常會直接讓 Puppet 自簽 Internal CA，如果你用的是 Internet CA 則必須停用 Internal CA。

整個認證的交易就和我們一般 HTTP 的 SSL 申請狀況相同，Agent 就像是一般企業客戶端，而 Master 就是 CA 憑證中心。

![puppet-master-slave-communication](03_Setup_Puppet_安装和配置/03_02_证书认证配置/images/puppet-master-slave-communication.jpg)

1. 由 Agent 向 Master 發起 憑證簽署需求 (CSR  Certification Sign Request )
    1.  Agent send an CSR to Master
2. 由 Master 依照 Agent 提供的 CSR, 簽署一份 SSL 憑證 (Sign SSL Certification) 提交給 Agent
    1. According to the CSR from Agent, Master sign a SSL Certification and send it back to Agent
3. 由 Master 起的 CA 憑證 底下皆 信任所有簽署過的 CSR
    1. the Certificate Authority in Puppet Master, trust all signed CSR from itself   
4. Master 的 CA 憑證並不需要外部憑證信任，使用 local CA 信任
5. 整個的 SSL 的交易過程就像是我們向憑證中心 ( Certificate Authority ) 交易的過程，只不過 HTTPS 是 trust internet，而 Puppet 是 trust local。


然而 Agent 交付給 Master 需要經過 Master 的同意，簽署的方式分為：

- 手動簽署 (Signing) 
- 自動簽署 (Naive Autosigning) 
- 白名單簽署 (Basic Autosigning)
- 依照政策的簽署 (Policy-based Autosigning)
- 透過 API 簽署 (Policy executable API)

## 2.1 自動簽署 (Naive Autosigning)

在自動簽署中是最不安全的簽署方式，當使用 Naive Autosigning 的話，所有的 Agent CSR 要求皆會自動同意簽署，這並不適合用在正式環境，但相對的用於測試環境是非常方便。

設定自動簽署你只需要在 puppet.conf 的 [master] 中加入 autosign = true

```shell
$ vim /etc/puppetlabs/puppet/puppet.conf

[master]
autosign = true
```

## 2.2 白名單簽署 (Basic Autosigning)

Basic Autosigning 是繼承 Naive Autosigning 的概念在加上白名單限制，你可以針對 Agent 所申請的 domain 進行白名單限制，例如 *.example.com，你可以用 autosign.conf 將這些清單寫入

```shell
$ vim /etc/puppetlabs/puppet/puppet.conf
 
[master]
autosign = /etc/puppetlabs/puppet/autosign.conf
 
$ vim /etc/puppetlabs/puppet/autosign.conf
 
*.example.com
ami.puppet.com
```

## 2.3 依照政策的簽署 (Policy-based autosigning)

在 Policy-based autosigning 中你可以定義 Policy 簽署的條件，在 Agent 發出的 csr 請求中動手腳，加入一些可以提供驗證的資訊 (embedding additional data)，如 Password, token … etc，再由 Master 觸發 autosign 所執行的 script 進行驗證，這個 scirpt 不限語言，只要 return 給 Puppet 0 or 1 就行。

Puppet 官方極力推薦使用 Policy-based autosigning 這種方式進行驗證，是目前最安全且彈性的作法，甚至可以把 Policy-based autosigning 當成一個 trigger 去做許多事件。

## 2.4 透過 API 簽署  (Policy executable API)

延伸 Policy-based autosigning 概念的作法，你也可以透過 API 進行簽署認證，這個可以應用到更多的地方，但目前小弟還沒有實際應用。




### 2.4.1 Reference
- [SSL configuration: autosigning certificate requests](https://docs.puppet.com/puppet/5.3/config_ssl_external_ca.html)


# 3 具体认证流程 

## 3.1 客户端 向服务器端申请 证书 
![[03_Setup_Puppet_安装和配置/03_02_证书认证配置/images/Pasted image 20240412190909.png]]

--test 
有绿色的信息证明 通信成功 
![[03_Setup_Puppet_安装和配置/03_02_证书认证配置/images/Pasted image 20240412190917.png]]


## 3.2 自动认证/自动颁发证书

![[03_Setup_Puppet_安装和配置/03_02_证书认证配置/images/Pasted image 20240412190952.png]]

在服务端开启自动认证，自动出则证书  服务器端就不必再分发证书了
puppet.conf 中  Autosign = true 
![[03_Setup_Puppet_安装和配置/03_02_证书认证配置/images/Pasted image 20240412191032.png]]

![[03_Setup_Puppet_安装和配置/03_02_证书认证配置/images/Pasted image 20240412191039.png]]

![[03_Setup_Puppet_安装和配置/03_02_证书认证配置/images/Pasted image 20240412191044.png]]

现在两个端都删除已经有的证书 然后重新生成证书

![[03_Setup_Puppet_安装和配置/03_02_证书认证配置/images/Pasted image 20240412191059.png]]

新版本中 删除客户端的证书 puppet cert --clean 主机名
![[03_Setup_Puppet_安装和配置/03_02_证书认证配置/images/Pasted image 20240412191115.png]]



## 3.3 自动同步


### 3.3.1 服务器端自动推送 

![[03_Setup_Puppet_安装和配置/03_02_证书认证配置/images/Pasted image 20240412191328.png]]

![[03_Setup_Puppet_安装和配置/03_02_证书认证配置/images/Pasted image 20240412191335.png]]

![[03_Setup_Puppet_安装和配置/03_02_证书认证配置/images/Pasted image 20240412191340.png]]

![[03_Setup_Puppet_安装和配置/03_02_证书认证配置/images/Pasted image 20240412191345.png]]

![[03_Setup_Puppet_安装和配置/03_02_证书认证配置/images/Pasted image 20240412191350.png]]


![[03_Setup_Puppet_安装和配置/03_02_证书认证配置/images/Pasted image 20240412191355.png]]


在客户端执行 puppet kick -d hostnameofmachineWithPuppetAgent
在客户端 执行某个命令 ， 从而实行 对某个 node 的配置和部署 
![[03_Setup_Puppet_安装和配置/03_02_证书认证配置/images/Pasted image 20240412191410.png]]

### 3.3.2 客户端自动同步

![[03_Setup_Puppet_安装和配置/03_02_证书认证配置/images/Pasted image 20240412191423.png]]

Etc/sysconfig/puppet 
![[03_Setup_Puppet_安装和配置/03_02_证书认证配置/images/Pasted image 20240412191432.png]]

系统配置文件
只要把这些 都激活后， 不在commit。 之后 只要 agent 被启动。 就会立即和master 端 同步。然后之后， 每 30s 和 master node 同步一次 


## 3.4 initial cert 的初步配置 


run puppet agent --test on the agent to generate (and send) the initial cert request? That should put the agent in the certificate request list of your master.

If the agent's just complaining about not finding a cert then quitting, it may be thinking that it's already sent a request

方法: 
	•  just reset its memory as far as SSL is concerned by backing up 
	• then nuking (用核武器攻击， 就是删除 ) the configured puppet SSL directory (by default, /var/lib/puppet/ssl or /etc/puppetlabs/puppet/ssl), 
	• then running puppet agent --test (with --debug and --verbose if you want to make really sure) - this run should output that it's generating a new cert request, and it should be sent to the configured master.



To fix this, remove the CSR from both the master and the agent and then start a puppet run, which will automatically regenerate a CSR.

On the master:
  puppet cert clean ip-172-31-27-12.us-west-2.compute.internal

On the agent:
  1a. On most platforms: find /home/ubuntu/.puppetlabs/etc/puppet/ssl -name ip-172-31-27-12.us-west-2.compute.internal.pem -delete
  1b. On Windows: del "\home\ubuntu\.puppetlabs\etc\puppet\ssl\certs\ip-172-31-27-12.us-west-2.compute.internal.pem" /f
  2. puppet agent -t


解释上面 
1 Clear the nodes SSL folder in node machine. Not in puppet server 
	rm -rf /etc/puppetlabs/puppet/ssl
Note: This deletes all puppet SSL certificates, CSR, etc. So only do it on the node, not on the server.

2.1 puppet cert list --all 查看 有那些证书 in puppetmaster

2.2  Then, remove the CSR from the puppetmaster:
	sudo /opt/puppetlabs/bin/puppet cert clean `<hostname>`

3 Afterwards, do another puppet run on the node: 
	sudo /opt/puppetlabs/bin/puppet agent -tv

4   This will generate a new CSR.
