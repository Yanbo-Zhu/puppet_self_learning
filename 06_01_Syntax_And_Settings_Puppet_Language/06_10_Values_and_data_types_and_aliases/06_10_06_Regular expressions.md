
A regular expression (sometimes shortened to “regex” or “regexp”) is a pattern that can match some set of strings, and optionally capture parts of those strings for further use.

You can use regular expression values with the `=~` and `!~` match operators, case statements and selectors, node definitions, and functions like `regsubst` for editing strings, or `match` for capturing and extracting substrings. Regular expressions act like any other value, and can be assigned to variables and used in function arguments.

# 1 Syntax

Regular expressions are written as patterns enclosed within forward slashes. Unlike in Ruby, you cannot specify options or encodings after the final slash, like `/node .*/m`.

```
if $host =~ /^www(\d+)\./ {
  notify { "Welcome web server #$1": }
}
```

Puppet uses [Ruby’s standard regular expression implementation](http://ruby-doc.org/core/Regexp.html) to match patterns. Other forms of regular expression quoting, like Ruby’s `%r{^www(\d+)\.}`, are not allowed. You cannot interpolate variables or expressions into regex values.

If you are matching against a string that contains newlines, use `\A` and `\z` instead of `^` and `$`, which match the beginning and end of a line. This is a common mistake that can cause your regexp to unintentionally match multiline text.

Some places in the language accept both real regex values and stringified regexes — that is, the same pattern quoted as a string instead of surrounded by slashes.




# 2 Regular expression options

Regular expresions in Puppet cannot have options or encodings appended after the final slash. However, you can turn options on or off for portions of the expression using the `(?<ENABLED OPTION>:<SUBPATTERN>)` and `(?-<DISABLED OPTION>:<SUBPATTERN>)` notation. The following example enables the `i` option while disabling the `m` and `x` options:

```
$packages = $operatingsystem ? {
  /(?i-mx:ubuntu|debian)/        => 'apache2',
  /(?i-mx:centos|fedora|redhat)/ => 'httpd',
}
```

使用格式：（?选项:模式）

```
　(?<ENABLED OPTION>:<PATTERN>)
　　(?-<DISABLED OPTION>:<PATTERN>)
　　OPTIONS：
　　　　i：忽略字符大小写；
　　　　m：把.当换行符；
　　　　x：忽略<PATTERN>中的空白字符；
　　(?i-mx:PATTERN）  （enable I, disable m和 x ， 以这个规则去 匹配 pattern 中对应的字符，例子  /(?i-mx:debian)/）
注意：不能赋值给变量，仅能用在接受=~或!~操作符的位置；

```


The following options are available:

i
Ignore case.

m
Treat a new line as a character matched by `.`

x
Ignore whitespace and comments in the pattern.

# 3 Regular expression capture variables

Within [conditional statements](https://www.puppet.com/docs/puppet/8/lang_conditional#lang_conditional "Conditional statements let your Puppet code behave differently in different situations. They are most helpful when combined with facts or with data retrieved from an external source. Puppet supports if and unless statements, case statements, and selectors.") and [node definitions](https://www.puppet.com/docs/puppet/8/lang_node_definitions "A node definition, also known as a node statement, is a block of Puppet code that is included only in matching nodes\"), substrings withing parentheses `()` in a regular expression are available as numbered variables inside the associated code section. The first is `$1`, the second is `$2`, and so on. The entire match is available as `$0`.

These are not normal variables, and have some special behaviors:
- The values of the numbered variables do not persist outside the code block associated with the pattern that set them.
- You can’t manually assign values to a variable with only digits in its name; they can only be set by pattern matching.
- In nested conditionals, each conditional has its own set of values for the set of numbered variables. At the end of an interior statement, the numbered variables are reset to their previous values for the remainder of the outside statement. This causes conditional statements to act like [local scopes](https://www.puppet.com/docs/puppet/8/lang_scope#lang_scope "A scope is a specific area of code that is partially isolated from other areas of code."), but only with regard to the numbered variables.

# 4 The `Regexp` data type

The data type of regular expressions is `Regexp`. By default, `Regexp` matches any regular expression value. If you are looking for a type that matches strings which match arbitrary regular expressions, see the Pattern type. You can use parameters to restrict which values `Regexp` matches.

## 4.1 Parameters

The full signature for `Regexp` is:

```
Regexp[<SPECIFIC REGULAR EXPRESSION>]
```

The parameter is optional.

    
|Position|Parameter|Data type|Default value|Description|
|---|---|---|---|---|
|1|Specific regular expression|`Regexp`|none|If specified, this results in a data type that only matches one specific regular expression value. Specifying a parameter here doesn’t have a practical use.|

Examples:

`Regexp`
Matches any regular expression.

`Regexp[/<regex>/]`
Matches the regular expression `/<regex>/` only.

`Regexp` matches only literal regular expression values. Don't confuse it with the abstract `Pattern` data type, which uses a regular expression to match a limited set of `String` values.


# 5 例子 

1 $operatingsystem =~ /(?i-mx:(macox|majaro|debain|gentoo))

- $operatingsystem是puppet的内置变量
- =~ 是字符串的模式匹配
- 其中/(?i-mx:
- 此表达式表示操作系统的名称是否匹配maosx, majaro, debain, gentoo其中的一种
- /.../是必须的, 里面的(?i-mx:可以不要, 那么就采用默认的

2  /(?i-mx:Redhat|Centos)/
选项i（忽略字符大小写），但不使用m（把.当作换行符）和x（忽略模式中的空白字符和注释）。

case $osfamily {
	"RedHat": { $webserver='httpd' }
	/(?i-mx:debian)/: { $webserver='apache2' }
	default: { $webserver='httpd' }
}

3 判断操作系统类型，选择安装Web服务器的程序包，使用notify输出到屏幕。
```
$a =$operatingsystem ? {

 /(?i-mx:^(redhat|centos))/ => 'httpd',

 /(?i-mx:^(debian|ubuntu))/ => 'apache2',

}

notify {'notice':

        message => "Install $a",

}
```


