
Classes are named blocks of Puppet code that are stored in modules and applied later when they are invoked by name. You can add classes to a node’s catalog by either declaring them in your manifests or assigning them from an external node classifier (ENC). Classes generally configure large or medium-sized chunks of functionality, such as all of the packages, configuration files, and services needed to run an application.

# 1 Defining classes

Defining a class makes it available for later use. It doesn't add any resources to the catalog — to do that, you must declare the class or assign it from an external node classifier (ENC).

Create a class by writing a class definition in a manifest (`.pp`) file. Store class manifests in the `manifests/` directory of a module. Define only one class in a manifest, and give the manifest file the same name as the class. Puppet automatically loads any classes that are present in a valid module. See [module fundamentals](https://www.puppet.com/docs/puppet/8/modules_fundamentals#modules_fundamentals "You\") to learn more about module structure and usage.

(Class 是包含 若干个 resources 的)
A class contains all of its resources. This means any relationships formed with the class as a whole is extended to every resource in the class. Every resource in a class gets automatically tagged with the class’s name and each of its namespace segments.

( Class 里面 包含 其他 class, 要使用 contain 这个 函数  )
Classes can contain other classes, but you must use the `contain` function to explicitly specify when a class is contained. For more information, see the documentation about [containing classes](https://www.puppet.com/docs/puppet/8/lang_containment#language_contained_classes "Unlike with resources, Puppet does not automatically contain classes when they are declared inside another class (by using the include function or resource-like declaration). But in certain situations, having classes contain other classes can be useful, especially in larger modules where you want to improve code readability by moving chunks of implementation into separate files."). A contained class is automatically tagged with the name of its container.

## 1.1 define class 中要包含的元素 

The general form of a class definition is:
- The `class` keyword.
- The name of the class.
- An optional parameter list, which consists of:
    - An opening parenthesis.
    - A comma-separated list of parameters, such as`String $myparam = "value"`. Each parameter consists of:
        - An optional data type, which restricts the allowed values for the parameter. If not specified, the data type defaults to `Any`.
        - A variable name to represent the parameter, including the dollar sign (`$`) prefix
        - An optional equals sign (`=`) and default value, which must match the data type, if one was specified.
    - An optional trailing comma after the last parameter.
    - A closing parenthesis.
- Optionally, the `inherits` keyword followed by a single class name.
- An opening curly brace.
- A block of arbitrary Puppet code, which generally contains at least one resource declaration.
- A closing curly brace.

## 1.2 example

this class definition specifies no parameters
```puppet
class base::linux {
  file { '/etc/passwd':
    owner => 'root',
    group => 'root',
    mode  => '0644',
  }
  file { '/etc/shadow':
    owner => 'root',
    group => 'root',
    mode  => '0440',
  }
}

```


2
This class definition creates a version parameter ($version) that accepts a String data type with a default value of 'latest'. It also includes file content from an embedded Ruby (ERB) template from the apache module. 

```
class apache (String $version = 'latest') {
  package {'httpd':
    ensure => $version, # Using the version parameter from above
    before => File['/etc/httpd.conf'],
  }
  file {'/etc/httpd.conf':
    ensure  => file,
    owner   => 'httpd',
    content => template('apache/httpd.conf.erb'), # Template from a module
  }
  service {'httpd':
    ensure    => running,
    enable    => true,
    subscribe => File['/etc/httpd.conf'],
  }
}


```

## 1.3 Class parameters and variables

Parameters allow a class to request external data. If a class needs to use data other than facts for configuration, use a parameter for that data.

You can use class parameters as normal variables inside the class definition. The values of these variables are set based on user input when the class is declared, rather than with normal assignment statements.

Supply default values for parameters whenever possible. If a class parameter lacks a default value, the parameter is considered required and the user must set a value, either in external data or as an override.

If you set a data type for each parameter, Puppet checks the parameter's value at runtime to make sure that it is the correct data type, and raises an error if the value is illegal. If you do not provide a data type for a parameter, the parameter accepts values of any data type.

The variables `$title` and `$name` are both set to the class name automatically, so you can't use them as parameters.


## 1.4 Setting class parameter defaults with Hiera data

To set class parameter defaults with Hiera data in your modules, set up a hierarchy in your module's `hiera.yaml` file and include the referenced data files in the data directory.

For example, this `hiera.yaml` file, located in the root directory of the `ntp` module, uses the operating system fact to determine which class defaults to apply to the target system. Puppet first looks for a data file that matches the operating system of the target system: `path: "os/%{facts.os.family}.yaml"`. If no matching path is found, Puppet uses defaults from the "common" data file instead.

```
# ntp/hiera.yaml
---
version: 5
defaults:
  datadir: data
  data_hash: yaml_data
hierarchy:
  - name: "OS family"
    path: "os/%{facts.os.family}.yaml"

  - name: "common"
    path: "common.yaml"
```

The files in the example below specify the default values are located in the `data` directory:
- `Debian.yaml` specifies the defaults for systems that return an operating system fact of Debian.
- `common.yaml` specifies the defaults for all other systems.

```
# ntp/data/common.yaml
---
ntp::autoupdate: false
ntp::service_name: ntpd

# ntp/data/os/Debian.yaml
ntp::service_name: ntp
```

Tip:
If you are maintaining older modules, you might encounter cases where class parameter defaults are set with a parameter class, such as `params.pp`, and class inheritance. Update such modules to use Hiera data instead. Class inheritance can have unpredictable effects and makes troubleshooting difficult. For details about updating existing params classes to Hiera data, see [data in modules](https://www.puppet.com/docs/puppet/8/hiera_migrate#adding_hiera_data_module "Modules need default values for their class parameters. Before, the preferred way to do this was the “params.pp” pattern. With Hiera 5, you can use the “data in modules” approach instead. The following example shows how to replace params.pp with the new approach.").

**Related information**  

- [Variables](https://www.puppet.com/docs/puppet/8/lang_variables#lang_variables "Variables store values so that those values can be accessed in code later.")
- [Resources](https://www.puppet.com/docs/puppet/8/lang_resources#lang_resources "Resources are the fundamental unit for modeling system configurations. Each resource describes the desired state for some aspect of a system, like a specific service or package. When Puppet applies a catalog to the target system, it manages every resource in the catalog, ensuring the actual state matches the desired state.")
- [Tags](https://www.puppet.com/docs/puppet/8/lang_tags "Tags are useful for collecting resources, analyzing reports, and restricting catalog runs. Resources, classes, and defined type instances can have multiple tags associated with them, and they receive some tags automatically.")
- [Scope](https://www.puppet.com/docs/puppet/8/lang_scope#lang_scope "A scope is a specific area of code that is partially isolated from other areas of code.")
- [Values, data types, and aliases](https://www.puppet.com/docs/puppet/8/lang_data "Most of the things you can do with the Puppet language involve some form of data. An individual piece of data is called a value, and every value has a data type, which determines what kind of information that value can contain and how you can interact with it.")
- [Namespaces and autoloading](https://www.puppet.com/docs/puppet/8/lang_namespaces#lang_namespaces "Class and defined type names can be broken up into segments called namespaces which enable the autoloader to find the class or defined type in your modules.")



# 2 Declaring classes

Declaring a class in a Puppet manifest adds all of its resources to the catalog.
在 puppet manifest 文件中 declare a class, the included class will be added to the catalog


You can declare classes in node definitions, at top scope in the site manifest, and in other classes or defined types. Classes are singletons — although a given class can behave very differently depending on how its parameters are set, the resources in it are evaluated only once per compilation. You can also assign classes to nodes with an external node classifier (ENC) .

Puppet has two main ways to declare classes: include-like and resource-like. 
- Include-like declarations are the most common; they are flexible and idempotent, so you can safely repeat them without causing errors. 
- Resource-like declarations are mostly useful if you want to pass parameters to the class but can't or don't use Hiera. 

Most ENCs assign classes with include-like behavior, but others assign them with resource-like behavior. See the [ENC interface](https://puppet.com/docs/puppet/6.4/nodes_external.html) documentation or the documentation of your specific ENC for details.

CAUTION: Do not mix include-like and resource-like declarations for a given class. If you declare or assign a class using both styles, it can cause compilation failures.

## 2.1 Include-like declarations

Include-like resource declarations allow you to declare a class multiple times — but no matter how many times you add the class, it is added to the catalog only once. This allows classes or defined types to manage their own dependencies and allows you create overlapping role classes, in which a given node can have more than one role.

Include-like behavior relies on external data and defaults for class parameter values, which allows the external data source to act like cascading configuration files for all of your classes.

You can declare a class with this behavior with one of <mark>three functions: `include`, `require`, and `contain`. </mark>

When a class is declared with an include-like declaration, Puppet takes the following actions, in order, for each of the class parameters:
1. Requests a value from the external data source, using the key `<class name>::<parameter name>`. For example, to get the `apache` class's `version` parameter, Puppet searches for `apache::version`.
2. Uses the default value, if one exists.
3. Fails compilation with an error, if no value is found.


### 2.1.1 The `include` function

The `include` function is the most common way to declare classes. Declaring a class with this function includes the class in the catalog.

> Tip: The `include` function refers only to inclusion in the catalog. You can include a class in another class's definition, but doing so does not mean one class contains the other; it only means the included class will be added to the catalog. If you want one class to contain another, use the `contain` function instead.

This function uses include-like behavior, so you can make multiple declarations and Puppet relies on external data for parameters.

The `include` function accepts one of the following:
- A single class name, such as `apache`.
- A single class reference, such as `Class['apache']`.
- A comma-separated list of class names or class references.
- An array of class names or class references.

This single class name declaration declares the class only once and has no additional effect:
```
include base::linux
```

This example declares a single class with a class reference:
```
include Class['base::linux']
```

This example declares two classes in a list:
```
include base::linux, apache
```

This example declares two classes in an array:
```
$my_classes = ['base::linux', 'apache']
include $my_classes
```


### 2.1.2 The `require` function

The `require` function declares one or more classes, then causes them to become a dependency of the surrounding container. This function uses include-like behavior, so you can make multiple declarations, and Puppet relies on external data for parameters. 

Tip: The `require` function is used to declare classes and defined types. Do not confuse it with the `require` metaparameter, which is used to establish relationships between resources.

The `require` function accepts one of the following:
- A single class name, such as `apache`.
- A single class reference, such as `Class['apache']`.
- A comma-separated list of class names or class references.
- An array of class names or class references.

In this example, Puppet ensures that every resource in the `apache` class is applied before any resource in any `apache::vhost` instance:

```
define apache::vhost (Integer $port, String $docroot, String $servername, String $vhost_name) {
  require apache
  ...
}
```


### 2.1.3 The `contain` function

The `contain` function is used inside another class definition to declare one or more classes and contain those classes in the surrounding class. This enforces ordering of classes. When you contain a class in another class, the relationships of the containing class extend to the contained class as well. For details about containment, see the documentation on [containing classes](https://www.puppet.com/docs/puppet/8/lang_containment#language_contained_classes "Unlike with resources, Puppet does not automatically contain classes when they are declared inside another class (by using the include function or resource-like declaration). But in certain situations, having classes contain other classes can be useful, especially in larger modules where you want to improve code readability by moving chunks of implementation into separate files.").

This function uses include-like behavior, so you can make multiple declarations, and Puppet relies on external data for parameters.

The `contain` function accepts one of the following:
- A single class name, such as `apache`.
- A single class reference, such as `Class['apache']`.
- A comma-separated list of class names or class references.
- An array of class names or class references.

In this example class declaration, the `ntp` class contains the `ntp::service` class. Any resource that forms a relationship with the `ntp` class also has the same relationship to the `ntp::service` class.

```
class ntp {
  file { '/etc/ntp.conf':
    ...
    require => Package['ntp'],
    notify  => Class['ntp::service'],
  }
  contain ntp::service
  package { 'ntp':
    ...
  }
}
```

For example, if a resource has a `before` relationship with the `ntp` class, that resource will also be applied before the `ntp::service` class. Similarly, any resource that forms a `require` relationship with `ntp` will be applied after `ntp::service`.

### 2.1.4 The `hiera_include` function

The legacy Hiera function `hiera_include` has been deprecated. In Hiera 5 and later, instead of using `hiera_include` use `lookup` with the `include` function. For example: use `lookup('classes', {merge => unique}).include` instead of `hiera_include('classes')`.


## 2.2 Resource-like declarations

Resource-like class declarations require that you declare a given class only once. They allow you to override class parameters at compile time — for any parameters you don't override, Puppet falls back to external data.

Resource-like declarations must be unique to avoid conflicting parameter values. Repeated overrides cause catalog compilation to be unreliable and dependent on order evaluation. This is because overridden values from the class declaration:
- Always take precedence.
- Are computed at compile time.
- Do not have a built-in hierarchy for resolving conflicts.

When a class is declared with a resource-like declaration, Puppet takes the following actions, in order, for each of the class parameters:
1. Uses the override value from the declaration, if present.
2. Requests a value from the external data source, using the key `<class name>::<parameter name>`. For example, to get the `apache` class's `version` parameter, Puppet searches for `apache::version`.
3. Uses the default value.
4. Fails compilation with an error, if no value is found.

Resource-like declarations look like normal resource declarations, using the `class` pseudo-resource type. You can provide a value for any class parameter by specifying it as a resource attribute.

You can also specify a value for any metaparameter. In such cases, every resource contained in the class will also have that metaparameter. However:

- Any resource can specifically override metaparameter values received from its container.
- Metaparameters that can take more than one value, such as the relationships metaparameters, merge the values from the container and any resource-specific values.
- You cannot apply the `noop` metaparameter to resource-like class declarations.
    

For example, this resource-like declaration declares a class with no parameters:

```
class {'base::linux':}
```

This declaration declares a class and specifies the version parameter:

```
class {'apache':
  version => '2.2.21',
}
```

