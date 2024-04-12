
https://www.puppet.com/docs/puppet/8/lang_template

templates 是基於組合語言， 擅長用來管理複雜的設定，你只要提供簡單的輸入就可以輸出一個複雜的設定檔，templates 用於 file resource 的 content。

Templates are documents that combine code, data, and literal text to produce a final rendered output. The goal of a template is to manage a complicated piece of text with simple inputs.


# 1 Templates 支援的語言

目前 templates 支援兩種語言寫法
EPP (Embedded Puppet) 基於 Puppet 自身的語言，目前只在 Puppet 4 以上的版本支援。
ERB (Embedded Ruby) 基於 Ruby 語言的寫法，適用所有版本。
兩者看起來 Puppet 想用 EPP 來取代 ERB，寫起來 EPP 和 ERB 的差異不大，但是如果你還不太熟 Puppet，或是比較熟 Ruby 建議可以從 ERB 開始寫起，畢竟網路資料目前還是以 ERB 為主。


# 2 When to use (and not use) templates

|                                                                                                |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| ---------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1 用 interpolate variables and expressions                                                      | 1 Strings in the Puppet language already allow you to interpolate variables and expressions into text.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| 2 用 heredoc and interpolate a few variables, or even something like ${ $my_array.join(', ') }. | 2 So if you’re managing a short and simple config file, you can often use a heredoc and interpolate a few variables, or even something like ${ $my_array.join(', ') }.<br><br>见 [https://puppet.com/docs/puppet/7/lang_data_string.html#lang_data_string_heredocs](https://puppet.com/docs/puppet/7/lang_data_string.html#lang_data_string_heredocs)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| 3 用 template                                                                                   | 3  if you’re doing complex transformations (especially iterating over collections) or working with very large config files, you might need the extra capabilities of a template.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| 4 用 Augeas, concat, or file_line resource.                                                     | （不同的 modules 都用同一个 file ）<br><br>Sometimes you end up in a situation where several different modules need to manage parts of the same config file, which is incredibly difficult to coordinate with either templates or interpolated strings.  <br><br>For shared configuration like this, you should try to model each setting in that file as an individual resource, with either a custom resource type or an Augeas, concat, or file_line resource. (This approach is similar to how core resource types like ssh_authorized_key and mount work.)<br><br>[Augeas](https://puppet.com/docs/puppet/7/type.html), [concat](https://forge.puppet.com/puppetlabs/concat?_ga=2.150473599.874476915.1655829474-1777409666.1646987604), or [file_line](https://forge.puppet.com/puppetlabs/stdlib?_ga=2.150473599.874476915.1655829474-1777409666.1646987604) |


# 3 Using templates的细节 

Once you have a template, you can pass it to a function that will evaluate it and return a final string.

(
	例子：content => epp('ntp/ntp.conf.epp', {'service_name' => 'xntpd', 'iburst_enable' => true}
	Epp 为 function
	Content 就是 returen a final string
)

The actual template can be either a separate file or a string value.

![[06_01_Syntax_And_Settings_Puppet_Language/images/Pasted image 20240412153501.png]]


## 3.1 Using with a template file: 使用 template() and epp () Function 

### 3.1.1 With a template file: template and epp

To use a template file, evaluate it with the template (ERB) or epp function:
```
# epp(<FILE REFERENCE>, [<PARAMETER HASH>])
file { '/etc/ntp.conf':
  ensure  => file,
  content => epp('ntp/ntp.conf.epp', {'service_name' => 'xntpd', 'iburst_enable' => true}),
  # Loads /etc/puppetlabs/code/environments/production/modules/ntp/templates/ntp.conf.epp
}

# template(<FILE REFERENCE>, [<ADDITIONAL FILES>, ...])
file { '/etc/ntp.conf':
  ensure  => file,
  content => template('ntp/ntp.conf.erb'),
  # Loads /etc/puppetlabs/code/environments/production/modules/ntp/templates/ntp.conf.erb
}
```


### 3.1.2 With a template string: inline_template and inline_epp

If you have a string value that contains template content, you can evaluate it with the inline_template (ERB) or inline_epp functions as follows:

```
$ntp_conf_template = @(END)
...template content goes here...
END

# inline_epp(<TEMPLATE STRING>, [<PARAMETER HASH>])
file { '/etc/ntp.conf':
  ensure  => file,
  content => inline_epp($ntp_conf_template, {'service_name' => 'xntpd', 'iburst_enable' => true}),
}

# inline_template(<TEMPLATE STRING>, [<ADDITIONAL STRINGS>, ...])
file { '/etc/ntp.conf':
  ensure  => file,
  content => inline_template($ntp_conf_template),
}
```


### 3.1.3 使用 epp() funtion 
  content => epp('ntp/ntp.conf.epp', {'service_name' => 'xntpd', 'iburst_enable' => true}),

1 第一个参数为 Referencing files 
The first argument to these functions should be a string like` '<MODULE>/<FILE>',` which will load `<FILE>` from `<MODULE>`ﾡﾯs templates directory.

|   |   |
|---|---|
|File Reference|Actual File|
|ntp/ntp.conf.epp|<MODULES DIRECTORY>/ntp/templates/ntp.conf.epp|
|activemq/amq/activemq.xml.erb|<MODULES DIRECTORY>/activemq/templates/amq/activemq.xml.erb|

2 第二个参数 为  EPP parameters

EPP templates can declare parameters, and you can provide values for them by passing a parameter [hash](https://puppet.com/docs/puppet/5.5/lang_data_hash.html) to the epp function.

The keys of the hash must be [valid local variable names](https://puppet.com/docs/puppet/5.5/lang_reserved.html#variables) (minus the $). Inside the template, Puppet will create variables with those names and assign their values from the hash. (With a parameter hash of {'service_name' => 'xntpd', 'iburst_enable' => true}, an EPP template would receive variables called $service_name and $iburst_enable.)

- If a template declares any mandatory parameters, you must set values for them with a parameter hash.
- If a template declares any optional parameters, you can choose to provide values or let them use their defaults.
- If a template declares no parameters, you can pass any number of parameters with any names; otherwise, you can only choose from the parameters requested by the template.


3 使用 template() funtion  (只有 Extra ERB file 需要用这个function ) 

content => template('ntp/ntp.conf.erb'),
The template function can take any number of additional template files, and will concatenate their outputs together to produce the final string.


## 3.2 Using with a template string: inline_template() (epp 用这个) and inline_epp() (ERB 用这个)  funtion 

If you have a string value that contains template content, you can evaluate it with the inline_template (ERB) or inline_epp functions as follows:

```
$ntp_conf_template = @(END)  // 这个就是 string value that contains template content 
...template content goes here...
END

# inline_epp(<TEMPLATE STRING>, [<PARAMETER HASH>])
file { '/etc/ntp.conf':
  ensure  => file,
  content => inline_epp($ntp_conf_template, {'service_name' => 'xntpd', 'iburst_enable' => true}),
}

# inline_template(<TEMPLATE STRING>, [<ADDITIONAL STRINGS>, ...])
file { '/etc/ntp.conf':
  ensure  => file,
  content => inline_template($ntp_conf_template),
}
```

1 EPP parameters  in  inline_template()   
EPP templates can declare parameters, and you can provide values for them by passing a parameter hash to the epp function.
The keys of the hash must be valid local variable names (minus the $). Inside the template, Puppet will create variables with those names and assign their values from the hash. (With a parameter hash of {'service_name' => 'xntpd', 'iburst_enable' => true}, an EPP template would receive variables called $service_name and $iburst_enable.)
	• If a template declares any mandatory parameters, you must set values for them with a parameter hash.
	• If a template declares any optional parameters, you can choose to provide values or let them use their defaults.
If a template declares no parameters, you can pass any number of parameters with any names; otherwise, you can only choose from the parameters requested by the template.


2 Extra ERB strings in  inline_epp()
The inline_template function can take any number of additional template strings, and will concatenate their outputs together to produce the final value.



# 4 Validating and previewing templates

可以用 相关的命令， 取试一下啊 这个 templates 写的好不好 ， 有没有 syntax problem, 输出的结果是什么样的
命令 
puppet epp for epp templates
Erb for ruby templates 

Puppet includes the puppet epp command-line tool for EPP templates, 
while Ruby can check ERB syntax after trimming the template with its erb command.


## 4.1 puppet epp for epp templates

puppet-epp  命令解释 
https://puppet.com/docs/puppet/5.5/man/epp.html

### 4.1.1 EPP validation
实验  有没有 syntax problem

```
puppet epp validate <TEMPLATE NAME>

1
The puppet epp command includes an action that checks EPP code for syntax problems. The <TEMPLATE NAME> can be a file reference or can refer to a <MODULE NAME>/<TEMPLATE FILENAME> as the epp function. If a file reference can also refer to a module, Puppet validates the module’s template instead.

2
You can also pipe EPP code directly to the validator:

cat example.epp | puppet epp validate

3
The command is silent on a successful validation. It reports and halts on the first error it encounters; to modify this behavior, check the command’s man page.
```


### 4.1.2 EPP rendering
作用： previewing templates

`puppet epp render <TEMPLATE NAME>`

You can render EPP from the command line with puppet epp render. If Puppet can evaluate the template, it outputs the result.

If your template relies on specific parameters or values to function, you can simulate those values by passing a hash to the --values option:

puppet epp render example.epp --values '{x => 10, y => 20}'

You can also render inline EPP by using the -e flag or piping EPP code to puppet epp render, and even simulate facts using YAML. For details, see the command’s man page.



## 4.2 ERB validation

erb -P -x -T '-' example.erb | ruby -c

You can use Ruby to check the syntax of ERB code by piping output from the erb command into ruby. The -P switch ignores lines that start with ‘%’, the -x switch outputs the template’s Ruby script, and the -T '-' sets the trim mode to be consistent with Puppet’s behavior. This output gets piped into Ruby’s syntax checker (-c).

If you need to validate many templates quickly, you can implement this command as a shell function in your shell’s login script, such as .bashrc, .zshrc, or .profile:

	validate_erb() {
	  erb -P -x -T '-' $1 | ruby -c
	}

You can then use validate_erb example.erb to validate an ERB template.

