# 1 Day 12 - 手把手系列 - 常用的 Resource

本系列文資料可參考以下：

- [Github](https://github.com/shazi7804/ops-puppet-30-days)
- [Gitbook](https://gitbook.com/book/shazi7804/puppet-manage-guide/details)
- [Mr.沙先生](https://shazi.info)

---

如果以等級來說，前面幾天可以說是 Puppet 的新手村，你大概可以用簡單的範例來達成一般性的需求，但是 **"會用"** 跟 **"用的好"** 是兩回事。

準備好認識 Puppet 更多了嗎 ?


## 1.1 建議

這個篇幅屬於可 LAB 性質，可以參考 [shazi7804/puppet-master-docker](https://github.com/shazi7804/puppet-master-docker) 來實作 Puppet code。

---

Resource 是 Puppet 最基本組成的元件，在整個 Puppet code 中，我們會利用各種 Resource 來控制幾乎所有我們想要控制的元件，例如 cron、file、user .. 等等。

但是前面我們用到的 package、service、file 明顯不足阿 !! 今天還會另外介紹一些也很常用到的 Resource。

## 1.2 package

package 是用來管理套件的 resource，像是 apt / yum，也支援 rpm / dpkg 這類型的工具，針對不同的平台有各自的 [provider][package-provider]。

最簡單的安裝 apache2 方式：

```puppet
package { 'apache2':
  ensure => installed,
}
```

> 等同於 `apt-get install apache2`。

或是想要安裝特別版本的話可以用 `ensure` 來指定版本：

```puppet
package { 'apache2': 
  ensure => '2.4.18-2ubuntu3.5',
}
```

> 等同於 `apt-get install apache2=2.4.18-2ubuntu3.5`

## 1.3 service

service 用來管理 daemon，舉 Ubuntu / CentOS 來說就是 `service` / `systemd` 一樣支援不同的 [provider][service-provider] 

- 啟動 Apache2

```puppet
service { 'apache2':
  ensure => running,
  enable => true,
}
```

ensure 定義這個服務的狀態，enable 則是定義在 on boot 的時候的狀態。

## 1.4 file

file 可以用來管理檔案、目錄、檔案內容、以及權限，還有 symlink。
在 Puppet 裡使用 file 就跟吃飯一樣常常接觸，畢竟這是一套 Configuration 組態管理系統嘛！

- 建立一個 Hello world !!

```puppet
file { '/tmp/bar':
  ensure  => file,
  mode    => '0644',
  owner   => 'root',
  group   => 'root',
  content => 'hello world!!'
}
```

實際上在 instances 會產生一個 /tmp/bar 的檔案，並且：

```shell
$ cat /tmp/bar
hello world!!
```

檔案權限的部份會變成這樣：

```shell
$ ll /tmp/bar
-rw-r--r-- 1 root root 13 Oct 24 11:45 bar
```

- 建立樹狀結構目錄

在 Linux 你可以用 `mkdir -p` 搞定，但是 Puppet 你要一層一層建立，就像這樣：

```puppet
file { ['/tmp/bar', '/tmp/bar/foo']:
  ensure => directory,
}
```

或是用變數：

```puppet
$sample_dirs = ['/tmp/bar', '/tmp/bar/foo']
file { $sample_dirs:
  ensure => directory,
}
```

- 處理檔案符號連結 (symlink)

```puppet
file { '/tmp/link-to-motd':
  ensure => link,
  target => '/etc/motd', 
}
```

file 多數用在你想 `完全掌控` 檔案或目錄的情況下使用，如果你只想針對內容的某一段修改，則應該使用 [augeas][resource-augeas] 或是 [file_line][file_line]。

[package-provider]: https://puppet.com/docs/puppet/5.3/types/package.html#package-attribute-provider
[service-provider]: https://puppet.com/docs/puppet/5.3/types/service.html#service-attribute-provider
[resource-augeas]: https://puppet.com/docs/puppet/5.3/type.html#augeas
[file_line]: https://forge.puppet.com/puppetlabs/stdlib#file_line







