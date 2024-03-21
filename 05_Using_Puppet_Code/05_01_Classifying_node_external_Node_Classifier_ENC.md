https://www.puppet.com/docs/puppet/8/nodes_external#writing-node-classifiers

ENC 可以取代 node definitions in the main site manifest 
You can classify nodes using an external node classifier (ENC), which is a script or application that tells Puppet which classes a node must have. It can replace or work in concert with the node definitions in the main site manifest (`site.pp`).


script 就把 node name 作为输入量 
The external_nodes script receives the name of the node to classify as its first argument


# 1 External node classifiers

enc 的脚本 可以为 ruby的 也可以 为 其他 language
script input argument 为 node name. 
script 的输出值 为 YAML document describing the node
An external node classifier is an executable that Puppet Server or `puppet apply` can call; it doesn’t have to be written in Ruby. Its only argument is the name of the node to be classified, and it returns a YAML document describing the node.

Inside the ENC, you can reference any data source you want, including [PuppetDB](https://puppet.com/docs/puppetdb/latest/index.html). From Puppet’s perspective, the ENC submits a node name and gets back a hash of information.

External node classifiers 可以和  node definitions in `site.pp`  一起使用 
External node classifiers can co-exist with standard node definitions in `site.pp`; the classes declared in each source are merged together.

# 2 Comparing ENCs and node definitions


If you're trying to decide whether to use an ENC or main manifest node definitions (or both), consider the following:

- The YAML returned by an ENC isn’t an exact equivalent of a node definition in `site.pp` — it can’t declare individual resources, declare relationships, or do conditional logic. An ENC can only declare classes, assign top-scope variables, and set an environment. So, an ENC is most effective if you’ve done a good job of separating your configurations out into classes and modules.
    
- ENCs can set an environment for a node, overriding whatever environment the node requested.
    
- Unlike regular node definitions, where a node can match a less specific definition if an exactly matching definition isn’t found (depending on Puppet’s `strict_hostname_checking` setting), an ENC is called only once, with the node’s full name.

# 3 Connect an ENC

Configure two settings to have Puppet Server connect to an external node classifier.

/etc/puppetlabs/puppet/puppet.conf
In the primary server's `puppet.conf` file:

1. Set the `node_terminus` setting to `exec`.
2. Set the `external_nodes` setting to the path to the ENC executable.

Results

For example:

```
[server]
  node_terminus = exec
  external_nodes = /usr/local/bin/puppet_node_classifier
```



# 4 ENC output format

见 https://www.puppet.com/docs/puppet/8/nodes_external#enc_output_format