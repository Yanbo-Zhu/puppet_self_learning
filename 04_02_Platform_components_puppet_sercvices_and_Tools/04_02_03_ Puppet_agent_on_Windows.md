
https://www.puppet.com/docs/puppet/8/services_agent_windows#services_agent_windows

Puppet agent is the application that manages configurations on your nodes. It requires a Puppet primary server to fetch configuration catalogs.

For more information about invoking the Puppet agent command, see the [>puppet agent man page](https://www.puppet.com/docs/puppet/8/man/agent).

# 1 Puppet agent's run environment

Puppet agent runs as a specific user, by default `LocalSystem`, and initiates outbound connections on port 8140.

## 1.1 Ports

By default, Puppet’s HTTPS traffic uses port 8140. Your operating system and firewall must allow Puppet agent to initiate outbound connections on this port.

If you want to use a non-default port, change the `serverport` setting on all agent nodes, and ensure that you change your Puppet primary server’s port as well.

## 1.2 User

Puppet agent runs as the `LocalSystem` user, which lets it manage the configuration of the entire system, but prevents it from accessing files on UNC shares.

Puppet agent can also run as a different user. You can change the user in the Service Control Manager (SCM). To start the SCM, click Start -> Run… and then enter Services.msc.

You can also specify a different user when installing Puppet. To do this, install using the CLI and specify the required MSI properties: `PUPPET_AGENT_ACCOUNT_USER`,`PUPPET_AGENT_ACCOUNT_PASSWORD`, and `PUPPET_AGENT_ACCOUNT_DOMAIN`.

Puppet agent’s user can be a local or domain user. If this user isn’t already a local administrator, the Puppet installer adds it to the `Administrators` group. The installer also grants [Logon as Service](http://msdn.microsoft.com/en-us/library/ms813948.aspx) to the user.

# 2 Managing systems with Puppet agent

In a normal Puppet configuration, every node periodically does configuration runs to revert unwanted changes and to pick up recent updates.

On Windows nodes, there are two main ways to do this:

Run Puppet as a service.
The easiest method. The Puppet agent service does configuration runs at a set interval, which can be configured.

Run Puppet agent on demand.
You can also use [Bolt](https://www.puppet.com/docs/bolt/) or deploy[MCollective](https://puppet.com/docs/mcollective/) to run on demand on many nodes.

Because the Windows version of the Puppet agent service is much simpler than the nix version, there’s no real performance to be gained by running Puppet as a scheduled task. If you want scheduled configuration runs, use the Windows service.

## 2.1 Running Puppet agent as a service

The Puppet installer configures Puppet agent to run as a Windows service and starts it. No further action is needed. Puppet agent does configuration runs at a set interval.

### 2.1.1 Configuring the run interval

The Puppet agent service defaults to doing a configuration run every 30 minutes. If you don’t need frequent configuration runs, a longer run interval lets your Puppet primary servers handle many more agent nodes.

You can configure this with the `[runinterval](https://www.puppet.com/docs/puppet/8/configuration)` setting in `puppet.conf`:

```
# C:\ProgramData\PuppetLabs\puppet\etc\puppet.conf
[agent]
  runinterval = 2h
```

After you change the run interval, the next run happens on the previous schedule, and subsequent runs happen on the new schedule.

### 2.1.2 Configuring the service start up type

设置  Puppet agent service 是否自己自动 启动 

The Puppet agent service defaults to starting automatically. If you want to start it manually or disable it, you can configure this during installation.

To do this, install using the CLI and specify the `[PUPPET_AGENT_STARTUP_MODE](https://www.puppet.com/docs/puppet/8/install_agents#install_agents "You can install agents on *nix, Windows, or macOS.")` MSI property.

You can also configure this after installation with the Service Control Manager (SCM). To start the SCM, click Start -> Run... and enter Services.msc.

You can also configure agent service with the `sc.exe` command. To prevent the service from starting on boot, run the following command from the Command Prompt (`cmd.exe`):

```
sc config puppet start= demand
```

Important: The space after `start=` is mandatory and must be run in cmd.exe. This command won’t work from PowerShell.

To stop and restart the service, run the following commands:

```
sc stop puppet
sc start puppet
```

To change the arguments used when triggering a Puppet agent run, add flags to the command:

```
sc start puppet --debug --logdest eventlog
```

This example changes the level of detail that gets written to the Event Log.

## 2.2 Running Puppet agent on demand

Some sites prefer to run Puppet agent on demand, and others occasionally need to do an on-demand run.

You can start Puppet agent runs while logged in to the target system, or remotely with Bolt or MCollective.

### 2.2.1 While logged in to the target system

On Windows, log in as an administrator, and start the configuration run by selecting Start -> Run Puppet Agent. If Windows prompts for User Account Control confirmation, click Yes. The status result of the run is shown in a command prompt window.

### 2.2.2 Running other Puppet commands

To run other Puppet-related commands, start a command prompt with administrative privileges. You can do so by right-clicking the Command Prompt or Start Command Prompts with Puppet program and clicking Run as administrator. Click Yes if the system asks for UAC confirmation.

### 2.2.3 Remotely

Open source Puppet users can use [Bolt](https://puppet.com/docs/bolt) to run tasks and commands on remote systems.

Alternatively, you can install MCollective and the [puppet agent plugin](https://github.com/puppetlabs/mcollective-puppet-agent) to get similar capabilities, but Puppet doesn't provide standalone MCollective packages for Windows.

Important: As of Puppet agent 5.5.4, MCollective is deprecated and will be removed in a future version of Puppet agent. If you use Puppet Enterprise, consider migrating from [MCollective to Puppet orchestrator](https://puppet.com/docs/pe/latest/orchestrating_puppet_and_tasks.html). If you use open source Puppet, migrate MCollective agents and filters using tools like [Bolt](https://puppet.com/docs/bolt) and the [PuppetDB Puppet Query Language.](https://puppet.com/docs/puppetdb)

# 3 Disabling and re-enabling Puppet runs

Whether you’re troubleshooting errors, working in a maintenance window, or developing in a sandbox environment, you might need to temporarily disable the Puppet agent from running.

1. Start a command prompt with Run as administrator.
2. To disable the agent, run:
    
    ```
    puppet agent --disable "<MESSAGE>"
    ```
    
3. To enable the agent, run:
    
    ```
    puppet agent --enable
    ```
    

# 4 Configuring Puppet agent on Windows

The Puppet agent comes with a default configuration that you might want to change.

Configure Puppet agent with [puppet.conf](https://www.puppet.com/docs/puppet/8/config_file_main "The puppet.conf file is Puppet’s main config file. It configures all of the Puppet commands and services, including Puppet agent, the primary Puppet server, Puppet apply, and puppetserver ca. Nearly all of the settings listed in the configuration reference can be set in puppet.conf."), using the `[agent]` section, the `[main]` section, or both. For more information on which settings are relevant to Puppet agent, see [important settings](https://www.puppet.com/docs/puppet/8/config_important_settings#config_important_settings "Puppet has about 200 settings, all of which are listed in the configuration reference. Most of the time, you interact with only a couple dozen of them. This page lists the most important ones, assuming that you\").

## 4.1 Logging for Puppet agent on Windows systems
Puppet agent logging 在哪里
如何设置其他的保存为止

When running as a service, Puppet agent logs messages to the Windows Event Log. You can view its logs by browsing the Event Viewer. Click Control Panel -> System and Security -> Administrative Tools -> Event Viewer.

By default, Puppet logs to the `Application` event log. However, you can configure Puppet to log to a separate Puppet log instead.

To enable the Puppet log, create the requisite registry key by opening a command prompt and running one of the following commands:

Bash:
```
reg add HKLM\System\CurrentControlSet\Services\EventLog\Puppet\Puppet /v EventMessageFile /t REG_EXPAND_SZ /d "%SystemRoot%\System32\EventCreate.exe"
```

PowerShell and the `New-EventLog` cmdlet:
```
if ([System.Diagnostics.Eventlog]::SourceExists("puppet")) { Remove-EventLog -Source 'puppet' } & New-EventLog -Source puppet -LogName Puppet
```

Note that for agents older than 5.5.17 on the 5.5.x stream, 6.4.4 on the 6.4.x stream and 6.8.0 on the primary server stream, use the same Bash command listed above, but the following PowerShell command instead:
```
if ([System.Diagnostics.Eventlog]::SourceExists("puppet")) { Remove-EventLog -Source 'puppet' } & New-EventLog -Source puppet -LogName Puppet  -MessageResource "%SystemRoot%\System32\EventCreate.exe" 
```

After you add the registry key, you need to reboot your machine for the logging to be redirected.

Note: If you are using an older version of Puppet, double check that you have the most up to date path to `EventCreate.exe`.

For existing agents, these commands can be placed in an exec resource to configure agents going forward.

Note: Any previously recorded event log messages are not moved; only new messages are recorded in the newly created Puppet log.

You can adjust how verbose the logs are with the [`log_level`](https://www.puppet.com/docs/puppet/8/configuration) setting, which defaults to `notice`.

When running in the foreground with the `--verbose`, `--debug`, or `--test` options, Puppet agent logs directly to the terminal.

When started with the `--logdest <FILE>` option, Puppet agent logs to the file specified by `<FILE>`. Note that there are no file size checks for the `--logdest <FILE>` option.

## 4.2 Reporting for Puppet agent on Windows systems

可以自己设置 Puppet agent 是否 report  自己的状态 to Puppet server 


In addition to local logging, Puppet agent submits a report to the primary server after each run. This can be disabled by setting `report = false` in [puppet.conf](https://www.puppet.com/docs/puppet/8/config_file_main "The puppet.conf file is Puppet’s main config file. It configures all of the Puppet commands and services, including Puppet agent, the primary Puppet server, Puppet apply, and puppetserver ca. Nearly all of the settings listed in the configuration reference can be set in puppet.conf.").

## 4.3 Setting Puppet agent CPU priority

When CPU usage is high, lower the priority of the Puppet agent service by using the [process priority](https://www.puppet.com/docs/puppet/8/configuration) setting, a cross platform configuration option. Process priority can also be set in the primary server configuration.


# 5 使用 puppet agent 中 出现的问题 
Run Puppet agent 在刚安装Puppet agent完成的时候的时候不了

https://serverfault.com/questions/845875/cant-connect-puppet-agent-to-master


![[03_Setup_Puppet_安装和配置/03_01_安装/images/Pasted image 20240412183742.png]]

"Error: Could not request certificate: Failed to open TCP connection to puppet:8140 (getaddrinfo: Name or service not known)"


这是因为 安装Puppet agent 后 
C:/ProgramData/PuppetLabs/puppet/etc/puppet.conf 文件中 

![[03_Setup_Puppet_安装和配置/03_01_安装/images/Pasted image 20240412183803.png]]

- 被自动设置成 Server= puppet  (default value for server was puppet,)  电脑无法解析 puppet 的ip adrresse. 并且我们也没有讲 puppet master 的 信息 在这个文件中匹配好
- puppet was an alias for localhost:8140
- The name puppet is not resolvable by your laptop. I assume it should be attempting to contact the puppet master at a different, fully qualifed address, specifically one that matches the subject name on the puppet master certificate
- Given the master and agent are on two separate servers it wouldn't be possible for them to communicate without having the master IP address somewhere

解决方法
- added this line to /etc/hosts： 比如 174.89.000.000 puppet （174.89.000.000 不知道是啥 ）
- 在 Puppet.conf 中 写入 [server] 区域， 并且在其中 配置好 puppet server 的ip 地址

Cert 校验
puppet server 的ip 地址 后 ，在第一次链接的时候， 会出现证书校验的步骤
when SSL is involved you use names so you can verify the name/address used against the subject name in the cert.

