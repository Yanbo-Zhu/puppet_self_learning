
 1 Day 3 - Puppet 的架構

本系列文資料可參考以下：

- [Github](https://github.com/shazi7804/ops-puppet-30-days)
- [Gitbook](https://gitbook.com/book/shazi7804/puppet-manage-guide/details)
- [Mr.沙先生](https://shazi.info)

---

前面兩天講了不怎麼技術的廢話之後，終於要在以下的 27 天會有比較"技術性"的語言了啊 ...

因為我習慣了解一個新事務時先從最簡單的開始看起 XDD，所以第三天我打算先來談談 Puppet 的整體架構，這樣看起來比較不容易放棄 ...


# 1 Puppet Master and Puppet Server

1 
Puppet Master 是由 Ruby 開發的 Application，Puppet Master 主要是用來跑整個 Puppet Code 編譯的核心，單單 Puppet Master 是無法運行的，還必須搭配 Server 當平台。

2 
Puppet Server 是由 Java 開發，運行在 JVM 的 Application 並且提供和 Puppet Master 相同功能的服務，採用 JRuby 可以直接運行 Puppet Code，所以在 Puppet Server 本身就已經內建 Puppet Master。

目前 Puppet Server 支援在任何 POSIX 平台下安裝，以下平台可以直接用套件管理工具安裝 (i.e. yum, apt)

RedHat
Debian
Fedora
Ubuntu
除此之外最低必須使用 JDK 1.7 以上才跑的動。

无法在 mircosoft 上面 跑 


# 2 Master / Agent

master/agent模型实现的是集中式管理，即 agent 端周期性向 master 端发起请求，请求自己需要的数据。然后在自己的机器上运行，并将结果返回给 master 端。

![](image/Pasted%20image%2020231214165137.png)

![puppetmaster-agent](../images/puppetmaster-agent.png)

基於 `pull-based` 的架構 (官方建議)

Puppet 的 Master / Agent 架構通常適用於有規模的環境佈署，由 Puppet Master 提供 configuration，在每台要被佈署的 Node 安裝 Puppet agent 向 Master 獲取 configuration。

Puppet Master 通常由 N + 1 台 Puppet Server 構成，必須要能夠承受所有的 Puppet agent 訪問，而 Puppet agent 通常透過 service 運行或是 cron 來定期向 Puppet Master 更新 catalog。

Agent 透過 catalog 更新完畢後，將 report 回傳給 Puppet Master。

## 2.1 构架

![](image/Pasted%20image%2020231215205312.png)

![](image/Pasted%20image%2020231215205317.png)

为什么寻找过需要dns 服务， node 后面写的是 hostname : master 需要通过 FQDN找到agent 端， 所以要依赖于dns 服务
![](image/Pasted%20image%2020231215205325.png)


## 2.2 工作原理

puppet的C/S模式工作原理

![](image/Pasted%20image%2020231215205402.png)

![](image/Pasted%20image%2020231215205407.png)



![puppet-catalog](06_01_Syntax_And_Setting_Puppet_Language/images/puppet-catalog.png)


![](06_01_Syntax_And_Setting_Puppet_Language/images/Pasted%20image%2020231211231801.png)


Manifest: 
在 Puppet master 上撰寫 manifests，而 manifests 是可以被即時生效，不需要 reload service。
![](image/Pasted%20image%2020231215210321.png)


Master 和 Agent 之間取得佈署清單流程：
1. Agent 索要 catalog:  Agent 內容包含 certname (節點名稱) 和 facts (由 facter  所獲取的系統參數)
1. Master 從 catalog 提供的資訊將 manifests 編譯重新打包 catalog。
1. Agent 收到 catalog 後執行佈署工作，並且回應 Report 給 Master 執行結果。

Master 和 Agent 提交的關係有：
- 由 Agent 設定的 runinterval 觸發更新
- MCollective 的 MQ (Message Queue) 更新
- 採用 random time 的方式觸發更新
- etc


Master端会检索给被管理的节点定义好的类，然后到模块中把需要执行的类给抽取出来编译成catalog发送给agent，agent再到本地应用一遍。
Puppet start and compile a catalog for your manifest.

![](image/Pasted%20image%2020231215210217.png)

包括如下步骤：
1. Agent 用facter 探测出 主机的一些信息， 发送给  puppet server
2. 解析 facter 发过来的信息，到 manifest里面找对应的node 配置。 编译配置
3. 将编译的后的配置发送到 Agent 上
4. 在 Agent 上应用编译后的配置。 获取从master 端获得的配置， 按照这个配置 完成对本机的配置
5. 将运行结果上报给 Master， 写入 master 段的日志

事物层是 Puppet 的工作引擎，一个 Puppet 事物包含配置一台 Agent 主机的完整过程，
包括如下步骤：
1. 解析并编译配置
2. 将编译的后的配置发送到 Agent 上
3. 在 Agent 上应用编译后的配置
4. 将运行结果上报给 Master
# 3 Masterless

实现定义多个manifests --> complier --> catalog --> apply
![](image/Pasted%20image%2020231215205239.png)

![puppet-apply](../images/puppet-apply.png)

基於 `push-based` 的架構

無 Master 又稱獨立佈署的架構，透過 `Puppet apply` 進行單機佈署的方式達成。

Masterless 很常被應用於 Docker or Vagrant 等 image 環境使用，通常適用於數量少的環境。

# 4 Tasks and Plans

![puppet-bolt](../images/puppet-bolt.png)

Puppet bolt tasks and plans 是 puppet 推出基於 `push-based` 的一次性的臨時佈署，與 Ansible 的方式相似，彌補 Puppet 長期以來使用 `pull-based` 必須等待 deploy time 的缺點。

可以使用 [Bolt](https://github.com/puppetlabs/bolt) 開源專案或是企業版的 Puppet Enterprise Task Management (available in Puppet Enterprise 2017.3) 來實現 Tasks and Plans。

# 5 不負責碎碎念

佈署架構上主要分為三種，每個佈署的方式都適應不同的場景，從能適應**超大型架構**的 Master / Agent 架構和常用在 Images 的 Masterless 架構，或是最近才剛出的 Tasks and Plans 用來解決臨時性的需求，Puppet 在這些階段都算是都有解決方案了。

經過時間的演變，Puppet 不再是以前那個老牌頑固不變的組態工具，從這幾年可以看到 Puppet 有聽到使用者的聲音而做了改變，現在變化的作法更多也更廣。

[puppet-tasks-and-plans]: https://puppet.com/blog/easily-automate-ad-hoc-work-new-puppet-tasks

