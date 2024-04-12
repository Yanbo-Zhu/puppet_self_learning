

管理 user: Client 中建立那些user

文件: /etc/puppet/manifest/user.pp


# 1 User

## 1.1 属性 


name：用户名（不写时表示与title同名）；
uid: UID;
gid：基本组ID；
groups：附加组，不能包含基本组；
comment：注释； 
expiry：过期时间 ；
home：家目录； 
shell：默认shell类型；
system：是否为系统用户 ；
ensure：present/absent；
password：加密后的密码串；

Uid 和gid 只能在其 unix 中使用， Windows 中无法使用
Under Unix you can specifiy gid and uid properties respectively to ensure that the unique ID for users and groups are consistent across systems. You can’t do this on Windows


## 1.2 命令 
查询 user
`puppet resource user <name>`


## 1.3 例子

/etc/puppet/manifest/user.pp

```

user{'sjj':
        ensure => present,
        system => false,
        comment => 'A good Girl',
        home => '/app/sjj',
        uid => 2000,
}

puppet apply -v --noop user.pp #只测试，不真正执行，-v：显示详细信息
puppet apply -v  user.pp  #执行时将-noop去掉，-v可以将执行过程中的信息显示；


```

```
vim user1.pp
	user{'keerr':
        ensure => present,
        system => false,
        comment => 'Test User',
        shell => '/bin/tcsh',
        home => '/data/keerr',
        managehome => true,
        groups => 'mygrp',
        uid => 3000,
}
```



# 2 user group

## 2.1 属性：

```
name：组名；
gid：GID；
system：是否为系统组，true OR false；
ensure：目标状态，present/absent；
members：成员用户;

```

```
vim group.pp
	group{'mygrp':
        name => 'mygrp',
        ensure => present,
        gid => 2000,
}
```


## 2.2 例子


```
group { 'Avengers':
  ensure => present,
}

user { 'TonyStark':
  ensure => present,
  password => 'password123',
  groups => ['Avengers'],
}

```
