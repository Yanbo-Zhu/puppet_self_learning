
https://www.puppet.com/docs/puppet/7/bgtm#writing_modules_overview-defining-your-module

# 1 Day 9 - 手把手系列 - 第一個 module


## 1.1 建議

這個篇幅屬於可 LAB 性質，可以參考 [shazi7804/puppet-master-docker](https://github.com/shazi7804/puppet-master-docker) 來實作 Puppet code。

---

## 1.2 動手寫 Module

如果覺得線上的所有 module 都不符合你的需求，可以自己動手寫，在這篇會以基礎的 module 方向去寫。

你可以把 module 放在以下位置：

 - /etc/puppetlabs/code/environments/production/modules
 - /etc/puppetlabs/code/modules
 - /opt/puppetlabs/puppet/modules

### 1.2.1 基礎 module 檔案結構

以一個 apache module 為例，我會需要產生以下這些檔案結構

```shell
$ tree /etc/puppetlabs/code/modules/apache
├── files
|   └── default.conf
└── manifests
    ├── init.pp
    ├── install.pp
    ├── config.pp
    ├── service.pp
```

创建一个module_name目录, 在该目录中包含manifests, lib, tests, spec, files, templates目录,

 其中在manifests中放的文件是.pp文件, 且和python一样有类似的要求, 需要有一个init.pp文件, 并且在该文件中需要定义一个class, 他的class name需要和module_name一致

![[07_Developing_modules/07_01_Module/images/Pasted image 20240412163304.png]]

### 1.2.2 module 的 manifests

init.pp 是預設被讀取的檔案，通常用來定義變數、引用 class。

```puppet
class apache (
  String $package_name      = 'apache2',
  String $package_ensure    = 'installed',
  String $service_name      = 'apache2',
  String $service_ensure    = 'running',
  String $default_site_conf = '/etc/apache2/sites-enabled/default.conf',
  String $run_user          = 'www-data',
){
  contain apache::install
  contain apache::config
  contain apache::service

  Class['::apache::install']
  -> Class['::apache::config']
  ~> Class['::apache::service']
}
```

install.pp 用來定義如何安裝 package。

```puppet
class apache::install inherits apache {
  package { $apache::package_name:
    ensure => $apache::package_ensure
  }
}
```

service.pp 用來處理服務

```puppet
class apache::service inherits apache {
  service { $apache::service_name:
    ensure    => $apache::service_ensure,
    subscribe => Package['apache'],
    require   => Class['apache::install']
  }
}
```

config.pp 用來處理設定檔

```puppet
class apache::config inherits apache {
  file { $apache::default_site_conf:
    ensure => file,
    owner  => $apache::run_user,
    source => "puppet:///modules/${module_name}/default.conf"
  }
}
```

## 1.3 怎麼引用 module

要引用 module 只需要使用 include 就可以把已經寫好的 apache module 引用進來

```puppet
node default {
  include ::apache
}
```
只要三行、三行、三行 !! 就完成佈署安裝 apache 了

這樣 Puppet code 就簡潔許多了，恭喜你已經學會了基本的 module 寫法，可以再往進階 module 前進。

[github]: https://github.com
[puppetforge]: https://forge.puppet.com/
[puppetlabs]: https://github.com/puppetlabs/
[voxpupuli]: https://github.com/voxpupuli
[puppet-nginx]: https://github.com/voxpupuli/puppet-nginx
[elastic]: https://github.com/elastic


# 2 create modul via PDK command 

使用 pdk build 命令

1 生成 tar.gz package 
The pdk build command performs a series of checks on your module and builds a tar.gz package so that you can upload your module to the Forge.

Naming convention
PDK names module packages with the convention forgeusername-modulename-version.tar.gz.


2 .gitignore or .pdkignore files
 run the pdk build command, PDK checks your module metadata, looks for any symlinks, and excludes from the package any files listed in the .gitignore or .pdkignore files. If PDK finds any issues with metadata or symlinks, it prompts you to fix these issues.

By default, the .pdkignore file contains a list of commonly ignored files, such as temporary files. This file is located in the module's main directory. 

To add or remove files to this list, define them in the module's .sync.yml file and run pdk update on your module.


3 步骤
1. From the command line, change into the module's directory with cd <MODULE_NAME>
2. Run pdk build and respond to any prompts.  
    To change the behavior of the build command, add option flags to the command. For example, to create the package to a custom location, run pdk build --target-dir=`<PATH>` . For a complete list of command options and usage information, see the PDK [command reference](https://puppet.com/docs/pdk/2.x/pdk_reference.html).



# 3 Class design

A good module is made up of small, self-contained classes that each do only one thing. Classes within a module are similar to functions in programming, using parameters to perform related steps that create a coherent whole.

In general, files must have the same named as the class or definition that it contains, and classes must be named after their function. The one exception to this rule is the main class of a module, which is defined in the `init.p`p file, but is called by the same name as the module. Generally, a module includes:
- The `<MODULE>` class: The main class of the module shares the name of the module and is defined in the `init.pp` file.
- The `install` class: Contains all of the resources related to installing the software that the module manages.
- The `config` class: Contains resources related to configuring the installed software.
- The `service` class: Contains service resources, as well as anything else related to the running state of the software.

For more information and an example of this structure and the code contained in classes, see the topic about module classes.

# 4 Parameters

Parameters form the public API of your module. They are the most important interface you expose, so be sure to balance to the number and variety of parameters so that users can customize their interactions with the module.

Name your parameters in a consistent `thing_property` pattern, such as `package_ensure`. Consistency in names helps users understand your parameters and aids in troubleshooting and collaborative development. If you have a parameter that manages the entire installation of a package, you can use the `package_manage` convention. The `package_manage` pattern allows you to wrap all of the resources in an `if $package_manage {}` test, as shown in this `ntp` example:

```
class ntp::install {

  if $ntp::package_manage {
    package { $ntp::package_name:
      ensure => $ntp::package_ensure,
    }
  }
}            
```

To make sure users can customize your module as needed, add parameters. Do not hardcode data in your module, because this makes it inflexible and harder to use in even slightly different circumstances. For the same reason, avoid adding parameters that allow users to override templates. When you allow template overrides, users can override your template with a custom template containing additional hardcoded parameters. Instead, it's better to add flexible, user configurable parameters as needed.

For an example of a module that offers many parameters to increase flexibility, see the [puppetlabs-apache](http://forge.puppet.com/puppetlabs/apache) module.

# 5 Ordering

Base all order-related dependencies (such as `require` and `before`) on classes rather than resources. Class-based ordering allows you to isolate the implementation details of each class. For example, rather than specifiying `require` for several packages, you can use one class dependency. This allows you to make adjustments to the `module::install` class only, instead of adjusting multiple class manifests:

```
file { 'configuration':
      ensure  => present,
      require => Class['module::install'],
    }
```

# 6 Containment

Ensure that your main classes explicitly contain any subordinate classes they declare. Classes do not automatically contain the classes they declare, because classes can be declared in several places via `include` and similar functions. If your classes contain the subordinate classes, it makes it easier for other modules to form ordering relationships with your module.

To contain classes, use the `contain` function. For example, the `puppetlabs-ntp` module uses containment in the main `ntp` class:

```
contain ntp::install
contain ntp::config
contain ntp::service

Class['ntp::install']
-> Class['ntp::config']
~> Class['ntp::service']
```

For more information about containment, see the [containment documentation](https://www.puppet.com/docs/puppet/7/lang_containment#lang_containment "Containment is what controls the order in which the various parts of your Puppet code are executed. Containment is the relationship that resources have to classes and defined types, determining what has to happen before other things can happen.").

# 7 Dependencies

If your module's functionality depends on another module, list these dependencies in the module and include them directly in the module's main class with an `include` statement. This ensures that the dependency is included in the catalog. List the dependency to the module's `metadata.json` file and the `.fixtures.yml` file used for RSpec unit testing.


# 8 Testing modules

Test your module to make sure that it works in a variety of conditions and that its options and parameters work together. PDK includes tools for validating and running unit tests on your module, including RSpec, RSpec Puppet, and Puppet Spec Helper.

Write unit tests to verify that your module works as intended in a variety of circumstances. For example, to ensure that the module works in different operating systems, write tests that call the `osfamily` fact to verify that the package and service exist in the catalog for each operating system your module supports.

To learn more about how to write unit tests, see the [RSpec testing tutorial](http://rspec-puppet.com/tutorial). For more information on testing tools, see the tools list below.

`rspec-puppet`

Extends the RSpec testing framework to understand and work with Puppet catalogs, the artifact it specializes in testing. This allows you to write tests that verify that your module works as intended. This tool is included in PDK.

For example, you can call facts, such as `osfamily`, with RSpec, iterating over a list of operating systems to make sure that the package and service exist in the catalog for every operating system your module supports.

To learn more about `rspec-puppet` use and unit testing, see the [rspec-puppet page](http://rspec-puppet.com/).

`puppetlabs_spec_helper`

Automates some of the tasks required to test modules. This is especially useful in conjunction with `rspec-puppet`, because `puppetlabs_spec_helper` provides default Rake tasks that allow you to standardize testing across modules. It also provides some code to connect `rspec-puppet` with modules. This tool is included in PDK.

To learn more, see the [puppetlabs_spec_helper](https://github.com/puppetlabs/puppetlabs_spec_helper) project.

**litmus**

Allows you to provision test platforms such as containers/images, install a Puppet agent, install a module and run tests against your systems. For more information, see the [puppet_litmus](https://puppetlabs.github.io/content-and-tooling-team/docs/litmus/usage/commands/litmus-core-commands/#test-automation-group) documentation.

# 9 Publish module 

[https://puppet.com/docs/puppet/4.9/modules_publishing.html#removing-symlinks-from-your-module](https://puppet.com/docs/puppet/4.9/modules_publishing.html#removing-symlinks-from-your-module)

You can now upload your module to the Forge.

# 10 Documenting your module

|   |   |
|---|---|
|Documenting modules|[https://puppet.com/docs/puppet/7/modules_documentation.html#modules_documentation](https://puppet.com/docs/puppet/7/modules_documentation.html#modules_documentation)|
|Documenting modules with Puppet Strings<br><br>Puppet string|[https://puppet.com/docs/puppet/7/puppet_strings.html](https://puppet.com/docs/puppet/7/puppet_strings.html)|
|Puppet Strings style guide|[https://puppet.com/docs/puppet/7/puppet_strings_style.html#puppet_strings_style](https://puppet.com/docs/puppet/7/puppet_strings_style.html#puppet_strings_style)|

Document your module's use cases, usage examples, and parameter details with `README.md` and `REFERENCE.md` files. In the README, explain why and how users would use your module, and provide usage examples. Use Puppet Strings to create the REFERENCE, which is a detailed list of information about your module's classes, defined types, functions, tasks, task plans, and resource types and providers. For more about writing your README and creating the REFERENCE, see our [module documentation guide](https://www.puppet.com/docs/puppet/7/modules_documentation#modules_documentation "Document any module you write, whether your module is for internal use only or for publication on the Forge. Complete, clear documentation helps your module users understand what your module can do and how to use it.") and the [Strings documentation](https://www.puppet.com/docs/puppet/7/puppet_strings#puppet_strings "Produce complete, user-friendly module documentation by using Puppet Strings. Strings uses tags and code comments, along with the source code, to generate documentation for a module\").

# 11 Versioning your module

Whenever you make changes to your module, update the version number. Version your module semantically to help users understand the level of changes in your updated module. To learn more about the specific rules of semantic versioning, see the [semantic versioning specification](https://semver.org).

After you've decided on the new version number, adjust the version number in the `metadata.json` file. This allows you to create a list of dependencies in the `metadata.json` file of your modules with specific versions of dependent modules, which ensures your module isn't used with an old dependency that won't work. Versioning also enables workflow management by allowing you to easily use different versions of modules in different environments.

# 12 Releasing your module

Publish your modules on the Forge to share your modules with other Puppet users. Sharing modules allows other users to not only download and use your module to solve their infrastructure problems, but also to contribute their own improvements to your modules. Sharing modules fosters community among Puppet users, and helps improve the quality of modules available to everyone. To learn how to publish your modules to the Forge, see the [module publishing documentation](https://www.puppet.com/docs/puppet/7/modules_publishing#modules_publishing "To share your module with other Puppet users, get contributions to your modules, and maintain your module releases, publish your module on the Puppet Forge. The Forge is a community repository of modules, written and contributed by open source Puppet and Puppet Enterprise users.").


# 13 Module classes

A typical module contains a main module class, as well as classes for managing installation, configuration, and the running state of the managed software. The `puppetlabs-ntp` module provides examples of the classes in such a module structure.

## 13.1 `module`

The main class of any module shares the name of the module, but the file itself is named `init.pp`. This class is the module's main interface point with Puppet. If possible, make the main class the only parameterized class in your module. Limiting the parameterized classes to only the main class means that you only have to include a single class to control usage of the entire module. This class provides sensible defaults so that a user can get going by just declaring the main class with `include module`.

For instance, the main `ntp` class in the `puppetlabs-ntp` module is the only parameterized class in the module:

```
class ntp (
  Boolean $broadcastclient,
  Stdlib::Absolutepath $config,
  Optional[Stdlib::Absolutepath] $config_dir,
  String $config_file_mode,
  Optional[String] $config_epp,
  Optional[String] $config_template,
  Boolean $disable_auth,
  Boolean $disable_dhclient,
  Boolean $disable_kernel,
  Boolean $disable_monitor,
  Optional[Array[String]] $fudge,
  Stdlib::Absolutepath $driftfile,
 ...
```

## 13.2 `module::install`

The install class must be located in the `install.pp` file. It contains all of the resources related to getting the software that the module manages onto the node. The install class must be named `module::install`. In the `puppetlabs-ntp` module, this class is private, which means users do not interact with the class directly.

```
class ntp::install {

  if $ntp::package_manage {
    package { $ntp::package_name:
      ensure => $ntp::package_ensure,
    }
  }
}
```

## 13.3 `module::config`

Place the resources related to configuring the installed software in a config class. The config class must be named `module::config` and must be located in the `config.pp` file. In the `puppetlabs-ntp` module, this class is private, which means users do not interact with the class directly.

```
class ntp::config {

  # The servers-netconfig file overrides NTP config on SLES 12, interfering with our configuration.
  if $facts['operatingsystem'] == 'SLES' and $facts['operatingsystemmajrelease'] == '12' {
    file { '/var/run/ntp/servers-netconfig':
      ensure => 'absent'
    }
  }

  if $ntp::keys_enable {
    case $ntp::config_dir {
      '/', '/etc', undef: {}
      default: {
        file { $ntp::config_dir:
          ensure  => directory,
          owner   => 0,
          group   => 0,
          mode    => '0775',
          recurse => false,
        }
      }
    }

    file { $ntp::keys_file:
      ensure  => file,
      owner   => 0,
      group   => 0,
      mode    => '0644',
      content => epp('ntp/keys.epp'),
    }
  }
...
```

## 13.4 `module::service`

Put the remaining service resources, and anything else related to the running state of the software in the service class. The service class must be named `module::service` and must be located in the `service.pp` file. In the `puppetlabs-ntp` module, this class is private, which means users do not interact with the class directly.

```
class ntp::service {

  if ! ($ntp::service_ensure in [ 'running', 'stopped' ]) {
    fail('service_ensure parameter must be running or stopped')
  }

  if $ntp::service_manage == true {
    service { 'ntp':
      ensure     => $ntp::service_ensure,
      enable     => $ntp::service_enable,
      name       => $ntp::service_name,
      provider   => $ntp::service_provider,
      hasstatus  => true,
      hasrestart => true,
    }
  }
}
```


