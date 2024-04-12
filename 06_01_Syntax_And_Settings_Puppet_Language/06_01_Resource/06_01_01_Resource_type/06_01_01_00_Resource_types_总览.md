
https://www.puppet.com/docs/puppet/8/resource_types

八种核心资源
user、group、file、package、service、cron、exec、notify；

|                  |       |                                      |
| ---------------- | ----- | ------------------------------------ |
| ( Resource )资源类型 | 资源描述  |                                      |
| file             | 文件或目录 | file 可以用來管理檔案和目錄                     |
| user             | 用户    |                                      |
| group            | 组     |                                      |
| service          | 服务    | 其中一个使用就是 service 用來決定啟動服務與是否 ONBOOT。 |
| cron             | 计划任务  |                                      |
| exec             | 命令    |                                      |
| package          | 软件包   | package 用來安裝套件。                      |
| yumrepo          | 软件仓库  |                                      |

![[06_01_Syntax_And_Settings_Puppet_Language/06_01_Resource/06_01_01_Resource_type/images/Pasted image 20240411131320.png]]

----


`package`, `file`, `service`, `notify`, `exec`, `user`, and `group`.

`filebucket` , `overview`, `resources`, `schedule`, `tidy`

Every resource (file, user, service, package, and so on) is associated with a resource type within the Puppet language. The resource type defines the kind of configuration it manages. This section provides information about the resource types that are built into Puppet.

- **[Resource Type Reference (Single-Page)](https://www.puppet.com/docs/puppet/8/type)**  
    
- **[Built-in types](https://www.puppet.com/docs/puppet/8/cheatsheet_core_types#cheatsheet_core_types)**  
    This page provides a reference guide for Puppet's built-in types: `package`, `file`, `service`, `notify`, `exec`, `user`, and `group`.
- **[Optional resource types for Windows](https://www.puppet.com/docs/puppet/8/resources_windows_optional)**  
    In addition to the resource types included with Puppet, you can install custom resource types as modules from the Forge. This is especially useful when managing Windows systems, because there are several important Windows-specific resource types that are developed as modules rather than as part of core Puppet.
- **[Resource Type: exec](https://www.puppet.com/docs/puppet/8/types/exec)**  
    
- **[Using exec on Windows](https://www.puppet.com/docs/puppet/8/resources_exec_windows)**  
    Puppet uses the same `exec` resource type on both *nix and Windows systems, and there are a few Windows-specific best practices and tips to keep in mind.
- **[Resource Type: file](https://www.puppet.com/docs/puppet/8/types/file)**  
    
- **[Using file on Windows](https://www.puppet.com/docs/puppet/8/resources_file_windows)**  
    Use Puppet's built-in `file` resource type to manage files and directories on Windows, including ownership, group, permissions, and content, with the following Windows-specific notes and tips.
- **[Resource Type: filebucket](https://www.puppet.com/docs/puppet/8/types/filebucket)**  
    
- **[Resource Type: group](https://www.puppet.com/docs/puppet/8/types/group)**  
    
- **[Using user and group on Windows](https://www.puppet.com/docs/puppet/8/resources_user_group_windows)**  
    Use the built-in `user` and `group` resource types to manage user and group accounts on Windows.
- **[Resource types overview](https://www.puppet.com/docs/puppet/8/types/overview)**  
    
- **[Resource Type: notify](https://www.puppet.com/docs/puppet/8/types/notify)**  
    
- **[Resource Type: package](https://www.puppet.com/docs/puppet/8/types/package)**  
    
- **[Using package on Windows](https://www.puppet.com/docs/puppet/8/resources_package_windows#resources_package_windows)**  
    The built-in `package` resource type handles many different packaging systems on many operating systems, so not all features are relevant everywhere. This page offers guidance and tips for working with `package` on Windows.
- **[Resource Type: resources](https://www.puppet.com/docs/puppet/8/types/resources)**  
    
- **[Resource Type: schedule](https://www.puppet.com/docs/puppet/8/types/schedule)**  
    
- **[Resource Type: service](https://www.puppet.com/docs/puppet/8/types/service)**  
    
- **[Using service](https://www.puppet.com/docs/puppet/8/resources_service#resources_service)**  
    Puppet can manage services on nearly all operating systems.This page offers operating system-specific advice and best practices for working with `service`.
- **[Resource Type: stage](https://www.puppet.com/docs/puppet/8/types/stage)**  
    
- **[Resource Type: tidy](https://www.puppet.com/docs/puppet/8/types/tidy)**  
    
- **[Resource Type: user](https://www.puppet.com/docs/puppet/8/types/user)**




# 1 详细



https://www.puppet.com/docs/puppet/8/type

[About resource types

---

](https://www.puppet.com/docs/puppet/8/type#about-resource-types)

- [Built-in types and custom types](https://www.puppet.com/docs/puppet/8/type#built-in-types-and-custom-types)
- [Declaring resources](https://www.puppet.com/docs/puppet/8/type#declaring-resources)
- [Namevars and titles](https://www.puppet.com/docs/puppet/8/type#namevars-and-titles)
- [Attributes, parameters, properties](https://www.puppet.com/docs/puppet/8/type#attributes-parameters-properties)
- [Providers](https://www.puppet.com/docs/puppet/8/type#providers)
- [Features](https://www.puppet.com/docs/puppet/8/type#features)

[Puppet 6.0 type changes

---

](https://www.puppet.com/docs/puppet/8/type#puppet-60-type-changes)

- [Supported type modules in `puppet-agent`](https://www.puppet.com/docs/puppet/8/type#supported-type-modules-in-puppet-agent)
- [Type modules available on the Forge](https://www.puppet.com/docs/puppet/8/type#type-modules-available-on-the-forge)
- [Deprecated types](https://www.puppet.com/docs/puppet/8/type#deprecated-types)

[Puppet core types

---

](https://www.puppet.com/docs/puppet/8/type#puppet-core-types)

[exec

---

](https://www.puppet.com/docs/puppet/8/type#exec)

- [Description](https://www.puppet.com/docs/puppet/8/type#exec-description)
- [Attributes](https://www.puppet.com/docs/puppet/8/type#exec-attributes)
- [Providers](https://www.puppet.com/docs/puppet/8/type#exec-providers)

[file

---

](https://www.puppet.com/docs/puppet/8/type#file)

- [Description](https://www.puppet.com/docs/puppet/8/type#file-description)
- [Attributes](https://www.puppet.com/docs/puppet/8/type#file-attributes)
- [Providers](https://www.puppet.com/docs/puppet/8/type#file-providers)
- [Provider Features](https://www.puppet.com/docs/puppet/8/type#file-provider-features)

[filebucket

---

](https://www.puppet.com/docs/puppet/8/type#filebucket)

- [Description](https://www.puppet.com/docs/puppet/8/type#filebucket-description)
- [Attributes](https://www.puppet.com/docs/puppet/8/type#filebucket-attributes)

[group

---

](https://www.puppet.com/docs/puppet/8/type#group)

- [Description](https://www.puppet.com/docs/puppet/8/type#group-description)
- [Attributes](https://www.puppet.com/docs/puppet/8/type#group-attributes)
- [Providers](https://www.puppet.com/docs/puppet/8/type#group-providers)
- [Provider Features](https://www.puppet.com/docs/puppet/8/type#group-provider-features)

[notify

---

](https://www.puppet.com/docs/puppet/8/type#notify)

- [Description](https://www.puppet.com/docs/puppet/8/type#notify-description)
- [Attributes](https://www.puppet.com/docs/puppet/8/type#notify-attributes)

[package

---

](https://www.puppet.com/docs/puppet/8/type#package)

- [Description](https://www.puppet.com/docs/puppet/8/type#package-description)
- [Attributes](https://www.puppet.com/docs/puppet/8/type#package-attributes)
- [Providers](https://www.puppet.com/docs/puppet/8/type#package-providers)
- [Provider Features](https://www.puppet.com/docs/puppet/8/type#package-provider-features)

[resources

---

](https://www.puppet.com/docs/puppet/8/type#resources)

- [Description](https://www.puppet.com/docs/puppet/8/type#resources-description)
- [Attributes](https://www.puppet.com/docs/puppet/8/type#resources-attributes)

[schedule

---

](https://www.puppet.com/docs/puppet/8/type#schedule)

- [Description](https://www.puppet.com/docs/puppet/8/type#schedule-description)
- [Attributes](https://www.puppet.com/docs/puppet/8/type#schedule-attributes)

[service

---

](https://www.puppet.com/docs/puppet/8/type#service)

- [Description](https://www.puppet.com/docs/puppet/8/type#service-description)
- [Attributes](https://www.puppet.com/docs/puppet/8/type#service-attributes)
- [Providers](https://www.puppet.com/docs/puppet/8/type#service-providers)
- [Provider Features](https://www.puppet.com/docs/puppet/8/type#service-provider-features)

[stage

---

](https://www.puppet.com/docs/puppet/8/type#stage)

- [Description](https://www.puppet.com/docs/puppet/8/type#stage-description)
- [Attributes](https://www.puppet.com/docs/puppet/8/type#stage-attributes)

[tidy

---

](https://www.puppet.com/docs/puppet/8/type#tidy)

- [Description](https://www.puppet.com/docs/puppet/8/type#tidy-description)
- [Attributes](https://www.puppet.com/docs/puppet/8/type#tidy-attributes)

[user

---

](https://www.puppet.com/docs/puppet/8/type#user)

- [Description](https://www.puppet.com/docs/puppet/8/type#user-description)
- [Attributes](https://www.puppet.com/docs/puppet/8/type#user-attributes)
- [Providers](https://www.puppet.com/docs/puppet/8/type#user-providers)
- [Provider Features](https://www.puppet.com/docs/puppet/8/type#user-provider-features)