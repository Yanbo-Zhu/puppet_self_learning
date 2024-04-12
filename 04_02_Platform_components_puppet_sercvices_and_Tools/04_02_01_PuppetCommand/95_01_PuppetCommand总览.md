
https://www.puppet.com/docs/puppet/8/services_commands#services_commands

![](04_02_Platform_components_puppet_sercvices_and_Tools/04_02_01_PuppetCommand/image/Pasted%20image%2020231216154124.png)

每个命令行的细则见 
obsidian://open?vault=puppet_self_learning&file=06_02_Syntax_And_Settings_Puppet_Man_Pages_Command%2F06_02_02_CoreTools%2F06_02_02_CoreTools 

# 1 puppet基本语法格式


`puppet <subcommand> [options] <action> [options]`

subcommand：
help             Display Puppet help.
apply             Apply Puppet manifests locally
describe       Display help about resource types
agent            The puppet agent daemon
master          The puppet master daemon
module        Creates, installs and searches formodules on the Puppet Forge

puppet --version  我装的是 7.13.1  


Puppet’s command line interface (CLI) consists of a single `puppet` command with many subcommands.

Puppet Server and Puppet’s companion utilities [Facter](https://puppet.com/docs/facter/3.11/index.html) and [Hiera](https://www.puppet.com/docs/puppet/8/hiera "Hiera is a built-in key-value configuration data lookup system, used for separating data from Puppet code.") , have their own CLI.

# 2 Puppet agent

Puppet agent is a core service that manages systems, with the help of a Puppet primary server. It requests a configuration catalog from a Puppet primary server server, then ensures that all resources in that catalog are in their desired state.

For more information, see:

- [Overview of Puppet’s architecture](https://www.puppet.com/docs/puppet/8/architecture "Puppet is configured in an agent-server architecture, in which a primary server node controls configuration information for a fleet of managed agent nodes.")
- Puppet [Agent on *nix systems](https://www.puppet.com/docs/puppet/8/services_agent_unix#services_agent_unix "Puppet agent is the application that manages the configurations on your nodes. It requires a Puppet primary server to fetch configuration catalogs from.")
- Puppet [Agent on Windows systems](https://www.puppet.com/docs/puppet/8/services_agent_windows#services_agent_windows "Puppet agent is the application that manages configurations on your nodes. It requires a Puppet primary server to fetch configuration catalogs.")
- Puppet [Agent’s man page](https://www.puppet.com/docs/puppet/8/man/agent)

# 3 Puppet Server

Using Puppet code and various other data sources, Puppet Server compiles configurations for any number of Puppet agents.

Puppet Server is a core service and has its own subcommand, `puppetserver`, which isn’t prefaced by the usual `puppet` subcommand.

For more information, see:

- [Overview of Puppet’s architecture](https://www.puppet.com/docs/puppet/8/architecture "Puppet is configured in an agent-server architecture, in which a primary server node controls configuration information for a fleet of managed agent nodes.")
- [Puppet Server](https://puppet.com/docs/puppetserver/latest/services_master_puppetserver.html)
- [Puppet Server subcommands](https://puppet.com/docs/puppetserver/latest/subcommands.html)

# 4 Puppet apply

Puppet apply is a core command that manages systems without contacting a Puppet primary server. Using Puppet modules and various other data sources, it compiles its own configuration catalog, and then immediately applies the catalog.

For more information, see:

- [Overview of Puppet’s architecture](https://www.puppet.com/docs/puppet/8/architecture "Puppet is configured in an agent-server architecture, in which a primary server node controls configuration information for a fleet of managed agent nodes.")
- [Puppet apply](https://www.puppet.com/docs/puppet/8/services_apply#services_apply "Puppet apply is an application that compiles and manages configurations on nodes. It acts like a self-contained combination of the Puppet primary server and Puppet agent applications.")
- [Puppet apply’s man page](https://www.puppet.com/docs/puppet/8/man/apply)

# 5 Puppet ssl

Puppet ssl is a command for managing SSL keys and certificates for Puppet SSL clients needing to communicate with your Puppetinfrastructure.

Puppet ssl usage: `puppet ssl <action> [--certname <name>]`

Possible actions:

- `submit request`: Generate a certificate signing request (CSR) and submit it to the CA. If a private and public key pair already exist, they are used to generate the CSR. Otherwise, a new key pair is generated. If a CSR has already been submitted with the given `certname,` then the operation fails.
    
- `download_cert`: Download a certificate for this host. If the current private key matches the downloaded certificate, then the certificate is saved and used for subsequent requests. If there is already an existing certificate, it is overwritten.
    
- `verify`: Verify that the private key and certificate are present and match. Verify the certificate is issued by a trusted CA, and check the revocation status
    
- `bootstrap`: Perform all of the steps necessary to request and download a client certificate. If autosigning is disabled, then puppet will wait every `waitforcert` seconds for its certificate to be signed. To only attempt once and never wait, specify a time of 0. Since `waitforcert` is a Puppet setting, it can be specified as a time interval, such as 30s, 5m, 1h.

For more information, see the [SSL man page](https://www.puppet.com/docs/puppet/8/man/ssl).

# 6 Puppet module

Puppet module is a multi-purpose administrative tool for working with Puppet modules. It can install and upgrade new modules from the Puppet [Forge](https://forge.puppetlabs.com/), help generate new modules, and package modules for public release.

For more information, see:

- [Module fundamentals](https://www.puppet.com/docs/puppet/8/modules_fundamentals#modules_fundamentals "You\")
- [Installing modules](https://www.puppet.com/docs/puppet/8/modules_installing#modules_installing "Install, upgrade, and uninstall Forge modules from the command line with the puppet module command.")
- [Publishing modules on the Puppet Forge](https://www.puppet.com/docs/puppet/8/modules_publishing#modules_publishing "To share your module with other Puppet users, get contributions to your modules, and maintain your module releases, publish your module on the Puppet Forge. The Forge is a community repository of modules, written and contributed by open source Puppet and Puppet Enterprise users.")
- Puppet [Module’s man page](https://www.puppet.com/docs/puppet/8/man/module)

# 7 Puppet resource

Puppet resource is an administrative tool that lets you inspect and manipulate resources on a system. It can work with any resource type Puppet knows about. For more information, see Puppet [Resource’s man page](https://www.puppet.com/docs/puppet/8/man/resource).

# 8 Puppet config

Puppet config is an administrative tool that lets you view and change Puppet settings.

For more information, see:

- [About Puppet’s settings](https://www.puppet.com/docs/puppet/8/config_about_settings#config_about_settings "Customize Puppet settings in the main configuration file, called puppet.conf.")
    
- [Checking values of settings](https://www.puppet.com/docs/puppet/8/config_print "Puppet settings are highly dynamic, and their values can come from several different places. To see the actual settings values that a Puppet service uses, run the puppet config print command.")
    
- [Editing settings on the command line](https://www.puppet.com/docs/puppet/8/config_set "Puppet loads most of its settings from the puppet.conf config file. You can edit this file directly, or you can change individual settings with the puppet config set command.")
    
- [Short list of important settings](https://www.puppet.com/docs/puppet/8/config_important_settings#config_important_settings "Puppet has about 200 settings, all of which are listed in the configuration reference. Most of the time, you interact with only a couple dozen of them. This page lists the most important ones, assuming that you\")
    
- Puppet [Config’s man page](https://www.puppet.com/docs/puppet/8/man/config)
    

# 9 Puppet parser

Puppet parser lets you validate Puppet code to make sure it contains no syntax errors. It can be a useful part of your continuous integration toolchain. For more information, see Puppet [Parser’s man page](https://www.puppet.com/docs/puppet/8/man/parser).

# 10 Puppet help and Puppet man

Puppet help and Puppet man can display online help for Puppet’s other subcommands.

For more information, see:

- [Puppet help’s man page](https://www.puppet.com/docs/puppet/8/man/help)
    

# 11 Full list of subcommands

For a full list of Puppet subcommands, see [Puppet’s subcommands](https://www.puppet.com/docs/puppet/8/man/overview).