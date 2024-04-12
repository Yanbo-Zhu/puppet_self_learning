


# 1 resource-ordering 處理 Resource 之間的順序

本系列文資料可參考以下：

- [Github](https://github.com/shazi7804/ops-puppet-30-days)
- [Gitbook](https://gitbook.com/book/shazi7804/puppet-manage-guide/details)
- [Mr.沙先生](https://shazi.info)

---

開始寫 Puppet code 之後總是會遇到有些命令要先執行、後執行等相依性的東西，例如：要先安裝 nginx 後才能處理 service 的啟動，如果先執行 service，就會出現找不到 nginx package 的問題。

puppet中也提供了before、require、notify和subscribe四个参数来定义资源之间的依赖关系和通知关系。
before：表示需要依赖于某个资源 
require：表示应该先执行本资源，在执行别的资源 
notify：A notify B：B依赖于A，且A发生改变后会通知B； 
subscribe：B subscribe A：B依赖于A，且B监控A资源的变化产生的事件；
依赖关系还可以使用->和～>来表示：-> 表示后资源需要依赖前资源 ~> 表示前资源变动通知后资源调用

## 1.1 建議

這個篇幅屬於可 LAB 性質，可以參考 [shazi7804/puppet-master-docker](https://github.com/shazi7804/puppet-master-docker) 來實作 Puppet code。

---

## 1.2 before

`before` 這個可以用來對指定的資源執行之前應用，例如：要設定 `sshd_config` 之前，必須先安裝好 `openssh-server` 這個 package。


例如：要設定 sshd_config 之前，必須先安裝好 openssh-server 這個 package。
```puppet
package { 'openssh-server':
  ensure => present,
  before => File['/etc/ssh/sshd_config'],
}
```

这表示当前资源执行完成以后，才能运行Service['httpd']资源。也就是必须先安装了httpd才能启动httpd。
```puppet
package {'httpd':

        ensure=> 'installed',

        before=> Service['httpd'],

        }

service {'httpd':

        ensure=> 'running',

        }

```


## 1.3 require

`require` 顧名思義就是一定要指定的資源有執行過，才會應用。需要的资源执行以后，当前资源才能执行。

`require` 和 `before` 可以相呼應來做到防呆機制。

```puppet
file { '/etc/ssh/sshd_config':
  ensure  => file,
  mode    => '0600',
  source  => 'puppet:///modules/sshd/sshd_config',
  require => Package['openssh-server'],
}
```


需要`Package[httpd`]资源执行完成后，才能运行当前资源。两个例子达到的效果相同，但是表诉方法可以不同。
```puppet
package {'httpd':

       ensure => 'installed',

       }

service {'httpd':

       ensure => 'running',

       require => Package[httpd],

       }
```

## 1.4 notify

`notify` 通常用來應用當 `設定檔` 被修改時，要觸發 service restart 的情境。
Notify（通知）：定义在前资源中，当资源执行时可以通知某个资源。

subscribe 和 notify 可以相呼應避免設定檔沒有生效。

```puppet
file { '/etc/ssh/sshd_config':
  ensure => file,
  mode   => '0600',
  source => 'puppet:///modules/sshd/sshd_config',
  notify => Service['sshd'],
}
```


The example above also introduces the notify attribute. This is a way to define dependencies between different parts of your configuration. 
It ensure that if the package or configuration file change that the service is also restarted. Puppet wouldn’t do this by default because the service would already be running so (due to idempotency) it would see no reason to modify it’s state. 

Using notify in this instance ensures that config or install changes take an immediate effect by also restarting the service. It performs a restart because that is the default action performed on a service when it is notified.

```puppet
package { 'mysql':
  ensure   => installed,
  provider => chocolatey,
  notify   => Service['mysql']
}

file { 'C:/mysql-5.5/mysql.cnf':
  source => 'c:/source/mysql.cnf',
  notify => Service['mysql']
}

service { 'mysql':
  ensure => running,
  enable => true,
}
```

## 1.5 subscribe

`subscribe` 也是拿來用於指定目標的觸發，和 `notify` 相反。
`subscribe` 和 `notify` 可以相呼應避免設定檔沒有生效。
Subscribe（订阅），定义在后资源中，当订阅的某个资源执行时，重启当前服务资源。
subscribe 和 notify 可以相呼應避免設定檔沒有生效。
资源相关性，一般用于当配置文件修改，需要通知服务重启的场景。

```puppet
service { 'sshd':
  ensure    => running,
  enable    => true,
  subscribe => File['/etc/ssh/sshd_config'],
}

######################

file{'/etc/httpd/conf/httpd.conf':

        ensure => file,

        source =>'/backup/httpd/httpd.conf',

        mode => '0644',

        owner => 'root',

        group => 'tuchao',

        notify => Service['httpd'],

     }

service {'httpd':

  ensure => running,

# subscribe =>File['/etc/httpd/conf/httpd.conf'],

      }

```



## 1.6 箭頭符號

如果是在寫 module，常用在 init.pp `->` 和 `~>` 來處理 class 的順序
“—>”用于定义次序链，而”~>”用于定义通知链，它们既可以用于资源引用间，也可以用于资源申报之间。

如果是在寫 module，常用在 init.pp -> 和 ~> 來處理 class 的順序

```puppet
class nginx (){
  contain nginx::install
  contain nginx::config
  contain nginx::service

  Class['::nginx::install']
  -> Class['::nginx::config']
  ~> Class['::nginx::service']
}
```

或是直接拿來處理 resource

```puppet
Package['nginx'] -> File['/etc/nginx/nginx.conf'] ~> Service['nginx']
```


- `->` 代表左邊的資源會先應用後，再執行右邊資源。
- `~>` 代表當左邊資源有異動的時候，觸發右邊資源更新。


```puppet
Package[‘ntp’] —> File[‘/etc/ntp.conf’] ~> Service[‘ntpd’]
```

![[06_01_Syntax_And_Settings_Puppet_Language/06_01_Resource/images/Pasted image 20240411085614.png]]

## 1.7 例子

![[06_01_Syntax_And_Settings_Puppet_Language/06_01_Resource/images/Pasted image 20240411090019.png]]

我们还可以使用在最下面统一写依赖关系的方式来定义：

![[06_01_Syntax_And_Settings_Puppet_Language/06_01_Resource/images/Pasted image 20240411090027.png]]




# 2 tag

puppet 也可以定义“标签”——tag，
打了标签以后，我们在运行资源的时候就可以只运行某个打过标签的部分，而非全部。这样就更方便于我们的操作。
一个资源中，可以有一个tag也可以有多个。


格式：
```puppet

为资源定义tag；

            type{'title':
                ...
                tag => 'TAG1',
            }

            type{'title':
                ...
                tag => ['TAG1','TAG2',...],
            }      


```

手动调用：
```
puppet apply --tags TAG1,TAG2,... FILE.PP
```

## 2.1 实例

首先，我们去修改一下redis.pp文件，添加一个标签进去

![[06_01_Syntax_And_Settings_Puppet_Language/06_01_Resource/images/Pasted image 20240411090201.png]]


然后，我们手动先开启redis服务：
systemctl start redis


现在，我们去修改一下file目录下的配置文件：
```
vim file/redis.conf 
requirepass keerya
```


接着，我们就去运行redis.pp，我们的配置文件已经修改过了，现在想要实现的就是重启该服务，实现，需要使用密码keer登录
puppet apply -v --tags instconf redis.pp
![[06_01_Syntax_And_Settings_Puppet_Language/06_01_Resource/images/Pasted image 20240411090429.png]]
![[06_01_Syntax_And_Settings_Puppet_Language/06_01_Resource/images/Pasted image 20240411090237.png]]
redis.pp运行结果
现在，我们就去登录一下redis看看是否生效：
![[06_01_Syntax_And_Settings_Puppet_Language/06_01_Resource/images/Pasted image 20240411090437.png]]
redis验证
验证成功，实验完成。
