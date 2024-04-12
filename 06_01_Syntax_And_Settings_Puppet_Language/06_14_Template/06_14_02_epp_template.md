

# 1 总览 

Intersperse 穿插
precede 超过

templates 是基於組合語言， 擅長用來管理複雜的設定，你只要提供簡單的輸入就可以輸出一個複雜的設定檔，templates 用於 file resource 的 content。

Embedded Puppet (EPP) is a templating language based on the Puppet language. You can use EPP in Puppet 4 and higher, and with Puppet 3.5 through 3.8 with the future parser enabled. 
Puppet evaluates EPP templates with the epp and inline_epp functions.

Epp 和 inline_epp 这两个 functions 的作用就是用来 evalueate epp template ， 让他把 epp file 中的内容 输出出来 

![[06_01_Syntax_And_Settings_Puppet_Language/images/Pasted image 20240412154409.png]]


# 2 EPP structure and syntax

2 部分 
第一部分 定义 有那些   template’s parameters. 通過 EPP tags  (Declare the template’s parameters.)
第二部分 是  plain-text document interspersed with tags containing Puppet expressions。 When evaluated, these tagged expressions can modify text in the template.

```
<%- | Boolean $keys_enable,
      String  $keys_file,
      Array   $keys_trusted,
      String  $keys_requestkey,
      String  $keys_controlkey
| -%>
<% if $keys_enable { -%>

<%# Printing the keys file, trusted key, request key, and control key: -%>
keys <%= $keys_file %>
<% unless $keys_trusted =~ Array[Data,0,0] { -%>
trustedkey <%= $keys_trusted.join(' ') %>
<% } -%>
<% if $keys_requestkey =~ String[1] { -%>
requestkey <%= $keys_requestkey %>
<% } -%>
<% if $keys_controlkey =~ String[1] { -%>
controlkey <%= $keys_controlkey %>
<% } -%>

<% } -%>
```

# 3 EPP TAGS

The following example shows 
	• parameter tags (<% |), 
	• non-printing expression tags (<%), 
	• expression-printing tags (<%=), and 
	• comment tags (<%#). 
A hyphen in a tag (-) strips leading or trailing whitespace when printing the evaluated template


main tag types used with EPP.
![[06_01_Syntax_And_Settings_Puppet_Language/images/Pasted image 20240412154456.png]]

---

## 3.1 如果這一段文本沒有加 tags , 只是写在  datai 中

Text outside a tag is treated as literal text, but is subject to any tagged Puppet code surrounding it. 

For example, text surrounded by a tagged if statement, this text only appears in the output if the condition is true.

比如说 
<% if $broadcastclient == true { -%>
Broadcastclient  <-- 就是指这一段
<% } -%>

<% if $broadcastclient == true { -%>
Context
<% } -%>


----

## 3.2 Literal tag delimiters  如何在  template’s final output 显示  literal <% , 
<% 不作为 tag 使用 

```
If you need the template’s final output to contain a literal <% or %>, you can escape the characters as <%% or %%>. 

The first literal tag is taken, and the rest of the line is treated as a literal. 
This means that <%% Test %%> in an EPP template would turn out as <% Test %%>, not <% Test %>.
```

---

## 3.3 Expression printing tags 取参数值 并且 要把参数值 显示出来

An expression-printing tag inserts the value of a single Puppet expression into the output.

1 An expression-printing tag must contain any single Puppet expression that resolves to a value, including plain variables, function calls, and arithmetic expressions. 
2 If the value isn’t a string, Puppet automatically converts it to a string based on the rules for value interpolation in double-quoted strings.
3 All facts are available in EPP templates. 
For example, to insert the value of the fqdn and hostname facts in an EPP template for an Apache config file
```
ServerName <%= $facts[fqdn] %>
ServerAlias <%= $facts[hostname] %>
```

==============================
Example tag: <%= $fqdn %>

![[06_01_Syntax_And_Settings_Puppet_Language/images/Pasted image 20240412154631.png]]

## 3.4 Non-printing tags  取参数值 并且 不要把参数值 显示出来 

A non-printing tag executes the code it contains, but doesn’t insert a value into the output.
![[06_01_Syntax_And_Settings_Puppet_Language/images/Pasted image 20240412154722.png]]

1
Non-printing tags that contain iterative and conditional expressions can affect the untagged text they surround.
For example, to insert text only if a certain variable was set, write:
<% if $broadcastclient == true { -%>
broadcastclient
<% } -%>

2
Expressions in non-printing tags don’t have to resolve to a value or be a complete statement, but the tag must close at a place where it would be legal to write another expression. 

```
For example, this doesn't work:
<%# Syntax error: %>
<% $servers.each -%>
# some server
<% |$server| { %> server <%= server %>
<% } -%>

You must keep |$server| { inside the first tag ( <% |$server| { %> ) , because you can’t insert an arbitrary statement between a function call and its required block.

```



3 写 group 
写成这样就可以了：
![[06_01_Syntax_And_Settings_Puppet_Language/images/Pasted image 20240412154801.png]]

下面这样的是不行的
https://serverfault.com/questions/478120/puppet-and-array-loop
<% (1..servername.length).each do |i| -%>
必须从1 开始 

![[06_01_Syntax_And_Settings_Puppet_Language/images/Pasted image 20240412154813.png]]

![[06_01_Syntax_And_Settings_Puppet_Language/images/Pasted image 20240412154821.png]]


## 3.5 Parameter tag  
卸载 epp 文件 开头、 定义 parameters  in this template 


A parameter tag declares which parameters the template accepts. Each parameter can be typed and can have a default value.
![[06_01_Syntax_And_Settings_Puppet_Language/images/Pasted image 20240412154843.png]]

Example tag:  
<%- | 
Boolean $keys_enable = false, 
String $keys_file = '' 
| -%>


1
The parameter tag is optional; if used, it must be the first content in a template. 

2
Always close the parameter tag with a right-trimmed delimiter (-%>) to avoid outputting a blank line. 
使用 -%> 在 | -%> 中  。 就可以避免 在 文本中有空行 

3
Literal text, line breaks, and non-comment tags cannot precede the template’s parameter tag. (Comment tags that precede a parameter tag must use the right-trimming tag to trim trailing whitespace.)
在epp文件中 Parameter tag  之前 不能有 任何空行， 文本， 或者其他的 tags
只有有 Comment tags 才可以出现在 parameter tag 之前 


Each parameter follows this format:
Boolean $keys_enable = false
Optional[Boolean] $keys_enable = false


4 参数的给出 的格式
	• 参数之间用 逗号隔开 The parameter tag’s pair of pipe characters (|) contains a comma-separated list of parameters. 
	• An optional data type, which restricts the allowed values for the parameter (defaults to Any)
	• A variable name
	• An optional equals (=) sign and default value, which must match the data type, if one was specified  如果 一个 parameter 有 default value, 则这个 parameter 就是 optional 的 
	• Parameters with default values are optional, and can be omitted when the template is evaluated. 
		○ If you want to use a default value of undef, make sure to also specify a data type that allows undef. 
For example, Optional[String] accepts undef as well as any string.


## 3.6 Comment tags

在 template's output 中不会显示任何 

只是作为 epp 文件中的 comment 

A comment tag’s contents do not appear in the template's output.

![[06_01_Syntax_And_Settings_Puppet_Language/images/Pasted image 20240412154920.png]]


# 4 Accessing EPP variables
Embedded Puppet (EPP) templates can access variables with the $variable syntax used in Puppet.

A template works like a defined type:
	• It has its own anonymous local scope.
	• The parent scope is set to node scope (or top scope if there’s no node definition).
	• When you call the template (with the epp or inline_epp functions), you can use parameters to set variables in its local scope.
Unlike Embedded Ruby (ERB) templates, EPP templates cannot directly access variables in the calling class without namespacing. Fully qualify variables or pass them in as parameters.

已有的 global variables 出现在 epp 文件中 
EPP templates can use short names to access global variables (like $os or $trusted) and their own local variables, but must use qualified names (like $ntp::tinker) to access variables from any class. The exception to this rule is inline_epp.


## 4.1 Special scope rule for inline_epp 

evaluate a template with the inline_epp function

If you evaluate a template with the inline_epp function (见 ), and if the template has no parameters, either passed or declared, you can access variables from the calling class in the template by using the variables’ short names.

This exceptional behavior is only allowed if all of the above conditions are true.


## 4.2 Should I use a parameter or a class variable 
Templates have two ways to use data:
	• Directly access class variables, such as $ntp::tinker
	• Use parameters passed at call time

1 Use class variables, 不在 epp 文件一开头就 定义 paramater 
Use class variables when a template is closely tied to the class that uses it, you don’t expect it to be used anywhere else, and you need to use a lot of variables.

2在 epp 文件一开头就 定义 paramater  
Use parameters when a template is used in several different places and you want to keep it flexible.
 Remember that declaring parameters with a tag makes a template’s data requirements visible at a glance.


## 4.3 EPP parameters

When you pass parameters when you call a template, the parameters become local variables inside the template. 
To use a parameter in this way, pass a hash as the last argument of the epp or inline_epp functions.

![[06_01_Syntax_And_Settings_Puppet_Language/06_14_Template/images/Pasted image 20240412160010.png]]



1
The keys of the hash match the variable names you’ll be using in the template, minus the leading $ sign. Parameters must follow the normal rules for local variable names.

2 Passing parameters  to a epp file (like 這個語句 epp('example/example.epp', { 'logfile' => "/var/log/ntp.log" })    )
	• If the template uses a parameter tag, it must be the first content in a template and you can only pass the parameters it declares. Passing any additional parameters is a syntax error.    
		○ Passing any 沒有定義過的 parameters 是會報錯的
	• However, if a template omits the parameter tag, you can pass it any parameters.   
		○ 没有 任何的 paramters 被定义in epp file, 则可以传入 任何 的 paramter

3 parameter 没有给出 default value, 则 在 calliing 这个 template 的时候， 必须给出 这个 parameter 的值 
If a template’s parameter tag includes any parameters without default values, they are mandatory. You must pass values for them when calling the template.


## 4.4 access  Sensitive data 

Puppet (version 6.20 and later) can interpolate sensitive values. In your EPP template, you can access the sensitive variable without unwrapping.

ERB templates do not support interpolation of sensitive values — you have to manually unwrap and re-wrap these.
![[06_01_Syntax_And_Settings_Puppet_Language/06_14_Template/images/Pasted image 20240412160037.png]]


# 5 Validating and previewing EPP templates With puppet epp command 

用来 epp compand-line  测试一些 自己写的 epp 好不好用 

Before deploying a template, validate its syntax and render its output to make sure the template is producing the results you expect. 
Use the puppet epp compand-line tool for validating and rendering Embedded Puppet (EPP) templates.
see the command’s man page:  https://puppet.com/docs/puppet/7/man/epp.html

## 5.1 EPP validation

看是否报错 

` puppet epp validate <TEMPLATE NAME>`

To validate your template, run: puppet epp validate `<TEMPLATE NAME>`

1
The puppet epp command includes an action that checks EPP code for syntax problems. 
The command is silent on a successful validation. It reports and halts on the first error it encounters. 
For information on how to modify this default behavior, see the command’s man page https://puppet.com/docs/puppet/7/man/epp.html
见 puppet epp validate 命令

2 `<TEMPLATE NAME>  `的给入: The` <TEMPLATE NAME> `can be a file reference or can refer to a `<MODULE NAME>/<TEMPLATE FILENAME>` as the epp function. 
If a file reference can also refer to a module, Puppet validates the module’s template instead.

3
You can also pipe EPP code directly to the validator: cat example.epp | puppet epp validate


## 5.2 EPP rendering

在 rendering之前 会outputs  the template final outputs 

puppet epp render 



To render your template, run: puppet epp render `<TEMPLATE NAME>`

You can render EPP from the command line with puppet epp render. If Puppet can evaluate the template, it outputs the result.

1

If your template relies on specific parameters or values to function, you can simulate those values by passing a hash to the --values option. For example:

puppet epp render example.epp --values '{x => 10, y => 20}'

You can also render inline EPP by using the -e flag or piping EPP code to puppet epp render, and even simulate facts using YAML.

2 更多见 

[puppet epp render 命令](onenote:PuppetCLI.one#Puppet%20epp&section-id={92BA47C4-EF0F-4462-A883-32CE61C69679}&page-id={796F2A82-ED8D-48BE-B5DD-8934B2BFB8F5}&object-id={951F844D-C259-074C-1A83-E567320C76EB}&14&base-path=https://d.docs.live.net/7290c9c9925b56f3/Onenote/运维工具/Puppet)

# 6 Evaluating EPP templates

After you have an EPP template, you can pass it to a function that evaluates it and returns a final string. The actual template can be either a separate file or a string value.
就是用 epp() function  或者 inline_epp function  将 epp template + paramter value 的结果输出来 


## 6.1 Evaluating EPP templates that are in a template file

Function

` epp(<FILE REFERENCE>, [<PARAMETER HASH>])`

1 将  epp file 放到  templates directory of a module
Put template files in the templates directory of a module. EPP files use the .epp extension.

2 example
例子1 
```
`# epp(<FILE REFERENCE>, [<PARAMETER HASH>])`
file { '/etc/ntp.conf':
  ensure  => file,
  path    => D:/Fitnesse  (注意 new csv file can be new created through epp template file， 就是说 file之前不需要存在  . The Directory of this new file muss already exist, 但是 diretory 必须之前已经存在， 不然会报错  )
  content => epp('ntp/ntp.conf.epp', {'service_name' => 'xntpd', 'iburst_enable' => true}),
# 7 Loads /etc/puppetlabs/code/environments/production/modules/ntp/templates/ntp.conf.epp
}
```

例子2 

    $servers_file_content = epp('test/servers.txt.epp', {
      tag => $tag,
      schema => $schema,
      db_server => $db_server,
      port => $port,
      app_server => $app_server,
      schema_pw => $schema_pw
    })


      file { "D:\fitnesse\FitNesseRoot\FrontPage\PerformanceTests\Includes\Servers\content.txt":
        ensure  => file,
        content => $servers_file_content
      }

3
`解释 epp(<FILE REFERENCE>, [<PARAMETER HASH>]) 这个function`

FILE REFERENCE: 
The first argument to the function is the file reference: 
a string in the form `'<MODULE>/<FILE>',` which loads `<FILE>` from `<MODULE>`’s templates directory. 
For example, the file reference ntp/ntp.conf.epp loads the `<MODULES DIRECTORY>/ntp/templates/ntp.conf.epp` file.

 parameter hash: 
Some EPP templates declare parameters, and you can provide values for them by passing a parameter hash to the epp function.
The keys of the hash must be valid local variable names (minus the $). Inside the template, Puppet creates variables with those names and assign their values from the hash. 
For example, with a parameter hash of {'service_name' => 'xntpd', 'iburst_enable' => true}, an EPP template would receive variables called $service_name and $iburst_enable.
When structuring your parameter hash, remember:
	• If a template declares any mandatory parameters, you must set values for them with a parameter hash.
		○ 或者  没有  Optional[Boolean] $keys_enable ， 也没有 给出 default value ， 则这个 paramter 就是 mandatory param 
	• If a template declares any optional parameters, you can choose to provide values or let them use their defaults.
		○ Optional[Boolean] $keys_enable = false
		○ 或者 如果 一个 parameter 有 default value, 则这个 parameter 就是 optional 的 
	• If a template declares no parameters, you can pass any number of parameters with any names; otherwise, you can only choose from the parameters requested by the template.



## 6.2 Evaluating EPP template strings (in heredoc syntax )

 inline_epp function.

If you have a string value that contains template content  (like  heredoc  ), you can evaluate it with the inline_epp function.

In modern versions of Puppet, inline templates are usable in some of the same situations template files are.

Because the heredoc syntax makes it easy to write large and complicated strings in a manifest, you can use inline_epp to reduce the number of files needed for a simple module that manages a small config file.

2 example

$ntp_conf_template = @(END)
...template content goes here...
END


```

# 8 inline_epp(<TEMPLATE STRING>, [<PARAMETER HASH>])

file { '/etc/ntp.conf':

  ensure  => file,

  content => inline_epp($ntp_conf_template, {'service_name' => 'xntpd', 'iburst_enable' => true}),

}

```

3

参照上一个zeile 的 3



# 7 Example EPP template

call this template
```
To call this template from a manifest (assuming that the template file is located in the templates directory of the puppetlabs-ntp module), add the following code to the manifest:

# epp(<FILE REFERENCE>, [<PARAMETER HASH>])
file { '/etc/ntp.conf':
  ensure  => file,
  content => epp('ntp/ntp.conf.epp'),
  # Loads /etc/puppetlabs/code/environments/production/modules/ntp/templates/ntp.conf.epp
}
```


Epp file 中的内容 
```
The following example is an EPP translation of the ntp.conf.erb template from the puppetlabs-ntp module.

# ntp.conf: Managed by puppet.
#
<% if $ntp::tinker == true and ($ntp::panic or $ntp::stepout) { -%>
# Enable next tinker options:
# panic - keep ntpd from panicking in the event of a large clock skew
# when a VM guest is suspended and resumed;
# stepout - allow ntpd change offset faster
tinker<% if $ntp::panic { %> panic <%= $ntp::panic %><% } %><% if $ntp::stepout { -%> stepout <%= $ntp::stepout %><% } %>
<% } -%>

<% if $ntp::disable_monitor == true { -%>
disable monitor
<% } -%>
<% if $ntp::disable_auth == true { -%>
disable auth
<% } -%>

<% if $ntp::restrict =~ Array[Data,1] { -%>
# Permit time synchronization with our time source, but do not
# permit the source to query or modify the service on this system.
<% $ntp::restrict.flatten.each |$restrict| { -%>
restrict <%= $restrict %>
<% } -%>
<% } -%>

<% if $ntp::interfaces =~ Array[Data,1] { -%>
# Ignore wildcard interface and only listen on the following specified
# interfaces
interface ignore wildcard
<% $ntp::interfaces.flatten.each |$interface| { -%>
interface listen <%= $interface %>
<% } -%>
<% } -%>

<% if $ntp::broadcastclient == true { -%>
broadcastclient
<% } -%>

# Set up servers for ntpd with next options:
# server - IP address or DNS name of upstream NTP server
# iburst - allow send sync packages faster if upstream unavailable
# prefer - select preferrable server
# minpoll - set minimal update frequency
# maxpoll - set maximal update frequency
<% [$ntp::servers].flatten.each |$server| { -%>
server <%= $server %><% if $ntp::iburst_enable == true { %> iburst<% } %><% if $server in $ntp::preferred_servers { %> prefer<% } %><% if $ntp::minpoll { %> minpoll <%= $ntp::minpoll %><% } %><% if $ntp::maxpoll { %> maxpoll <%= $ntp::maxpoll %><% } %>
<% } -%>

<% if $ntp::udlc { -%>
# Undisciplined Local Clock. This is a fake driver intended for backup
# and when no outside source of synchronized time is available.
server   127.127.1.0
fudge    127.127.1.0 stratum <%= $ntp::udlc_stratum %>
restrict 127.127.1.0
<% } -%>

# Driftfile.
driftfile <%= $ntp::driftfile %>

<% unless $ntp::logfile =~ Undef { -%>
# Logfile
logfile <%= $ntp::logfile %>
<% } -%>

<% unless $ntp::peers =~ Array[Data,0,0] { -%>
# Peers
<% [$ntp::peers].flatten.each |$peer| { -%>
peer <%= $peer %>
<% } -%>
<% } -%>

<% if $ntp::keys_enable { -%>
keys <%= $ntp::keys_file %>
<% unless $ntp::keys_trusted =~ Array[Data,0,0] { -%>
trustedkey <%= $ntp::keys_trusted.join(' ') %>
<% } -%>
<% if $ntp::keys_requestkey =~ String[1] { -%>
requestkey <%= $ntp::keys_requestkey %>
<% } -%>
<% if $ntp::keys_controlkey =~ String[1] { -%>
controlkey <%= $ntp::keys_controlkey %>
<% } -%>

<% } -%>
<% [$ntp::fudge].flatten.each |$entry| { -%>
fudge <%= $entry %>
<% } -%>

<% unless $ntp::leapfile =~ Undef { -%>
# Leapfile
leapfile <%= $ntp::leapfile %>
<% } -%>

```

