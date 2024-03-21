
The `puppet module` command provides an interface for managing modules from the Forge. Its interface is similar to other common package managers, such as `gem`, `apt-get`, or `yum`. You can install, upgrade, uninstall, list, and search for modules with this command.

# 1 Setting up `puppet module` behind a proxy

To use the `puppet module` command behind a proxy, set the proxy's IP address and port by running the following two commands:

```
export http_proxy=http://<PROXY IP>:<PROXY PORT>
export https_proxy=http://<PROXY IP>:<PROXY PORT>
```

For instance, for an proxy at 192.168.0.10 on port 8080, run:

```
export http_proxy=http://192.168.0.10:8080
export https_proxy=http://192.168.0.10:8080
```

Alternatively, you can set these two proxy settings in the `puppet.conf` file, by setting `http_proxy_host` and `http_proxy_port` in the `user` section of `puppet.conf`. For more information, see the Puppet [configuration reference](https://www.puppet.com/docs/puppet/7/configuration).

Important: Set these two proxy settings only in the `[user`] section of the `puppet.conf` file. Setting them in other sections can cause problems.


# 2 Installing modules from the command line

The `puppet module install` command installs a module and all of its dependencies. You can install modules from the Forge, a module repository, or a release tarball.

By default, this command installs modules into the first directory in the Puppet modulepath, which defaults to `$codedir/environments/production/modules`. For example, to install the `puppetlabs-apache` module, run:

```
puppet module install puppetlabs-apache
```

You can customize the module version, installation directory, or environment, get debugging information, or ignore dependencies by passing options with the `puppet module install` command.


## 2.1 Installing from another module repository

You can install modules from other repositories that mimic the Forge interface. You can change the module repository for one installation, or you can change your default repository.

To change the module repository for a single module installation, specify the base URL of the repository on the command line with the `--module_repository` option. For example:

```
puppet module install --module_repository http://dev-forge.example.com puppetlabs-apache
```

To change the default module repository, edit the `module_repository` setting in the `puppet.conf` to the base URL of the repository you want to use. The default value for the `module_repository` is the Forge URL, `https://forgeapi.puppetlabs.com`. See the `module_repository` setting in the [`puppet.conf` configuration](https://www.puppet.com/docs/puppet/7/configuration) documentation.


# 3 Install modules on nodes without internet

To manually install a module on a node with no internet, download the module on a connected machine, and then move a module package to the unconnected node. If the module is a PE-only module, the download machine must have a valid PE license.

> Before you begin
	Make sure you have PDK installed. You'll use PDK to build a module package that you can move to your unconnected node. For installation instructions, see the [PDK install docs](https://puppet.com/docs/pdk/1.x/pdk_install.html).

> Tip: On machines with no internet access, you must install any module dependencies manually. Check your dependencies at the beginning of this process, so that you can move all of the necessary modules to the unconnected node at one time.

1. On a node with internet access, run `puppet module install puppetlabs-<MODULE>`
2. Change into the module's directory by running `cd <MODULE_NAME>`
3. Build a package from the installed module by running `pdk build`
4. Move the `*.tar.gz` to the machine on which you want to install the module.
5. Install the `tar.gz` package with the `puppet module install` command. For example:
    
    ```
    puppet module install puppetlabs-pe_module-0.1.0.tar.gz
    ```
    
6. Manually install the module's dependencies. Without internet access, `puppet module install` cannot install dependencies automatically.


# 4 Upgrading modules

To upgrade a module to a newer version, use the `puppet module upgrade` command.

This command upgrades modules to the most recent released version of the module. This includes upgrading the module to the most recent major version.

Specify the module you want to upgrade with the module's full name. For example:

```
puppet module upgrade puppetlabs-apache
```

To upgrade to a specific version, specify the version you want with the `--version` option. For example, to upgrade `puppetlabs-apache` version 2.2.0 without any breaking changes, specify the 2.x release to upgrade to:

```
puppet module upgrade puppetlabs-apache --version 2.3.1
```

You can also ignore changes or dependencies when upgrading with command line options. See the `puppet module` command reference for a complete list of options.


# 5 Uninstalling modules

Completely remove installed modules with the `puppet module uninstall` command.

This command uninstalls modules from the modulepath specified in the `puppet.conf` file. To remove a module, run the uninstall command with the full name of the module. For example:

```
puppet module uninstall puppetlabs-apache
```

By default, the command exits and returns an error if you try to uninstall a module that other modules depend on or if the module's files have been modified after it was installed. You can forcibly uninstall dependencies or changed modules with command line options.

For example, to uninstall a module that other modules depend on, run:

```
puppet module uninstall --force
```

See the `puppet module` command reference for a complete list of options.


# 6 `puppet module` command reference

The puppet module command manages modules with several actions and options.

## 6.1 `puppet module` actions

|Action|Description|Arguments|Example|
|---|---|---|---|
|`build`|Deprecated. Prepares a local module for release on the Forge by building a ready-to-upload archive file. Will be removed in a future release; use Puppet Development Kit instead.|A valid directory path to a module.|`puppet module build modules/apache`|
|`changes`|Compares the files on disk to the md5 checksums and returns an array of paths of modified files.|A valid directory path to a module.|`puppet module changes /etc/code/modules/stdlib`|
|`install`|Installs a module.|The full name <`username-module_name`> of the module to uninstall.|`` `puppet module install puppetlabs-apache` ``|
|`list`|Lists the modules installed in the modulepath specified in the `[main]` block in the `puppet.conf` file.|None.|`puppet module list`|
|`search`|**Deprecated. Visit the [Forge](https://forge.puppet.com/) to search for modules.** This command searched the Forge for modules matching search values.|Deprecated.|Deprecated|
|`uninstall`|Uninstalls a module.|The full name <`username-module_name`> of the module to uninstall.|`puppet module uninstall puppetlabs-apache`|
|`upgrade`|Upgrades a module to the most recent release or to the specified version. Does not upgrade dependencies.|The full name <`username-module_name`> of the module to upgrade.|`puppet module upgrade puppetlabs-apache --version 0.0.3`|

## 6.2 `puppet module install` action

Installs a module from the Forge or another specified release archive.

Usage:

```
puppet module install [--debug] [--environment] [--force | -f] [--ignore-dependencies]
  [--module_repository <REPOSITORY_URL>] [--strict-semver]
  [--target-dir <DIRECTORY/PATH> | -i <DIRECTORY/PATH>] [--version <x.x.x> | -v <x.x.x>]
  <full_module_name>
```

For example:

```
puppet module install --environment testing --ignore-dependencies 
  --version 1.0.0-pre1 --strict-semver false puppetlabs-apache
```

   
|Option|Description|Value|Default|
|---|---|---|---|
|`--debug`, `-d`|Displays additional information about what the `puppet module` command is doing.|None.|If not specified, additional information is not displayed.|
|`--environment`|Installs the module into the specified environment.|An environment name.|By default, installs the module into the default environment specified in the `puppet.conf` file.|
|`--force`, `-f`|Installs the module regardless of dependency tree, checksum changes, or whether the module is already installed. By default, installs the module in the default modulepath, even if the module is already installed in another directory. Does not install dependencies.|None.|If not specified, `puppet module install` exits and returns information if it encounters installation errors or conflicts.|
|`--ignore-dependencies`|Does not install any modules required by this module.|None.|If not specified, the `puppet module install` action installs the module and its dependencies.|
|`--module_repository`|Specifies a module repository.|A valid URL for a module repository.|If not specified, installs modules from the module repository specified in from the `puppet.conf` file. By default, this is the URL for the Forge.|
|`--strict-semver`|Whether to exclude pre-release versions. A value of false allows installation of pre-release versions.|`true`, `false`|Defaults to `true`, excluding pre-release versions.|
|`--target-dir`, `-i`|Specifies a directory to install modules.|A valid directory path.|By default, installs modules into `$codedir/environments/production/modules`|
|`--version`, `-v`|Specifies the module version to install.|A semantic version number, such as 1.2.1 or a string specifying a requirement, such as ">=1.0.3".|If not specified, installs the most recent version available on the Forge.|

## 6.3 `puppet module list` action

Lists the Puppet modules installed in the modulepath specified in the `puppet.conf` file's `[main]` block. Use the `--modulepath` option to change which directories are scanned.

Usage:

```
puppet module list [--tree] [--strict-semver]
```

For example:

```
puppet module list --tree --modulepath etc/testing/modules
```

   
|Option|Description|Value|Default|
|---|---|---|---|
|`--modulepath`|Specifies another modulepath to scan for modules.|A valid directory path.|By default, scans the default modulepath from the `[main]` block in the `puppet.conf` file.|
|`--strict-semver`|Whether to exclude pre-release versions. A value of false allows uninstallation of pre-release versions.|`true`, `false`|Defaults to `true`, excluding pre-release versions.|
|`--tree`|Displays the module list as a tree showing dependencies.|None.|By default, `puppet module list` lists installed modules but does not show dependency relationships.|

## 6.4 `puppet module uninstall` action

Uninstalls a module from the default modulepath.

Usage:

```
puppet module uninstall [--force | -f] [--ignore-changes | -c] [--strict-semver] [--version=]  <full_module_name>
```

For example:

```
puppet module uninstall --ignore-changes --version 0.0.2 puppetlabs-apache
```

   
|Option|Description|Value|Default|
|---|---|---|---|
|`--force`, `-f`|Uninstalls the module regardless of dependency tree or checksum changes.|None.|By default, `puppet module uninstall` exits and returns an error if it encounters changes, namespace errors, or dependencies.|
|`--ignore-changes`|Does not use the checksum and uninstalls regardless of modified files.|None.|By default, if the `puppet module uninstall` action finds modified files in the module, it exits and returns an error.|
|`--strict-semver`|Whether to exclude pre-release versions. A value of false allows uninstallation of pre-release versions.|`true`, `false`|Defaults to `true`, excluding pre-release versions.|
|`--version`, `-v`|Specifies the module version to uninstall.|A semantic version number, such as 1.2.1 or a string specifying a requirement, such as ">=1.0.3"..|By default, `puppet module uninstall` uninstalls the version installed in the modulepath.|

## 6.5 `puppet module upgrade` action

This command upgrades modules to the most recent released version of the module. This includes upgrades to the most recent major version.

Usage:

```
puppet module upgrade [--force | -f] [--ignore-changes | -c] [--ignore-dependencies] 
  [--strict-semver] [--version=] <full_module_name>
```

For example:

```
puppet module upgrade --force --version 2.1.2 puppetlabs-apache
```

   
|Option|Description|Value|Default|
|---|---|---|---|
|`--force`, `-f`|Upgrades the module regardless of dependency tree or checksum changes.|None.|By default, `puppet module upgrade` exits and returns an error if it encounters changes, namespace errors, or dependencies.|
|`--ignore-changes`|Does not use the checksum and upgrades regardless of modified files.|None.|By default, if the `puppet module upgrade` action finds modified files in the module, it exits and returns an error.|
|`--ignore-dependencies`|Does not attempt to install any missing modules required by this module.|None.|If not specified, the `puppet module upgrade` action installs missing module dependencies.|
|`--strict-semver`|Whether version ranges must exclude pre-release versions.|`true`, `false`|Defaults to `true`, excluding pre-release versions.|
|`--version`, `-v`|Specifies the module version to uninstall.|A semantic version number, such as 1.2.1 or a string specifying a requirement, such as ">=1.0.3".|By default, `puppet module uninstall` uninstalls the version installed in the modulepath.|


