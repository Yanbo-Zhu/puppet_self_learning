
https://confluence.ivu.de/display/SYS/How+to+write+acceptance+tests+for+puppet+modules

This tutorial explains how to write acceptance tests for Puppet modules and takes the module ivu_monitoring is an example. We use here the test harness Beaker and the standard test framework beaker-rspec.

To run theses tests locally on a Windows machine, you can follow the instructions to install the necessary software in a Windows Linux Subsystem: [How to install a puppet developer environment in a Windows Linux Subsystem](https://confluence.ivu.de/display/SYS/How+to+install+a+puppet+developer+environment+in+a+Windows+Linux+Subsystem)


# 1 What are acceptance tests

Contrary to [unit tests](https://confluence.ivu.de/display/SYS/How+to+write+unit+tests+for+Puppet+modules), during acceptance tests, puppet manifests are applied on real machines - test nodes - in the same way as they are applied on a machine in production. All operations that take place during this process are happening for real: packages are retrieved from repositories and installed, security keys are exchanged, services are set up and started etc.

Beaker 的作用: 
The test harness Beaker is responsible for **creating the nodes, provisioning them and destroying them at the end of tests**. 

beaker-rspec: contains Rspec and Serverspec 
The test framework beaker-rspec is an extension of the test framework RSpec that has been designed especially for the acceptance testing of Puppet modules. It includes the framework Serverspec, that containes many helper methods to test infrastructure.

## 1.1 Create the necessary folder structure

The first step involved in writing acceptance tests for a Puppet module is to create the folder structure recommended by beaker-rspec:

![[images/folder_structure_acc.png]]

In particular, to execute successfully,  beaker-rspec expects a number of files in specific locations. Some of those files have a fixed name and a specific role.

- spec_helper_acceptance.rb: this file contains the configuration of beaker-rspec and in particular the steps to be executed before the tests are run (copying files or modules on the SUT - system under test - for example)
- acceptance: acceptance tests have to be stored in that directory. Those tests are specified in files which name ends with _spec
- nodesets: in this director are stored the files that specify which kind of machines should be used for the tests: which hypervisor (Docker or Vagrant for exemple), which operating system etc. It normally contains a default.yml file in which the default configuration is specified.

It also presupposes that the files Gemfile and Rakefile exist (normally created while writing [unit tests](https://confluence.ivu.de/display/SYS/How+to+write+unit+tests+for+Puppet+modules)). The Gemfile must contain the gems beaker and, depending on the hypervisor, the  beaker-vagrant and/or beaker-docker gem.

It is possible to run Hiera data to run the tests. In this case, the data can be stored in a config/data directory. The exact location of this file can be defined in the spec_helper_acceptance.rb file.



# 2 Write the nodesets files

大部分情况用 docker 去创建 node. 但有些情况 需要用到 vagrant 去 创建. 

Beaker acceptance tests are run on virtual systems (virtual machines or containers) that are created before the tests and typically destroyed afterwards. In the Beaker documentation, these are referred to as SUTs (system under test). Part of the test configuration implies specifying which hypervisor should be used to build the SUT and the operating system that should be installed on it. This configuration is stored in the files placed under the directory spec/nodesets. Among others, Beaker makes it possible to use Vagrant and Docker to spin up SUTs, and these two options are the most commonly used.

---
用 docker: 

We use docker in the CI because Docker containers are quicker to spin up than traditional virtual machines. Docker containers do not run a full-blown operating system. They are a lightweight version of a VM  shipped without standard services such as cron or systemd. While this brings a clear advantage in terms of speed, this can generate problems as the SUTs might slightly differ from real systems. 
However,  it is normally possible to install the missing software/services on the container to make it behave like a full-blown VM when needed (with the option "docker_image_commands", see example below). The default node for the acceptance tests of the monitoring Puppet modules is a Docker container on which CentOS 7 is installed.

The configuration for this default node is stored in the default.yml file in the nodesets directory:
![[98_puppet_test_tools/images/nodeset_default.png]]

---

用 vagrant

ivu_monitoring needs access to cron, which is not typically installed on a docker container. It is here installed with the package cronie.

For testing locally we typically use a vagrant node:
![[98_puppet_test_tools/images/vagrant nodeset.png]]

With the option "box", we specify the vagrant box that will can be retrieved from the Vagrant Hub (the equivalent of the Docker Hub). In that case, we do not have to install extra software as it is included in the operating system. It is also possible to run tests with multiple hypervisors by placing one nodeset file per hypervisor in the nodeset directory (for example a vagrant.yml and a docker.yml file). When using the command "rake beaker", the default configuration is used (usually docker), otherwise it is necessary to specify the nodeset by running "rake beaker:vagrant"

# 3 Write the spec_helper_acceptance file

The only files to be affected by a change in hypervisor are the files included in the nodeset directory. The rest of the setup is specified in the spec_helper_acceptance.rb file and stays constant whatever the hypervisor. This file plays a similar role for acceptance tests as the file spec_helper.rb for unit tests. 
It defines the configuration for the tests and particularly, in the *"before :suite"* part, the steps needed to prepare the SUT(s) before the tests are executed: which files should be copied on it, which dependencies installed etc. 

Some Puppet modules can only been tested on a machine if previous software is present on it, and this installation step should take place before the tests are executed. As it is possible to specify in spec_helper_acceptance.rb any (shell) command that should be run during setup, including Puppet manifests applied to the system, very complex environments can be produced as part of this setup step.

In the case of ivu_monitoring, to prepare the SUT for the tests, the latest version of Puppet 5 must be installed on it. To make this installation easier, the gem Beaker provides the helper method "run_puppet_install_helper". 

Per default, this method installs the latest version of Puppet (currently Puppet 6), but this behavior can be changed with the environment variables BEAKER_PUPPET_AGENT_VERSION (to install a specific minor version) or BEAKER_PUPPET_COLLECTION (to install the latest minor version of a major release). In our case, we can set this variable to BEAKER_PUPPET_COLLECTION=puppet5.

![[98_puppet_test_tools/images/spechelperacceptance2.png]]

<u>上一个截图中 23,24 行, 拷贝 module and data files 到 rigt place for testing </u>
To test the module on the SUT, we use the standalone version of Puppet rather than the distributed version, as the later option would require to spin up a Puppet master along the monitoring host(s), which would slow down the process and is not strictly necessary at this level of testing. 
For a standalone use of Puppet, all Puppet modules (the module under test and its dependencies) must be stored in the directory /etc/puppetlabs/code/environment/production/modules and the hiera data in the directory /etc/puppetlabs/code/environment/production/data on the SUT. Copying these files to the right place on the SUT is therefore part of the necessary setup. 
The gem Beaker offers the method *scp_to* to copy files on the SUT. This method can take as an argument a list of files to ignore so it is possible to speed up the copying process by excluding unnecessary files (lines 23-28).

---
<u>Install module dependecies: </u>
The gem beaker also provides a helper method "install_module_dependencies". However, the latter only fetches dependency modules if they are stored on the official Forge. 
As some of the dependencies of the module ivu_monitoring are internal and can only be retrieved from our internal Puppet Forge, we can not use this method. But we can use the "puppet install module" method and specify the url of the local repository with the option "--module_repository" (lines 34-41).

![[98_puppet_test_tools/images/acceptancehelper4.png]]


# 4 Write the tests

When all the steps specified in the "before :suite" part of the RSpec configuration are executed, the tests that are stored in the folder "acceptance" are executed.


![[98_puppet_test_tools/images/acceptance_test.png]]


Here, we use the RSpec let helper method to create on the SUT a Puppet manifest (a file with the extension .pp) which will contain the line specified within the let(:pp) method (line 4-6). Usually, this manifest only contains the name of the class/module to be tested but any Puppet code can be added to it. 
The first test "should apply without errors" (line 9-12) simply applies the manifest on the SUT and fails if it can not be applied successfully. 

The second test ("package icinga2 should be installed", line 15-17) uses the Serverspec framework which translates this expectation to a ssh command to actually check if this package is present on the machine. 

Tests three and four ("service icinga 2 should be installed" and "service icinga 2 should be running") also translate to a command that checks the status of that service on the SUT (for example with "systemctl status icinga2" on a host running a Red Hat Linux distribution - the actual command will depend on the operating system of the SUT).

<u>command "rake beaker" to run all test</u>
To run all acceptance tests of a module, one can simply use the command "rake beaker". It is also possible to run the tests with a specific nodeset with the command `"rake beaker:<name of the nodeset file>"`

<u>是否删除用于测试才产生的nodes</u>
At the end of all tests, all VMs and containers are normally destroyed. However, Beaker offers the possibility to use an environment variable, *BEAKER_destroy* , that can be set to "no" or "onpass". 
The advantage of setting this environment variable in this way is that one can then log in on the SUT with the command *"rake beaker:ssh"* to inspect the machine and understand what went wrong when a test failed. However, this setting should normally only be used for troubleshooting reasons, as not destroying the VMs /containers at the end of each test suite can quickly provoke space shortage problems.

