自定义一个 resource types 

Defined resource types, sometimes called defined types or defines, are blocks of Puppet code that can be evaluated multiple times with different parameters.

Create a defined resource type by writing a `define` statement in a manifest (`.pp`) file. You can declare a resource of a defined type in the same way you would declare a resource of a built-in type.

新的 resource type 被定义在 一个 新的 module 中: 
Store defined resource type manifests in the `manifests/` directory of a module. Define only one defined type in a manifest, and give the manifest file the same name as the defined type. Puppet automatically loads any defined types that are present in a valid module. See [module fundamentals](https://www.puppet.com/docs/puppet/8/modules_fundamentals#modules_fundamentals "You\") to learn more about module structure and usage.

If a defined type is present and loadable, you can declare resources of that defined type anywhere in your manifests. Declaring a new resource of the defined type causes Puppet to re-evaluate the block of code in the definition, using different values for the parameters.

Every instance of a defined type contains all of its unique resources. This means that any relationships formed between the instance and another resource are extended to every resource that makes up the instance. See the topics about [containment](https://www.puppet.com/docs/puppet/8/lang_containment#lang_containment "Containment is what controls the order in which the various parts of your Puppet code are executed. Containment is the relationship that resources have to classes and defined types, determining what has to happen before other things can happen.") and [relationships](https://www.puppet.com/docs/puppet/8/lang_relationships#lang_relationships "Resources are included and applied in the order they are defined in their manifest, but only if the resource has no implicit relationship with another resource, as this can affect the declared order. To manage a group of resources in a specific order, explicitly declare such relationships with relationship metaparameters, chaining arrows, and the require function.") for more information.

> Tip: Unlike many parts of Puppet code, define statements aren't expressions, so you can't use them where a value is expected.

# 1 Defining types

The general form of a define statement is:

- The `define` keyword.
- The name of the defined type.
- An optional parameter list, which consists of:
    - An opening parenthesis.
    - A comma-separated list of parameters, such as: `String $myparam = "default value"`. Each parameter consists of:
        - An optional data type, which restricts the allowed values for the parameter. If no data type is specified, values of any data type are permitted.
        - A variable name to represent the parameter, including the `$` prefix, such as `$parameter`.
        - An optional equals `=` sign and default value, which must match the data type, if one was specified. If no default value is specified, the parameter is considered required and the user must specify a value.
    - An optional trailing comma after the last parameter.
    - A closing parenthesis.
- An opening curly brace.
- A block of arbitrary Puppet code, which generally contains at least one resource declaration
- A closing curly brace

The definition does not cause the code in the block to be added to the catalog; it only makes it available. To add the code to the catalog, you must declare one or more resources of the defined type.

This example creates a new resource type called `apache::vhost`:
```
# /etc/puppetlabs/puppet/modules/apache/manifests/vhost.pp
define apache::vhost (
  Integer $port,
  String[1] $docroot,
  String[1] $servername = $title,
  String $vhost_name = '*',
) {
  include apache # contains package['httpd'] and service['httpd']
  include apache::params # contains common config settings

  $vhost_dir = $apache::params::vhost_dir

  # the template used below can access all of the parameters and variable from above.
  file { "${vhost_dir}/${servername}.conf":
    ensure  => file,
    owner   => 'www',
    group   => 'www',
    mode    => '0644',
    content => template('apache/vhost-default.conf.erb'),
    require  => Package['httpd'],
    notify    => Service['httpd'],
  }
}
```


# 2 Declaring defined type resources

就是 在其他地方 使用 defined type resources 

You can declare instances of a defined type—usually just called resources—the same way you declare any other resource: with a resource type, a title, and a set of attribute-value pairs. The parameters you added when defining the type, such as `$port`, become resource attributes, such as `port`, when you declare resources of the defined type.

Parameters that have a default value are considered optional parameters: if you don't specify them in the resource declaration, the default value is used. Parameters without defaults are required parameters, and you must specify a value for them when you declare the resource.

To declare a resource of the `apache::vhost` defined type from the example above:

```
apache::vhost {'homepages':
 port    => 8081,
 docroot => '/var/www-testhost', 
}
```

If a defined type is present and loadable, you can declare resources of that defined type anywhere in your manifests. Declaring a new resource of the defined type causes Puppet to re-evaluate the block of code in the definition, using the new declaration's values for the parameters.

Just as with a normal resource type, you can declare resource defaults for a defined type. In this example, every `apache::vhost` resource defaults to port 80 unless specifically overridden:

```
# /etc/puppetlabs/puppet/manifests/site.pp 
Apache::Vhost {
 port => 80,
}
```

You can include any metaparameter in the declaration of a defined type instance. If you do:
- Every resource contained in the resource declaration also has that metaparameter. Metaparameters that can accept more than one value, such as the relationship metaparameters, merge the values from the container and any specific values from the individual resource.
- The value of the metaparameter can be used as a variable in the definition, as though it were a normal parameter. For example, in an instance declared with `require => Class['ntp']`, the local value of `$require` would be `Class['ntp']`.


# 3 Naming

Defined type names can consist of one or more namespace segments, which indicate the defined type's location in a module. Each segment must adhere to the [naming and reserved names](https://www.puppet.com/docs/puppet/8/lang_reserved#lang_reserved_words) guidelines.

Each namespace segment must be capitalized when writing a resource reference, collector, or resource default. For example, a reference to the `apache::vhost` resource would be `Apache::Vhost['homepages']`.

Because you can declare multiple instances of a defined type in your manifests, every resource in the definition must be different in every instance. Duplicate resource instances result in compilation failures with a "duplicate resource declaration" error. To make resources different across instances, include the value of `$title` or another parameter in the resource's title and name.

Because `$title` is unique per instance, this ensures the resources are unique as well. For example, this segment of a file declaration makes resources unique by adding the `vhost_dir` and `servername` attributes to the resource title:

```
file { "${vhost_dir}/${servername}.conf": 
```

# 4 Parameters and attributes

When you create a defined type, you can precede each parameter in the define statement with an optional data type. If you include a data type, Puppet checks the resource parameter's value at runtime to make sure that it has the right data type; if the value is illegal, Puppet raises an error. If you don't specify a data type in the definition statement, the parameter accepts values of any data type.

You can use the parameters of a defined type as local variables inside the definition. Rather than the usual assignment statement, each instance of the defined type uses its parameter attributes to set the value of the variable. In this example declaration, the value of the `port` parameter, 8081, becomes the value assigned to the `$port` variable. Likewise, the path for the `docroot` parameter becomes the value for the `$docroot` variable.

```
apache::vhost {'homepages': 
  port    => 8081,  
  docroot => '/var/www-testhost', 
}
```

Note:

The `$title` and `$name` variables are both set to the defined type's name automatically, so they cannot be used as parameters.


## 4.1 `$title` and `$name`

The `$title` and `$name` attributes are always available to a defined type and are not explicitly added to the definition. These attributes are both set to the defined type's name automatically:

- `$title` is always set to the title of the instance. Because it is always unique for each instance, it is useful for making sure that contained resources are unique.
- `$name` defaults to the value of `$title`. You can specify a different value when you declare an instance of the defined type, but this is rarely useful.

Because the values of `$title` and `$name` are already available inside the defined type's parameter list, you can use `$title` as all or part of the default value for another attribute. In this example, `$title` is used as the value of `$servername` to ensure the server name is always unique:

```
define apache::vhost ( 
  Integer $port, 
  String[1] $docroot, 
  String $servername = $title, 
  String[1] $vhost_name = '*', 
) { # ...
```
