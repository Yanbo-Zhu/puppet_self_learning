
https://www.puppet.com/docs/puppet/8/lang_windows_file_paths.html#lang_windows_file_paths-dir-separators-file-paths


问题所在 

Several resource types (including file, exec, and package) take file paths as values for various attributes. 
The Puppet language uses the backslash (\) as an escape character in quoted strings. 

However, Windows also uses the backslash to separate directories in file paths, such as C:\Program Files\PuppetLabs.

Additionally, Windows file system APIs accept both backslashes and forward slashes in file paths, but some Windows programs accept only backslashes.


# 1 Directory separators in file paths

Generally, if Puppet itself is interpreting the file path, or if the file path is meant for the primary server, use forward slashes. 

If the file path is being passed directly to a Windows program, use backslashes. 

![[06_01_Syntax_And_Settings_Puppet_Language/images/Pasted image 20240412133346.png]]

1 Attributes" Path" in resource  "file " 
On Windows, the path should include the drive letter and should use / as the separator character (rather than \).


2 在Single-quoted strings  写 windows路径 
用 / , 不会报错
e:/PuppetTest/ranorex_dataSource_templateFile.csv

用 `\` 不会报错 
例子
path => 'e:\PuppetTest\ranorex_dataSource_templateFile.csv'

3 在Double-quoted strings  写 windows路径 
用 / , 不会报错
e:/PuppetTest/ranorex_dataSource_templateFile.csv

用 `\` 会报错 
例子
path => 'e:\PuppetTest\ranorex_dataSource_templateFile.csv'

用 `\\` 不会报错 
path => 'e:\\PuppetTest\\norex_dataSource_templateFile.csv'


# 2 Line endings in files

Windows uses CRLF line endings instead of *nix's LF line endings. Be aware of the following issues:

- If you specify the contents of a file with the `content` attribute, Puppet writes the content in binary mode. To create files with CRLF line endings, specify the `\r\n` escape sequence as part of the content.
    
- When downloading a file to a Windows node with the `source` attribute, Puppet transfers the file in binary mode, leaving the original newlines untouched.
    
- If you are using version control, such as Git, ensure that it is configured to use CRLF line endings.
- Non-`file` resource types that make partial edits to a system file, such as the [`host` resource type](https://forge.puppet.com/puppetlabs/host_core), which manages the `%windir%\system32\drivers\etc\hosts` file, manage their files in text mode and automatically translate between Windows and *nix line endings.

