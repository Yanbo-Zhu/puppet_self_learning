
https://www.puppet.com/docs/puppet/8/lang_facts_and_builtin_vars.html

# 1 总览 

1 什么是 facts
Before requesting a catalog for a managed node, or compiling one with puppet apply, Puppet collects system information, called facts, by using the Facter tool. 
就是 target 系统的一些 system information


2 Facts 的用途
The facts are assigned as values to variables that you can use anywhere in your manifests. 


3 built-in variables
Puppet also sets some additional special variables, called built-in variables, which behave a lot like facts.


4 
1  Facter's core facts 
 2 Facter's custom facts
3 Built-in variable: 
Puppet creates several variables for a node to facilitate managing it. 
These variables are called trusted facts, server facts, agent facts, server variables, and compiler variables.
Built-in variable  https://puppet.com/docs/puppet/7/lang_facts_builtin_variables.html


那些 facts when compiling a catalog 可以获取
Puppet code can access the following facts when compiling a catalog:
Core facts from Facter.
Custom facts and external facts that are present in your modules.


如何 看  see the fact values for a node
To see the fact values for a node, 
1 run facter -p on the command line, or 
2 browse facts on node detail pages in the Puppet Enterprise console. 
3 You can also use the PuppetDB API to explore or build tools to search and report on your infrastructure's facts.


Fact value 的数据类型 是 任意的： 
Puppet honors fact values of of any data type. It does not convert Boolean, numeric, or structured facts to strings.  



# 2 Accessing facts from Puppet code

When you write Puppet code, you can access facts in two ways: with the `$fact_name` syntax, or with the `$facts['fact_name']` hash.


1 Using the $fact_name syntax
Example:

```
if $osfamily == 'RedHat' {    # $osfamily 就是 $fact_name 的 一个 代表 
	# 3 ...
}

```



或者也可以使用 $::osfamily, 原因见下面 

When you code with this fact syntax, it's not immediately obvious that you're using a fact — someone reading your code needs to know which facts exist to guess that you're accessing a top-scope variable. 
To make your code easier for others to read, use the $::fact_name syntax as a hint, to show that it's accessing a top-scope variable.


2 Using `$facts['fact_name']` hash syntax 

Alternatively, facts are structured in a $facts hash, and your manifest code can access them as $facts['fact_name']. 

The variable name $facts is reserved, so local scopes cannot re-use it. Structured facts show up as a nested structure inside the $facts namespace, and can be accessed using Puppet's normal hash access syntax.

Accessing facts using this syntax makes for more readable and maintainable code, by making facts visibly distinct from other variables. It eliminates confusion that is possible when you use a local variable whose name happens to match that of a common fact.
Because of ambiguity with function invocation, the dot-separated access syntax that is available in Facter commands is not available with the `$facts` hash access syntax.  
(比如说 facter networking.ip  ， 但是在  `$facts['fact_name']`  不用 以 dot 的方使用  )
However, you can instead use the get function built into core Puppet. For more information, see README.

For example:
```
if $facts['os']['family'] == 'RedHat' {
	# 4 ...
}
```


# 3 Built-in variables

In addition to Facter's core facts and custom facts, Puppet creates several variables for a node to facilitate managing it. These variables are called trusted facts, server facts, agent facts, server variables, and compiler variables.

## 3.1 Trusted facts

Normal facts are self-reported by the node, and nothing guarantees their accuracy. Trusted facts are extracted from the node's certificate, which can prove that the certificate authority checked and approved them, making them useful for deciding whether a given node can receive the sensitive data in its catalog.

Trusted facts is a hash that contains trusted data from the node's certificate. You can access the data using the syntax `$trusted['fact_name']`. The variable name `$trusted` is reserved, so local scopes cannot reuse it.

 
|Keys in the `$trusted` hash|Possible values|
|---|---|
|`authenticated`|An indication of whether the catalog request was authenticated, as well as how it was authenticated. The value will be one of these:<br><br>- `remote` for authenticated remote requests, as with agent-server Puppet configurations.<br>    <br>- `local` for all local requests, as with standalone Puppet apply nodes.<br>    <br>- `false` for unauthenticated remote requests, possible if your `auth.conf` configuration allows unauthenticated catalog requests.|
|`certname`|The node’s subject certificate name, as listed in its certificate. When first requesting its certificate, the node requests a subject certificate name matching the value of its `certname` setting.<br><br>- If `authenticated` is `remote`, the value is the subject certificate name extracted from the node’s certificate.<br>- If `authenticated` is `local`, the value is read directly from the `certname` setting.<br>- If `authenticated` is `false`, the value will be an empty string.|
|`domain`|The node’s domain, as derived from its validated certificate name. The value can be empty if the certificate name doesn’t contain a fully qualified domain name.|
|`extensions`|A hash containing any [custom extensions](https://www.puppet.com/docs/puppet/8/ssl_attributes_extensions#ssl_attributes_extensions "When Puppet agent nodes request their certificates, the certificate signing request (CSR) usually contains only their certname and the necessary cryptographic information. Agents can also embed additional data in their CSR, useful for policy-based autosigning and for adding new trusted facts.") present in the node’s certificate. The keys of the hash are the extension OIDs. OIDs in the ppRegCertExt range appear using their short names, and other OIDs appear as plain dotted numbers. If no extensions are present, or `authenticated` is `local` or `false`, this is an empty hash.|
|`hostname`|The node’s hostname, as derived from its validated certificate name|

A typical `$trusted` hash looks something like this:

```
{
  'authenticated' => 'remote',
  'certname'      => 'web01.example.com',
  'domain'        => 'example.com',
  'extensions'    => {
                      'pp_uuid'                         => 'ED803750-E3C7-44F5-BB08-41A04433FE2E',
                      'pp_image_name'           => 'storefront_production'
                      '1.3.6.1.4.1.34380.1.2.1' => 'ssl-termination'
                   },
  'hostname'      => 'web01'
}
```

Here is some example Puppet code using a certificate extension:

```
if $trusted['extensions']['pp_image_name'] == 'storefront_production' {
  include private::storefront::private_keys
}
```

Here's an example of a `hiera.yaml` file using certificate extensions in a hierarchy:

```
---
version: 5
hierarchy:
  - name: "Certname"
    path: "nodes/%{trusted.certname}.yaml"
  - name: "Original VM image name"
    path: "images/%{trusted.extensions.pp_image_name}.yaml"
  - name: "Machine role (custom certificate extension)"
    path: "role/%{trusted.extensions.'1.3.6.1.4.1.34380.1.2.1'}.yaml"
  - name: "Common data"
    path: "common.yaml"
```

## 3.2 Server facts

The `$server_facts` variable provides a hash of server-side facts that cannot be overwritten by client side facts. This is important because it enables you to get trusted server facts that could otherwise be overwritten by client-side facts.

For example, the primary Puppet server sets the global `$::environment` variable to contain the name of the node's environment. However, if a node provides a fact with the name `environment`, that fact's value overrides the server-set `environment` fact. The same happens with other server-set global variables, like `$::servername` and `$::serverip`. As a result, modules can't reliably use these variables for whatever their intended purpose was.

A warning is issued any time a node parameter is overwritten.

Here is an example `$server_facts` hash:

```
{
  serverversion => "4.1.0",
  servername    => "v85ix8blah.delivery.example.com",
  serverip      => "192.0.2.10",
  environment   => "production",
}
```

## 3.3 Agent facts

Puppet agent and Puppet apply both add several extra pieces of info to their facts before requesting or compiling a catalog. Like other facts, these are available as either top-scope variables or elements in the `$facts` hash.

 
|Agent facts|Values|
|---|---|
|`$clientcert`|The node’s [`certname` setting](https://www.puppet.com/docs/puppet/8/configuration). This is self-reported; for the verified certificate name, use`$trusted['certname']`.|
|`$clientversion`|The current version of Puppet agent.|
|`$puppetversion`|The current version of Puppet on the node.|
|`$clientnoop`|The value of the node’s [`noop` setting](https://www.puppet.com/docs/puppet/8/configuration) (true or false) at the time of the run.|
|`$agent_specified_environment`|The value of the node’s [`environment` setting](https://www.puppet.com/docs/puppet/8/configuration). If the primary server’s node classifier specified an environment for the node, `$agent_specified_environment` and `$environment` can have different values. If no value was set for the `environment` setting (in `puppet.conf` or with `--environment`), the value of `$agent_specified_environment`is `undef`. That is, it doesn't default to `production` like the setting does.|

## 3.4 Server variables

Several variables are set by the compiling Puppet server. These are most useful when managing Puppet with Puppet, for example, managing the `puppet.conf` file with a template. Server variables are **not** available in the `$facts` hash.

 
|Server variables|Values|
|---|---|
|`$environment` (also available to`puppet apply`)|The agent node’s environment. Note that nodes can accidentally or purposefully override this with a custom fact; the `$server_facts['environment']` variable always contains the correct environment, and can’t be overridden.|
|`$servername`|The compiling server’s fully-qualified domain name (FQDN). Note that this information is gathered from the compiling server by Facter, rather than read from the config files. Even if the compiling server’s certname is set to something other than its FQDN, this variable still contains the server’s FQDN.|
|`$serverip`|The compiling server’s IP address.|
|`$serverversion`|The current version of Puppet on the compiling server.|
|`$settings::<SETTING_NAME>`(also available to `puppet apply`)|The value of any of the compiling server’s [configuration settings](https://www.puppet.com/docs/puppet/8/config_about_settings#config_about_settings "Customize Puppet settings in the main configuration file, called puppet.conf."). This is implemented as a special namespace and these variables must be referred to by their qualified names. Other than `$environment` and `$clientnoop`, the agent node’s settings are not available in manifests. If you wish to expose them to the compiling server, you must create a custom fact.|
|`$settings::all_local`|Contains all variables in the `$settings`namespace as a hash of `<SETTING_NAME> => <SETTING_VALUE>`. This helps you reference settings that might be missing, because a direct reference to such a missing setting raises an error when `--strict_variables`is enabled.|

## 3.5 Compiler variables

Compiler variables are set in every local scope by the compiler during compilation. They are mostly used when implementing complex defined types. Compiler variables are **not** available in the `$facts` hash.

By default, compiler variables have a value of `undef` (undefined). If you reference an undefined compiler variable, and you have specified the `strict_variables=true` setting, an error message flags the undefined variable.

 
|Compiler variables|Values|
|---|---|
|`$module_name`|The name of the module that contains the current class or defined type.|
|`$caller_module_name`|The name of the module in which the _specific instance_ of the surrounding defined type was declared. This is useful when creating versatile defined types that will be reused by several modules.|


# 4 facter Commnd
facter 是 Puppet 用來收集系統資訊使用，除了內建的一些變數以外，你還可以自己寫 facter 或使用外部 facter


## 4.1 facter command line
先講簡單的 cli 工具，只要你有裝 Puppet agent，那麼你已經有 facter 可以用，沒意外會在 /opt/puppetlabs/bin/facter 這邊找到 softlink 的 facter。

facter -p: 显示所有的变量
the dot-separated access syntax  is available in Facter commands
例子： facter networking.ip

直接跑 facter 會出現內建的 variable （只输入 facter 这个指令）
```
$ facter
aio_agent_version => 1.10.1
augeas => {
  version => "1.4.0"
}
dmi => {
  product => {
    name => "MacBookPro13,2"
    ...
```

## 4.2 Core Facts  Facts 

除了這些以外可以直接看 Core Facts 更多內建 facter
https://puppet.com/docs/facter/3.6/core_facts.html


(facts 的种类
1  Facter's core facts 
 2 Facter's custom facts
3 Built-in variable: 
)


會看到一堆系統參數，但是要怎樣取得有用的參數 ?，以 IP 為例
$ facter networking.ip
10.0.0.101

或是目前的 OS 訊息。
```
$ facter os
{
  architecture => "x86_64",
  family => "Darwin",
  hardware => "x86_64",
  macosx => {
    build => "17C88",
    product => "Mac OS X",
    version => {
      full => "10.13.2",
      major => "10.13",
      minor => "2"
    }
  },
  name => "Darwin",
  release => {
    full => "17.3.0",
    major => "17",
    minor => "3"
  }
}
```



## 4.3 Custom factor

如果你有 custom facter 也可以 include dir 用 --custom-dir 指定位置，一般會在 /opt/puppetlabs/puppet/cache/lib/facter/*.rb 這邊。

或是直接利用 FACTERLIB 這個變數來 include custom-dir
$ export FACTERLIB=/opt/puppetlabs/puppet/cache/lib/facter


## 4.4 自己寫 facter 來收集資料

除了內建的 facter 以外，還可以自己寫 facter 來抓資料，你可以寫在 module plugin 內。
/etc/puppetlabs/code/modules/<module_name>/lib/facter/*.rb

https://puppet.com/docs/facter/3.6/core_facts.html


範例 Facter.add
簡單用個範例來取 resolv.conf 裡的 name server。
```
#!/usr/bin/env ruby
# /etc/puppetlabs/code/modules/profile/lib/facter/name_server.rb
Facter.add(:nameserver) do
  setcode do
    # Find all the nameserver values in /etc/resolv.conf
    nameserver_data = Facter::Core::Execution.exec('/bin/cat /etc/resolv.conf | grep nameserver | awk \'{print $2}\'')
    nameserver = nameserver_data.split(" ")
  end
end
```


利用 Facter.add 的方式加入新的 facter，這邊是直接用 exec 來跑 shell 取參數，很方便。更多範例
https://github.com/puppetlabs/docs-archive/tree/main/facter/3.6



## 4.5 其他

1 Facts 的值传给 Puppetdb 
當 facter 成功被 deploy 到 Node 之後，在每次的 deploy，Agent 都會回穿 facter 給 Puppet Server，這時如果你有建立 Puppetdb 就會存入 Puppetdb 裡面。

透過 Dashboard 你可以看到被拋回來的資料

![[06_01_Syntax_And_Settings_Puppet_Language/images/Pasted image 20240412150537.png]]

2 Improving performance by blocking or caching built-in facts

https://puppet.com/docs/puppet/7/lang_facts_accessing.html#lang_facts_accessing-blocking-built-in-facts

If Facter is slowing down your code, you can configure Facter to block or cache built-in facts. When a system has a lot of something — for example, mount points or disks — Facter can take a long time to collect the facts from each one. 
When this is a problem, you can speed up Facter’s collection by configuring these settings in the facter.conf file:
- blocklist for blocking built-in facts you’re uninterested in.
- ttls for caching built-in facts you don’t need retrieved frequently.


# 5 Puppet API 来获取 infrastructure's facts.

You can also use the PuppetDB API to explore or build tools to search and report on your infrastructure's facts.
或是直接透過 Puppet API 來取得。
https://puppet.com/docs/puppet/5.5/http_api/http_api_index.html