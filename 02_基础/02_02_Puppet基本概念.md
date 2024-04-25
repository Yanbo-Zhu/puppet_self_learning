
# 1 什么是 Puppet

puppet是一个IT基础设施自动化管理工具，帮助系统管理员管理基础设施的整个生命周期：provisioning（供应）、configuration（配置）、orchestration（联动）及reporting（报告)

puppet是什么？
是puppet labs 基于 ruby 语言开发的自动化系统配置工具。 是c/s 模式或者牡蛎模式运行 
puppet是一种Linux、Unix、Windows平台的集中配置管理系统，使用自有的puppet描述语言，可管理配置文件、用户、cron任务、软件包、系统服务等
puppet将自己所管理的系统实体（文件、用户、软件包等）称之为资源
puppet是开源的可以免费使用的，基于ruby语言开发
Puppet 是一個集合所有 Configure 的工具，能夠協助你了解 Infrastructure 的內容與架構，以及在傳統 Data Center 中所有的實體設備進行組態設定，包含虛擬化、和雲的基礎設施，甚至能夠管理你的 Container。

puppet的作用和目的是什么？
作用：是实现资源的自动化批量部署配置管理，最常用的是集群服务和配置文件的管理
目的：提高运维人员的工作效率，降低了运维人员的工作难度



|   |   |
|---|---|
|特性|自動化的可靠性、重複性並且可以輕鬆的複製設定到多台機器。<br><br>你可以用 Git 來管理 Puppet 來做到版本控管及透明化。<br><br>你可以將所有的 Configuration 都放在 Puppet Server 上，若是遇到相同的設定可以直接分配設定檔給所有的 Server，只要修改對應的參數就好了。|
|Pupet 是基于主机名来查找的|不是基于 ip 地址来查找的。<br><br>所以一定要 在 hosts 这个 file 中写入正确的对应关系|
|Puppet端口|Master 端 ： puppet server 占用 8140 这个端口<br><br>Client 端： puppet agent 占用 8139 这个端口|

# 2 常用工具比较

## 2.1 Puppet vs. Ansible

Ansible靠模块实现，而puppet靠的是资源实现；puppet的模块类似于Ansible的角色roles； 定义的模块是为了复用，不是为了管控；

"ssh in a for loop is not a solution" – Luke Kanies, Puppet developer
最明顯的差別就是 Ansible 採用的是 ssh 的方式來 push，而 Puppet 是依靠 agent 來 pull。
而官方強調 Ansible 是 by task 類型的工具，適合完成單一性任務，Puppet 則強調在於 持續管理 的概念並且能有效的 scale 架構，比起 Ansible，Puppet 針對大型架構也能有效的管理。

## 2.2 Puppet vs. Chef

Puppet 和 Chef 彼此相似，在選擇上開發者多數喜愛 Chef，而系統人員偏愛 Puppet / Ansible，這是由於使用 Chef 需要比較多的 Ruby 經驗，開發者容易上手，系統人員反之。
Puppet 和 Chef 之間存在著微妙的關係，有許多企業會同時使用 Puppet 和 Chef 彌補彼此的缺點。

# 3 Day 4 - 你不能不知道的 Puppet 小常識

本系列文資料可參考以下：

- [Github](https://github.com/shazi7804/ops-puppet-30-days)
- [Gitbook](https://gitbook.com/book/shazi7804/puppet-manage-guide/details)
- [Mr.沙先生](https://shazi.info)

---

這篇要來帶大家認識在使用 Puppet 之前，你不可不知的小常識

## 3.1 各種 Puppet 名詞

|   |   |
|---|---|
|资源|是puppet的核心，通过资源申报，定义在资源清单中。相当于ansible中的模块，只是抽象的更加彻底。|
|类|一组资源清单<br><br>资源组成类，类封装起来形成模块。|
|模块|包含多个类。相当于ansible中的角色。|
|站点清单|以主机为核心，应用哪些模块。|


Ansible靠模块实现，而puppet靠的是资源实现；puppet的模块类似于Ansible的角色roles； 定义的模块是为了复用，不是为了管控；
定义模块的文件叫资源清单（manifest）；
为每个站点主机定义具体使用哪个模块的叫站点清单（sitemanifest）；

### 3.1.1 Domain

第一件事情就是要講 Domain，Domain 在 Puppet 的世界非常重要，**所有 Node 都必須擁有自己的 Domain**，由於 Puppet 和 Node 之間的 trust 是使用 CA 憑證，所以跟 Node 之間會換發憑證，這個動作就需要 Domain 的存在，如果 Master / Node 更換 Domain 那麼你的憑證就會信任失敗，這時所有的 Deploy 都將失敗。

---

### 3.1.2 [Catalogs](https://puppet.com/docs/puppet/5.3/subsystem_catalog_compilation.html)

==a manifest compiles to the catalog (a manifest file can compile to the catalog)==

==A catalog is a document that describes the desired state for each resource that Puppet manages on a node. ==
==A primary server typically compiles a catalog from manifests of Puppet code.==
The catalog is the aggregation of the desired state of all the resources managed on the node. The responsibility for compiling the catalog depends on Puppet's operating mode: In a server-agent operating mode it is the responsibility of the Puppet server to compile the catalog for the agent.

Puppet 會把 Node 所需要的設定封裝成 Catalogs，再由 Node 將 Catalogs 解析後 Deploy，Catalogs 又分為兩個階段來 Deploy catalogs：

- Compile catalog
- Apply catalog

在 Puppet Master 時會將程式碼編譯成 Compile catalog，由 Node 將 Catalogs apply 進行佈署，這樣可以保證在傳輸過程中即使被擷取也僅是被編譯過的資料。


### 3.1.7 [Manifest](https://puppet.com/docs/puppet/5.3/lang_summary.html)

==A manifest is a file containing Puppet configuration language that describes how resources should be configured. ==
The manifest is the closest thing to what one might consider a Puppet program. It declares resources that define state to be enforced on a node 

==a manifest compiles to the catalog (a manifest file can compile to the catalog)==

Manifest 是 Puppet 的倉庫，所有的 Resource 都會在 manifest 裡面去定義，就好比 Ansible 的 playbook。

### 3.1.3 [Node](https://puppet.com/docs/puppet/5.3/lang_node_definitions.html)

Node 通稱為**被佈署的節點**，即是 catalog apply 的最終目標。

---

### 3.1.4 [Resource](https://puppet.com/docs/puppet/5.3/type.html)

身為一個 Puppet 的 Developer 你不能不知道 Resource，Resource 是用來定義**系統資源**的基本元件，例如 file、service、package .. 等等這類的東西。

---

### 3.1.5 [Hiera](https://puppet.com/docs/puppet/5.3/hiera_intro.html)

Hiera 是 Puppet 內建的數據查找系統，透過 Three layout 架構並且實現 `defaults, with overrides` 在不同的環境給予不同的參數，讓 Node 可以找到適合自己的值 (例如：在 Dev 時取到的參數和 Production 的不同)

如果你會寫 Resource 而不會 Hiera，那麼我大概會稱你是 **會用 Puppet**，但如果你也會 Hiera 的話那麼我會說 **Puppet 用的不錯**

---

### 3.1.6 [Facter](https://puppet.com/docs/puppet/5.3/lang_facts_and_builtin_vars.html)

Facter 是 Puppet 的小幫手，Facter 會隨著 Puppet agent 安裝在 Node 裡面，並且協助 Puppet 收集 Node 的系統資源。

---


## 3.2 Puppet 的 3, 4, 5 版本

在目前 Internet 上能找到的資料大概就是 3-5 這幾個版本，但對於這些版本的差異也必須要有認知，否則你在參考網路上的資訊時就會浪費非常多寶貴的時間。

首先各版本最大的差異就是 Ruby 支援的版本。
 - `Puppet 3`：Ruby 1.x
 - `Puppet 4`：Ruby 2.x (Puppet Collections)
 - `Puppet 5`：Ruby 2.4 (Performance)

在這幾個版本中主要的差異還是在於 3 和 4，在 `Puppet 4` 中針對整體架構大幅更新並且捨棄相當多 `Puppet 3` 的要素，也重構命名為 PC1 版本 (Puppet Collections)，而 Puppet 4 和 5 差異則不太大，`Puppet 5` 針對翻新後的 Puppet 4 提昇 Performance 以及過去的 Bug，並且在 Puppet 5 開始聆聽使用者的需求新增了許多 features。

簡單提了幾個在學習 Puppet 的過程中必須要知道的知識，希望藉由這 30 天把經驗分享出來，讓大家早早下班！！



