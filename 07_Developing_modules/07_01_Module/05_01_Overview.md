

https://www.puppet.com/docs/puppet/7/modules_fundamentals.html#module-structure

Modules contain Puppet classes, defined types, tasks, task plans, functions, resource types and providers, and plug-ins such as custom types or facts. 
Modules must be installed in the Puppet modulepath. Puppet loads all content from every module in the modulepath, making this code available for use.

# Module names

Module names must match the expression: `[a-z][a-z0-9_]*`. In other words, they can contain only lowercase letters, numbers, and underscores, and begin with a lowercase letter.

These restrictions are similar to those that apply to class names, with the added restriction that module names cannot contain the namespace separator (`::`), because modules cannot be nested. Certain module names are disallowed; see the list of [reserved words and names](https://www.puppet.com/docs/puppet/7/lang_reserved#lang_reserved "You can use only certain characters for naming variables, modules, classes, defined types, and other custom constructs. Additionally, some words in the Puppet language are reserved and cannot be used as bare word strings or names.").

# Manifests

Manifests, contained in the module's `manifests/` folder, each contain one class or defined type.

The `init.pp` manifest is the main class of a module and, unlike other classes or defined types, it is referred to only by the name of the module itself. For example, the class in `init.pp` in the `puppetlabs-motd` module is the `mot`d class. You cannot name a class `init`.

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

# Files in modules

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

To access files with the `file` function, pass the reference `<MODULE NAME>/<FILE NAME>` to the function, which returns the content of the requested file from the module's `files/` directory. Puppet URLs work for both `puppet agent` and `puppet apply`; in either case they retrieve the file from a module.

To learn more about the `file` function, see the [function reference](https://www.puppet.com/docs/puppet/7/function).

# Templates in modules

You can use ERB or EPP templates in your module to manage the content of configuration files. Templates combine code, data, and literal text to produce a string output, which can be used as the content attribute of a `file` resource or as a variable value. Templates are contained in the module's `templates/` directory.

For ERB templates, which use Ruby, use the `template` function. For EPP templates, which use the Puppet language, use the `epp` function. See the page about [templates](https://www.puppet.com/docs/puppet/7/lang_template "Templates are written in a specialized templating language that generates text from data. Use templates to manage the content of your Puppet configuration files via the content attribute of the file resource type.") for detailed information.

The `template` and `epp` functions look up templates identified by module and template name, passed as a string in parentheses: `function('module_name/template_name.extension')`. For example:

```
template('my_module/component.erb')
```

```
epp('my_module/component.epp')
```


