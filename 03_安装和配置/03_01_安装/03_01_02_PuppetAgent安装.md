
# 1 Day 6 - Master-Agent 架構 - Agent 安裝

本系列文資料可參考以下：

- [Github](https://github.com/shazi7804/ops-puppet-30-days)
- [Gitbook](https://gitbook.com/book/shazi7804/puppet-manage-guide/details)
- [Mr.沙先生](https://shazi.info)

---

繼 Day-5 寫完 Puppet Master / Server 之後，這篇要來跑 Agent 向 Server 更新。

目前主流大概是 RedHat / Debian 家族居多，所以這篇就來寫 Open Source 的 Ubuntu / CentOS OS 環境的安裝方式

---

## 1.1 LAB 環境

在一開始先定義好 Puppet Master 和 Agent 運行的環境：

**Master**

- Operating system: Ubuntu 16.04
- IP Address: 192.168.10.10
- Domain : master.puppet.com

**Agent**

Node.1 (Ubuntu)

- Operating system: Ubuntu 16.04
- IP Address: 192.168.10.11
- Domain : ubuntu.puppet.com

Node.2 (CentOS)

- Operating system: Ubuntu 16.04
- IP Address: 192.168.10.12
- Domain : centos.puppet.com

## 1.2 安裝 ( Agent server 为 Ubuntu Server )

1. Puppet agent 一樣要定義 Domain，測試可以先寫在 hosts

    ```shell
    $ cat /etc/hosts
    192.168.10.10 master.puppet.com
    192.168.10.11 ubuntu.puppet.com
    ```

1. Puppet agent 必須準確校時。

    ```shell
    $ sudo ntpdate time.stdtime.gov.tw
    $ sudo timedatectl set-timezone Asia/Taipei
    ```

1. 安裝 Puppet agent

    從官方 [apt repository][apt-repository] 取得 Puppet package。

    ```shell
    $ wget https://apt.puppetlabs.com/puppet5-release-xenial.deb
    $ sudo dpkg -i puppet5-release-xenial.deb
    $ sudo apt-get update
    $ sudo apt-get install puppet-agent
    ```

1. 修改 Puppet 的主要設定檔 [puppet.conf][puppet-conf]

    ```shell
    sudo vim /etc/puppetlabs/puppet/puppet.conf
    
    [main]
    certname = ubuntu.puppet.com
    server = master.puppet.com
    environment = production
    runinterval = 2h  
    ```
    
    這邊和 master 不同之處在於：
    - environment: 這台 node 是屬於哪個環境，所以 Puppet master 可以同時管理 dev/staging/production/pre-production 等環境。
    - runinterval: 當啟動 puppet daemon 時，會按照設定的時間定時和 master 更新 config，預設為 30m。

1. Puppet agent 產生 certificate

    ```shell
    $ sudo /opt/puppetlabs/bin/puppet agent --test
    ```
    
    這個動作會嘗試將 certificate 和 Master 進行 signin。
  
1. 在 Puppet master 中  signin ubuntu.puppet.com 這個 node，否則會無法取得 catalog。

    ```shell
    $ sudo /opt/puppetlabs/bin/puppet cert sign agent.puppet.com
    ```
    
1. 回到 Agent 再跑一次 `puppet agent -t` 來測試和 master 的溝通
    
    ```shell
    $ sudo /opt/puppetlabs/bin/puppet agent -t
    ...
    ...
    Info: Applying configuration version '1503680249'
    ```
    出現 Applying configuration version 代表能成功要到 catalog。

1. 啟動 puppet agent daemon 常駐。

    ```shell
    $ sudo systemctl start puppet
    $ sudo systemctl enable puppet
    ```

## 1.3 安裝 ( Agent Server 为 CentOS)

步驟和 Ubuntu 安裝大致上相同，但是有幾個設定必須修改

- step.1 中 hosts 修改為

    ```shell
    $ cat /etc/hosts
    
    192.168.10.10 master.puppet.com
    192.168.10.12 centos.puppet.com
    ```

- step.3 改為 [yum repository][yum-repository] 取得 Puppet package。
- step.4 設定的 `certname` 改為自己的 Domain `centos.puppet.com`。
- step.6 在這邊 Puppet Master 則要 sign in `centos.puppet.com`。

[yum-repository]: https://yum.puppetlabs.com/
[apt-repository]: https://apt.puppetlabs.com/
[puppet-conf]: https://docs.puppet.com/puppet/5.3/configuration.html

## 1.4 回顧

從這篇我們安裝了兩台 Node Agent，這兩台 Node 每 2h 會向 Puppet Master 更新設定，但這邊有個小小的陷阱 ... 嘿嘿 !!

之後再回來談這個 **陷阱** 該怎麼處理。

下集待續 ...




