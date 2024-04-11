
https://www.puppet.com/docs/puppet/8/lang_node_definitions

# 1 总览 

A node definition, also known as a node statement, is a block of Puppet code that is included only in matching nodes' catalogs. This allows you to assign specific configurations to specific nodes.

node 的 definition 放在 site.pp 中. Put node definitions in the main manifest, which can be a single `site.pp` file, or a directory containing many files.  (/etc/puppetlabs/code/environments/production/manifests/site.pp )

If the main manifest contains at least one node definition, it must have one for _every_ node. Compilation for a node fails if a node definition for it cannot be found. Either specify no node definitions, or use the `default` node definition, as described below, to avoid this situation.


```
## site.pp ##

## Node Definitions ##

# The default node definition matches any node lacking a more specific node
# definition. If there are no other node definitions in this file, classes
# and resources declared in the default node definition will be included in
# every node's catalog.
#
# Note that node definitions in this file are merged with node data from the
# Puppet Enterprise console and External Node Classifiers (ENC's).
#
# For more on node definitions, see: https://puppet.com/docs/puppet/latest/lang_node_definitions.html
node default {
  # This is where you can declare classes for all nodes.
  # Example:
  #   class { 'my_class': }
}

```

# 2 node definition 和 external node classifier 结合使用 

Node definitions are an optional feature of Puppet. You can use them instead of or in combination with an [external node classifier (ENC)](https://www.puppet.com/docs/puppet/8/nodes_external#writing-node-classifiers "You can classify nodes using an external node classifier (ENC), which is a script or application that tells Puppet which classes a node must have. It can replace or work in concert with the node definitions in the main site manifest (site.pp)."). Alternatively, you can use conditional statements with facts to classify nodes. Unlike more general conditional structures, node definitions match nodes only by name. By default, the name of a node is its certname, which defaults to the node's fully qualified domain name.

Although you can use node definitions in conjunction with an ENC, it's simpler to choose one method or the other. If you do use them together, Puppet merges their data as follows:
- Variables from an ENC are set at top scope and can be overridden by variables in a node definition.
- Classes from an ENC are declared at node scope, so they are affected by any variables set in the node definition.

# 3 Syntax 

Node definitions look like class definitions. The general form of a node definition is:
- The `node` keyword.
- The node definition name: a quoted string, a regular expression, or `default`.
- An opening curly brace.
- Any mixture of class declarations, variables, resource declarations, collectors, conditional statements, chaining relationships, and functions.
- A closing curly brace.

A node definition name must be one of the following:
- A quoted string containing only letters, numbers, underscores (`_`), hyphens (`-`), and periods (`.`).
- A regular expression.
- The bare word `default`. If no other node definition matches a given node, the `default` node definition will be used for that node.

1 
In the following example, only `www1.example.com` receives the `apache` and `squid` classes, and only `db1.example.com` receives the `mysql` class:

```
# <ENVIRONMENTS DIRECTORY>/<ENVIRONMENT>/manifests/site.pp
node 'www1.example.com' {
  include common
  include apache
  include squid
}
node 'db1.example.com' {
  include common
  include mysql
}
```


2 
You can use a comma-separated list of names to match a group of nodes with a single node definition: 

```
node 'www1.example.com', 'www2.example.com', 'www3.example.com' {
  include common
  include apache, squid
}
```

## 3.1 Regex 

If you use a regular expression for a node definition name, it also has the potential to match multiple nodes. For example, the following node definition matches `www1`, `www13`, and any other node whose name consists of `www` and one or more digits:

```
node /^www\d+$/ {
  include common
}
```

The following example of a regex node definition name matches `one.example.com` and `two.example.com`, but no other nodes:

```
node /^(one|two)\.example\.com$/ {
  include common
}
```


regex match 千万不要重复 
Important: Make sure all of your node definition name regexes match non-overlapping sets of node names. If a node’s name matches more than one regex, Puppet makes no guarantee about which matching definition it will get.



You can use regex capture variables by enclosing parts of your regex node definition name in parentheses `()`, and then referencing them in order as `$1`, `$2` and so on, as variables within the body of the node definition. For example:
```
node /^www(\d+)$/ {
  $wwwnumber = $1  #assigns the value of the (\d+) from a regex match to the variable $wwwnumber
}
```


# 4 Matching 

如果 a node can gets the contents of more than on  node definition, 但是规定 A given node 只能 match 一个 node definition . 这种情况下 如何能确定 那个 node definition 


A given node gets the contents of only **one** node definition, even if multiple node definitions could match its name. Puppet does the following checks, in this order, until it finds one that matches:

1. If there is a node definition with the node's exact name, Puppet uses it.
    
2. If there is a regular expression node definition that matches the node's name, Puppet uses it. If more than one regex node matches, Puppet uses one of them, but we can't predict which. Make your node definition name regexes non-overlapping to avoid this problem.
    
3. By default, the primary server's `strict_hostname_checking` is set to `true`, which means the nodes are always matched by the certname.
    
    If `strict_hostname_checking` is set to `false` and the node's name looks like a fully qualified domain name (it has multiple period-separated groups of letters, numbers, underscores, and dashes), Puppet chops off the final group and starts again at step 1. This is also true for fqdn. If the fqdn fact is not found, it will combine the hostname and domain facts.
    
4. Puppet uses the `default` node.
    

For example, when compiling a catalog for a node with certname `www01.example.com`, with fuzzy checking, Puppet looks for a node definition with the following name, in this order:

- `www01.example.com`
    
- A regex that matches `www01.example.com`
    
- `www01.example`
    
- A regex that matches `www01.example`
    
- `www01`
    
- A regex that matches `www01`
    
- `default`
    

If it doesn't find one, catalog compilation fails. It's a good idea to always have a `default` node definition.
