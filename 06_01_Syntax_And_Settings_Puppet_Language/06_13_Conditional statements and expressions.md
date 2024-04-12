
https://www.puppet.com/docs/puppet/7/lang_conditional#lang_conditional

条件里面可以是表达式、变量、多个表达式做逻辑运算and，or，not、有返回值的函数。
puppet 支持if 语句，case 语句和selector 语句。


# 1 if


```
if CONDITION {
    # statementlikenotice("Hello, World!")
} elsif CONDITION {
} else{
}


```


if语句支持单分支，双分支和多分支。具体语法如下
```
单分支：
	if CONDITION {
		statement
		……
	}

双分支：
	if CONDITION {
		statement
		……
	}
	else{
		statement
		……		
	}

多分支：
	if CONDITION {
		statement
		……
	}
elsif CONDITION{
	statement
	……
	}
	else{
		statement
		……		
}
```

## 1.1 例子


```
if $::timezone == 'UTC' {
  notify { 'Universal Time Coordinated': }
} elseif $::timezone == 'GMT' {
  notify { 'Greenwich Mean Time': }
} else {
  notify { "$::timezone is not UTC": }
}

============

if ($::uptime_days > 365) and ($::kernel == 'Linux') {
  …
}

if ($role == 'webserver') and ( ($datacenter == 'A') or ($datacenter == 'B') ) {
  …
}
```



其中，CONDITION的给定方式有如下三种：
```
vim if.pp
	if $operatingsystemmajrelease == '7' {
		$db_pkg='mariadb-server'
	}else{
		$db_pkg='mysql-server'
	}

	package{"$db_pkg":
		ensure => installed,
}
```


```
    $tuchao=20

    if $tuchao > 30{

        notice ('Old man')

    }

      else {

        notice ('young man')

    }
```


判断当前操作系统，选择安装合适的web程序包。

```
if  $operatingsystem =~ /(?i-mx:^(Redhat|Centos))/ {

        $webserver='httpd'

}

  elsif $operatingsystem =~ /(?i-mx:^(debain|ubuntu))/ {

        $webserver='apache2'

}

  else {

     notice ('Unknow OS')

}

 

package{"$webserver":
    ensure => installed,
}

```


# 2 case 语句

类似 if 语句，case 语句会从多个代码块中选择一个分支执行，这跟其它编程语言中的 case 语句功能一致。
case 语句会接受一个控制表达式和一组 case 代码块，并执行第一个匹配到控制表达式的块。


```
case CONTROL_EXPRESSION {
	case1: { ... }
	case2: { ... }
	case3: { ... }
	……
	default: { ... }
}

--------

case{
    case1, case2: {
}
    case3: {
}
    case4, case5, case6: {
}
}
```


其中，CONTROL_EXPRESSION的给定方式有如下三种:
变量
表达式
有返回值的函数

各case的给定方式有如下五种：
直接字串；
变量
有返回值的函数
正则表达式模式；
default



## 2.1 example

```
case $operatingsystem {

        'Solaris':      { notice ("Welcome to Solaris")}

        'RedHat','Centos': { notice("Welcome to Redhat OSFamily")}

        /(?i-mx:^(Debian|Ubuntu))/: { notice("Welcome to $1 linux")}

        default:        { notice ("Unknow OS")}

        }
```



![[06_01_Syntax_And_Settings_Puppet_Language/images/Pasted image 20240412151403.png]]


# 3 selector语句

selector只能用于期望出现直接值的地方，这包括变量赋值、资源属性、函数参数、资源标题、其他selector的值及表达式
但是不能用于一个已经嵌套于selector的case中，也不能用于一个已经嵌套于case的case语句中。

## 3.1 语法

```
CONTROL_VARIABLE ? {
	case1 => value1,
	case2 => value2,
	...
	default => valueN,
}
```

CONTROL_EXPRESSION的给定方式有如下三种:
变量
表达式
有返回值的函数


各case的给定方式有如下五种：
直接子串；
变量；
有返回值的函数；
正则表达式模式；
default


使用要点：
1、整个selector语句会被当作一个单独的值。
2、如果没有任何一个case与控制变量匹配时，puppet在编译时将会返回一个错误，因此，实践中，其必须提供default case。
3、Selector的控制变量只能是变量或有返回值的函数，不能使用表达式。
4、其各case可以是直接值（需要加引号）、变量、能调用返回值的函数，正则表达式模式或default。
5、与case语句不同的是，selector的各case不能使用列表。
6、Selector的各case的值可以是一个除了hash以为的直接值、变量、能调用返回值的函数或其它的selector。


## 3.2 例子

1 
```
Example:


$a =$operatingsystem ? {

 /(?i-mx:^(redhat|centos))/ => 'httpd',

 /(?i-mx:^(debian|ubuntu))/ => 'apache2',

}

 

notify {'notice':

        message => "Install $a",

}
```

2 
```
$rootgroup= $osfamily? {
        'Solaris'=> 'wheel',
        /(Darwin|FreeBSD)/ => 'wheel',
        default => 'root',
}
原型为:
variable = $var? {
    var1 => value1,
    var2 => value2
}
```


3
![[06_01_Syntax_And_Settings_Puppet_Language/images/Pasted image 20240412151625.png]]
```
vim selector.pp
	$pkgname = $operatingsystem ? {
		/(?i-mx:(ubuntu|debian))/       => 'apache2',
		/(?i-mx:(redhat|fedora|centos))/        => 'httpd',
		default => 'httpd',
	}
	package{"$pkgname":
		ensure  => installed,
}
```

# 4 Unless 


"Unless" statements work like reversed `if` statements. They take a Boolean condition and an arbitrary block of Puppet code, evaluate the condition, and if it is false, execute the code block. They cannot include `elsif` clauses.

**Behavior**

The condition is evaluated first and, if it is false, the code block is executed. If the condition is true, Puppet does nothing and moves on.

In addition to executing the code in a block, an `unless` statement is also an expression that produces a value, and it can be used wherever a value is allowed. The value of an `unless` expression is the value of the last expression in the executed block. If no block was executed, the value is `undef`.

**Syntax**

The general form of an `unless` statement is:

- The `unless` keyword.
    
- A condition (any expression resolving to a Boolean value).
    
- A pair of curly braces containing any Puppet code.
    
- Optionally: the `else` keyword and a pair of curly braces containing Puppet code.
    

You cannot include an `elsif` clause in an `unless` statement. If you do, compilation fails with a syntax error.

```
unless $facts['memory']['system']['totalbytes'] > 1073741824 {
  $maxclient = 500
}
```

**Conditions**

The condition of an `unless` statement can be any expression that resolves to a Boolean value. This includes:

- [Variables](https://www.puppet.com/docs/puppet/7/lang_variables#lang_variables "Variables store values so that those values can be accessed in code later.").
    
- [Expressions](https://www.puppet.com/docs/puppet/7/lang_expressions#lang_expressions "Expressions are statements that resolve to values. You can use expressions almost anywhere a value is required. Expressions can be compounded with other expressions, and the entire combined expression resolves to a single value."), including arbitrarily nested `and` and `or` expressions.
    
- [Functions](https://www.puppet.com/docs/puppet/7/lang_functions#lang_functions "Functions are plug-ins, written in Ruby, that you can call during catalog compilation. A call to any function is an expression that resolves to a value. Most functions accept one or more values as arguments, and return a resulting value.") that return values.
    

Expressions that resolve to non-Boolean values are automatically converted to Booleans. For more information, see the [Booleans documentation](https://www.puppet.com/docs/puppet/7/lang_data_boolean "Booleans are one-bit values, representing true or false. The condition of an if statement expects an expression that resolves to a Boolean value. All of Puppet\").
