


這篇會以 ERB 做示範怎麼寫 templates，首先先拿 file 來 include template。
```
lass ntp::config (
  String $driftfile = '/var/lib/ntp/drift',
  Boolean $tinker = true,
  Array $restrict = [ '127.0.0.1', '::1' ],
){
  file { '/etc/ntp.conf':
    ensure => file,
    content => templates("${module_name}"/ntp.conf.erb),
  }
}

把要 input 到 ntp.conf.erb 的參數先定義好 type。
```


ERB 用變數寫設定
```
driftfile <%= @driftfile %>

Output
driftfile /var/lib/ntp/drift
```


---

ERB 的 if 用法
```
# ntp.conf: Managed by puppet.
<% if @tinker == true -%>
tinker panic 0
<% end -%>

Output
# ntp.conf: Managed by puppet.
tinker panic 0
```

ERB 用 if 判斷 type
```
<% if @restrict != [] -%>
<% @restrict.flatten.each do |restrict| -%>
restrict <%= restrict %>
<% end -%>
<% end -%>

Output
restrict 127.0.0.1
restrict ::1

先用簡單的範例來寫 ERB，因為有了判斷式的幫助所以設定檔就能按照不同的 input 而變化。
```




