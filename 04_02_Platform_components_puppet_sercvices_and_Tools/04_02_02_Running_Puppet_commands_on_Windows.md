
https://www.puppet.com/docs/puppet/8/services_commands_windows#services_commands_windows

Puppet was originally designed to run on *nix systems, so its commands generally act the way *nix admins expect. Because Windows systems work differently, there are a few extra things to keep in mind when using Puppet commands.

# 1 Supported commands

Not all Puppet commands work on Windows. Notably, Windows nodes can’t run the `puppet server` or `puppetserver ca` commands.

The following commands are designed for use on Windows:
- [`puppet agent`](https://www.puppet.com/docs/puppet/8/man/agent)
- [`puppet apply`](https://www.puppet.com/docs/puppet/8/man/apply)
- [`puppet module`](https://www.puppet.com/docs/puppet/8/man/module)
- [`puppet resource`](https://www.puppet.com/docs/puppet/8/man/resource)
- [`puppet config`](https://www.puppet.com/docs/puppet/8/man/config)
- [`puppet lookup`](https://www.puppet.com/docs/puppet/8/man/lookup)
- [`puppet help`](https://www.puppet.com/docs/puppet/8/man/help)
# 2 Running Puppet's commands

The installer adds Puppet commands to the PATH. After installing, you can run them from any command prompt (cmd.exe) or PowerShell prompt.

Open a new command prompt after installing. Any processes that were already running before you ran the installer do not pick up the changed PATH value.

# 3 Running with administrator privileges

You usually want to run Puppet’s commands with administrator privileges.

Puppet has two privilege modes:

- Run with limited privileges, only manage certain resource types, and use a user-specific [`confdir`](https://www.puppet.com/docs/puppet/8/dirs_confdir "Puppet’s confdir is the main directory for the Puppet configuration. It contains configuration files and the SSL data.") and [codedir](https://www.puppet.com/docs/puppet/8/dirs_codedir "The codedir is the main directory for Puppet code and data. It is used by the primary Puppet server and Puppet apply, but not by Puppet agent. It contains environments (which contain your manifests and modules) and a global modules directory for all environments.")
    
- Run with administrator privileges, manage the whole system, and use the system [`confdir`](https://www.puppet.com/docs/puppet/8/dirs_confdir "Puppet’s confdir is the main directory for the Puppet configuration. It contains configuration files and the SSL data.") and [codedir](https://www.puppet.com/docs/puppet/8/dirs_codedir "The codedir is the main directory for Puppet code and data. It is used by the primary Puppet server and Puppet apply, but not by Puppet agent. It contains environments (which contain your manifests and modules) and a global modules directory for all environments.")
    

On *nix systems, Puppet defaults to running with limited privileges, when not run by `root`, but can have its privileges raised with the standard sudo command.

Windows systems don’t use sudo, so escalating privileges works differently.

Newer versions of Windows manage security with User Account Control (UAC), which was added in Windows 2008 and Windows Vista. With UAC, most programs run by administrators still have limited privileges. To get administrator privileges, the process has to request those privileges when it starts.

To run Puppet's commands in adminstrator mode, you must first start a Powershell command prompt with administrator privileges.

Right-click the Start (or apps screen tile) -> Run as administrator:  
![](https://www.puppet.com/docs/puppet/8/run_as_admin.png)  

Click Yes to allow the command prompt to run with elevated privaleges:  
![](https://www.puppet.com/docs/puppet/8/uac.png)  

The title bar on the comand prompt window begins with Administrator. This means Puppet commands that run from that window can manage the whole system.  
![](https://www.puppet.com/docs/puppet/8/windows_administrator_prompt.png)  
## 3.1 The Puppet Start menu items

Puppet’s installer adds a folder of shortcut items to the Start Menu.

  
![](https://www.puppet.com/docs/puppet/8/start_menu.png)  

These items aren’t necessary for working with Puppet, because Puppet agent runs a normal [Windows service](https://www.puppet.com/docs/puppet/8/services_agent_windows#services_agent_windows "Puppet agent is the application that manages configurations on your nodes. It requires a Puppet primary server to fetch configuration catalogs.") and the Puppet commands work from any command or PowerShell prompt. They’re provided solely as conveniences.

The Start menu items do the following:

Run Facter
This shortcut requests UAC elevation and, using the CLI, runs [Facter](https://puppet.com/docs/facter/) with administrator privileges.

Run Puppet agent
This shortcut requests UAC elevation and, using the CLI, performs a single Puppet agent command with administrator privileges.

Start Command Prompt with Puppet
This shortcut starts a normal command prompt with the working directory set to Puppet's program directory. The CLI window icon is also set to the Puppet logo. This shortcut was particularly useful in previous versions of Puppet, before Puppet's commands were added to the PATH at installation time.

Note: This shortcut does not automatically request UAC elevation; just like with a normal command prompt, you'll need to right-click the icon and choose Run as administrator.

# 4 Configuration settings

Configuration settings can be viewed and modified using the CLI.
To get configuration settings, run: `puppet agent --configprint <SETTING>`
To set configuration settings, run: `puppet config set <SETTING VALUE> --section <SECTION`>

When running Puppet commands on Windows, note the following:
- The location of `puppet.conf` depends on whether the process is running as an administrator or not.
- Specifying file owner, group, or mode for file-based settings is not supported on Windows.
- The `puppet.conf` configuration file supports Windows-style CRLF line endings as well as nix-style LF line endings. It does not support Byte Order Mark (BOM). The file encoding must either be UTF-8 or the current Windows encoding, for example, Windows-1252 code page.
- Common configuration settings are `certname`, `server`, and `runinterval`.
- You must restart the Puppet agent service after making any changes to Puppet’s `runinterval` config file setting.
