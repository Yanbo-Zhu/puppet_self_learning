
# 1 Day 14 - 用 eyaml-hiera 來處理機敏資料

本系列文資料可參考以下：

- [Github](https://github.com/shazi7804/ops-puppet-30-days)
- [Gitbook](https://gitbook.com/book/shazi7804/puppet-manage-guide/details)
- [Mr.沙先生](https://shazi.info)

---

既然是 Configure 有存入 data，就會有敏感資訊的議題，像是 User/Password 這類型的東西就可以使用 hiera-eyaml 來加密。

## 1.1 hiera-eyaml 介紹

[hiera-eyaml][hiera-eyaml] 是由 Github 在 voxpupuli 維護的 Open Source 專案，前身是 [hiera-gpg][hiera-gpg]，但是 hiera-gpg 在 2015/08/25 EOL，所以就出現了 hiera-eyaml。

hiera-eyaml 相較於 hiera-gpg 的 feature：
  - only encrypts the values (which allows files to be swiftly reviewed without decryption)
  - encrypts the value of each key individually (this means that git diff is meaningful)
  - includes a command line tool for encrypting, decrypting, editing and rotating keys (makes it almost as easy as using clear text files)
  - uses basic asymmetric encryption (PKCS#7) by default (doesn't require any native libraries that need to be compiled & allows users without the private key to encrypt values that the puppet master can decrypt)
  - has a pluggable encryption framework (e.g. GPG encryption (hiera-eyaml-gpg) can be used if you have the need for multiple keys and easier key rotation)
  
除了官方以上的介紹以外，最大的特點是他可以支援**片段加密**，在 hiera-data 中你可以針對你想要的字串進行加密，但其他顯示明碼，就像是：

```yaml
profile::users:
  user: 'shazi'
  password: >
    ENC[PKCS7,MIIB2gYJKoZIhvcNAQcDoIIByzCCAccCAQAxggEhMIIBHQIBADAFMAACAQEw
    DQYJKoZIhvcNAQEBBQAEggEAX5J09MMulrg7zUvwDamARR0oYaq7RNkxZyvV
    SQWBSrBq+4C+u9wALGxZHatQpCOMzPlz3gRJKWq7KOqLHgrPehAofJUpDx1T
    KI32Hbac8rkWGx5gOAghSew3cufTgZg8nMZ1z0fngXwC5/ENZbmZbA/99udx
    gGMz18QUwVY9KwDw+FtYyxX5eV/ndcF2XAGNIsQwRYM09SBRYd5LEIxhC6+o
    b8JSVt2VuxggCRn9RQOwqUhMAI0YfgfUpARD94HOXWIqoI+S2HB18TZGVXQE
    ZuVu3G0HGhesovgEs/hoy95TUsSZekZWYoFE+oelkfDNQQCtHRhcTQJeMnXi
    swtqajCBnAYJKoZIhvcNAQcBMB0GCWCGSAFlAwQBKgQQgd7ZFA2MHAv2d0EQ
    u+u1yoBwYAz1z6FrIaRO3bZZjms0t+PG6CK3+kzisLpe7a/Fc7FROo94XuMn
    PtYksgfanjR9P87xIXgr8V0dVM0+xjyXBX/dHv6Qyy5avYZg3N/p+c8d7PD8
    Tp8FhVr7ayAU8TuoUoeZbF3YL/iydMPDtJZ7DA==]
```

## 1.2 安裝

hiera-eyaml 必須在 Master 安裝

1. 用 gem 安裝 hiera-eyaml

```shell
$ sudo gem install hiera-eyaml
```

1. 替 puppetserver 安裝 hiera-eyaml

```shell
$ puppetserver gem install hiera-eyaml 
```

1. 使用 eyaml createkeys 生成 Public/Private key

```shell
$ cd /etc/puppetlabs/puppet
$ sudo eyaml createkeys
```

這個步驟會在當下的目錄產生：

- keys/private_key.pkcs7.pem
- keys/public_key.pkcs7.pem

這兩把 Key 是用來加解密 hiera-eyaml 使用。

1. 因為安全所以會特別處理權限的部份 only read

```shell
$ sudo chown -R puppet:puppet /etc/puppetlabs/puppet/keys/
$ sudo chmod -R 0500 /etc/puppetlabs/puppet/keys/
$ sudo chmod 0400 /etc/puppetlabs/puppet/keys/*.pem
$ ls -lha /etc/puppetlabs/puppet/keys
-r-------- 1 puppet puppet 1679 May 31 14:12 private_key.pkcs7.pem
-r-------- 1 puppet puppet 1050 May 31 14:12 public_key.pkcs7.pem
```

1. 如果你要使用 eyaml 的 command line 的話必須要設定 config 指定 key 的位置

```yaml
# /etc/eyaml/config.yaml
---
pkcs7_private_key: /etc/puppetlabs/puppet/keys/private_key.pkcs7.pem
pkcs7_public_key:  /etc/puppetlabs/puppet/keys/public_key.pkcs7.pem
```

eyaml 讀取設定檔的順序是：

- /etc/eyaml/config.yaml
- ~/.eyaml/config.yaml
- $EYAML_CONFIG 變數

## 1.3 設定

hiera 本身可以 hiera-eyaml 的設定，如果拿來跟 hiera 放在一起，你的 hiera.yml 會長這樣：

```yaml
---
version: 5
 
defaults:
  datadir: data
  data_hash: yaml_data
 
hierarchy:
  - name: "Secret data: per-node, per-datacenter, common"
    lookup_key: eyaml_lookup_key
    paths:
      - "secrets/nodes/%{trusted.certname}.eyaml"
      - "common.eyaml"
    options:
      pkcs7_private_key: /etc/puppetlabs/puppet/keys/private_key.pkcs7.pem
      pkcs7_public_key:  /etc/puppetlabs/puppet/keys/public_key.pkcs7.pem
 
  - name: "Per-node data"
    path: "nodes/%{trusted.certname}.yaml"
 
  - name: "Common data"
    path: "common.yaml"
``` 

hiera-eyaml 預設是 .eyaml 為檔名，你可以把有被 hiera-eyaml 加密的 String  放在 common.eyaml。


## 1.4 加密與解密

### 1.4.1 透過 eyaml encrypt 來加密字串或檔案。

- -s 加密字串
- -f 加密檔案

```shell
$ eyaml encrypt -s '123'
 
[hiera-eyaml-core] Loaded config from /Users/scottliao/.eyaml/config.yaml
string: ENC[PKCS7,MIIBeQYJKoZIhvcNAQcDoIIBajCCAWYCAQAxggEhMIIBHQIBADAFMAACAQEwDQYJKoZIhvcNAQEBBQAEggEALG08ai79eR+tE1Ilh7i+Hnw585ZCGDrcL+/ae54x5em2IA0EzrtS+yZVED3x1hGQPzsi/vwDvZf0VXuFQCDa14OXVsrcz1YVY2dDRYu0lTQEb1RGFUlb1aarQdIOrrfuvMcC/awDRXkNuLl/0IbHfH20P5IRo6Zd/lDhBzFCN9hSzvJ187TmwNqXDV3HzZ/d76SNW1RA6SP/0T2iZ/8x2YOhcyYqz+l/BjSMxffGzlLFFsiiOndPqHtuldZGfqwnzvzkvMSml8no0695et8wwsFz1nAG3oyQFVWCi7TtF7nJ46SdY6pyy0GCHI0tVMyXlO5W0IBzSLkHfvh1Ukoy7DA8BgkqhkiG9w0BBwEwHQYJYIZIAWUDBAEqBBBFC79eC77bcWazzCrKwncpgBAeU+vF/bzMD9ZYjblt9WUT]
 
OR
 
block: >
    ENC[PKCS7,MIIBeQYJKoZIhvcNAQcDoIIBajCCAWYCAQAxggEhMIIBHQIBADAFMAACAQEw
    DQYJKoZIhvcNAQEBBQAEggEALG08ai79eR+tE1Ilh7i+Hnw585ZCGDrcL+/a
    e54x5em2IA0EzrtS+yZVED3x1hGQPzsi/vwDvZf0VXuFQCDa14OXVsrcz1YV
    Y2dDRYu0lTQEb1RGFUlb1aarQdIOrrfuvMcC/awDRXkNuLl/0IbHfH20P5IR
    o6Zd/lDhBzFCN9hSzvJ187TmwNqXDV3HzZ/d76SNW1RA6SP/0T2iZ/8x2YOh
    cyYqz+l/BjSMxffGzlLFFsiiOndPqHtuldZGfqwnzvzkvMSml8no0695et8w
    wsFz1nAG3oyQFVWCi7TtF7nJ46SdY6pyy0GCHI0tVMyXlO5W0IBzSLkHfvh1
    Ukoy7DA8BgkqhkiG9w0BBwEwHQYJYIZIAWUDBAEqBBBFC79eC77bcWazzCrK
    wncpgBAeU+vF/bzMD9ZYjblt9WUT]
```

你可以有兩種形式寫在 common.eyaml，ENC 代表已加密。

### 1.4.2 透過 eyaml decrypt 來解密字串或檔案。

```shell
$ eyaml decrypt -e common.eyaml
 
users:
  shazi:
    password: >
        DEC::PKCS7[123]!
```

解密後會以 DEC 來表示已經被解密，而 123 就是解密後的值。

### 1.4.3 使用 hiera-eyaml 後的 KEY 管理

在用了 hiera-eyaml 後同時也必須思考 Public / Private key 存放的問題，基本上 repository 不會存放 key，只提供開發 Puppet 的人員 Public key (有 Public key 就可以 encrypt)，讓系統管理者把 Public / Private key 塞到 Puppet master。

- 怎麼 Deploy Key

當使用 Puppet 後通常都會講 code 保存在 Version Control，但 Key 並不適合保存在 repository，所以可以嘗試在 CI 的時候打包，或是 CD 的時候再從儲存 Key 的地方 deployment 到 puppetserver。

如果你的環境有是用 AWS 則可以使用 [Parameter Store][parameter-store] 來保存，並且在 deployment 時拉下來，由於 Parameter Store 可以支援 IAM 的權限控管與 KMS 的加密，所以你可以將 Key 控制在**有權限**的 Server 上才能獲取到這把 Key。

[hiera-eyaml]: https://github.com/voxpupuli/hiera-eyaml
[hiera-gpg]: https://github.com/crayfishx/hiera-gpg
[parameter-store]: https://aws.amazon.com/tw/ec2/systems-manager/parameter-store/

