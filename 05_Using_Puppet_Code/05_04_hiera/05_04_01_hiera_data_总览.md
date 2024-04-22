
https://www.puppet.com/docs/puppet/8/hiera_intro#hierarchies-interpolate


# 1 什麼是 Hiera

Hiera 是 Puppet 內建的數據查詢系統，如果要管理好 Puppet，Hiera 絕對是不可或缺的項目，Hiera 支援 YAML 和 JSON 的格式編輯。

将一个 node-specific 的值 放在外部的文件中 
Puppet’s strength is in reusable code. Code that serves many needs must be configurable: put site-specific information in external configuration data files, rather than in the code itself.

Puppet uses Hiera to do two things:
- Store the configuration data in key-value pairs
- Look up what data a particular module needs for a given node during catalog compilation

This is done via:
- Automatic Parameter Lookup for classes included in the catalog
- Explicit lookup calls

Hiera’s hierarchical lookups follow a “defaults, with overrides” pattern, meaning you specify common data one time, and override it in situations where the default won’t work. Hiera uses Puppet’s facts to specify data sources, so you can structure your overrides to suit your infrastructure. While using facts for this purpose is common, data-sources can also be defined without the use of facts.

Hierarchies are configured in a `hiera.yaml` configuration file


# 2 为什么使用hiera

https://shazi7804.gitbooks.io/puppet-manage-guide/content/basic/how-to-use-hiera-data.html

Hiera 擁有多層架構的特性，你可以為尚未定義的 data 設定一個 default hiera data，後續再給另一個 hiera data。

舉例：

hiera.yaml 是 hiera 預設讀取的檔案。

```
# hiera.yaml
---
version: 5

defaults:
  datadir: data
  data_hash: yaml_data

hierarchy:
  - name: "Per-node data"
    path: "nodes/%{::trusted.certname}.yaml"

  - name: "Common data"
    path: "common.yaml"
```

定義 command.yaml 的 ntp_server

```
# data/command.yaml
profile::base::ntp_server: 'time.stdtime.gov.tw'
```

但 web.puppet.com 必須修改 ntp_server 的值。

```
# data/nodes/web.puppet.com.yaml
profile::base::ntp_server: 'time.google.com'
```

按照上面的範例，common.yaml 和 web.puppet.com.yaml 都擁有 ntp_server，但是依照 hiera 的讀取順序，在 web.puppet.com.yaml 就取到了，所以 common.yaml 的 ntp_server 就被丟掉了。

這是因為在 Hiera 是按照 facts 讀到的 **第一個** 數據，讀取的順序：

- data/nodes/web.puppet.com.yaml
- data/location/portland/ops.yaml
- data/groups/ops.yaml
- data/os/RedHat.yaml
- data/common.yaml

除了 facts 的讀取順序以外，在 Hiera 5 之後還支援了 three layers 的架構：

- Global ($confdir/hiera.yaml)
- Environment ($ENVIRONMENT/hiera.yaml)
- Module ($MODULE/hiera.yaml)

透過這麼多層的 data 設計，就能讓你的 hiera 符合各種環境。



# 3 Hiera hierarchies

Hiera 擁有多層架構的特性，你可以為尚未定義的 data 設定一個 default hiera data，後續再給另一個 hiera data。

Hierarchies are configured in a hiera.yaml configuration file. Each level of the hierarchy tells Hiera how to access some kind of data source. A hierarchy is usually organized like this:


# 4 Hiera configuration layers

## 4.1 layered hierarchies 的顺序 
Each layer can configure its own independent hierarchy. Before a lookup, Hiera combines them into a single super-hierarchy: global → environment → module.

 Note: There is a fourth layer - default_hierarchy - that can be used in a module’s hiera.yaml. It only comes into effect when there is no data for a key in any of the other regular hierarchies

==Hiera skips empty data sources, and either returns the first found value or merges all found values.==
==By default, Hiera returns the first value it finds, but it can also continue searching and merge all the found values together.==
1. global (这个层级的 会被先看, 一旦找到了相关的参数的值 就不往下看了 )
2. environment
3. module
4. default_hierarchy


Hiera uses three independent layers of configuration. Each layer has its own hierarchy, and they’re linked into one super-hierarchy before doing a lookup.

The three layers are searched in the following order: global → environment → module. Hiera searches every data source in the global layer’s hierarchy before checking any source in the environment layer.

## 4.2 there layer

### 4.2.1 The global layer

`/etc/puppetlabs/puppet`

==The configuration file for the global layer is located, by default, in `$confdir/hiera.yaml`. You can change the location by changing the `hiera_config` setting in `puppet.conf`.==



Hiera has one global hierarchy. Because it goes before the environment layer, it’s useful for temporary overrides, for example, when your ops team needs to bypass its normal change processes.

The global layer is the only place where legacy Hiera 3 backends can be used - it’s an important piece of the transition period when you migrate you backends to support Hiera 5. It supports the following config formats: `hiera.yaml` v5, `hiera.yaml` v3 (deprecated).

Other than the above use cases, try to avoid the global layer. Specify all normal data in the environment layer.


#### 4.2.1.1 `hiera_config` setting in `puppet.conf



### 4.2.2 The environment layer

`/etc/puppetlabs/code/environments/production/hiera.yaml`

The configuration file for the environment layer is located, by default, in `<ENVIRONMENT DIR>/hiera.yaml`.

The environment layer is where most of your Hiera data hierarchy definition happens. Every Puppet environment has its own hierarchy configuration, which applies to nodes in that environment. Supported config formats include: v5, v3 (deprecated).

### 4.2.3 The module layer

`/etc/puppetlabs/code/environments/production/modules/ivu_ptp_kubernetes/hiera.yaml`

The configuration file for a module layer is located, by default, in a module's `<MODULE>/hiera.yaml`.

The module layer sets default values and merge behavior for a module’s class parameters. It is a convenient alternative to the `params.pp` pattern.

Note: To get the exact same behaviour as `params.pp`, use the `default_hierarchy`, as those bindings are excluded from merges. When placed in the regular hierarchy in the module’s hierarchy the bindings are merged when a merge lookup is performed.

It comes last in Hiera’s lookup order, so environment data set by a user overrides the default data set by the module’s author.

Every module can have its own hierarchy configuration. You can only bind data for keys in the module’s namespace.

## 4.3 Tips for making a good hierarchy

- Make a short hierarchy. Data files are easier to work with.
- Use the roles and profiles method to manage less data in Hiera. Sorting hundreds of class parameters is easier than sorting thousands.
- If the built-in facts don’t provide an easy way to represent differences in your infrastructure, make custom facts. For example, create a custom datacenter fact that is based on information particular to your network layout so that each datacenter is uniquely identifiable.
- Give each environment – production, test, development – its own hierarchy.


# 5 例子 

## 5.1 例子1

global hierarchy hiera.yaml: 
```
---
version: 5
hierarchy:
  - name: "Data exported from our old self-service config tool"
    path: "selfserve/%{trusted.certname}.json"
    data_hash: json_data
    datadir: data
```


environmental hierarchy  hiera.yaml: 
```
---
version: 5
defaults:  # Used for any hierarchy level that omits these keys.
  datadir: data         # This path is relative to hiera.yaml's directory.
  data_hash: yaml_data  # Use the built-in YAML backend.

hierarchy:
  - name: "Per-node data"                   # Human-readable name.
    path: "nodes/%{trusted.certname}.yaml"  # File path, relative to datadir.
                                   # ^^^ IMPORTANT: include the file extension!

  - name: "Per-datacenter business group data" # Uses custom facts.
    path: "location/%{facts.whereami}/%{facts.group}.yaml"

  - name: "Global business group data"
    path: "groups/%{facts.group}.yaml"

  - name: "Per-datacenter secret data (encrypted)"
    lookup_key: eyaml_lookup_key   # Uses non-default backend.
    path: "secrets/%{facts.whereami}.eyaml"
    options:
      pkcs7_private_key: /etc/puppetlabs/puppet/eyaml/private_key.pkcs7.pem
      pkcs7_public_key:  /etc/puppetlabs/puppet/eyaml/public_key.pkcs7.pem

  - name: "Per-OS defaults"
    path: "os/%{facts.os.family}.yaml"

  - name: "Common data"
    path: "common.yaml"
```



module level
```
---
version: 5
hierarchy:
  - name: "OS values"
    path: "os/%{facts.os.name}.yaml"
  - name: "Common values"
    path: "common.yaml"
defaults:
  data_hash: yaml_data
  datadir: data
```

Then in a lookup for the `ntp::servers` key, `thrush.example.com` would use the following combined hierarchy:
 global → environment → module.
- `<CODEDIR>/data/selfserve/thrush.example.com.json`
- `<CODEDIR>/environments/production/data/nodes/thrush.example.com.yaml`
- `<CODEDIR>/environments/production/data/location/belfast/ops.yaml`
- `<CODEDIR>/environments/production/data/groups/ops.yaml`
- `<CODEDIR>/environments/production/data/os/Debian.yaml`
- `<CODEDIR>/environments/production/data/common.yaml`
- `<CODEDIR>/environments/production/modules/ntp/data/os/Ubuntu.yaml`
- `<CODEDIR>/environments/production/modules/ntp/data/common.yaml`

The combined hierarchy works the same way as a layer hierarchy. ==Hiera skips empty data sources, and either returns the first found value or merges all found values.==

## 5.2 舉例2

hiera.yaml 是 hiera 預設讀取的檔案。

```yaml
# hiera.yaml
---
version: 5

defaults:
  datadir: data
  data_hash: yaml_data

hierarchy:
  - name: "Per-node data"
    path: "nodes/%{::trusted.certname}.yaml"

  - name: "Common data"
    path: "common.yaml"
```

定義 command.yaml 的 ntp_server

```yaml
# data/command.yaml
profile::base::ntp_server: 'time.stdtime.gov.tw'
```

但 web.puppet.com 必須修改 ntp_server 的值。

```yaml
# data/nodes/web.puppet.com.yaml
profile::base::ntp_server: 'time.google.com'
```

按照上面的範例，common.yaml 和 web.puppet.com.yaml 都擁有 ntp_server，但是依照 hiera 的讀取順序，在 web.puppet.com.yaml 就取到了，所以 common.yaml 的 ntp_server 就被丟掉了。


這是因為在 Hiera 是按照 facts 讀到的 **第一個** 數據，讀取的順序：

  - data/nodes/web.puppet.com.yaml
  - data/location/portland/ops.yaml
  - data/groups/ops.yaml
  - data/os/RedHat.yaml
  - data/common.yaml

除了 facts 的讀取順序以外，在 Hiera 5 之後還支援了 three layers 的架構：

  - Global ($confdir/hiera.yaml)
  - Environment ($ENVIRONMENT/hiera.yaml)
  - Module ($MODULE/hiera.yaml)

透過這麼多層的 data 設計，就能讓你的 hiera 符合各種環境。



# 6 在 Hiera 中處理變數

當你常用 Hiera 在比較複雜的環境，你可能會遇到在不同的變數需要相同的值，或是比較奇怪的符號，Hiera 就支援以下這幾項讓你用來處理這些情境。

## 6.1 Hierarchies interpolate variables

Most levels of a hierarchy interpolate variables into their configuration:

```
path: "os/%{facts.os.family}.yaml"
```

The percent-and-braces `%{variable}` syntax is a Hiera interpolation token. It is similar to the Puppet language’s `${expression}` interpolation tokens. Wherever you use an interpolation token, Hiera determines the variable’s value and inserts it into the hierarchy.

The `facts.os.family` uses the Hiera special `key.subkey` notation for accessing elements of hashes and arrays. It is equivalent to `$facts['os']['family']` in the Puppet language but the 'dot' notation produces an empty string instead of raising an error if parts of the data is missing. Make sure that an empty interpolation does not end up matching an unintended path.

You can only interpolate values into certain parts of the config file. For more info, see the `hiera.yaml` format reference.

With node-specific variables, each node gets a customized set of paths to data. The hierarchy is always the same.

## 6.2 用 lookup 來取得其他的 hiera data

lookup 適合讓你用來取得別的 data 後塞入另一個 data 的內容中，例如：

```yaml
---
php_version: '7.0'
php_package: "php%{lookup('php_version')}"
```

> php_package 的值等於 'php7.0'。

## 6.3 用 alias 直接對應相同的 hiera data

或是直接把變數 alias 到另一個 data

```yaml
---
original:
  - 'one'
  - 'two'
aliased: "%{alias('original')}"
```

> aliased 的值等於 original

## 6.4 literal 用來處理有 ‘%’ 的 hiera data

由於 hiera 的變數是 '%'，但如果你想儲存有 '%' 的值的話預設會被當成變數處理，要跳脫 '%' 就會用上 literal

```yaml
---
server_name: "%{literal('%')}{SERVER_NAME}"
```

> server_name 的值等於 ${SERVER_NAME}。

## 6.5 用來解釋 facts 的 scope

個人覺得 scope 沒什麼用途，主要是用來解釋 facts 的變數，實際上對 data 沒有特別改變

```yaml
smtpserver: "mail.%{facts.domain}"
smtpserver: "mail.%{scope('facts.domain')}"
```
> 以上兩者的值相同。
