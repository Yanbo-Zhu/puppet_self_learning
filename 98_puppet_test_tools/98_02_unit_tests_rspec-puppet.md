https://confluence.ivu.de/display/SYS/How+to+write+unit+tests+for+Puppet+modules

https://www.puppet.com/docs/puppet/7/bgtm#writing_modules_overview-modules-containment


# 1 总览
https://www.puppet.com/docs/pdk/3.x/pdk_testing.html#unit-testing-modules

<mark>Create and run unit tests to verify that your Puppet code compiles on supported operating systems and includes all declared resources in the catalog.</mark>
PDK runs your unit tests to ensure that your code compiles correctly and works as you expect it to, but it cannot test changes to the managed system or services.
PDK can create ==a basic unit test file that tests whether a manifest compiles to the catalog on the operating systems== specified in the module's metadata.json file. PDK creates these unit test files when you

==they can’t test the result of the manifest on a live system ==


---
这个解释好
Rspec-puppet tests are there to test the behaviour of Puppet when it compiles your manifests into a catalogue of Puppet resources. For example, you might want to test that your `apache::vhost` defined type creates a `file` resource with a `path` of `/etc/apache2/sites-available/foo` when run on a Debian host.

When writing your test cases, you should only test the first level of resources in your manifest. ==By this I mean, when testing your ‘webserver’ role class, you would test for the existence of the `apache::vhost` types, but not for the `file` resources created by them, that’s the job of the tests for `apache::vhost`==

--- 


Test your module to make sure that it works in a variety of conditions and that its options and parameters work together. PDK includes tools for validating and running unit tests on your module, including RSpec, RSpec Puppet, and Puppet Spec Helper.

Write unit tests to verify that your module works as intended in a variety of circumstances. For example, to ensure that the module works in different operating systems, write tests that call the `osfamily` fact to verify that the package and service exist in the catalog for each operating system your module supports.

To learn more about how to write unit tests, see the [RSpec testing tutorial](http://rspec-puppet.com/tutorial). For more information on testing tools, see the tools list below.

rspec-puppet

	Extends the RSpec testing framework to understand and work with Puppet catalogs, the artifact it specializes in testing. This allows you to write tests that verify that your module works as intended. This tool is included in PDK.
	
	For example, you can call facts, such as `osfamily`, with RSpec, iterating over a list of operating systems to make sure that the package and service exist in the catalog for every operating system your module supports.
	
	To learn more about `rspec-puppet` use and unit testing, see the [rspec-puppet page](http://rspec-puppet.com/).

puppetlabs_spec_helper

	Automates some of the tasks required to test modules. This is especially useful in conjunction with `rspec-puppet`, because `puppetlabs_spec_helper` provides default Rake tasks that allow you to standardize testing across modules. It also provides some code to connect `rspec-puppet` with modules. This tool is included in PDK.
	
	To learn more, see the [puppetlabs_spec_helper](https://github.com/puppetlabs/puppetlabs_spec_helper) project.

# 2 **What are unit tests**

Contrary to [acceptance tests](https://confluence.ivu.de/display/SYS/How+to+write+acceptance+tests+for+puppet+modules), executing unit tests for a puppet module does not imply installing this module on a real machine. Unit testing a module means testing it in a sandbox to check if it compiles succesfully on the basis of given facts. Because, during unit testing, the module has no access to a real system, facts about the operating system on which the installation would normally take place are mocked.

Apart from catching syntax mistakes, undefined variables/functions and logical errors, rspec-puppet unit tests are very useful to catch missing module dependencies or incompatible module versions, and this in a very short amount of time. They would also find out if template files are missing or files are installed before ensuring that a directory exist. However, they are no guarantee that the manifest under test produces the expected installation/configuration. For this reason, they are usually run before acceptance tests, to catch errors quickly, as acceptance tests are considerably slower.


# 3 IVU 内部puppet module 的 unittest 

https://git.ivu-ag.com/projects/PUPPET/repos/ivu_ptp_software/browse/spec/defines
https://git.ivu-ag.com/projects/PUPPET/repos/ivu_kubernetes/browse/spec


# 4 IVU实践中 Common errors and possible solutions

"Errno::ENOENT: No such file or directory - git"
try to install git on the system ("apt install git")


# 5 Rspec简介

https://rspec-puppet.com/
https://rspec-puppet.com/tutorial/
https://github.com/puppetlabs/rspec-puppet/?tab=readme-ov-file#defined-types-and-classes
http://rspec.info/

Related projects
- [puppetlabs_spec_helper](https://github.com/puppetlabs/puppetlabs_spec_helper): shared spec helpers to setup puppet
- Fact providers
    - [rspec-puppet-facts](https://github.com/voxpupuli/rspec-puppet-facts): Simplify your unit tests by looping on every supported Operating System and populating facts.

For a list of other module development tools see [DevX Tools](https://puppetlabs.github.io/content-and-tooling-team/tools/), or from our trusted Voxpupuli community [here](https://voxpupuli.org/plugins/).

---

Unit test 最推薦的測試工具，這算是單機測試中最重要的一環，



# 6 gem 安裝 rspec-puppet
$ gem install rspec-puppet



# 7 Create new unit test with command 
https://www.puppet.com/docs/pdk/3.x/pdk_testing

PDK can create a basic unit test file that tests whether a manifest compiles to the catalog on the operating systems specified in the module's `metadata.json` file. PDK creates these unit test files when you:

- Create a new class or defined type with the `pdk new class` or `pdk new defined_type` command.
- Convert a module with the `--add-tests` option, such as `pdk convert --add-tests`.
- Create new unit tests for an existing class or defined type with the `pdk new test --unit` command.
    
The unit test file is located in your module's `/spec/classes` (for classes) or `/spec/defines` (for defined types) folder. In addition to testing whether your code compiles, this file also serves as a template for writing further unit tests to ensure that your code does what you expect it to do.


# 8 How to Run Test 


## 8.1 rake spec 

For running unit tests for a module, it is then enough to execute the command `rake spec`. 
When running the tests locally, it might be sometimes useful to run the command `rake spec_clean `when conflicts occur between fixtures.

## 8.2 pdk test unit 
https://www.puppet.com/docs/pdk/3.x/pdk_testing

The `pdk test unit` command runs all of the tests in your module's `/spec/` directory.


The `pdk test unit` command runs all the unit tests in your module.

> **Before you begin** Ensure that the `/spec/` directory contains the unit tests you want to run. Unit tests generated by PDK test only whether the manifest compiles on the module's supported operating systems, and you can write tests that test whether your code correctly performs the functions you expect it to.

1. From the command line, change into the module's directory with `cd <MODULE_NAME>`
2. Run `pdk test unit`
    To change unit test behavior, add option flags to the command. For example, to run only certain unit tests, run:
    ```
    pdk test unit --tests=<TEST1>,<TEST2>
    ```
    
    To unit test against a specific version of Puppet, add a version option flag.
    For example. To test against Puppet 5.5.12, run:
    
    ```
    pdk test unit --puppet-version 5.5.12
    ```
    

**Result:**
PDK reports what Ruby and Puppet versions it is testing against, and after tests are completed, test results.


# 9 Rspec配置


## 9.1 文件组织结构 

生成所有必备的文件 When you start out on a new module, create a metadata.json file for your module and then run `rspec-puppet-init` to create the necessary files to configure rspec-puppet for your module's tests.

The first step involved in writing unit tests for a Puppet module is to create the folder structure recommended by rspec-puppet.

![[images/Pasted image 20240119180950.png]]

In particular, to execute successfully, rspec-puppet expects a number of files in specific locations. Some of those files have a fixed name and a specific role.
- spec_helper.rb: this file contains the configuration of rspec-puppet such as the output directory for test reports, the format of the output, the gems and files to load prior to the tests etc.
- .fixtures.yml: this file contains information about the dependencies of the module and from where they can be retrieved.
- Gemfile: this file contains the list of gems necessary to run the tests and their version.
- Rakefile: this file contains a list of tasks that can be added to the standard test tasks.

```
module/
  ├── manifests/
  ├── lib/
  └── spec/
       ├── spec_helper.rb
       │
       ├── classes/
       │     └── <class_name>_spec.rb
       │
       ├── defines/
       │     └── <define_name>_spec.rb
       │
       ├── functions/
       │     └── <function_name>_spec.rb
       │
       ├── types/
       │     └── <type_name>_spec.rb
       │
       ├── type_aliases/
       │     └── <type_alias_name>_spec.rb
       │
       └── hosts/
             └── <host_name>_spec.rb
```

---

If you use the above directory structure, your examples will automatically be placed in the correct groups and have access to the custom matchers. If you choose not to, you can force the examples into the required groups as follows.

```
describe 'myclass', :type => :class do
  ...
end

describe 'mydefine', :type => :define do
  ...
end

describe 'myfunction', :type => :puppet_function do
  ...
end

describe 'mytype', :type => :type do
  ...
end

describe 'My::TypeAlias', :type => :type_alias do
  ...
end

describe 'myhost.example.com', :type => :host do
  ...
end
```


## 9.2 假定的数据给入 `spec/default_facts.yml`

<u>自己假定的 facts 的值 给入通过 default_facts.yml</u> 
It is also possible to specify default facts that are stored in a yaml file (usually called "default_facts.yml"). The location for this file can be set in spec_helper.rb.
Similarly, it is possible to set anew the values of the parameters used by the manifests in each example with  let(:params)  or to specify them in a hiera file that is referenced in the spec_helper.rb file:

## 9.3 Configuration via spec/spec_helper.rb

Configuration is typically done in a `spec/spec_helper.rb `file which each of your spec will require. Example code:

```puppet
RSpec.configure do |c|
  c.module_path     = File.join(File.dirname(File.expand_path(__FILE__)), 'fixtures', 'modules')
  c.environmentpath = File.join(Dir.pwd, 'spec')
  c.manifest        = File.join(File.dirname(File.expand_path(__FILE__)), 'fixtures', 'manifests', 'site.pp')
  # Coverage generation
  c.after(:suite) do
    RSpec::Puppet::Coverage.report!
  end
end
```

rspec-puppet can be configured by modifying the RSpec.configure block in your spec/spec_helper.rb file.
```
RSpec.configure do |c|
  c.<config option> = <value>
end
```


---


![[images/RspecHelper2.png]]

Gemfile 文件: 
At the beginning of the spec_helper.rb file, a set of gems is loaded with the keyword "require". Before executing RSpec tests for the first time, these gems must be installed. (Gemfile 这个 file 的作用) This is usually done by listing all required gems in the "Gemfile" situated at the root of the module and using the gem manager "Bundler" with the command "bundle install". 

puppetlabs_spec_helper  : 
Some of these gems are very useful to set up and execute unit tests in an easy and efficient way. This is the case for the gem "puppetlabs_spec_helper". It is particularly useful to deal with module dependencies. 

### 9.3.1 更多的 config option 

更多的 config option 见 https://github.com/puppetlabs/rspec-puppet/?tab=readme-ov-file#configuration

rspec-puppet can be configured by modifying the RSpec.configure block in your spec/spec_helper.rb file.
```
RSpec.configure do |c|
  c.<config option> = <value>
end
```

#### 9.3.1.1 module_path

[](https://github.com/puppetlabs/rspec-puppet/?tab=readme-ov-file#module_path)

|Type|Default|Puppet Version(s)|
|---|---|---|
|String|Required|any|

The path to the directory containing your Puppet modules.

#### 9.3.1.2 default_facts

[](https://github.com/puppetlabs/rspec-puppet/?tab=readme-ov-file#default_facts)

|Type|Default|Puppet Version(s)|
|---|---|---|
|Hash|`{}`|any|

A hash of default facts that should be used for all the tests.

#### 9.3.1.3 hiera_config

[](https://github.com/puppetlabs/rspec-puppet/?tab=readme-ov-file#hiera_config)

|Type|Default|Puppet Version(s)|
|---|---|---|
|String|`"/dev/null"`|any|

The path to your `hiera.yaml` file (if used).

#### 9.3.1.4 manifest

[](https://github.com/puppetlabs/rspec-puppet/?tab=readme-ov-file#manifest)

|Type|Default|Puppet Version(s)|
|---|---|---|
|String|Puppet's default value|any|

Path to test manifest. Typically `spec/fixtures/manifests/site.pp`.

#### 9.3.1.5 default_node_params

[](https://github.com/puppetlabs/rspec-puppet/?tab=readme-ov-file#default_node_params)

|Type|Default|Puppet Version(s)|
|---|---|---|
|Hash|`{}`|any|

A hash of default node parameters that should be used for all the tests.

#### 9.3.1.6 default_trusted_facts

[](https://github.com/puppetlabs/rspec-puppet/?tab=readme-ov-file#default_trusted_facts)

|Type|Default|Puppet Version(s)|
|---|---|---|
|Hash|`{}`|any|

A hash of default trusted facts that should be used for all the tests (available in the manifests as the `$trusted` hash).

#### 9.3.1.7 confdir

[](https://github.com/puppetlabs/rspec-puppet/?tab=readme-ov-file#confdir)

|Type|Default|Puppet Version(s)|
|---|---|---|
|String|`"/etc/puppet"`|any|

The path to the main Puppet configuration directory.

#### 9.3.1.8 config

[](https://github.com/puppetlabs/rspec-puppet/?tab=readme-ov-file#config)

|Type|Default|Puppet Version(s)|
|---|---|---|
|String|Puppet's default value|any|

The path to `puppet.conf`.

#### 9.3.1.9 environmentpath

[](https://github.com/puppetlabs/rspec-puppet/?tab=readme-ov-file#environmentpath)

|Type|Default|Puppet Version(s)|
|---|---|---|
|String|`"/etc/puppetlabs/code/environments"`|any|

The search path for environment directories.

#### 9.3.1.10 strict_variables

[](https://github.com/puppetlabs/rspec-puppet/?tab=readme-ov-file#strict_variables)

|Type|Default|Puppet Version(s)|
|---|---|---|
|Boolean|`false`|any|

Makes Puppet raise an error when it tries to reference a variable that hasn't been defined (not including variables that have been explicitly set to `undef`).

#### 9.3.1.11 stringify_facts

[](https://github.com/puppetlabs/rspec-puppet/?tab=readme-ov-file#stringify_facts)

|Type|Default|Puppet Version(s)|
|---|---|---|
|Boolean|`true`|any|

Makes rspec-puppet coerce all the fact values into strings (matching the behaviour of older versions of Puppet).

#### 9.3.1.12 enable_pathname_stubbing

[](https://github.com/puppetlabs/rspec-puppet/?tab=readme-ov-file#enable_pathname_stubbing)

|Type|Default|Puppet Version(s)|
|---|---|---|
|Boolean|`false`|any|

Configures rspec-puppet to stub out `Pathname#absolute?` with it's own implementation. This should only be enabled if you're running into an issue running cross-platform tests where you have Ruby code (types, providers, functions, etc) that use `Pathname#absolute?`.

#### 9.3.1.13 setup_fixtures

[](https://github.com/puppetlabs/rspec-puppet/?tab=readme-ov-file#setup_fixtures)

|Type|Default|Puppet Version(s)|
|---|---|---|
|Boolean|`true`|any|

Configures rspec-puppet to automatically create a link from the root of your module to `spec/fixtures/<module name>` at the beginning of the test run.

#### 9.3.1.14 derive_node_facts_from_nodename

[](https://github.com/puppetlabs/rspec-puppet/?tab=readme-ov-file#derive_node_facts_from_nodename)

|Type|Default|Puppet Version(s)|
|---|---|---|
|Boolean|`true`|any|

If `true`, rspec-puppet will override the `fdqn`, `hostname`, and `domain` facts with values that it derives from the node name (specified with `let(:node)`.

In some circumstances (e.g. where your nodename/certname is not the same as your FQDN), this behaviour is undesirable and can be disabled by changing this setting to `false`.

#### 9.3.1.15 facter_implementation

[](https://github.com/puppetlabs/rspec-puppet/?tab=readme-ov-file#facter_implementation)

|Type|Default|Puppet Version(s)|
|---|---|---|
|String|`facter`|any|

Configures rspec-puppet to use a specific Facter implementation for running unit tests. If the `rspec` implementation is set and Puppet does not support it, rspec-puppet will warn and fall back to the `facter` implementation. Setting an unsupported option will make rspec-puppet raise an error.

- `facter` - Use the default implementation, honoring the Facter version specified in the Gemfile
- `rspec` - Use a custom hash-based implementation of Facter defined in rspec-puppet (this provides a considerable gain in speed if tests are run with Facter 4)

## 9.4 .fixtures.yml 这个 file 的作用
Indeed, for the catalog to compile successfully, all module dependencies must be installed locally. They must therefore be retrieved at the beginning of each test run, either from a Puppet Forge (private or public) or a git repository. For this to take place,  a file named ".fixtures.yml" must be added at the root of the module and all dependencies and their versions listed.

![[Pasted image 20240120115844.png]]

In our case, we can then specify at the top of the file the URL of our internal stable Puppet Repository and retrieve all our dependencies (internal and third party dependencies) from it.  Unfortunately, puppetlabs_spec_helper does not allow to automatically retrieve the list of dependencies which is stored in the metadata file to create the .fixtures file from it. All dependencies must therefore be managed at two different places in the module, which can easily be the source of errors.


## 9.5 puppetlabs_spec_helper
puppetlabs_spec_helper comes with a set of predefined rake tasks such as "rake spec" (that executes all the tests contained in the "spec" folder) which itself contains sub-tasks such as "rake spec_prep" (that populates the "fixtures" directory with all required dependency modules), "rake spec_standalone" (that runs the actual tests) and "rake spec_clean" (that cleans the fixtures directory). 

Rakefile 的作用 
These rake tasks can be imported to the Rakefile that is located at the root of the module by adding the line "require puppetlabs_spec_helper/rake_tasks" at the top of it.



# 10 一个 `<Classname_to_betested>_spec.rb` 里面的内容 

整体文件结构

```ruby
require 'spec_helper'

describe '<function name， 但是也是 要检测的class 的名字 >' do
  ...
end
```

Specifying the name of the function to test
The name of the function must be provided in the top level description, e.g.

```ruby
describe 'split' do
```


範例 rspec-puppet 的作法
```ruby
# 4 role_spec.rb
require 'spec_helper'

describe 'role::base' do

  let(:title) { 'nginx' }

  it { is_expected.to contain_class('logrotate::setup') }

  it do
    is_expected.to contain_file('/etc/logrotate.d/nginx').with({
      'ensure' => 'present',
      'owner'  => 'root',
      'group'  => 'root',
      'mode'   => '0444',
    })
  end

  context 'with defaults for all parameters' do
    let(:facts) { global_facts }
    it do
      should contain_class('profile::base')
      should contain_class('profile::users')
    end
  end

  context 'with defaults for all parameters 2' do
    let(:facts) { global_facts }
    it do
      should contain_class('profile::base')
      should contain_class('profile::users')
    end
  end
  
end
```

這個範例會測試有 include 以下 class
include profile::base
include profile::user

# 11 rspect syntax: Matchers

https://github.com/puppetlabs/rspec-puppet/?tab=readme-ov-file#defined-types-and-classes

## 11.1 普通Matchers

### 11.1.1 Checking if the catalog compiles

You can test whether the subject catalog compiles cleanly with `compile`.

```ruby
it { is_expected.to compile }
```

To check the error messages of your class, you can check for raised error messages.

```ruby
it { is_expected.to compile.and_raise_error(/error message match/) }
```

### 11.1.2 Checking if a resource exists

You can test if a resource exists in the catalogue with the generic `contain_<resource type>` matcher.

```ruby
it { is_expected.to contain_augeas('bleh') }
```

You can also test if a class has been included in the catalogue with the same matcher.

```ruby
it { is_expected.to contain_class('foo') }
```

Note that rspec-puppet does none of the class name parsing and lookup that the puppet parser would do for you. The matcher only accepts fully qualified classnames without any leading colons. That is a class `foo::bar` will only be matched by `foo::bar`, but not by `::foo::bar`, or `bar` alone.

If your resource type includes :: (e.g. `foo::bar` simply replace the :: with __ (two underscores).

```ruby
it { is_expected.to contain_foo__bar('baz') }
```

You can further test the parameters that have been passed to the resources with the generic `with_<parameter>` chains.

```ruby
it { is_expected.to contain_package('mysql-server').with_ensure('present') }
```

If you want to specify that the given parameters should be the only ones passed to the resource, use the `only_with_<parameter>` chains.

```ruby
it { is_expected.to contain_package('httpd').only_with_ensure('latest') }
```

You can use the `with` method to verify the value of multiple parameters.

```ruby
it do
  is_expected.to contain_service('keystone').with(
    'ensure'     => 'running',
    'enable'     => 'true',
    'hasstatus'  => 'true',
    'hasrestart' => 'true'
  )
end
```

The same holds for the `only_with` method, which in addition verifies the exact set of parameters and values for the resource in the catalogue.

```ruby
it do
  is_expected.to contain_user('luke').only_with(
    'ensure' => 'present',
    'uid'    => '501'
  )
end
```

You can also test that specific parameters have been left undefined with the generic `without_<parameter>` chains.

```ruby
it { is_expected.to contain_file('/foo/bar').without_mode }
```

You can use the without method to verify that a list of parameters have not been defined

```ruby
it { is_expected.to contain_service('keystone').without(
  ['restart', 'status']
)}
```

### 11.1.3 Checking the number of resources

You can test the number of resources in the catalogue with the `have_resource_count` matcher.

```ruby
it { is_expected.to have_resource_count(2) }
```

The number of classes in the catalogue can be checked with the `have_class_count` matcher.

```ruby
it { is_expected.to have_class_count(2) }
```

You can also test the number of a specific resource type, by using the generic `have_<resource type>_resource_count` matcher.

```ruby
it { is_expected.to have_exec_resource_count(1) }
```

This last matcher also works for defined types. If the resource type contains ::, you can replace it with __ (two underscores).

```ruby
it { is_expected.to have_logrotate__rule_resource_count(3) }
```

_NOTE_: when testing a class, the catalogue generated will always contain at least one class, the class under test. The same holds for defined types, the catalogue generated when testing a defined type will have at least one resource (the defined type itself).

## 11.2 Relationship matchers

The following methods will allow you to test the relationships between the resources in your catalogue, regardless of how the relationship is defined. This means that it doesn’t matter if you prefer to define your relationships with the metaparameters (**require**, **before**, **notify** and **subscribe**) or the chaining arrows (**->**, **~>**, **<-** and **<~**), they’re all tested the same.

```ruby
it { is_expected.to contain_file('foo').that_requires('File[bar]') }
it { is_expected.to contain_file('foo').that_comes_before('File[bar]') }
it { is_expected.to contain_file('foo').that_notifies('File[bar]') }
it { is_expected.to contain_file('foo').that_subscribes_to('File[bar]') }
```

An array can be used to test a resource for multiple relationships

```ruby
it { is_expected.to contain_file('foo').that_requires(['File[bar]', 'File[baz]']) }
it { is_expected.to contain_file('foo').that_comes_before(['File[bar]','File[baz]']) }
it { is_expected.to contain_file('foo').that_notifies(['File[bar]', 'File[baz]']) }
it { is_expected.to contain_file('foo').that_subscribes_to(['File[bar]', 'File[baz]']) }
```

You can also test the reverse direction of the relationship, so if you have the following bit of Puppet code

```ruby
notify { 'foo': }
notify { 'bar':
  before => Notify['foo'],
}
```

You can test that **Notify[bar]** comes before **Notify[foo]**

```ruby
it { is_expected.to contain_notify('bar').that_comes_before('Notify[foo]') }
```

Or, you can test that **Notify[foo]** requires **Notify[bar]**

```ruby
it { is_expected.to contain_notify('foo').that_requires('Notify[bar]') }
```

## 11.3 Match target syntax

Note that this notation does not support any of the features you're used from the puppet language. Only a single resource with a single, unquoted title can be referenced here. Class names need to be always fully qualified and not have the leading `::`. It currently does not support inline arrays or quoting.

These work

- `Notify[foo]`
- `Class[profile::apache]`

These will not work

- `Notify['foo']`
- `Notify[foo, bar]`
- `Class[::profile::apache]`

### 11.3.1 Recursive dependencies

The relationship matchers are recursive in two directions:
- vertical recursion, which checks for dependencies with parents of the resource (i.e. the resource is contained, directly or not, in the class involved in the relationship). E.g. where `Package['foo']` comes before `File['/foo']`:

```puppet
class { 'foo::install': } ->
class { 'foo::config': }

class foo::install {
  package { 'foo': }
}

class foo::config {
  file { '/foo': }
}
```

- horizontal recursion, which follows indirect dependencies (dependencies of dependencies). E.g. where `Yumrepo['foo']` comes before `File['/foo']`:

```puppet
class { 'foo::repo': } ->
class { 'foo::install': } ->
class { 'foo::config': }

class foo::repo {
  yumrepo { 'foo': }
}

class foo::install {
  package { 'foo': }
}

class foo::config {
  file { '/foo': }
}
```

### 11.3.2 Autorequires
Autorequires are considered in dependency checks.

## 11.4 Type matcher

When testing custom types, the `be_valid_type` matcher provides a range of expectations:

- `with_provider(<provider_name>)`: check that the right provider was selected
- `with_properties(<property_list>)`: check that the specified properties are available
- `with_parameters(<parameter_list>)`: check that the specified parameters are available
- `with_features(<feature_list>)`: check that the specified features are available
- `with_set_attributes(<param_value_hash>)`: check that the specified attributes are set

## 11.5 Type alias matchers

When testing type aliases, the `allow_value` and `allow_values` matchers are used to check if the alias accepts particular values or not:

```ruby
describe 'MyModule::Shape' do
  it { is_expected.to allow_value('square') }
  it { is_expected.to allow_values('circle', 'triangle') }
  it { is_expected.not_to allow_value('blue') }
end
```


# 12 Writing tests: Set parameter with `let()`

https://github.com/puppetlabs/rspec-puppet/?tab=readme-ov-file#writing-tests

==使用 let() 如何在 rpect set一个新的值 ==

## 12.1 Basic test structure


To test that

```
sysctl { 'baz'
  value => 'foo',
}
```

Will cause the following resource to be in included in catalogue for a host

```
exec { 'sysctl/reload':
  command => '/sbin/sysctl -p /etc/sysctl.conf',
}
```

We can write the following testcase (in `spec/defines/sysctl_spec.rb`)

```ruby
describe 'sysctl' do
  let(:title) { 'baz' }
  let(:params) { { 'value' => 'foo' } }

  it { is_expected.to contain_exec('sysctl/reload').with_command("/sbin/sysctl -p /etc/sysctl.conf") }
end
```

## 12.2 Specifying the title of a resource: let(:title) { 'foo' }

let(:title) { 'foo' }

## 12.3 Specifying the parameters to pass to a resources or parameterised class: let(:params) 


Parameters of a defined type or class can be passed defining `:params` in a let, and passing it a hash as seen below.

```ruby
let(:params) { {'ensure' => 'present', ...} }
```

For passing Puppet's `undef` as a paremeter value, you can simply use `:undef` and it will be translated to `undef` when compiling. For example:

```ruby
let(:params) { {'user' => :undef, ...} }
```

For passing a sensitive value you can use the sensitive function with a value in brackets. For example

```ruby
let(:params) { {'password' =>sensitive('secret') } }
```

For references to nodes or resources as seen when using `require` or `before` properties, you can pass the string as an argument to the `ref` helper:

```ruby
let(:params) { 'require' => ref('Package', 'sudoku') }
```

Which translates to:

```puppet
mydefine { 'mytitle': require => Package['sudoku'] }
```

## 12.4 Specifying the FQDN of the test node let(:node)

If the manifest you're testing expects to run on host with a particular name, you can specify this as follows

```ruby
let(:node) { 'testhost.example.com' }
```

## 12.5 Specifying the environment name


If the manifest you're testing expects to evaluate the environment name, you can specify this as follows

```ruby
let(:environment) { 'production' }
```

## 12.6 Specifying the facts that should be available to your manifest


By default, the test environment contains no facts for your manifest to use. You can set them with a hash

```ruby
let(:facts) { {'operatingsystem' => 'Debian', 'kernel' => 'Linux', ...} }
```

Facts may be expressed as a value (shown in the previous example) or a structure. Fact keys may be expressed as either symbols or strings. A key will be converted to a lower case string to align with the Facter standard

```ruby
let(:facts) { {'os' => { 'family' => 'RedHat', 'release' => { 'major' => '7', 'minor' => '1', 'full' => '7.1.1503' } } } }
```

You can also create a set of default facts provided to all specs in your spec_helper:

```ruby
RSpec.configure do |c|
  c.default_facts = {
    'operatingsystem' => 'Ubuntu'
  }
end
```

Any facts you provide with `let(:facts)` in a spec will automatically be merged on top of the default facts.

## 12.7 Specifying top-scope variables that should be available to your manifest

You can create top-scope variables much in the same way as an ENC.

```ruby
let(:node_params) { { 'hostgroup' => 'webservers', 'rack' => 'KK04', 'status' => 'maintenance' } }
```

You can also create a set of default top-scope variables provided to all specs in your spec_helper:

```ruby
RSpec.configure do |c|
  c.default_node_params = {
    'owner'  => 'itprod',
    'site'   => 'ams4',
    'status' => 'live'
  }
end
```

## 12.8 Specifying extra code to load (pre-conditions)


If the manifest being tested relies on another class or variables to be set, these can be added via a pre-condition. This code will be evaluated before the tested class.

```ruby
let(:pre_condition) { 'include other_class' }
```

This may be useful when testing classes that are modular, e.g. testing `apache::mod::foo` which relies on a top-level `apache` class being included first.

The value may be a raw string to be inserted into the Puppet manifest, or an array of strings (manifest fragments) that will be concatenated.

## 12.9 Specifying extra code to load (post-conditions)

In some cases, you may need to ensure that the code that you are testing comes **before** another set of code. Similar to the `:pre_condition` hook, you can add a `:post_condition` hook that will ensure that the added code is evaluated **after** the tested class.

```ruby
let(:post_condition) { 'include other_class' }
```

This may be useful when testing classes that are modular, e.g. testing class `do_strange_things::to_the_catalog` which must come before class `foo`.

The value may be a raw string to be inserted into the Puppet manifest, or an array of strings (manifest fragments) that will be concatenated.

## 12.10 Specifying the path to find your modules


I recommend setting a default module path by adding the following code to your `spec_helper.rb`

```ruby
RSpec.configure do |c|
  c.module_path = '/path/to/your/module/dir'
end
```

However, if you want to specify it in each example, you can do so

```ruby
let(:module_path) { '/path/to/your/module/dir' }
```

## 12.11 Specifying trusted facts

The trusted facts hash will have the standard trusted fact keys (certname, domain, and hostname) populated based on the node name (as set with `:node`).

By default, the test environment contains no custom trusted facts (as usually obtained from certificate extensions) and found in the `extensions` key. If you need to test against specific custom certificate extensions you can set those with a hash. The hash will then be available in `$trusted['extensions']`

```ruby
let(:trusted_facts) { {'pp_uuid' => 'ED803750-E3C7-44F5-BB08-41A04433FE2E', '1.3.6.1.4.1.34380.1.2.1' => 'ssl-termination'} }
```

You can also create a set of default certificate extensions provided to all specs in your spec_helper:

```ruby
RSpec.configure do |c|
  c.default_trusted_facts = {
    'pp_uuid'                 => 'ED803750-E3C7-44F5-BB08-41A04433FE2E',
    '1.3.6.1.4.1.34380.1.2.1' => 'ssl-termination'
  }
end
```

## 12.12 Specifying trusted external data

The trusted facts hash will have an `external` key for trusted external data.

By default, the test environment contains no trusted external data (as usually obtained from trusted external commands and found in the `external` key). If you need to test against specific trusted external data you can set those with a hash. The hash will then be available in `$trusted['external']`

```ruby
let(:trusted_external_data) { {'foo' => 'bar'} }
```

You can also create a set of default trusted external data provided to all specs in your spec_helper:

```ruby
RSpec.configure do |c|
  c.default_trusted_external_data = {
    'foo' => 'bar'
  }
end
```

## 12.13 Testing Exported Resources

You can test if a resource was exported from the catalogue by using the `exported_resources` accessor in combination with any of the standard matchers.

You can use `exported_resources` as the subject of a child context:

```ruby
context 'exported resources' do
  subject { exported_resources }

  it { is_expected.to contain_file('foo') }
end
```

You can also use `exported_resources` directly in a test:

```ruby
it { expect(exported_resources).to contain_file('foo') }
```


# 13 Writing tests: Function 


就是在 
```
it 'titel of this function' do
	xxx 这里面可以定义多个 testcase 
end
```

https://github.com/puppetlabs/rspec-puppet/?tab=readme-ov-file#matchers-1

![[images/IvuMonitoringUnit3.png]]

---

https://rspec-puppet.com/tutorial/

```ruby
require 'spec_helper'

describe 'logrotate::rule' do
  let(:title) { 'nginx' }

  it { is_expected.to contain_class('logrotate::setup') }

  it do
    is_expected.to contain_file('/etc/logrotate.d/nginx').with({
      'ensure' => 'present',
      'owner'  => 'root',
      'group'  => 'root',
      'mode'   => '0444',
    })
  end

  context 'with compress => true' do
    let(:params) { {'compress' => true} }

    it do
      is_expected.to contain_file('/etc/logrotate.d/nginx') \
        .with_content(/^\s*compress$/)
    end
  end
end
```


## 13.1 Matchers

All of the standard RSpec matchers are available for you to use when testing Puppet functions.

```ruby
it 'should be able to do something' do
  subject.execute('foo') == 'bar'
end
```

For your convenience though, a `run` matcher exists to provide easier to understand test cases.

```ruby
it { is_expected.to run.with_params('foo').and_return('bar') }
```

## 13.2 Writing tests


#### 13.2.1.1 Specifying the arguments to pass to the function
下面的都是介绍如何 run 这个 定义的function 

You can specify the arguments to pass to your function during the test(s) using either the `with_params` chain method in the `run` matcher

```ruby
it { is_expected.to run.with_params('foo', 'bar', ['baz']) }
```

Or by using the `execute` method on the subject directly

```ruby
it 'something' do
  subject.execute('foo', 'bar', ['baz'])
end
```

### 13.2.2 Passing lambdas to the function

A lambda (block) can be passed to functions that support either a required or optional lambda by passing a block to the `with_lambda` chain method in the `run` matcher.

```ruby
it { is_expected.to run.with_lambda { |x| x * 2 }
```

### 13.2.3 Testing the results of the function

You can test the result of a function (if it produces one) using either the `and_returns` chain method in the `run` matcher

```ruby
it { is_expected.to run.with_params('foo').and_return('bar') }
```

Or by using any of the existing RSpec matchers on the subject directly

```ruby
it 'something' do
  subject.execute('foo') == 'bar'
  subject.execute('baz').should be_an Array
end
```

### 13.2.4 Testing the errors thrown by the function

You can test whether the function throws an exception using either the `and_raises_error` chain method in the `run` matcher

```ruby
it { is_expected.to run.with_params('a', 'b').and_raise_error(Puppet::ParseError) }
it { is_expected.not_to run.with_params('a').and_raise_error(Puppet::ParseError) }
```

Or by using the existing `raises_error` RSpec matcher

```ruby
it 'something' do
  expect { subject.execute('a', 'b') }.should raise_error(Puppet::ParseError)
  expect { subject.execute('a') }.should_not raise_error(Puppet::ParseError)
end
```

### 13.2.5 Accessing the parser scope where the function is running

Some complex functions require access to the current parser's scope, e.g. for stubbing other parts of the system.

```ruby
context 'when called with top-scope vars foo and bar set' do
  before do
    # :lookupvar is the method on scope that puppet calls internally to
    # resolve the value of a variable.
    allow(scope).to receive(:lookupvar).and_call_original
    allow(scope).to receive(:lookupvar).with('::foo').and_return('Hello')
    allow(scope).to receive(:lookupvar).with('::bar').and_return('World')
  end

  it { is_expected.to run.with_params().and_return('Hello World') }
end
```

Note that this does not work when testing manifests which use custom functions. Instead, you'll need to create a replacement function directly.

```ruby
before(:each) do
    Puppet::Parser::Functions.newfunction(:custom_function, :type => :rvalue) { |args|
        raise ArgumentError, 'expected foobar' unless args[0] == 'foobar'
        'expected value'
    }
end
```


# 14 Hiera integration

At some point, you might want to make use of Hiera to bring in custom parameters for your class tests. In this section, we will provide you with basic guidance to setup Hiera implementation within rspec testing. For more information on Hiera, you should check our official [documentation](https://www.puppet.com/docs/puppet/latest/hiera.html).

## 14.1 Configuration


The first step is to create the general hiera configuration file. Since we want this to be exclusive for testing, we recommend creating it inside your spec folder. Something along the lines of `spec/fixtures/hiera/hiera-rspec.yaml`. It should look something like this:

```yaml
---
version: 5
defaults:               # Used for any hierarchy level that omits these keys.
  datadir: data         # This path is relative to hiera.yaml's directory.
  data_hash: yaml_data  # Use the built-in YAML backend.

hierarchy:
  - name: 'rspec'
    path: 'rspec-data.yaml'
```

It is often recommended to use dummy data during testing to avoid real values from being entangled. In order to create these values, we will need a new file containing this data exclusively, normally existing within a subfolder called `data`, ending up with `spec/fixtures/hiera/data/rspec-data.yaml`. Here is an example of its contents:

```yaml
---
# We will be using this data in later examples
message: 'Hello world!'
dummy:message2: 'foobar' # autoloaded parameter
```

Finally, we make the target class spec file load the Hiera config, at which point we will be able to freely access it:

```ruby
let(:hiera_config) { 'spec/fixtures/hiera/hiera-rspec.yaml' }
```

Or alternatively, you could load the hiera configuration in the spec_helper to ensure it is available through all test files:

```ruby
RSpec.configure do |c|
  c.hiera_config = 'spec/fixtures/hiera/hiera-rspec.yaml'
end
```

## 14.2 Test usage examples

这个例子 用来测试 hiera 是否能正确提取我们想要的值

注意之前已经在  `spec/fixtures/hiera/data/rspec-data.yaml` 中 定义了 its contents:

```yaml
---
# We will be using this data in later examples
message: 'Hello world!'
dummy:message2: 'foobar' # autoloaded parameter
```

----


Unlike with Hiera 3, Hiera 5 comes packaged with our Puppet agent and runs during Puppet runtime. This means that it is not really possible to call the lookup function in the same way it previously worked. However, you can still test its functionality via dummy class instantiation:

The following test creates a dummny class that uses the lookup function within it. This should allow you to confirm that the lookup() function works correctly (remember that this test uses your custom hiera parameters, and not your real ones).

```ruby
context 'dummy hiera test is implemented' do
      let(:pre_condition) do
        "class dummy($message) { }
         class { 'dummy': message => lookup('message') }"
      end
      let(:hiera_config) { 'spec/fixtures/hiera/hiera-rspec.yaml' } # Only needed if the config has not been established in spec_helper

      it { is_expected.to compile }

      it 'loads ntpserver from Hiera' do
        is_expected.to contain_class('dummy').with_message('Hello world!')
      end
    end
```

The next test ensures that autoloaded parameters work correctly within your classes:

```ruby
    context 'dummy hiera test is implemented a second time' do
      let(:pre_condition) do
        "class dummy($message2) { }
        include dummy"
      end
      let(:hiera_config) { 'spec/fixtures/hiera/hiera-rspec.yaml' } # Only needed if the config has not been established in spec_helper

      it { is_expected.to compile }

      it 'loads ntpserver from Hiera' do
        is_expected.to contain_class('dummy').with_message2('foobar')
      end
    end
```

**Please note:** In-module hiera data depends on having a correct metadata.json file. It is strongly recommended that you use [metadata-json-lint](https://github.com/voxpupuli/metadata-json-lint) to automatically check your metadata.json file before running rspec.

# 15 Best Practices

## 15.1 标准的 

```ruby
# frozen_string_literal: true

require 'spec_helper'

describe 'ivu_kubernetes::extensions' do
  on_supported_os.each do |os, os_facts|
    context "on #{os}" do
      let(:facts) { os_facts }

      it { is_expected.to compile.with_all_deps }
    end
  end
end
```


## 15.2 检查
https://rspec-puppet.com/tutorial/
```
https://rspec-puppet.com/tutorial/
```

## 15.3 IVU.Monitoring: Define testing class in the sepc/classes
The tests themselves are files with names ending with "`_spec`" and are stored in the "spec" directory. They are usually testing classes and therefore stored in the sub-folder "classes", but rspec-puppet also makes it possible (and recommends) to use separate folders to test, among others, functions and types.

Our first unit test for the module ivu_monitoring will test the class init.pp and indirectly the manifests that are included it.

![[images/IvuMonitoringUnit3.png]]

describle, context, it 的作用
Compared with common frameworks such as JUnit, the keywords used by RSpec ("describe", "context") highlight the fact that tests are a form of specification. They allow us to declare what is expected from the module depending on the context of execution (with which facts or which parameters the module is compiled) in a language easy to read and understand. Each test (called "example" in the RSpec terminology) starts with the keyword "it". The keywords "describe" and "context" are used to organize examples in example groups which are equivalent to test suites in the JUnit terminology.


let 的作用: 给入自己假定的 facts 的值
Because unit tests for Puppet modules are executed in a sandbox, there are no facts (such as the version of the operating system, the name of the host...)  that can be retrieved by Puppet to decide on how to resolve a specific configuration/installation. To verify that manifests produce the desired catalogs, it is therefore necessary to mock these facts. This can easily be done with the helper method let(:facts) that takes as argument a hash of key/values. The RSpec method" let" makes it possible to set a state for the test that is only valid for the "it" block that follows. One can then add a second test to the init_spec file with another group of facts :

![[images/ivuMonitoringUnit2.png]]

<u>自己假定的 facts 的值 给入通过 default_facts.yml</u> 
It is also possible to specify default facts that are stored in a yaml file (usually called "default_facts.yml"). The location for this file can be set in spec_helper.rb.
Similarly, it is possible to set anew the values of the parameters used by the manifests in each example with  let(:params)  or to specify them in a hiera file that is referenced in the spec_helper.rb file:

## 15.4 Unit test for the OS versions supported by a module

Most of our modules support a specific set of operating systems and operating system versions. Usually this is specifies in the file metadata.json of the Puppet module: 

metadata.json
```
{
...
  "operatingsystem_support": [
    {
      "operatingsystem": "CentOS",
      "operatingsystemrelease": [ "7", "8" ]
    },
    {
      "operatingsystem": "RedHat",
      "operatingsystemrelease": [ "7", "8" ]
    }
  ],
...
 }
```


It is possible to use this information by including the following module in the **spec_helper.rb** file of the module:
```
...
require 'rspec-puppet-facts'
...
```

**rspec-puppet-facts** provides a variable on_supported_os that can be used in tests to loop through the OS versions supported by the module and to get the correct set of facts for each OS version. 

This can be used to write tests for all supported OS versions:
```
require 'spec_helper'
 
describe 'ivu_monitoring', :type => 'class' do
  on_supported_os.each do |os, facts|
    context "on #{os}" do
      let(:facts) do
        facts
      end
 
      it { is_expected.to compile }
    end
 
  end
end
```


## 15.5 Using Hiera in Unit Tests

Hier can be used in unit tests to supply parameters and test against them.

### 15.5.1 To supply parameters:
1 create a file spec/fixtures/hiera.yaml with the Hiera hirarchy definition
```

---
:backends:
  - yaml
:yaml:
  :datadir: './spec/fixtures/hiera'
:hierarchy:
  - 'default'

```

2 create a file spec/fixtures/hiera/default.yaml with the parameters to be used during the test

```
---
ivu_monitoring::params::icinga2_version: 1.2.3

```

### 15.5.2 To use hiera in you tests:

1 include the module hiera in your spec_helper.rb:
```
...
require 'hiera'
...

```


2 initialize hiera in your test spec and test against it:
init_spec 是一个 testing class 的名字 

init_spec.rb
```
let(:hiera_config) { 'spec/fixtures/hiera.yaml' }
hiera = Hiera.new( { :config => 'spec/fixtures/hiera.yaml' } )
...
#
# Versions of installed packages
#
case facts[:osfamily]
   when 'windows'
     it { is_expected.to contain_package('Icinga 2').with(
       'ensure' => hiera.lookup('ivu_monitoring::params::icinga2_version', nil, nil),
     ) }
   else
     it { is_expected.to contain_package('icinga2').with(
       'ensure' => hiera.lookup('ivu_monitoring::params::icinga2_version', nil, nil),
     ) }
   end
end
```

# 16 Producing coverage reports

[](https://github.com/puppetlabs/rspec-puppet/?tab=readme-ov-file#producing-coverage-reports)

输出报告 。 看那些Puppet resources  被测试的比较全面 

You can output a basic resource coverage report with the following in your `spec_helper.rb`

```ruby
RSpec.configure do |c|
  c.after(:suite) do
    RSpec::Puppet::Coverage.report!
  end
end
```

This checks which Puppet resources have been explicitly checked as part of the current test run and outputs both a coverage percentage and a list of untouched resources.

A desired code coverage level can be provided. If this level is not achieved, a test failure will be raised. This can be used with a CI service, such as Jenkins or Bamboo, to enforce code coverage. The following example requires the code coverage to be at least 95%.

```ruby
RSpec.configure do |c|
  c.after(:suite) do
    RSpec::Puppet::Coverage.report!(95)
  end
end
```

Resources declared outside of the module being tested (i.e. forge dependencies) are automatically removed from the coverage report.


==report 被输出在哪里？ ==









