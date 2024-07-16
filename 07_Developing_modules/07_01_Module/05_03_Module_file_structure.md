

# 1 Module file structure

https://www.puppet.com/docs/pdk/3.x/pdk_creating_modules

PDK creates a basic module skeleton with directories and templates to support writing, validating, and testing Puppet code. 

| Files and directories    | Description                                                                                                                                                                     |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Module directory         | Directory with the same name as the module. Contains all of the module's files and directories.                                                                                 |
| `appveyor.yml`           | File containing configuration for Appveyor CI integration.                                                                                                                      |
| `CHANGELOG.md`           | File in which you can document notable changes to this project.                                                                                                                 |
| `.devcontainer`          | File describing how a container should be configured to test this module.                                                                                                       |
| `./files`                | Directory containing static files, which managed nodes can download.                                                                                                            |
| `.fixtures.yml`          | File specifying where test dependencies are loaded from.                                                                                                                        |
| `Gemfile`                | File describing Ruby gem dependencies.                                                                                                                                          |
| `.gitattributes`         | Recommended defaults for using Git.                                                                                                                                             |
| `.gitignore`             | File listing module files that Git should ignore.                                                                                                                               |
| `.gitlab-ci.yml`         | File containing an example configuration for GitLab CI.                                                                                                                         |
| `./manifests`            | Directory containing module manifests, each of which defines one class or defined type. PDK creates manifests when you create new classes or defined types with `pdk` commands. |
| `metadata.json`          | File containing metadata for the module.                                                                                                                                        |
| `.pdkignore`             | File listing module files that PDK should ignore when building a module package for upload to the Forge.                                                                        |
| `puppet-lint.rc`         | File containing configuration for puppet-lint.                                                                                                                                  |
| `Rakefile`               | File containing configuration for the Ruby infrastructure. Used in CI and for backwards compatibility.                                                                          |
| `README.md`              | File containing a README template for your module.                                                                                                                              |
| `.rspec`                 | File containing the default configuration for RSpec.                                                                                                                            |
| `.rubocop.yml`           | File containing recommended settings for Ruby style checking.                                                                                                                   |
| `./spec`                 | Directory containing files and directories for unit testing.                                                                                                                    |
| `spec/default_facts.yml` | File specifying facts that are available to all tests.                                                                                                                          |
| `spec/spec_helper.rb`    | Helper code to set up preconditions for unit tests.                                                                                                                             |
| `./spec/classes`         | Directory containing testing templates for any classes you create with the `pdk new class` command.                                                                             |
| `.sync.yml`              | File to customize the PDK template in use.                                                                                                                                      |
| `./tasks`                | Directory containing task files and task metadata files for any tasks you create with the `pdk new task`command.                                                                |
| `./templates`            | Directory containing any ERB or EPP templates. Required when building a module to upload to the Forge.                                                                          |
| `.vscode`                | Directory containing configuration for Visual Studio code.                                                                                                                      |
| `.yardopts`              | File containing the default configuration for Puppet Strings.                                                                                                                   |


---


|                                      |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| ------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 以一個 apache module 為例，我會需要產生以下這些檔案結構  | $ tree /etc/puppetlabs/code/modules/apache<br><br>├── files<br><br>\|   └── default.conf<br><br>└── manifests<br><br>    ├── init.pp<br><br>    ├── install.pp<br><br>    ├── config.pp<br><br>    ├── service.pp                                                                                                                                                                                                                                                                                                                                                                       |
| module 的 manifests                   |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| init.pp 是預設被讀取的檔案，通常用來定義變數、引用 class。 | class apache (<br><br>  String $package_name      = 'apache2',<br><br>  String $package_ensure    = 'installed',<br><br>  String $service_name      = 'apache2',<br><br>  String $service_ensure    = 'running',<br><br>  String $default_site_conf = '/etc/apache2/sites-enabled/default.conf',<br><br>  String $run_user          = 'www-data',<br><br>){<br><br>  contain apache::install<br><br>  contain apache::config<br><br>  contain apache::service<br><br>  Class['::apache::install']<br><br>  -> Class['::apache::config']<br><br>  ~> Class['::apache::service']<br><br>} |
| install.pp 用來定義如何安裝 package。         | class apache::install inherits apache {<br><br>  package { $apache::package_name:<br><br>    ensure => $apache::package_ensure<br><br>  }<br><br>}                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| service.pp 用來處理服務                    | class apache::service inherits apache {<br><br>  service { $apache::service_name:<br><br>    ensure    => $apache::service_ensure,<br><br>    subscribe => Package['apache'],<br><br>    require   => Class['apache::install']<br><br>  }<br><br>}                                                                                                                                                                                                                                                                                                                                      |
| config.pp 用來處理設定檔                    | class apache::config inherits apache {<br><br>  file { `$apache::default_site_conf`:<br><br>    ensure => file,<br><br>    owner  => `$apache::run_user`,<br><br>    source => `"puppet:///modules/${module_name}/default.conf"`<br><br>  }<br><br>}                                                                                                                                                                                                                                                                                                                                    |
|                                      |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |


----


[https://puppet.com/docs/puppet/7/modules_fundamentals.html#module-structure](https://puppet.com/docs/puppet/7/modules_fundamentals.html#module-structure)

data/
Contains data files specifying parameter defaults.

examples/
Contains examples showing how to declare the module's classes and defined types.

	init.pp: The main class of the module.
	example.pp: Provide examples for major use cases.

facts.d/
Contains external facts, which are an alternative to Ruby-based custom facts. These are synced to all agent nodes, so they can submit values for those facts to the primary Puppet server.

files/
Contains static files, which managed nodes can download.

	files/service.conf
	This file's source => URL is puppet:///modules/my_module/service.conf. Its contents can also be accessed with the file function, such as content => file('my_module/service.conf').

functions/
Contains custom functions written in the Puppet language.

lib/
Contains plug-ins, such as custom facts and custom resource types. These are used by both the primary Puppet server and the Puppet agent, and they are synced to all agent nodes in the environment on each Puppet run.

	lib/facter/
	Contains custom facts, written in Ruby.
	
	lib/puppet/
	Contains custom functions, resource types, and resource providers:

		lib/puppet/functions/: Contains functions written in Ruby for the modern Puppet::Functions API.
		lib/puppet/parser/functions/: Contains functions written in Ruby for the legacy Puppet::Parser::Functions API.
		lib/puppet/provider/: Contains custom resource providers written in the Puppet language.
		lib/puppet/type/: Contains custom resource types written in the Puppet language.

locales/
Contains files relating to module localization into languages other than English.

manifests/
Contains all of the manifests in the module.

	manifests/init.pp
	The init.pp class, if used, is the main class of the module. This class's name must match the module's name.
	
	manifests/other_class.pp
	Classes and defined types are named with the namespace of the module and the name of the class or defined type. For example, this class is named my_module::other_class.
	
	manifests/implementation/
	You can group related classes and defined types in subdirectories of the manifests/ directory. The name of this subdirectory is reflected in the names of the classes and types it contains. Classes and defined types are named with the namespace of the module, any subdirectories, and the name of the class or defined type.

		manifests/implementation/my_defined_type.pp: This defined type is named my_module::implementation::my_defined_type.
		manifests/implementation/class.pp: This defined type is named my_module::implementation::class.

plans/
Contains Puppet task plans, which are sets of tasks that can be combined with other logic. Plans are written in the Puppet language.

readmes/
The module's README localized into languages other than English.

spec/
Contains spec tests for any plug-ins in the lib directory.

tasks/
Contains Puppet tasks, which can be written in any programming language that can be read by the target node.

templates/
Contains templates, which the module's manifests can use to generate content or variable values.

	templates/component.erb
	A manifest can render this template with template('my_module/component.erb').
	
	templates/component.epp
	A manifest can render this template with epp('my_module/component.epp').

types/
Contains resource type aliases.


# 2 Manifests

Manifests, contained in the module's `manifests/` folder, each contain one class or defined type.

The `init.pp` manifest is the main class of a module and, unlike other classes or defined types, it is referred to only by the name of the module itself. 
For example, the class in `init.pp` in the `puppetlabs-motd` module is the `motd` class. You cannot name a class `init`.

All other classes or defined types names are composed of name segments, separated from each other by a namespace separator, `::`
- The module short name, followed by the namespace separator.
- Any `manifests/` subdirectories that the class or defined type is contained in, followed by a namespace separato.
- The manifest file name, without the extension.
    

For example, each module class or defined type would have the following names based on their module name and location within the `manifests/` directory:

|Module name|Filepath to class or defined type|Class or defined type name|
|---|---|---|
|`username-my_module`|`my_module/manifests/init.pp`|`my_module`|
|`username-my_module`|`my_module/manifests/other_class.pp`|`my_module::other_class`|
|`puppetlabs-apache`|`apache/manifests/security/rule_link.pp`|`apache::security::rule_link`|
|`puppetlabs-apache`|`apache/manifests/fastcgi/server.pp`|`apache::fastcgi::server`|


# 3 Files in modules

You can serve files from a module's `files/` directory to agent nodes.
Download files to the agent by setting the `file` resource's `source` attribute to the `puppet:///` URL for the file. Alternately, you can access module files with the `file` function.

To download the file with a URL, use the following format for the `puppet:///` URL:
```
puppet:///<MODULE_DIRECTORY>/<MODULE_NAME>/<FILE_NAME>
```

For example, given a file located in `my_module/files/service.conf`, the URL is:
```
puppet:///modules/my_module/service.conf
```

To access files with the `file` function, pass the reference `<MODULE NAME>/<FILE NAME>` to the function, which returns the content of the requested file from the module's `files/` directory. 
Puppet URLs work for both `puppet agent` and `puppet apply`; in either case they retrieve the file from a module.

To learn more about the `file` function, see the [function reference](https://www.puppet.com/docs/puppet/7/function).


## 3.1 例子

1 

Files 文件夹中有这个文件夹 
ivu_kubernetes/files/common/helm/csi-driver-smb.yam·
```yaml
---
linux:
  kubelet: "/var/lib/k0s/kubelet"
```


然后再某个 manifests 中有 
```ruby
class ivu_kubernetes::extensions::csi_driver_smb (
  Pattern[/^[0-9]+\.[0-9]+\.[0-9]+$/] $version,
  Stdlib::Absolutepath                $custom_config_dir,
) {  ivu_kubernetes::helm::chart { 'csi-driver-smb':
    chart_ref      => 'csi-driver-smb',
    repository_url => 'https://raw.githubusercontent.com/kubernetes-csi/csi-driver-smb/master/charts',
    version        => $version,
    namespace      => 'kube-system',
    values_content => file('ivu_kubernetes/common/helm/csi-driver-smb.yaml'),  # 这里 files 就被引用了 
  }
```


----

2

文件 在ivu_kubernetes/files/common/manifests/kubernetes-dashboard.yaml


manifests/extensions/kubernetes_dashboard.pp
```ruby
Defaults to lookup('ivu_kubernetes::custom_config_dir').
class ivu_kubernetes::extensions::kubernetes_dashboard (
  Boolean                $deploy_admin_user,
  Optional[Stdlib::Port] $node_port,
  Stdlib::Absolutepath   $custom_config_dir,
) {
  if $node_port == undef {
    $nodeport_manifest_fragment = ''
  } else {
    $nodeport_manifest_fragment = template('ivu_kubernetes/common/manifests/kubernetes-dashboard-nodeport.yaml.erb')
  }
  if $deploy_admin_user {
    $deploy_admin_user_fragment = file('ivu_kubernetes/common/manifests/kubernetes-dashboard-admin-user.yaml')
  } else {
    $deploy_admin_user_fragment = ''
  }
	  $default_manifest_fragment = file('ivu_kubernetes/common/manifests/kubernetes-dashboard.yaml') # file 被引用了
  $manifest_content          = "${default_manifest_fragment}\n${nodeport_manifest_fragment}\n${deploy_admin_user_fragment}"
 
  ivu_kubernetes::manifest { 'kubernetes-dashboard':
    content           => $manifest_content,
```




# 4 Templates in modules

You can use ERB or EPP templates in your module to manage the content of configuration files. Templates combine code, data, and literal text to produce a string output, which can be used as the content attribute of a `file` resource or as a variable value. 

Templates are contained in the module's `templates/` directory.

For ERB templates, which use Ruby, use the `template` function. For EPP templates, which use the Puppet language, use the `epp` function. See the page about [templates](https://www.puppet.com/docs/puppet/7/lang_template "Templates are written in a specialized templating language that generates text from data. Use templates to manage the content of your Puppet configuration files via the content attribute of the file resource type.") for detailed information.

The `template` and `epp` functions look up templates identified by module and template name, passed as a string in parentheses: `function('module_name/template_name.extension')`. For example:

```
template('my_module/component.erb')
```

```
epp('my_module/component.epp')
```

