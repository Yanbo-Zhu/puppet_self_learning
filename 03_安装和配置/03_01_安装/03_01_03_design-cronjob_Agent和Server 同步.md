

# 1 Day 24 - 設計基於 cronjob 的分散式概念

本系列文資料可參考以下：

- [Github](https://github.com/shazi7804/ops-puppet-30-days)
- [Gitbook](https://gitbook.com/book/shazi7804/puppet-manage-guide/details)
- [Mr.沙先生](https://shazi.info)

再來要來談的是關於 Puppet Server 與 Agent 的設計，今天講的是 Agent 要怎麼向 Server 同步 !!

蝦米? 阿 Puppet Agent 起來不是就會自動向 Server 同步了嗎 ?

但是事情不是憨人所想的這麼簡單啦，先來看看 default agent 的設定

## 1.1 runinterval - 預設 agent 的同步模式

`runinterval` 這個參數在 puppet.conf 裡面，主要作用是會定時向 Puppet Server 更新。

這樣有錯嗎?

如果你只有 10 台以下的 agent 這樣弄可能還好，但是當數量是 10 倍、100 倍的時候就會有所差異，讓我們先看圖

![puppet-daemon-default](https://github.com/shazi7804/ops-puppet-30-days/raw/master/images/puppet-daemon-default.png)

這張圖我稱為 `agent 的逆襲` 啊 !!

因為 agent 設定 `runinterval = 30`，所以每 30 分鐘向 Server 更新，但是每一台 agent 都是相同的時間去向 Server 更新，但 `00:01 ~ 00:29` 這段時間 Server 是一個邊緣人的狀態，而 `00:30` 一到就被所有 agent DDos。

這個問題在所有 Server / Agent 的架構都會有，如果設計不良就會導致這樣的狀況。


## 1.2 基於 random 的 cronjob 設計

cronjob 的設計主要依靠 puppet 的 [fqdn_rand][puppet-fqdn_rand] 和 sleep 來做到

fqdn_rand 這個 function 是直接拿 puppet agent domain 來做 hash 產生出來的隨機數字，所以 puppet code 可以這樣寫：

```puppet
  cron { 'puppet-agent-reboot':
    ensure  => 'present',
    command => '/opt/puppetlabs/bin/puppet agent -t -l /var/log/puppetlabs/puppet/puppet.log',
    special => 'reboot',
    target  => 'root',
    user    => 'root',
  }
  $random_sleep = fqdn_rand(60)
  cron { 'puppet-agent':
    ensure  => 'present',
    command => "sleep ${random_sleep}; /opt/puppetlabs/bin/puppet agent -t -l /var/log/puppetlabs/puppet/puppet.log",
    minute  => [fqdn_rand(30),fqdn_rand(30)+30],
    target  => 'root',
    user    => 'root',
  }
```

這樣實際在 cronjob 會像這樣：

```
# Puppet Name: puppet-agent-reboot
@reboot /opt/puppetlabs/bin/puppet agent -t -l /var/log/puppetlabs/puppet/puppet.log
# Puppet Name: puppet-agent
5,35 * * * * sleep 10; /opt/puppetlabs/bin/puppet agent -t -l /var/log/puppetlabs/puppet/puppet.log
```

這樣的設計把執行的 `分` 和 `秒` 都隨機了，實務上你會發現這樣的作法對於 Puppet Server 非常有效的分散 Loading。

---

在早期還沒有 fqdn_rand 的時候，是直接自幹一段 md5 hash hostname 的方式

> \# Ref https://letsencrypt.tw/
> sleep $(expr $(printf "\%d" "0x$(hostname | md5sum | cut -c 1-8)") \% 86400)

這個方法有一個缺陷，如果不去改 hostname 的話就會大家都產生一樣的同步時間 ... 但 domain 在 internet 上則是相同。


## 1.3 後期加入的 splay 和 splaylimit

這是在 Puppet 後期在 agent 加入的參數 [splay][puppet-splay] 和 [splaylimit][puppet-splaylimit] 可以幫助你在啟動 agent daemon 的時候產生隨機的 sleep 來達到分散同步的效果。

如果 splay 設為 true，則在每次執行 catalog 的時候會隨機 sleep 一段時間再執行，而 splay 的最大秒數取決於 splaylimit 設定的值。

## 1.4 總結

目前小弟在公司是使用 cronjob 的設計，原因是這種設計你可以在 cronjob 看到實際向 Server 更新的時間，而不是選用 splay 這樣由 agent 產生的數字 (只能從 log 看到什麼時候更新)。

另外 cronjob 的設計可以在許多 Server / Agent 架構的工具看到相同的用法，例如 AWS 的 CodeDeploy Agent：

```
# Cron that starts once every hour or on reboot, which updates codedeploy-agent.
# This cron file has been fuzzed to run after 08:03:52 every hour.
SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
3 8 * * * root /bin/sleep 52; /opt/codedeploy-agent/bin/update --sanity-check --upgrade deb
33 * * * * root /bin/sleep 52; /opt/codedeploy-agent/bin/update --sanity-check --downgrade deb
@reboot root /bin/sleep 45; /opt/codedeploy-agent/bin/update --sanity-check deb
```

有時候參考較大的企業作法會找到許多技巧在裡面。

[puppet-fqdn_rand]: https://puppet.com/docs/puppet/5.3/function.html#fqdnrand
[puppet-splay]: https://docs.puppet.com/puppet/latest/configuration.html#splay
[puppet-splaylimit]: https://docs.puppet.com/puppet/latest/configuration.html#splaylimit


