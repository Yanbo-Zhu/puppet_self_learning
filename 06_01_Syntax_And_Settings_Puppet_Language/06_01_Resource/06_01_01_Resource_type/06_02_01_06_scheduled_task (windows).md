
The scheduled_task resource can be used to create Windows Scheduled Tasks. This is the equivalent of cron in Unix. 

a scheduled task to routinely apply any puppet configuration files that exist in the folder C:\Puppet:

You may also need to configure this scheduled task to run as a user with elevated permissions, particularly if your Puppet manifests are managing packages.

```
scheduled_task { 'Run Puppet Every 5 Minutes':
  ensure    => present,
  enabled   => true,
  command   => 'C:/Program Files/Puppet Labs/Puppet/bin/puppet.bat',
  arguments => 'apply C:/puppet',
  working_dir => 'C:/Program Files/Puppet Labs/Puppet/bin/',
  trigger   => {
    schedule   => daily,
	start_time => '08:00',
    minutes_interval => 5,
    minutes_duration => 60,
  }
}

```
