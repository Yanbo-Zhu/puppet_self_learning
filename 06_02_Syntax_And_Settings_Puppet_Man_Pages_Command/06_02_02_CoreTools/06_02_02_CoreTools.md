

[https://puppet.com/docs/puppet/7/man/agent.html](https://puppet.com/docs/puppet/7/man/agent.html)

Core tools

- [Man Page: puppet agent](https://puppet.com/docs/puppet/7/man/agent.html)
- [Man Page: puppet apply](https://puppet.com/docs/puppet/7/man/apply.html)
- [Man Page: puppet module](https://puppet.com/docs/puppet/7/man/module.html)
- [Man Page: puppet resource](https://puppet.com/docs/puppet/7/man/resource.html)
- [Man Page: puppet lookup](https://puppet.com/docs/puppet/7/man/lookup.html)



subcommand：
help             Display Puppet help.
apply             Apply Puppet manifests locally
describe       Display help about resource types
agent            The puppet agent daemon
master          The puppet master daemon
module        Creates, installs and searches formodules on the Puppet Forge




# Puppet agent

```
puppet agent [--certname <NAME>] [-D|--daemonize|--no-daemonize]
  [-d|--debug] [--detailed-exitcodes] [--digest <DIGEST>] [--disable [MESSAGE]] [--enable]
  [--fingerprint] [-h|--help] [-l|--logdest syslog|eventlog|<ABS FILEPATH>|console]
  [--serverport <PORT>] [--noop] [-o|--onetime] [--sourceaddress <IP_ADDRESS>] [-t|--test]
  [-v|--verbose] [-V|--version] [-w|--waitforcert <SECONDS>]
```


Puppet agent --test/ -t 
Puppt agent --server=192-168-1-12-jfedu.net --test
-t 的意义是
- 手动触发一次 从Server 端 pull  配置文件 然后在node 端 去实施部署 
- puppet agent --test on the agent to generate (and send) the initial cert request。That should put the agent in the certificate request list of your master.


puppet agent --help
puppet agent --enable/--disable: puppet agent --disable "my message"  --verbose  
puppet agent --configprint `<SETTING>`: To get configuration settings


# puppet apply

在单机模式下，应用资源的是apply，既编译又运行；
puppet  apply：Applies a standalone Puppet manifest to the local system.
用法：`puppetapply  [-d|--debug] [-v|--verbose][-e|--execute] [--noop] <file>`

Option

- -v: 证实运行 
	- puppet apply -v --noop user.pp #只测试，不真正执行，-v：显示详细信息
	- puppet apply -v  user.pp  #执行时将-noop去掉，-v可以将执行过程中的信息显示；
- --noop： dry run 2
	- puppet apply -v --noop your_file.pp
	- You can get Puppet to show you what it would modify without actually performing those changes by adding the --noop flag:
- --show_diff
	- To see what actual changes it would make to files, you can add the --show_diff switch. However while this simply works on a Unix system, for Windows you need to have a 3rd party diff tool and the Puppet diff setting to be properly set.
	- [https://puppet.com/docs/pe/2017.2/trouble_windows.html#diffs](https://puppet.com/docs/pe/2017.2/trouble_windows.html#diffs)

# Puppet module

Install a provider
Install the Chocolatey Package provider


![[06_02_Syntax_And_Settings_Puppet_Man_Pages_Command/06_02_02_CoreTools/images/Pasted image 20240410134541.png]]

![[06_02_Syntax_And_Settings_Puppet_Man_Pages_Command/06_02_02_CoreTools/images/Pasted image 20240410134545.png]]

# Puppet resource

- puppet resource package atom: See version of a package Puppet 
	- If you want to see what version of a package Puppet thinks is installed, you can use the Puppet Resource tool:
	- This will output a Puppet manifest with the current state of the named resource.
- puppet resource user `<name>`



# puppet lookup 


## NAME

**puppet-lookup** - Interactive Hiera lookup

## SYNOPSIS

Does Hiera lookups from the command line.

Since this command needs access to your Hiera data, make sure to run it on a node that has a copy of that data. This usually means logging into a Puppet Server node and running 'puppet lookup' with sudo.

The most common version of this command is:

'puppet lookup _KEY_ --node _NAME_ --environment _ENV_ --explain'

## USAGE

puppet lookup [--help] [--type _TYPESTRING_] [--merge first|unique|hash|deep] [--knock-out-prefix _PREFIX-STRING_] [--sort-merged-arrays] [--merge-hash-arrays] [--explain] [--environment _ENV_] [--default _VALUE_] [--node _NODE-NAME_] [--facts _FILE_] [--compile] [--render-as s|json|yaml|binary|msgpack] _keys_

## DESCRIPTION

The lookup command is a CLI for Puppet's 'lookup()' function. It searches your Hiera data and returns a value for the requested lookup key, so you can test and explore your data. It is a modern replacement for the 'hiera' command. Lookup uses the setting for global hiera.yaml from puppet's config, and the environment to find the environment level hiera.yaml as well as the resulting modulepath for the environment (for hiera.yaml files in modules). Hiera usually relies on a node's facts to locate the relevant data sources. By default, 'puppet lookup' uses facts from the node you run the command on, but you can get data for any other node with the '--node _NAME_' option. If possible, the lookup command will use the requested node's real stored facts from PuppetDB; if PuppetDB isn't configured or you want to provide arbitrary fact values, you can pass alternate facts as a JSON or YAML file with '--facts _FILE_'.

If you're debugging your Hiera data and want to see where values are coming from, use the '--explain' option.

If '--explain' isn't specified, lookup exits with 0 if a value was found and 1 otherwise. With '--explain', lookup always exits with 0 unless there is a major error.

You can provide multiple lookup keys to this command, but it only returns a value for the first found key, omitting the rest.

For more details about how Hiera works, see the Hiera documentation: https://puppet.com/docs/puppet/latest/hiera_intro.html

## OPTIONS

- --help: Print this help message.
    
- --explain Explain the details of how the lookup was performed and where the final value came from (or the reason no value was found).
    
- --node _NODE-NAME_ Specify which node to look up data for; defaults to the node where the command is run. Since Hiera's purpose is to provide different values for different nodes (usually based on their facts), you'll usually want to use some specific node's facts to explore your data. If the node where you're running this command is configured to talk to PuppetDB, the command will use the requested node's most recent facts. Otherwise, you can override facts with the '--facts' option.
    
- --facts _FILE_ Specify a .json or .yaml file of key => value mappings to override the facts for this lookup. Any facts not specified in this file maintain their original value.
    
- --environment _ENV_ Like with most Puppet commands, you can specify an environment on the command line. This is important for lookup because different environments can have different Hiera data. This environment will be always be the one used regardless of any other factors.
    
- --merge first|unique|hash|deep: Specify the merge behavior, overriding any merge behavior from the data's lookup_options. 'first' returns the first value found. 'unique' appends everything to a merged, deduplicated array. 'hash' performs a simple hash merge by overwriting keys of lower lookup priority. 'deep' performs a deep merge on values of Array and Hash type. There are additional options that can be used with 'deep'.
    
- --knock-out-prefix _PREFIX-STRING_ Can be used with the 'deep' merge strategy. Specifies a prefix to indicate a value should be removed from the final result.
    
- --sort-merged-arrays Can be used with the 'deep' merge strategy. When this flag is used, all merged arrays are sorted.
    
- --merge-hash-arrays Can be used with the 'deep' merge strategy. When this flag is used, hashes WITHIN arrays are deep-merged with their counterparts by position.
    
- --explain-options Explain whether a lookup_options hash affects this lookup, and how that hash was assembled. (lookup_options is how Hiera configures merge behavior in data.)
    
- --default _VALUE_ A value to return if Hiera can't find a value in data. For emulating calls to the 'lookup()' function that include a default.
    
- --type _TYPESTRING_: Assert that the value has the specified type. For emulating calls to the 'lookup()' function that include a data type.
    
- --compile Perform a full catalog compilation prior to the lookup. If your hierarchy and data only use the $facts, $trusted, and $server_facts variables, you don't need this option; however, if your Hiera configuration uses arbitrary variables set by a Puppet manifest, you might need this option to get accurate data. No catalog compilation takes place unless this flag is given.
    
- --render-as s|json|yaml|binary|msgpack Specify the output format of the results; "s" means plain text. The default when producing a value is yaml and the default when producing an explanation is s.
    

## EXAMPLE

To look up 'key_name' using the Puppet Server node's facts: $ puppet lookup key_name

To look up 'key_name' using the Puppet Server node's arbitrary variables from a manifest, and classify the node if applicable: $ puppet lookup key_name --compile

To look up 'key_name' using the Puppet Server node's facts, overridden by facts given in a file: $ puppet lookup key_name --facts fact_file.yaml

To look up 'key_name' with agent.local's facts: $ puppet lookup --node agent.local key_name

To get the first value found for 'key_name_one' and 'key_name_two' with agent.local's facts while merging values and knocking out the prefix 'foo' while merging: $ puppet lookup --node agent.local --merge deep --knock-out-prefix foo key_name_one key_name_two

To lookup 'key_name' with agent.local's facts, and return a default value of 'bar' if nothing was found: $ puppet lookup --node agent.local --default bar key_name

To see an explanation of how the value for 'key_name' would be found, using agent.local's facts: $ puppet lookup --node agent.local --explain key_name
