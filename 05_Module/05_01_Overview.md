

https://www.puppet.com/docs/puppet/7/modules_fundamentals.html#module-structure

Modules contain Puppet classes, defined types, tasks, task plans, functions, resource types and providers, and plug-ins such as custom types or facts. 
Modules must be installed in the Puppet modulepath. Puppet loads all content from every module in the modulepath, making this code available for use.

# Module names

Module names must match the expression: `[a-z][a-z0-9_]*`. In other words, they can contain only lowercase letters, numbers, and underscores, and begin with a lowercase letter.

These restrictions are similar to those that apply to class names, with the added restriction that module names cannot contain the namespace separator (`::`), because modules cannot be nested. Certain module names are disallowed; see the list of [reserved words and names](https://www.puppet.com/docs/puppet/7/lang_reserved#lang_reserved "You can use only certain characters for naming variables, modules, classes, defined types, and other custom constructs. Additionally, some words in the Puppet language are reserved and cannot be used as bare word strings or names.").




