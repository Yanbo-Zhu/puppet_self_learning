https://www.puppet.com/docs/puppet/8/lang_functions.html#lang_functions


Functions are plug-ins, written in Ruby, that you can call during catalog compilation. A call to any function is an expression that resolves to a value. Most functions accept one or more values as arguments, and return a resulting value.

The Ruby code in the function can do any number of things to produce the final value, including:
- Evaluate templates.
- Do mathematical calculations.
- Look up values from an external source.
- Cause side effects that modify the catalog.
- Evaluate a provided block of Puppet code, possibly using the function's arguments to modify that code or control how it runs.

Puppet includes several built-in functions. More functions are available in modules, such as `puppetlabs-stdlib`, on the [Forge](https://forge.puppet.com/). You can also write custom functions and put them in your own modules.

An entire function call—including the name, arguments, and lambda—constitutes an expression. It resolves to a single value, and can be used anywhere a value of that type is accepted. A function call might also have an effect, such as adding a class to the catalog. You can also use function calls on their own, which causes their effects to occur while their value is ignored.

All functions run during catalog compilation, which means they can access code and data only from the primary Puppet server. To make changes to an agent node, you must use a resource; to collect data from an agent node, you must use a custom fact.

Each function defines how many arguments it takes, what data types it expects those arguments to be, what values it returns, and any effects it has. For details about any functions built into Puppet, see the [function reference](https://www.puppet.com/docs/puppet/8/function). For details about a function included in a module, see that module's documentation.


# Statement functions

Statement functions are a group of built-in functions that are used only for their effects, rather than for any values. Puppet recognizes only its built-in statements; it doesn't allow adding new statement functions as plugins. The major difference between statement functions and other functions is that you can omit parentheses when calling a statement function with at least one argument, such as `include apache`.

Statement functions return a value like any other function, but they always return a value of `undef`. The built-in statement functions are:

Catalog statements
`include`: Includes the specified classes in a catalog.
`require`: Includes the specified classes in the catalog and adds them as a dependency of the current class or defined resource
`contain`: Includes the specified classes in the catalog and contains them in the current class.
`tag`: Adds the specified tag or tags to the containing class or defined resource.

Logging statements

`debug`: Logs message at the debug level.
`info`: Logs message at the info level.
`notice`: Logs message at the notice level.
`warning`: Logs message at the warning level
`err`: Logs message at the error level.

Failure statements
`fail`: Logs the error message and terminates compilation.

**Related information**  
- [Custom functions overview](https://www.puppet.com/docs/puppet/8/functions_basics#functions_basics "Puppet includes many built-in functions, and more are available in modules on the Forge. You can also write your own custom functions.")

# Functions syntax

Like any expression, a function call can be used anywhere the value it returns would be allowed. Function calls can also stand on their own, to cause their side effects, but ignore their returned value.

两种 语法： prefix calls and chained.calls
There are two ways to call functions in the Puppet language: _prefix calls_ as in `template("ntp/ntp.conf.erb")`, and _chained calls_ as in `"ntp/ntp.conf.erb".template`. There's also a modified form of prefix call that can only be used with certain functions.

The two function call styles have exactly the same capabilities, so you can choose whichever one is more readable. In general:
- For functions that take many arguments, prefix calls are easier to read.
- For functions that take one normal argument and a lambda, chained calls are easier to read.
- For a series of functions where each takes the last one's result as its argument, chained calls are easier to read, especially if at least one of those functions accepts a lambda.


Most functions have short, one-word names. The modern function API also allows qualified function names like `mymodule::myfunction`. Functions must always be called with their full names; you can't shorten a qualified function name.

## Prefix function calls

You call a function in the prefix style by writing its name and providing a list of arguments in parentheses. The general form of a prefix function call is:

```
function_name(argument, argument, ...) |$parameter, $parameter, ...| { code block }
```

- The full name of the function, as an unquoted word.
    
- An opening parenthesis `(`. Parentheses are optional when you're calling a built-in statement function with at least one argument, as in `include apache`. They're mandatory in all other cases.
    
- Zero or more arguments, separated by commas. Arguments can be any expression that resolves to a value. See each function's docs for the number of its arguments and their data types. Use the splat array operator `*` to convert an array into a comma-separated list of arguments.
    
- A closing parenthesis `)` if an opening parenthesis was used.
    
- Optionally, a lambda (code block), if the function accepts one.
    

In the following example, `template`, `include`, and `each` are all functions. The `template` function is used for its return value, `include` adds a class to the catalog, and `each` runs a block of code several times with different values.

```
file {"/etc/ntp.conf":
  ensure  => file,
  content => template("ntp/ntp.conf.erb"), # function call; resolves to a string
}

include apache # function call; modifies catalog

$binaries = [
  "facter",
  "hiera",
  "mco",
  "puppet",
  "puppetserver",
]

# function call with lambda; runs block of code several times
each($binaries) |$binary| {
  file {"/usr/bin/$binary":
    ensure => link,
    target => "/opt/puppetlabs/bin/$binary",
  }
}

```



## Chained function calls 

Alternatively, you call a function in the chained style by writing its first argument, a period, and the name of the function. The general form of a chained function call is:

```
argument.function_name(argument, ...) |$parameter, $parameter, ...| { code block }
```

- The first argument of the function, which can be any expression that resolves to a value.
- A period `.`
- The full name of the function, as an unquoted word.
- Optionally, parentheses containing a comma-separated list of additional arguments, starting with the second argument (because you already passed in the first argument). Use the splat array operator `*` to convert an array to a comma-separated list of arguments.
- Optionally, a lambda (code block), if the function accepts one.
    

In the following example, `template`, `include`, and `each` are all functions. The `template` function is used for its return value, `include` adds a class to the catalog, and `each` runs a block of code several times with different values.

```
puppet
file {"/etc/ntp.conf":
  ensure  => file,
  content => "ntp/ntp.conf.erb".template, # function call; resolves to a string
}

apache.include # function call; modifies catalog

$binaries = [
  "facter",
  "hiera",
  "mco",
  "puppet",
  "puppetserver",
]

# function call with lambda; runs block of code several times
$binaries.each |$binary| {
  file {"/usr/bin/$binary":
    ensure => link,
    target => "/opt/puppetlabs/bin/$binary",
  }
}


```


