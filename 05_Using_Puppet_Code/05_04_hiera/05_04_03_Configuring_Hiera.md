https://www.puppet.com/docs/puppet/8/hiera_config_yaml_5#hiera_eyaml
# 1 Location of `hiera.yaml` files

There are several `hiera.yaml` files in a typical deployment. Hiera uses three layers of configuration, and the module and environment layers typically have multiple instances.

The configuration file locations for each layer:

  
|Layer|Location|Example|
|---|---|---|
|Global|`$confdir/hiera.yaml`|`/etc/puppetlabs/puppet/hiera.yaml` `C:\ProgramData\PuppetLabs\puppet\etc\hiera.yaml`|
|Environment|`<ENVIRONMENT>/hiera.yaml`|`/etc/puppetlabs/code/environments/production/hiera.yaml` `C:\ProgramData\PuppetLabs\code\environments\production\hiera.yaml`|
|Module|`<MODULE>/hiera.yaml`|`/etc/puppetlabs/code/environments/production/modules/ntp/hiera.yaml` `C:\ProgramData\PuppetLabs\code\environments\production\modules\ntp\hiera.yaml`|

# 2 Config file syntax

The `hiera.yaml` file is a YAML file, containing a hash with up to four top-level keys.

The following keys are in a `hiera.yaml` file:

- `version` - Required. Must be the number 5, with no quotes.
- `defaults` - A hash, which can set a default `datadir`, `backend`, and `options` for hierarchy levels.
- `hierarchy` - An array of hashes, which configures the levels of the hierarchy.
- `default_hierarchy` - An array of hashes, which sets a default hierarchy to be used only if the normal hierarchy entries do not result in a value. Only allowed in a module's `hiera.yaml.`

```
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
    path: "secrets/nodes/%{trusted.certname}.eyaml"
    options:
      pkcs7_private_key: /etc/puppetlabs/puppet/eyaml/private_key.pkcs7.pem
      pkcs7_public_key:  /etc/puppetlabs/puppet/eyaml/public_key.pkcs7.pem

  - name: "Per-OS defaults"
    path: "os/%{facts.os.family}.yaml"

  - name: "Common data"
    path: "common.yaml"
```

Note: When writing in Hiera YAML files, do not use hard tabs for indentation.

### 2.1.1 The default configuration

If you omit the `hierarchy` or `defaults` keys, Hiera uses the following default values.

```
version: 5
hierarchy:
  - name: Common
    path: common.yaml
defaults:
  data_hash: yaml_data
  datadir: data
```

These defaults are only used if the file is present and specifies `version: 5`. If `hiera.yaml` is absent, it disables Hiera for that layer. If it specifies a different version, different defaults apply.
