
https://www.puppet.com/docs/puppet/8/lang_functions#lang_functions

什么是 Lambdas	
- Lambdas are blocks of Puppet code passed to functions.
- When a function receives a lambda, it provides values for the lambda’s parameters and evaluates its code.
- 会把 Funtion 的 values 提供给 lambada, 用作于 lambda’s parameters
- If you use other programming languages, think of lambdas as anonymous functions that are passed to other functions.


Location
- Lambdas are used only in function calls.
- Lambdas cannot be assigned to variables, and are not valid anywhere else in the Puppet language.
- While any function accepts a lambda, only some functions do anything with them.
- For information on useful lambda-accepting functions, see Iteration and loops.

Function calls: [https://puppet.com/docs/puppet/7/lang_functions.html#lang_functions](https://puppet.com/docs/puppet/7/lang_functions.html#lang_functions)
Iteration and loops:  [https://puppet.com/docs/puppet/7/lang_iteration.html](https://puppet.com/docs/puppet/7/lang_iteration.html)

# Syntax 

Lambdas consist of a list of parameters surrounded by pipe (|) characters, followed by a block of arbitrary Puppet code in curly braces. They must be used as part of a function call. 


```

$binaries = ['facter', 'hiera', 'mco', 'puppet', 'puppetserver']

# function call with lambda:
$binaries.each |String $binary| {
  file {"/usr/bin/${binary}":
    ensure => link,
    target => "/opt/puppetlabs/bin/${binary}",
  }
}

例子2
    $database_server = $terraform_output['hosts'].filter |$n, $h| {$h['role'] == 'database_server'}
      .reduce([]) |$list, $host| {
        $list  + $host[1]['fqdn']
    }

$h 就是 $terraform_output['hosts'] 中的内容 
```


The general form of a lambda is:

- A mandatory parameter list, which can be empty. This consists of:
- An opening pipe character (|).
- A comma-separated list of zero or more parameters (for example, String $myparam = "default value"). Each parameter consists of:
- An optional [data type](https://puppet.com/docs/puppet/7/lang_data_type.html#lang_data_type), which restricts the values it allows (defaults to Any).
- A [variable](https://puppet.com/docs/puppet/7/lang_variables.html#lang_variables) name to represent the parameter, including the $ prefix.
- An optional equals (=) sign and default value.
- A closing pipe character (|).
- Optionally, another comma and an extra arguments parameter (for example, String *$others = ["default one", "default two"]), which consists of:

- An optional [data type](https://puppet.com/docs/puppet/7/lang_data_type.html#lang_data_type), which restricts the values allowed for extra arguments (defaults to Any).
- An asterisk character (*).
- A [variable](https://puppet.com/docs/puppet/7/lang_variables.html#lang_variables) name to represent the parameter, including the $ prefix.
- An optional equals (=) sign and default value, which can be one value that matches the specified data type, or an array of values that all match the data type.

- An optional trailing comma after the last parameter.
- A closing pipe character (|).

- An opening curly brace.
- A block of arbitrary Puppet code.
- A closing curly brace.


# Parameters and variables


```
 $database_server = $terraform_output['hosts'].filter |$n, $h| {$h['role'] == 'database_server'}
      .reduce([]) |$list, $host| {
        $list  + $host[1]['fqdn']
    }
```

- filter 为 function , This Function are called .
- |$n, $h| 为 lambada, lambada pass value to Function

- $n, $h 为  lambda parameters。  lambda parameters的名字很重要， 顺序随便
- $h 就是 $terraform_output['hosts'] 中的内容

-  $terraform_output['hosts'] 为 发起 function 的对象。 values 提供给 lambada, 用作于 lambda’s parameters
- {} 中的内容为  A block of arbitrary Puppet code.



总览 
When functions call the lambda it sets values for the list of parameters that a lambda contains. and each parameter can be used as a variable.

Functions pass lambda parameters by position, similar to passing arguments in a function call.

Each function decides how many parameters, and in what order, it passes to a lambda. See the function’s documentation for details.

Important: The order of parameters is important and there are no restrictions on naming — unlike class or defined type parameters, where the names are the main interface for users.

（ parameters  的顺序很重要， 但是 命名不重要 ）


给不给出 lambada parameter  的 data type 随便。 如果给出了  data type 就必须按照这个data type
• Within the parameter list, the data type preceding a parameter is optional.
•  To ensure the correct data is included, Puppet checks the parameter value at runtime, and raises an error when the value is illegal. 
When no data type is provided, values of any data type are accepted by the parameter.



给不给出 lambada parameter  的 default value 随便。
When a parameter contains a default value, it’s optional — the lambda uses the default value when the caller doesn’t provide a value for that parameter.


Parameters  的顺序很重要 
Optional parameters must be poistioned after the required parameters
Important: Parameters are passed by position. Optional parameters must be poistioned after the required parameters, otherwise it causes an evaluation error. 
When you have multiple optional parameters, the later ones only receive values if all of the prior ones do.



最后一个 lambda  parameter  可以是 一个 a special extra arguments parameter
最后一个 lambda  parameter  可以是一个 收集无效多的 arguments 的 array 
The final parameter of a lambda can be a special extra arguments parameter, which collects an unlimited number of extra arguments into an array. This is useful when you don’t know in advance how many arguments the caller provides.

	• To specify that the last parameter collects extra arguments, write an asterisk (*) in front of its name in the parameter list (like *$others). 
	• An extra arguments parameter is always optional. You can’t put an asterisk (*) in front of any parameter except the last one. 
	• The value of an extra arguments parameter is always an array, containing every argument in excess of the earlier parameters. 
	• If there are no extra arguments and no default value, it will be an empty array.

An extra arguments parameter can contain a default value, which has automatic array wrapping for convenience:
	• When the provided default is a non-array value, the real default is a single-element array containing that value.
	• When the provided default is an array, the real default is that array.

An extra arguments parameter can also contain a data type. 
	• Puppet uses this data type to validate the elements of the array. 
	• When you specify a data type of String, the final data type of the extra arguments parameter will be Array[String].



# Behavior 

1 Lambda 和 defined type 的不同之处  
Similar to a defined type, a lambda delays evaluation of the Puppet code
it contains and makes it available for later. 

Unlike defined types, lambdas are not directly invoked by a user. The user provides a lambda to some other piece of code (a function), and that code decides:
	• Whether (and when) to call/evaluate the lambda.
	• How many times to call it.
	• What values its parameters must have.
	• What to do with any values it produces.

Some functions call a single lambda multiple times and provide different parameter values each time. For information on how a particular function uses its lambda, see its documentation. 
In this version of the Puppet language, calling a lambda is to pass it to a function that calls it.
( Lambda 将 值传给 Funtion )


unique resource declarations in the body of a lambda
Declarae resource 的时候 要使用 注意 他的 unique 姓 

You must use , duplicate resources cause compilation failures.

This means that when a function calls its lambda multiple times, any resource titles in the lambda must include a parameter value that changes with every call.

例子：
```
$binaries = ['facter', 'hiera', 'mco', 'puppet', 'puppetserver']
# function call with lambda:
$binaries.each |String $binary| {
  file {"/usr/bin/${binary}":
    ensure => link,
    target => "/opt/puppetlabs/bin/${binary}",
  }
}
```

"/usr/bin/$binary" 就是 resource titel . 
$binary 就是 parameter 他的值在每次互换的时候都会变 

In this example, we use the $binary parameter in the title of the lambda’s file resource:
file {"/usr/bin/$binary": 
ensure => link, 
target => "/opt/puppetlabs/bin/$binary", 
}
 
When the each function is called, the array we pass has no repeated values to ensure unique file resources. However, if we are working with an array that came from less reliable external data, we could use the unique function from stdlib to protect against duplicates. This uniqueness requirement is similar to defined types, which are also blocks of Puppet code that are evaluated multiple times



3 具有 lambda 的 function 的返回值 的类型 

• Each time a lambda is called it produces the value of the last expression in the code block 
	• 就是  |String $binary| {}  中 {} 中的内容.
• The function that calls the lambda has access to this value, but not every function does anything with it.
	• Each 就是 function 的名字 
• Some functions return it, some transform it, some ignore it, and some use it to do something else entirely.
	• Example 
	• The with function calls its lambda one time and returns the resulting value.
	• The map function calls its lambda multiple times and returns an array of every resulting value.
The each function throws away its lambda's values and returns a copy of its main argument.


```
$binaries = ['facter', 'hiera', 'mco', 'puppet', 'puppetserver']
# function call with lambda:
$binaries.each |String $binary| {
  file {"/usr/bin/${binary}":
    ensure => link,
    target => "/opt/puppetlabs/bin/${binary}",
  }
}
```


Scope of a lambda 
Every lambda creates its own local scope which is anonymous, and contains variables which can not be accessed by qualified names from any other scope. 
	• The parent scope of a lambda is the local scope in which that lambda is written. 
	• When a lambda is written inside a class definition, its code block accesses local variables from that class, as well as variables from that class’s ancestor scopes, and from the top scope.  (子lamba 可以ascc varable of parent lambda, 甚至是 variable of top scope)
Lambdas can contain other lambdas, which makes the outer lambda the parent scope of the inner one. (允许 lambda lambda)



lambda 本身是一个 value with 数据类型 Calleable 
A lambda is a value with the Callable data type, and functions using the modern function API (Puppet::Functions) use that data type to validate any lambda values it receives. 

However, the Puppet language doesn’t provide any way to store or interact with Callable values except as lambdas provided to a function. (lambdas  只能联合 function 使用， 没有其他的方法去 使用它)

