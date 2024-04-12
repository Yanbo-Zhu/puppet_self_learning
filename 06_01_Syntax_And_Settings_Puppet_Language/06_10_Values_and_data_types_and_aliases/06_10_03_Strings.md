
https://www.puppet.com/docs/puppet/8/lang_data_string#lang_data_string

# 1 Single-quoted strings 中的 

`path => 'e:\\PuppetTest\\norex_dataSource_templateFile.csv'`

Multi-word strings can be surrounded by single quotation marks, 'like this'.

1 例子 
if $autoupdate {
  notice('autoupdate parameter has been deprecated and replaced with package_ensure.  Set this to latest for the same behavior as autoupdate => true.')
}

2 Line breaks in ''
Quoted strings can continue over multiple lines 
Line breaks within the string are interpreted as literal line breaks. Line break 被自动翻译成 line break， 会被转行  ( 是肯定会被 转行 )


3 不能用 string subtitution 

Single-quoted strings can’t interpolate values.
就是不能用 string subtitution 
就是不能用参数带入


## 1.1 Escape sequences

0 只可以用 下面两个 escape sequences in  single-quoted strings
![[06_01_Syntax_And_Settings_Puppet_Language/06_10_Values_and_data_types_and_aliases/images/Pasted image 20240412135020.png]]


1
Within single quotation marks (在 ' ' 中 ), if a backslash is followed by any character other than another backslash or a single quotation mark, Puppet treats it as a literal backslash.

如果 `\` 后面是 其他字符 或者 ',  这个 `\` (back slash) 就会被翻译成 backslash 本身， 不具有转移的功能

2
To include a literal double backslash use a quadruple backslash. (use \\\\)

3
to include a backslash at the very end of a single-quoted string, use a double backslash instead of a single backslash. For example: path => 'C:\Program Files(x86)\\'

最后想加个 `\`, 就要写成 `\\`
 -> 我们可以统一习惯 携程 A good habit is to always use two backslashes where you want the result to be one backslash. For example, path => `'C:\\Program Files(x86)\\'`

# 2 Double-quoted strings

1 Line breaks
Line breaks within the string are interpreted as literal line breaks. 
Line break 被自动翻译成 line break， 会被转行 

You can also insert line breaks with` \n (Unix-style) or \r\n (Windows-style).`

2  interpolate values
Double-quoted strings can interpolate values. See Interpolation information below.


3 会被自动转义的字符  in Double-quoted strings
![[06_01_Syntax_And_Settings_Puppet_Language/06_10_Values_and_data_types_and_aliases/images/Pasted image 20240412135129.png]]


4 在 “” 中 如果 后面` \ `加的不是 可以被转义的字符， 则puppt 会报warn, 但不是 error

Within double quotation marks, if a backslash is followed by any character other than those listed above (that is, a character that is not a recognized escape sequence),  Puppet logs a warning: 
Warning: Unrecognized escape sequence, and treats it as a literal backslash.

# 3 写 windows路径 

在Single-quoted strings  写 windows路径
1 用 / , 不会报错  e:/PuppetTest/ranorex_dataSource_templateFile.csv          
2 用 `\` 不会报错: path => `'e:\PuppetTest\ranorex_dataSource_templateFile.csv' `


在Double-quoted strings  写 windows路径
1 用 / , 不会报错: e:/PuppetTest/ranorex_dataSource_templateFile.csv           
2 用 `\` 会报错:  path => `'e:\PuppetTest\ranorex_dataSource_templateFile.csv'` 
3 用 `\\` 不会报错                                                          |

# 4 养成统一的习惯 
Tip: A good habit is to always use two backslashes where you want the result to be one backslash.

# 5 Interpolation

|                                                                      |                                                                                                                                                                                                                                                                                                                   |
| -------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 如何使用                                                                 | You can interpolate expressions in double-quoted strings, and in heredocs with interpolation enabled.                                                                                                                                                                                                             |
| 使用 Dollar sign<br><br>"String content ${<EXPRESSION>} more content". | To interpolate an expression, start with a dollar sign and wrap the expression in curly braces, as in: "String content ${<EXPRESSION>} more content".<br><br>The dollar sign doesn’t have to have a space in front of it. It can be placed directly after any other character, or at the beginning of the string. |
| 使用 Dollar sign 可以使用 “”                                               | An interpolated expression can include quote marks that would end the string if they occurred outside the interpolation token. For example: "<VirtualHost *:${hiera("http_port")}>".                                                                                                                              |
| Preventing interpolation<br><br>\$.                                  | If you want a string to include a literal sequence that looks like an interpolation token, but you don't want Puppet to try to evaluate it, use a quoting syntax that disables interpolation (single quotes or a non-interpolating heredoc), or escape the dollar sign with \$.                                   |

## 5.1 Short forms for variable interpolation

The most common thing to interpolate into a string is the value of a variable. To make this easier, Puppet has some shorter forms of the interpolation syntax:

$myvariable
不写  curly braces
A variable reference (without curly braces) can be replaced with that variable’s value. This also works with qualified variable names like $myclass::myvariable.
Because this syntax doesn’t have an explicit stopping point (like a closing curly brace), Puppet assumes the variable name is everything between the dollar sign and the first character that couldn’t legally be part of a variable name. (Or the end of the string, if that comes first.)
This means you can’t use this style of interpolation when a value must run up against some other word-like text. And even in some cases where you can use this style, the following style can be clearer.

${myvariable}
A dollar sign followed by a variable name in curly braces can be replaced with that variable’s value. This also works with qualified variable names like ${myclass::myvariable}.

1 With this syntax, you can follow a variable name with any combination of chained function calls or hash access / array access / substring access expressions. 
For example: 
`"Using interface ${::interfaces.split(',')[3]} for broadcast"`

2 
However, this doesn’t work if the variable’s name overlaps with a language keyword. For example, if you had a variable called $inherits  (variable 的name 中本身就有$), you would have to use normal-style interpolation:
`"Inheriting ${$inherits.upcase}."`

## 5.2 Conversion of interpolated values: 
Puppet converts the value of any interpolated expression to a string using these rules:

|   |   |
|---|---|
|Data type|Conversion<br><br>Convese 这个 data type 的 值 , 产生什么值|
|String|The contents of the string, with any quoting syntax removed.|
|Undef|An empty string.|
|Boolean|The string 'true' or 'false'.|
|Number|The number in decimal notation (base 10). For floats, the value can vary on different platforms. Use [the sprintf function](https://puppet.com/docs/puppet/7/function.html) for more precise formatting.|
|Array|A pair of square brackets ([ and ]) containing the array’s elements, separated by a comma and a space (,), with no trailing comma. Each element is converted to a string using these rules.|
|Hash|A pair of curly braces ({ and }) containing a <KEY> => <VALUE> string for each key-value pair, separated by a comma and a space (,), with no trailing comma. Each key and value is converted to a string using these rules.|
|Regular expression|A stringified regular expression.|
|Resource reference or data type|The value as a string.|

# 6 Line breaks

Quoted strings can continue over multiple lines, and line breaks are preserved as a literal part of the string (会转行， Line breaks不会翻译成文字的).

Heredocs let you suppress these line breaks if you use the L escape switch.

Puppet does not attempt to convert line breaks, so whatever type of line break is used in the file (LF for *nix or CRLF for Windows) is preserved.

Use escape sequences to insert specific types of line breaks into strings:

- To insert a CRLF in a manifest file that uses *nix line endings, use the \r\n escape sequences in a double-quoted string, or a heredoc with those escapes enabled.
- To insert an LF in a manifest that uses Windows line endings, use the \n escape sequence in a double-quoted string, or a heredoc with that escape enabled.

# 7 Heredocs
Heredocs let you quote strings with more control over escaping, interpolation, and formatting.

They’re especially good for long strings with complicated content.

包含多个 herdoc tag 的情况

If a line of code includes more than one heredoc tag, Puppet reads all of those heredocs in order: the first one begins on the following line and continue until its end marker, the second one begins on the line immediately after the first end marker, and so on. Puppet won’t start evaluating additional lines of Puppet code until it reaches the end marker for the final heredoc tag from the original line.

## 7.1 Syntax

To write a heredoc, you place a heredoc tag in a line of code. This tag acts as a literal string value, but the content of that string is read from the lines that follow it.

The string ends when an end marker is reached.

----

1 heredoc tag
`@("ENDTEXT"/<X>)`
You can use a heredoc tag in Puppet code, anywhere a string value is accepted. In the above example, the heredoc tag @("GITCONFIG"/L) completes the line of Puppet code: $gitconfig = .

A heredoc tag starts with @( and ends with ).

Between those characters, the heredoc tag contains end text (see below) — text that is used to mark the end of the string. 
You can optionally surround this end text with double quotation marks to enable interpolation (see below). In the above example, the end text is GITCONFIG.
例子中 的 `@("ENDTEXT"/<X>) `， ENDTEXT 两两边 有 ""


`/<X>` 的作用：  x 可以是任意的character 
It also optionally contains escape switches (see below), which start with a slash /. In the above example, the heredoc tag has the escape switch /L. 

In 例子 @("GITCONFIG"/L) 中 end text 就是 Citconfig 这几个字母

-----

2 the string
This is the text that makes up my string.

The content of the string starts on the next line, and can run over multiple lines. If you specified escape switches in the heredoc tag, the string can contain the enabled escape sequences.

In the above example, the string starts with` [user]` and ends with default = upstream. It uses the escape sequence \ to add cosmetic line breaks, because the heredoc tag contained the /L escape switch. 


----

3 end marker
 | - ENDTEXT

On a line of its own, the end marker consists of:
- Optional indentation and a pipe character (`|`) to indicate how much indentation is stripped from the lines of the string (see below).
- An optional hyphen character (`-`), with any amount of space around it, to trim the final line break from the string (see below).
- The end text, repeating the same end text used in the heredoc tag, always without quotation marks.

In the above example, the end marker is `| GITCONFIG`.

## 7.2 End text

In heredoc 的例子 @("GITCONFIG"/L) 中 end text 就是 Citconfig 这几个字母

The heredoc tag contains piece of text called the end text. When Puppet reaches a line that contains only that end text (plus optional formatting control), the string ends.

1   End text 要一摸一样的文本 的 出现在 end marker

Both occurrences of the end text must match exactly, with the same capitalization and internal spacing.  中间包含的空格都必须一摸一行

2 End text 中 能包含 什么 不能包含什么

End text can be any run of text that doesn’t include line breaks, colons, slashes, or parentheses.

It can include spaces, and can be mixed case.

3

The following are all valid end text:

1. EOT
2. ...end...end...
3. Verse 8 of The Raven

4   end text 中的内容最好全大写

To help with code readability, make your end text stand out from the content of the heredoc string, for example, by making it all uppercase.

## 7.3 Enabling interpolation

By default, heredocs do not allow you to [interpolate values](https://www.puppet.com/docs/puppet/8/lang_data_string#lang_data_string_interpolation "Interpolation allows strings to contain expressions, which can be replaced with their values. You can interpolate any expression that resolves to a value, except for statement-style function calls. You can interpolate expressions in double-quoted strings, and in heredocs with interpolation enabled.") into the string content. You can enable interpolation by double-quoting the end text in the opening heredoc tag. That is:

- An opening tag like `@(EOT)` won’t allow interpolation.
    
- An opening tag like `@("EOT")` allows interpolation.
    

Note: If you enable interpolation, but the string has a dollar character (`$`) that you want to be literal, not interpolated, you must enable the literal dollar sign escape switch (`/$`) in the heredoc tag: `@("EOT"/$)`. Then use the escape sequence `\$` to specify the literal dollar sign in the string. See the information on enabling escape sequences, below, for more details.

## 7.4 Enabling escape sequences

By default, heredocs have no escape sequences and every character is literal (except interpolated expressions, if enabled). To enable escape sequences, add switches to the heredoc tag.

To enable individual escape sequences, add a slash (`/`) and one or more switches. For example, to enable an escape sequence for dollar signs (`\$`) and new lines (`\n`), add `/$n` to the heredoc tag :

```
@("EOT"/$n)
```

To enable all escape sequences, add a slash and no switches:

```
@("EOT"/)
```

![[06_01_Syntax_And_Settings_Puppet_Language/06_10_Values_and_data_types_and_aliases/images/Pasted image 20240412140342.png]]



4 注意的地方
A backslash (\) that isn't part of an escape sequence is treated as a literal backslash. Unlike in double-quoted strings, this does not log a warning.

 If a heredoc has escapes enabled, and includes several literal backslashes in a row, make sure each literal backslash is represented by the \\ escape sequence. So, for example, If you want the result to include a double backslash, use four backslashes.

例子1
sign escape switch (/$) in the heredoc tag: @("EOT"/$).  这样在 string  中 写 \$ 就代表  想要显示的 是 $，而不是取参数 （enable interploration）

如果不表明 /$ in heredoc tag. 则 就没有这样的效果

 If you enable interpolation, but the string has a dollar character ($) that you want to be literal, not interpolated, you must enable the literal dollar sign escape switch (/$) in the heredoc tag: @("EOT"/$).

Then use the escape sequence \$ to specify the literal dollar sign in the string.

例子2
@("EOT"/L)

则在后面的 string 中， 如果出现 \, 则在此处不进行 分行
or example, Puppet would read this as a single line:

例子 $gitconfig = @("GITCONFIG"/L)

lg = "log --pretty=format:'%C(yellow)%h%C(reset) %s \

%C(cyan)%cr%C(reset) %C(blue)%an%C(reset) %C(green)%d%C(reset)' --graph"

解释 ： \ 的作用就是 告诉他 这里不分行了

没有 \ 的话， 那个位置就要分行了


## 7.5 Enabling syntax chechking 


To enable syntax checking of heredoc text, add the name of the syntax to the heredoc tag. For example:
@(END:pp)
@(END:epp)
@(END:json)

1 By default, heredocs are treated as text unless otherwise specified in the heredoc tag.

2 
If Puppet has a syntax checker for the given syntax, it validates the heredoc text, but only if the heredoc is static text and does not contain any interpolations. 

3 
If Puppet has no checker available for the given syntax, it silently ignores the syntax tag.


## 7.6 end marker 属性

 | - ENDTEXT

### 7.6.1 Striping indentation 
|-EOT

• Optional indentation and a pipe character (|) to indicate how much indentation is stripped from the lines of the string (see below).

1 加上 | 上面的 string 部分 就会和 | 的位置对齐
To make your code easier to read, you can indent the content of a heredoc to separate it from the surrounding code. 
To strip this indentation from the resulting string value, put the same amount of indentation in front of the end marker and use a pipe character (|) to indicate the position of the first “real” character on each line.

2 
If a line has less indentation than you’ve indicated with the pipe, Puppet strips any spaces it can without deleting non-space characters.

3 
If a line has more indentation than you’ve indicated with the pipe, the excess spaces are included in the final string value.

4Important:
 Indentation can include tab characters, but Puppet won’t convert tabs to spaces, so make sure you use the exact same sequence of space and tab characters on each line.


### 7.6.2 Suppressing literal line breaks 

If you enable the L escape switch, you can end a line with a backslash (\) to exclude the following line break from the string value. 
This lets you break up long lines in your source code without adding unwanted literal line breaks to the resulting string value.

For example, Puppet would read this as a single line:
例子 $gitconfig = @("GITCONFIG"/L)

lg = "log --pretty=format:'%C(yellow)%h%C(reset) %s \
%C(cyan)%cr%C(reset) %C(blue)%an%C(reset) %C(green)%d%C(reset)' --graph"

解释 ： \ 的作用就是 告诉他 这里不分行了
没有 \ 的话， 那个位置就要分行了 

### 7.6.3 Suppressing dinal line breaks 

|-EOT

在 string 的末尾加上 一个 line break, 
默认情况下 没有 
By default, heredocs end with a trailing line break, but you can exclude this line break from the final string. 

To suppress it, add a hyphen (-) to the end marker, before the end text, but after the indentation pipe （ 就是 | 符号）  if you used one. 
This works even if you don’t have the L escape switch enabled.

For example, Puppet would read this as a string with no line break at the end:

$mytext = @("EOT")
    This is too inconvenient for ${double} or ${single} quotes, but must be one line.
    |-EOT


## 7.7 Example 

```puppet 
$gitconfig = @("GITCONFIG"/L)
    [user]
        name = ${displayname}
        email = ${email}
    [color]
        ui = true
    [alias]
        lg = "log --pretty=format:'%C(yellow)%h%C(reset) %s \
    %C(cyan)%cr%C(reset) %C(blue)%an%C(reset) %C(green)%d%C(reset)' --graph"
        wdiff = diff --word-diff=color --ignore-space-at-eol \
    --word-diff-regex='[[:alnum:]]+|[^[:space:][:alnum:]]+'
    [merge]
        defaultToUpstream = true
    [push]
        default = upstream
    | GITCONFIG

file { "${homedir}/.gitconfig":
  ensure  => file,
  content => $gitconfig,
}
```







