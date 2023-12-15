
[https://puppet.com/docs/puppet/7/modules_fundamentals.html#module-structure](https://puppet.com/docs/puppet/7/modules_fundamentals.html#module-structure)

data/
Contains data files specifying parameter defaults.

examples/
Contains examples showing how to declare the module's classes and defined types.
init.pp: The main class of the module.
example.pp: Provide examples for major use cases.

facts.d/
Contains external facts, which are an alternative to Ruby-based custom facts. These are synced to all agent nodes, so they can submit values for those facts to the primary Puppet server.

files/
Contains static files, which managed nodes can download.

files/service.conf
This file's source => URL is puppet:///modules/my_module/service.conf. Its contents can also be accessed with the file function, such as content => file('my_module/service.conf').

functions/
Contains custom functions written in the Puppet language.

lib/
Contains plug-ins, such as custom facts and custom resource types. These are used by both the primary Puppet server and the Puppet agent, and they are synced to all agent nodes in the environment on each Puppet run.

lib/facter/
Contains custom facts, written in Ruby.

lib/puppet/
Contains custom functions, resource types, and resource providers:

lib/puppet/functions/: Contains functions written in Ruby for the modern Puppet::Functions API.
lib/puppet/parser/functions/: Contains functions written in Ruby for the legacy Puppet::Parser::Functions API.
lib/puppet/provider/: Contains custom resource providers written in the Puppet language.
lib/puppet/type/: Contains custom resource types written in the Puppet language.

locales/
Contains files relating to module localization into languages other than English.

manifests/
Contains all of the manifests in the module.

manifests/init.pp
The init.pp class, if used, is the main class of the module. This class's name must match the module's name.

manifests/other_class.pp
Classes and defined types are named with the namespace of the module and the name of the class or defined type. For example, this class is named my_module::other_class.

manifests/implementation/
You can group related classes and defined types in subdirectories of the manifests/ directory. The name of this subdirectory is reflected in the names of the classes and types it contains. Classes and defined types are named with the namespace of the module, any subdirectories, and the name of the class or defined type.

manifests/implementation/my_defined_type.pp: This defined type is named my_module::implementation::my_defined_type.
manifests/implementation/class.pp: This defined type is named my_module::implementation::class.

plans/
Contains Puppet task plans, which are sets of tasks that can be combined with other logic. Plans are written in the Puppet language.

readmes/
The module's README localized into languages other than English.

spec/
Contains spec tests for any plug-ins in the lib directory.

tasks/
Contains Puppet tasks, which can be written in any programming language that can be read by the target node.

templates/
Contains templates, which the module's manifests can use to generate content or variable values.

templates/component.erb
A manifest can render this template with template('my_module/component.erb').

templates/component.epp
A manifest can render this template with epp('my_module/component.epp').

types/
Contains resource type aliases.


