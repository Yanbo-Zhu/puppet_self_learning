
https://www.puppet.com/docs/puppet/8/modules_metadata.html

The `metadata.json` file is located in the module's main directory, outside any subdirectories.
The metadata.json file uses standard JSON syntax and contains a single JSON object, mapping keys to values

# metadata.json example 

```json

{
  "name": "puppetlabs-ntp",
  "version": "6.1.0",
  "author": "puppetlabs",
  "summary": "Installs, configures, and manages the NTP service.",
  "license": "Apache-2.0",
  "source": "https://github.com/puppetlabs/puppetlabs-ntp",
  "project_page": "https://github.com/puppetlabs/puppetlabs-ntp",
  "issues_url": "https://tickets.puppetlabs.com/browse/MODULES",
  "dependencies": [
    { "name":"puppetlabs/stdlib","version_requirement":">= 4.13.1 < 5.0.0" }
  ],
  "operatingsystem_support": [
    {
      "operatingsystem": "RedHat",
      "operatingsystemrelease": [
        "5",
        "6",
        "7"
      ]
    },
    {
      "operatingsystem": "CentOS",
      "operatingsystemrelease": [
        "5",
        "6",
        "7"
      ]
    }
  ],
  "requirements": [
    {
      "name": "puppet",
      "version_requirement": ">= 4.5.0 < 5.0.0"
    }
  ]
}

```


# Specifying dependencies

If your module depends on functionality from another module, specify this in the `"dependencies"` key of the `metadata.json` file. The `"dependencies"` key accepts an array of hashes. This key is required, but if your module has no dependencies, you can pass an empty array.

Dependencies are not added to the metadata during module creation, so you must edit your `metadata.json` file to include dependency information. For information about how to format dependency versions, see the related topic about version specifiers in module metadata.

The hash for each dependency must contain the `"name"` and `"version_requirement"` keys. For example:

```
"dependencies": [
  { "name": "puppetlabs/stdlib", "version_requirement": ">= 3.2.0 < 5.0.0" },
  { "name": "puppetlabs/firewall", "version_requirement": ">= 0.0.4" },
  { "name": "puppetlabs/apt", "version_requirement": ">= 1.1.0 < 2.0.0" },
  { "name": "puppetlabs/concat", "version_requirement": ">= 1.0.0 < 2.0.0" }
]
```

When installing modules with the `puppet module install` command, Puppet installs any missing dependencies. When installing modules with Code Manager and the Puppetfile, dependencies are not automatically installed, so they must be explicitly specified in the Puppetfile.


# Specifying Puppet version requirements

The `requirements` key specifies external requirements for the module, particularly the Puppet version required. Although you can express any requirement here, the Forge module pages and search function support only the `"puppet"` value, which specifies the Puppet version.

The `"requirements"` key accepts an array of hashes with the following keys:
- `"name"`: The name of the requirement.
- `"version_requirement"`: A semantic version range, including lower and upper version bounds.
    

For example, this key specifies that the module works with any Puppet version of 5.5.0 or greater, but not with Puppet 6 or later:

```
"requirements": [
  {"name": "puppet”, “version_requirement”: ">= 5.5.0 < 6.0.0"}
]
```

Important: The Forge requires both lower and upper bounds for the Puppet version requirement. If you upload a module that does not specify an upper bound, the Forge adds an upper bound of the next major version. For example, if you upload a module that specifies a lower bound of 5.5.0 and no upper bound, the Forge applies an upper bound of `< 6.0.0` .

For Puppet Enterprise versions, specify the core Puppet version included in that version of PE. For example, PE 2017.1 contained Puppet 4.9. Do not express requirements for Puppet versions earlier than 3.0, because those versions do not follow semantic versioning. For information about formatting version requirements, see the related topic about version specifiers in module metadata.


# Specifying operating system compatibility

Specify the operating system your module is compatible with in the `operatingsystem_support` key. This key accepts an array of hashes, where each hash contains `operatingsystem` and `operatingsystemrelease` keys. The Forge uses these keys for search filtering and to display versions on module pages.

- The `operatingsystem` key accepts a string. The Forge uses this value for search filters.
- The `operatingsystemrelease` accepts an array of strings. The Forge displays these versions on module pages, and you can format them in whatever way makes sense for the operating system in question.
    

For example:

```
"operatingsystem_support": [
  {
  "operatingsystem":"RedHat",
  "operatingsystemrelease":[ "5.0", "6.0" ]
  },
  {
  "operatingsystem": "Ubuntu",
  "operatingsystemrelease": [
    "12.04",
    "10.04"
    ]
  }
]
```


# Specifying versions

Your module metadata specifies your own module's version as well as the versions for your module's dependencies and requirements. Version your module semantically; for details about semantic versioning (also known as SemVer), see the [Semantic Versioning specification](http://semver.org). This helps others know what to expect from your module when you make changes.

When you specify versions for a module dependencies or requirements, you can specify multiple versions.

If your module is compatible with only one major or minor version, use the semantic major and minor version shorthand, such as 1.x or 1.2.1. If your module is compatible with multiple major versions, you can set a supported version range.

For example, 1.x indicates that your module is compatible with any minor update of version 1, but is not compatible with version 2 or larger. Specifying a version range such as >= 1.0.0 < 3.0.0 indicates the the module is compatible with any version that greater than or equal to 1.0.0 and less than 3.0.0.

Always set an upper version boundary in your version range. If your module is compatible with the most recent released versions of a dependencies, set the upper bound to exclude the next, unreleased major version. Without this upper bound, users might run into compatibility issues across major version boundaries, where incompatible changes occur.

For example, to accept minor updates to a dependency but avoid breaking changes, specify a major version. This example accepts any minor version of `puppetlabs-stdlib` version 4:

```
"dependencies": [
  { "name": "puppetlabs/stdlib", "version_requirement": "4.x" },
]
```

In the example below, the current version of `puppetlabs-stdlib` is 4.8.0, and version 5.0.0 is not yet released. Because 5.0.0 might have breaking changes, the upper bound of the version dependency is set to that major version.

```
"dependencies": [
  { "name": "puppetlabs/stdlib", "version_requirement": ">= 3.2.0 < 5.0.0" }
]            
```

The version specifiers allowed in module dependencies are:

 |Format|Description|
|---|---|
|1.2.3|A specific version.|
|1.x|A semantic major version. This example includes 1.0.1 but not 2.0.1.|
|1.2.x|A semantic major and minor version. This example includes 1.2.3 but not 1.3.0.|
|> 1.2.3|Greater than the specified version.|
|< 1.2.3|Less than the specified version.|
|>= 1.2.3|Greater than or equal to the specified version.|
|<= 1.2.3|Less than or equal to the specified version.|
|>= 1.0.0 < 2.0.0|Range of versions; both conditions must be satisfied. This example includes version 1.0.1 but not version 2.0.1.|

Note: You cannot mix semantic versioning shorthand (such as .x) with syntax for greater than or less than versioning. For example, you could not specify `">= 3.2.x < 4.x"`

Note: Specification of minor versions is for semantic purposes only. Underlying tools, such as facterdb and rspec, only recognize major versions.


# Adding tags

Optionally, you can add tags to your metadata to help users find your module in Forge searches. Generally, include four to six tags for any given module.

Pass tags as an array, like `["msyql", "database", "monitoring"]`.

Tags cannot contain whitespace. Certain tags are prohibited, such as profanity or tags resembling the `$::operatingsystem` fact (such as `"redhat"`, `"rhel"`, `"debian"`, `"` `windows"`, or `"osx"`). Use of prohibited tags lowers your module's quality score on the Forge.


# Available `metadata.json` keys

Required and optional `metadata.json` keys specify metadata for your module.

|Key|Required?|Value|Example|
|---|---|---|---|
|`` `"name"` ``|Required.|The full name of your module, including your Forge username, in the format `username-module`.|``"`puppetlabs-stdlib`"``|
|`"version"`|Required.|The current version of your module. This must follow semantic versioning. For details, see the [Semantic Versioning specification](http://semver.org/spec/v1.0.0.html).|`"1.2.1`"|
|`"author"`|Required.|The person who gets credit for creating the module. If absent, this key defaults to the username portion of the name key.|`"puppetlabs`"|
|`"license"`|Required.|The license under which your module is made available. License metadata must match an identifier provided by SPDX. For a complete list, see the [SPDX license list](https://spdx.org/licenses/).|"`Apache-2.0`"|
|`"summary"`|Required.|A one-line description of your module.|`"Standard library of resources for Puppet modules."`|
|`"source"`|Required.|The source repository for your module.|`"https://github.com/puppetlabs/puppetlabs-stdlib"`|
|`"dependencies"`|Required.|An array of other modules that your module depends on to function. If the module has no dependencies, pass an empty array. See the related topic about specifying dependencies for more details.|```<br>"dependencies": [<br>    {<br>      "name": "puppetlabs/stdlib",<br>      "version_requirement": ">= 4.13.1 < 6.0.0"<br>    }<br>],<br>```|
|`"requirements"`|Optional.|A list of external requirements for your module, given as an array of hashes.|```<br>"requirements": [<br>    {<br>      "name": "puppet",<br>      "version_requirement": ">= 4.7.0 < 6.0.0"<br>    }<br>],<br>```|
|`` `"project_page"` ``|Optional.|A link to your module's website, to be included on the module's Forge page.|`"https://github.com/puppetlabs/puppetlabs-stdlib"`|
|`` `"issues_url"` ``|Optional.|A link to your module's issue tracker.|`"https://tickets.puppetlabs.com/browse/MODULES"`|
|`` `"operatingsystem_support"` ``|Optional.|An array of hashes listing the operating systems that your module is compatible with. See the topic about specifying operating compatibility for details.|```<br>{<br>      "operatingsystem": "RedHat",<br>      "operatingsystemrelease": [<br>        "5",<br>        "6",<br>        "7"<br>      ]<br>}<br>```|
|`` `"tags"` ``|Optional.|An array of four to six key words to help people find your module.|`["msyql", "database", "monitoring", "reporting"]`|

