
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

启动 重启 puppet agent
 /etc/init.d/puppet restart  或者 start


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


## 1.4 windows 中安装 puppet agent 

安装包 
我下载的是 
puppet-agent-x64-latest.msi	2020-10-20 10:27:25	36.4MiB	
下载位置 C:\Users\yzh\Downloads\puppet-agent-x64-latest.msi 


安装时间 am 2022.09.07 安装的 
安装位置  C:\Program Files\Puppet Labs\Puppet\
![[03_Setup_Puppet_安装和配置/03_01_安装/images/Pasted image 20240412180514.png]]

Path 中会自动加入 
The installer adds Puppet commands to the PATH. After installing, you can run them from any command prompt (cmd.exe) or PowerShell prompt.
`C:\Program Files\Puppet Labs\Puppet\bin`


## 1.5 回顧

從這篇我們安裝了兩台 Node Agent，這兩台 Node 每 2h 會向 Puppet Master 更新設定，但這邊有個小小的陷阱 ... 嘿嘿 !!

之後再回來談這個 **陷阱** 該怎麼處理。

下集待續 ...


# 2 Configure agents

Once you have installed your agents, you must complete the following three configuration steps.

## 2.1 Configure your `PATH` to access Puppet commands

Puppet's command line interface (CLI) consists of a [single Puppet command with many subcommands](https://puppet.com/docs/puppet/5.5/services_commands), for example `puppet --help`.

Puppet commands are located in the bin directory — `/opt/puppetlabs/bin/` on *nix and `C:\Program Files\Puppet Labs\puppet\bin` on Windows. The bin directory is not in your `PATH` environment variable by default. To have access to the Puppet commands, you must add the bin directory to your `PATH`.

Choose from the following options.

### 2.1.1 Linux: source a script for puppet-agent to install

If you are on Linux, you can source a script that `puppet-agent` installs. Run the following command:

```
source /etc/profile.d/puppet-agent.sh
```

### 2.1.2 *nix: Add the Puppet labs bin directory to your **PATH**

To add the bin directory to your `PATH` on *nix, run:

```
export PATH=/opt/puppetlabs/bin:$PATH
```

Alternatively, you can add this location wherever you configure your `PATH`, such as your `.profile` or `.bashrc` configuration files.

### 2.1.3 Windows: Add the Puppet labs bin directory to your **PATH**

To run Puppet commands on `Windows`, start a command prompt with administrative privileges. You can do so by right-clicking the Start Command Prompts with Puppet program and clicking Run as administrator. Click Yes if the system asks for UAC confirmation.

The Puppet agent `.msi` adds the Puppet bin directory to the system path automatically. If you are not using the Start Command Prompts, you may need to manually add the bin directory to your PATH using one of the following commands:

For cmd.exe, run:

```
set PATH=%PATH%;"C:\Program Files\Puppet Labs\Puppet\bin"
```

For PowerShell, run:

```
 $env:PATH += ";C:\Program Files\Puppet Labs\Puppet\bin" 
```

## 2.2 Configure the `server` setting

The `server` is setting, which allows you to connect the agent to the primary Puppet server, is the only mandatory setting.

You can add configuration to agents by using the `puppet config set` sub-command, which edits puppet.conf automatically, or editing `/etc/puppetlabs/puppet/puppet.conf` directly.

To configure the `server` setting, choose from one of the following options:

- On the agent node, run:
    
    ```
    puppet config set server puppetserver.example.com --section main
    ```
    
- Manually edit `/etc/puppetlabs/puppet/puppet.conf` or `C:\ProgramData\PuppetLabs\puppet\etc\puppet.conf`.
    
    Note that the location on Windows depends on whether you are running with administrative privileges. If you are not, it will be in home directory, not system location.
    

Results

This command adds the setting `server = puppetserver.example.com` to the `[main]` section of puppet.conf.

Note that there are other optional settings, for example, `serverport`, `ca_server`, `ca_port`, `report_server`, `report_port`, which you might need for more complicated Puppet deployments, such as when using a CA server and multiple compilers.

## 2.3 Connect the agent to the primary server and sign the certificate

Once you had added the `server`, you must connect the Puppet agent to the primary server so that it will check in at regular intervals to report its state, retrieve its catalog, and update its configuration if needed.

1. To connect the agent to the primary server, run:
    ```
    puppet ssl bootstrap
    ```
    
    Note: For Puppet 5 agents, run `puppet agent --test` instead.
    
    You will see a message that looks like:
    
    ```
    Info: Creating a new RSA SSL key for <agent node>
    ```
    
2. On the primary server node, sign the certificate:
    
    ```
    sudo puppetserver ca sign --certname <name>
    ```
    
3. On the agent node, run the agent again:
    
    ```
    puppet ssl bootstrap
    ```

