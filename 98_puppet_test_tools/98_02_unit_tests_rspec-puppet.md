https://confluence.ivu.de/display/SYS/How+to+write+unit+tests+for+Puppet+modules

https://www.puppet.com/docs/puppet/7/bgtm#writing_modules_overview-modules-containment


# 1 总览

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


# 3 

https://rspec-puppet.com/

Unit test 最推薦的測試工具，這算是單機測試中最重要的一環，

gem 安裝 rspec-puppet
$ gem install rspec-puppet


範例 rspec-puppet 的作法
```
# 4 role_spec.rb
require 'spec_helper'

describe 'role::base' do
  context 'with defaults for all parameters' do
    let(:facts) { global_facts }
    it do
      should contain_class('profile::base')
      should contain_class('profile::users')
    end
  end
End
```

這個範例會測試有 include 以下 class
include profile::base
include profile::user

# 4 **How to write them**

The first step involved in writing unit tests for a Puppet module is to create the folder structure recommended by rspec-puppet.

![[images/Pasted image 20240119180950.png]]

In particular, to execute successfully, rspec-puppet expects a number of files in specific locations. Some of those files have a fixed name and a specific role.
- spec_helper.rb: this file contains the configuration of rspec-puppet such as the output directory for test reports, the format of the output, the gems and files to load prior to the tests etc.
- .fixtures.yml: this file contains information about the dependencies of the module and from where they can be retrieved.
- Gemfile: this file contains the list of gems necessary to run the tests and their version.
- Rakefile: this file contains a list of tasks that can be added to the standard test tasks.

## 4.1 Define testing class in the sepc/classes
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


## 4.2 spec_helper.rb

![[images/RspecHelper2.png]]

Gemfile 文件: 
At the beginning of the spec_helper.rb file, a set of gems is loaded with the keyword "require". Before executing RSpec tests for the first time, these gems must be installed. (Gemfile 这个 file 的作用) This is usually done by listing all required gems in the "Gemfile" situated at the root of the module and using the gem manager "Bundler" with the command "bundle install". 

puppetlabs_spec_helper  : 
Some of these gems are very useful to set up and execute unit tests in an easy and efficient way. This is the case for the gem "puppetlabs_spec_helper". It is particularly useful to deal with module dependencies. 

## 4.3 .fixtures.yml 这个 file 的作用
Indeed, for the catalog to compile successfully, all module dependencies must be installed locally. They must therefore be retrieved at the beginning of each test run, either from a Puppet Forge (private or public) or a git repository. For this to take place,  a file named ".fixtures.yml" must be added at the root of the module and all dependencies and their versions listed.

![[Pasted image 20240120115844.png]]

In our case, we can then specify at the top of the file the URL of our internal stable Puppet Repository and retrieve all our dependencies (internal and third party dependencies) from it.  Unfortunately, puppetlabs_spec_helper does not allow to automatically retrieve the list of dependencies which is stored in the metadata file to create the .fixtures file from it. All dependencies must therefore be managed at two different places in the module, which can easily be the source of errors.


## 4.4 puppetlabs_spec_helper
puppetlabs_spec_helper comes with a set of predefined rake tasks such as "rake spec" (that executes all the tests contained in the "spec" folder) which itself contains sub-tasks such as "rake spec_prep" (that populates the "fixtures" directory with all required dependency modules), "rake spec_standalone" (that runs the actual tests) and "rake spec_clean" (that cleans the fixtures directory). 

Rakefile 的作用 
These rake tasks can be imported to the Rakefile that is located at the root of the module by adding the line "require puppetlabs_spec_helper/rake_tasks" at the top of it.


# 5 How to Run Test 
For running unit tests for a module, it is then enough to execute the command "rake spec". 
When running the tests locally, it might be sometimes useful to run the command *rake spec_clean* when conflicts occur between fixtures.


# 6 Best Practices

## 6.1 Unit test for the OS versions supported by a module

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


## 6.2 Using Hiera in Unit Tests

Hier can be used in unit tests to supply parameters and test against them.

### 6.2.1 To supply parameters:
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

### 6.2.2 To use hiera in you tests:

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



# 7 Common errors and possible solutions

"Errno::ENOENT: No such file or directory - git"
try to install git on the system ("apt install git")




# 8 IVU 内部puppet module 的 unittest 

[14:37] Florian Lamprecht
https://git.ivu-ag.com/projects/PUPPET/repos/ivu_ptp_software/browse/spec/defines
