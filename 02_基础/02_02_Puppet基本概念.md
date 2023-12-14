
# 1 What is Puppet 

![](image/Pasted%20image%2020231214164811.png)
# 2 Day 4 - 你不能不知道的 Puppet 小常識

本系列文資料可參考以下：

- [Github](https://github.com/shazi7804/ops-puppet-30-days)
- [Gitbook](https://gitbook.com/book/shazi7804/puppet-manage-guide/details)
- [Mr.沙先生](https://shazi.info)

---

這篇要來帶大家認識在使用 Puppet 之前，你不可不知的小常識

## 2.1 各種 Puppet 名詞

### 2.1.1 Domain

第一件事情就是要講 Domain，Domain 在 Puppet 的世界非常重要，**所有 Node 都必須擁有自己的 Domain**，由於 Puppet 和 Node 之間的 trust 是使用 CA 憑證，所以跟 Node 之間會換發憑證，這個動作就需要 Domain 的存在，如果 Master / Node 更換 Domain 那麼你的憑證就會信任失敗，這時所有的 Deploy 都將失敗。

---

### 2.1.2 [Catalogs](https://puppet.com/docs/puppet/5.3/subsystem_catalog_compilation.html)

Puppet 會把 Node 所需要的設定封裝成 Catalogs，再由 Node 將 Catalogs 解析後 Deploy，Catalogs 又分為兩個階段來 Deploy catalogs：

- Compile catalog
- Apply catalog

在 Puppet Master 時會將程式碼編譯成 Compile catalog，由 Node 將 Catalogs apply 進行佈署，這樣可以保證在傳輸過程中即使被擷取也僅是被編譯過的資料。

---

### 2.1.3 [Node](https://puppet.com/docs/puppet/5.3/lang_node_definitions.html)

Node 通稱為**被佈署的節點**，即是 catalog apply 的最終目標。

---

### 2.1.4 [Resource](https://puppet.com/docs/puppet/5.3/type.html)

身為一個 Puppet 的 Developer 你不能不知道 Resource，Resource 是用來定義**系統資源**的基本元件，例如 file、service、package .. 等等這類的東西。

---

### 2.1.5 [Hiera](https://puppet.com/docs/puppet/5.3/hiera_intro.html)

Hiera 是 Puppet 內建的數據查找系統，透過 Three layout 架構並且實現 `defaults, with overrides` 在不同的環境給予不同的參數，讓 Node 可以找到適合自己的值 (例如：在 Dev 時取到的參數和 Production 的不同)

如果你會寫 Resource 而不會 Hiera，那麼我大概會稱你是 **會用 Puppet**，但如果你也會 Hiera 的話那麼我會說 **Puppet 用的不錯**

---

### 2.1.6 [Facter](https://puppet.com/docs/puppet/5.3/lang_facts_and_builtin_vars.html)

Facter 是 Puppet 的小幫手，Facter 會隨著 Puppet agent 安裝在 Node 裡面，並且協助 Puppet 收集 Node 的系統資源。

---

### 2.1.7 [Manifest](https://puppet.com/docs/puppet/5.3/lang_summary.html)

Manifest 是 Puppet 的倉庫，所有的 Resource 都會在 manifest 裡面去定義，就好比 Ansible 的 playbook。

## 2.2 Puppet 的 3, 4, 5 版本

在目前 Internet 上能找到的資料大概就是 3-5 這幾個版本，但對於這些版本的差異也必須要有認知，否則你在參考網路上的資訊時就會浪費非常多寶貴的時間。

首先各版本最大的差異就是 Ruby 支援的版本。
 - `Puppet 3`：Ruby 1.x
 - `Puppet 4`：Ruby 2.x (Puppet Collections)
 - `Puppet 5`：Ruby 2.4 (Performance)

在這幾個版本中主要的差異還是在於 3 和 4，在 `Puppet 4` 中針對整體架構大幅更新並且捨棄相當多 `Puppet 3` 的要素，也重構命名為 PC1 版本 (Puppet Collections)，而 Puppet 4 和 5 差異則不太大，`Puppet 5` 針對翻新後的 Puppet 4 提昇 Performance 以及過去的 Bug，並且在 Puppet 5 開始聆聽使用者的需求新增了許多 features。

簡單提了幾個在學習 Puppet 的過程中必須要知道的知識，希望藉由這 30 天把經驗分享出來，讓大家早早下班！！



