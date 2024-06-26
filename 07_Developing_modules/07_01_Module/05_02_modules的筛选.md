

# 1 Day 10 - modules 的来源

本系列文資料可參考以下：

- [Github](https://github.com/shazi7804/ops-puppet-30-days)
- [Gitbook](https://gitbook.com/book/shazi7804/puppet-manage-guide/details)
- [Mr.沙先生](https://shazi.info)

---

除了 Day 9 我們自己寫的 module 以外，還可以直接用別人已經寫好的 module 當你開始會用 module 的時候，就會發現其實 Puppet 的 module 型態幾乎無所不在，利用 module 你可以輕鬆的兜出 infrastructure，但是你在使用別人 module 的時候 **篩選就是非常重要的議題** 用到對的 module 讓你上天堂，天天準時下班，用到錯的 module 你會恨到乾脆自己來寫 module。

今天這篇就來告訴大家該怎麼篩選 module，減少採雷的機會。


自己写 或者 外部获取

|   |   |
|---|---|
|從 Puppetforge 獲取 module|[https://forge.puppet.com/](https://forge.puppet.com/)<br><br>Modul withs tasks<br><br>[https://forge.puppet.com/modules?with_tasks=true](https://forge.puppet.com/modules?with_tasks=true)|
|Puppet 官方維護的 Puppetlabs|Puppetlabs 是官方提供的模組，不見得會是最新，但是會是最穩定的 module，基本上沒有太多問題，而且也沒有 EOL 的問題，而且 Test case 也算是最完整的。<br><br>[https://github.com/puppetlabs/](https://github.com/puppetlabs/)|
|Github 維護的 voxpupuli|[voxpupuli](https://github.com/voxpupuli) 是由 Github 內部維護的 module，也算是穩定中可選的 module，有許多由 Puppetlabs 官方維護的 module 會直接移轉給 voxpupuli 進行維護，例如 [puppet-nginx](https://github.com/voxpupuli/puppet-nginx)，算是除了 Puppetlabs 的第三方授權。|
|由套件本身官方維護的 module|各種套件本身會維護自己的 Puppet module，例如 [Elastic](https://github.com/elastic) 就維護 puppet-elasticsearch、puppet-kibana、puppet-logstash|


### 1.1.1 從 Puppetforge 獲取 module

[Puppetforge][puppetforge] 是 Puppet 提供的一個 module 管理平台，你可以透過這個平台來找到各種你想要的 Puppet module。

### 1.1.2 從 Github 獲取 module

除了 Puppetforge 以外，最多資源的就是在 [Github][github]，就因為資源太多更需要慎選。

## 1.2 如何挑選 Module

由於 Puppet 已經算是很成熟的組態管理工具，所以 module 也是琳琅滿目，但是要怎麼去篩選適合的 module 就是一個很重要的課題。

### 1.2.1 從維護者挑選

- Puppet 官方維護的 Puppetlabs
[Puppetlabs][puppetlabs] 是官方提供的模組，不見得會是最新，但是會是最穩定的 module，基本上沒有太多問題，而且也沒有 EOL 的問題，而且 Test case 也算是最完整的。

- Github 維護的 voxpupuli
[voxpupuli][voxpupuli] 是由 Github 內部維護的 module，也算是穩定中可選的 module，有許多由 Puppetlabs 官方維護的 module 會直接移轉給 voxpupuli 進行維護，例如 [puppet-nginx][puppet-nginx]，算是除了 Puppetlabs 的第三方授權。

- 由套件本身官方維護的 module
各種套件本身會維護自己的 Puppet module，例如 [Elastic][elastic] 就維護 puppet-elasticsearch、puppet-kibana、puppet-logstash

### 1.2.2 適用自己環境

 - OS platform
 - Features
 - Puppet version

除了以上建議的 module 來源以外，除非這個 module 你有能力做後續的維護，否則都不建議使用。


[github]: https://github.com
[puppetforge]: https://forge.puppet.com/
[puppetlabs]: https://github.com/puppetlabs/
[voxpupuli]: https://github.com/voxpupuli
[puppet-nginx]: https://github.com/voxpupuli/puppet-nginx
[elastic]: https://github.com/elastic
