
https://www.puppet.com/docs/puppet/8/lang_variables#lang_variables

# 1 重点 

Variable 无法重复赋值 

Puppet allows a given variable to be assigned a value only one time within a given scope. This is a little different from most programming languages. You cannot change the value of a variable, but you can assign a different value to the same variable name in a new scope:

1、使用$开头，无论是定义还是引用：
2、变量有其作用域，不在同一个作用域需要用FQN（长格式完全限定名称）引用。
puppet 变量以“$”开头，赋值操作符为“=”，语法为$variable_name=value。

facter -p //用于显示puppet顶级作用域所有变量。


# 2 变量的种类 

puppet 种类有三种，为facts，内建变量和用户自定义变量。

facts
由facter提供；top scope;

内建变量：
　　master端变量
　　　　$servername, $serverip, $serverversion
　　agent端变量
　　　　$clientcert, $clientversion, $environment
　　parser变量
　　　　$module_name

用户自定义变量



# 3 Assigning variables

Prefix variable names with a dollar sign (`$`). Assign values to variables with the equal sign (`=`) assignment operator.

Variables accept values of any data type. You can assign literal values, or you can assign any statement that resolves to a normal value, including expressions, functions, and other variables. The variable then contains the value that the statement resolves to, rather than a reference to the statement.

```
$content = "some content\n"
```

scope of variable 只局限于 该类中
Assign variables using their short name within their own scope. You cannot assign values in one scope from another scope. For example, you can assign a value to the `apache::params` class's `$vhostdir` variable only from within the `apache::params` class.

----
**array or hash**

You can assign multiple variables at the same time from an array or hash.

To assign multiple variables from an array, you must specify an equal number of variables and values. If the number of variables and values do not match, the operation fails. You can also use nested arrays. For example, all of the variable assignments shown below are valid.

```
[$a, $b, $c] = [1,2,3]      # $a = 1, $b = 2, $c = 3
[$a, [$b, $c]] = [1,[2,3]]  # $a = 1, $b = 2, $c = 3
[$a, $b] = [1, [2]]            # $a = 1, $b = [2]
[$a, [$b]] = [1, [2]]          # $a = 1, $b = 2
```

When you assign multiple variables with a hash, list the variables in an array on the left side of the assignment operator, and list the hash on the right. Hash keys must match their corresponding variable name. This example assigns a value of 10 to the `$a` variable and a value of 20 to the `$b` variable.

```
[$a, $b] = {a => 10, b => 20}  # $a = 10, $b = 20
```

You can include extra key-value pairs in the hash, but all variables to the left of the operator must have a corresponding key in the hash:

```
[$a, $c] = {a => 5, b => 10, c => 15, d => 22}  # $a = 5, $c = 15
```


--- 


Puppet allows a given variable to be assigned a value only one time within a given scope. This is a little different from most programming languages. You cannot change the value of a variable, but you can assign a different value to the same variable name in a new scope:

```
# scope-example.pp
# Run with puppet apply --certname www1.example.com scope-example.pp
$myvar = "Top scope value"
node 'www1.example.com' {
  $myvar = "Node scope value"
  notice( "from www1: $myvar" )
  include myclass
}
node 'db1.example.com' {
  notice( "from db1: $myvar" )
  include myclass
}
class myclass {
  $myvar = "Local scope value"
  notice( "from myclass: $myvar" )
}
```

# 4 Resolution

You can use the name of a variable in any place where a value of the variable's data type would be accepted, including expressions, functions, and resource attributes. Puppet replaces the name of the variable with its value. By default, unassigned variables have a value of `undef`. See the section about unassigned variables and strict mode for more details.

In these examples, the `content` parameter value resolves to whatever value has been assigned to the `$content` variable. The `$address_array` variable resolves to an array of the values assigned to the `$address1`, `$address2`, and `$address3` variables:

```
file {'/tmp/testing':
  ensure  => file,
  content => $content,
}

$address_array = [$address1, $address2, $address3]
```

# 5 Interpolation

Puppet can resolve variables that are included in double-quoted strings; this is called interpolation. Inside a double-quoted string, surround the name of the variable (the portion after the `$`) with curly braces, such as `${var_name}`. This syntax is optional, but it helps to avoid ambiguity and allows variables to be placed directly next to non-whitespace characters. These optional curly braces are permitted only inside strings.

For example, the curly braces make it easier to quickly identify the variable `$homedir`.

```
$rule = "Allow * from $ipaddress"
file { "${homedir}/.vim":
  ensure => directory,
  ...
}
            
```

# 6 Scope

The area of code where a given variable is visible is dictated by its scope. Variables in a given scope are available only within that scope and its child scopes, and any local scope can locally override the variables it receives from its parents. See the section on [scope](https://www.puppet.com/docs/puppet/8/lang_scope#lang_scope "A scope is a specific area of code that is partially isolated from other areas of code.") for complete details.

如何读取 out-of-scope variables 的值 
You can access out-of-scope variables from named scopes by using their qualified names, which include namespaces representing the variable's scope. The scope is where the variable is defined and assigned a value. For example, the qualified name of this `$vhost` variable shows that the variable is found and assigned a value in the `apache::params` class:

```
$vhostdir = $apache::params::vhostdir
```

---
**top scope variables**

Variables can be assigned outside of any class, type, or node definition. These top scope variables have an empty string as their first namespace segment, so that the qualified name of a top scope variable begins with a double colon, such as `$::osfamily`.

# 7 Unassigned variables and strict mode

By default, you cannot access variables that have never had values assigned to them. Such variables can be a problem, because their value would be `undef` and an unassigned variable is often an accident or a typo. To stop unassigned variable usage from returning an error, enable strict mode by setting `strict_variables = false` in the `puppet.conf` file on your primary server and on any nodes that run `puppet apply`. For details about this setting, see the [configuration](https://www.puppet.com/docs/puppet/8/configuration) page.

**Related information**  
- [Expressions and operators](https://www.puppet.com/docs/puppet/8/lang_expressions#lang_expressions "Expressions are statements that resolve to values. You can use expressions almost anywhere a value is required. Expressions can be compounded with other expressions, and the entire combined expression resolves to a single value.")
- [Function calls](https://www.puppet.com/docs/puppet/8/lang_functions#lang_functions "Functions are plug-ins, written in Ruby, that you can call during catalog compilation. A call to any function is an expression that resolves to a value. Most functions accept one or more values as arguments, and return a resulting value.")

# 8 Naming variables

Some variable names are reserved; for detailed information, see the [reserved name](https://www.puppet.com/docs/puppet/8/lang_reserved#lang_reserved "You can use only certain characters for naming variables, modules, classes, defined types, and other custom constructs. Additionally, some words in the Puppet language are reserved and cannot be used as bare word strings or names.") page.

## 8.1 Variable names

Variable names are case-sensitive and must begin with a dollar sign (`$`). Most variable names must start with a lowercase letter or an underscore. The exception is regex capture variables, which are named with only numbers.

Variable names can include:
- Uppercase and lowercase letters
- Numbers
- Underscores ( `_` ). If the first character is an underscore, access that variable only from its own local scope.

Qualified variable names are prefixed with the name of their scope and the double colon (`::`) namespace separator. For example, the `$vhostdir` variable from the `apache::params` class would be `$apache::params::vhostdir`.

Optionally, the name of the very first namespace can be empty, representing the top namespace. The main reason to namespace this way is to indicate to anyone reading your code that you're accessing a top-scope variable, such as `$::is_virtual`.

You can also use a regular expression for variable names. Short variable names match the following regular expression:

```
\A\$[a-z0-9_][a-zA-Z0-9_]*\Z
```

Qualified variable names match the following regular expression:

```
\A\$([a-z][a-z0-9_]*)?(::[a-z][a-z0-9_]*)*::[a-z0-9_][a-zA-Z0-9_]*\Z
```
