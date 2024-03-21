
https://www.puppet.com/docs/puppet/6/hiera_merging#interpolation_functions

# 1 Set the merge behavior for a lookup

When you look up a key in Hiera, it is common for multiple data sources to have different values for it. By default, Hiera returns the first value it finds, but it can also continue searching and merge all the found values together.

1. You can set the merge behavior for a lookup in two ways:
    - At lookup time. This works with the `lookup` function, but does not support automatic class parameter lookup.
    - In Hiera data, with the `lookup_options` key. This works for both manual and automatic lookups. It also lets module authors set default behavior that users can override.
    
2. With both of these methods, specify a merge behavior as either a string, for example, `'first'` or a hash, for example `{'strategy' => 'first'}`. The hash syntax is useful for `deep` merges (where extra options are available), but it also works with the other merge types.

**Related information**  

- [The Puppet lookup function](https://www.puppet.com/docs/puppet/6/hiera_automatic#puppet_lookup "The lookup function uses Hiera to retrieve a value for a given key.")

# 2 Merge behaviors

There are four merge behaviors to choose from: `first`, `unique`, `hash`, and `deep`.

When specifying a merge behavior, use one of the following identifiers:

- `'first'`, `{'strategy' => 'first'}`, or nothing.
- `'unique'` or `{'strategy' => 'unique'}`.
- `'hash'` or `{'strategy' => 'hash'}`.
- `'deep'` or `{'strategy' => 'deep', <OPTION> => <VALUE>, ...}`. Valid options:
    - `'knockout_prefix'` - string or `undef`; [disabled by default](https://www.puppet.com/docs/puppet/6/known_issues_puppet#known_issues_puppet "These are the known issues in this version of Puppet.").
    - `'sort_merged_arrays'` - Boolean; default is `false`
    - `'merge_hash_arrays'` - Boolean; default is `false`

## 2.1 First

A first-found lookup doesn’t merge anything: it returns the first value found for the key, and ignores the rest. This is Hiera’s default behavior.

Specify this merge behavior with one of these:

- `'first'`
    
- `{'strategy' => 'first'}`
    
- `lookup($key)`
- Nothing (because it’s the default)
    

## 2.2 Unique

A unique merge (also called an array merge) combines any number of array and scalar (string, number, boolean) values to return a merged, flattened array with all matching values for a key. All duplicate values are removed. The lookup fails if any of the values are hashes. The result is ordered from highest priority to lowest.

Specify this merge behavior with one of these:

- `'unique'`
    
- `lookup($key, { 'merge' => 'unique' })`
- `{'strategy' => 'unique'}`
    

## 2.3 Hash

A hash merge combines the keys and values of any number of hashes to return a merged hash of all matching values for a key. Every match must be a hash and the lookup fails if any of the values aren’t hashes.

If multiple source hashes have a given key, Hiera uses the value from the highest priority data source: it won’t recursively merge the values.

Hashes in Puppet preserve the order in which their keys are written. When merging hashes, Hiera starts with the lowest priority data source. For each higher priority source, it appends new keys at the end of the hash and updates existing keys in place.

```
# web01.example.com.yaml
mykey:
  d: "per-node value"
  b: "per-node override"
# common.yaml
mykey:
  a: "common value"
  b: "default value"
  c: "other common value"


`lookup('mykey', {merge => 'hash'})
```

Returns the following:

```
{
  a => "common value",
  b => "per-node override", # Using value from the higher-priority source, but
                            # preserving the order of the lower-priority source.
  c => "other common value",
  d => "per-node value",
}
```

Specify this merge behavior with one of these:

- `'hash'`
    
- `lookup($key, { 'merge' => 'hash' })`
- `{'strategy' => 'hash'}`
    

## 2.4 Deep

A deep merge combines the keys and values of any number of hashes to return a merged hash. It contains an array of class names and can be used as a lightweight External Node Classifier (ENC).

If the same key exists in multiple source hashes, Hiera recursively merges them:

- Hash values are merged with another deep merge.
- Array values are merged. This differs from the unique merge. The result is ordered from lowest priority to highest, which is the reverse of the unique merge’s ordering. The result is not flattened, so it can contain nested arrays. The `merge_hash_arrays` and `sort_merged_arrays` options can make further changes to the result.
- Scalar (String, Number, Boolean) values use the highest priority value, like in a first-found lookup.

Specify this merge behavior with one of these:

- `'deep'`
- `include(lookup($key, { 'merge' => 'deep' }))`
- `{'strategy' => 'deep', <OPTION> => <VALUE>, ...}` — Adjust the merge behavior with these additional options:
    - `'knockout_prefix'` (String or `undef`) - Use with a string prefix to indicate a value to remove from the final result. Note that this option is disabled by default due to a known issue that causes it to be ineffective in hierarchies more than three levels deep. For more information, see [Puppet 6 known issues](https://www.puppet.com/docs/puppet/6/known_issues_puppet#known_issues_puppet "These are the known issues in this version of Puppet.").
    - `'sort_merged_arrays'` (Boolean) - Whether to sort all arrays that are merged together. Defaults to `false`.
    - `'merge_hash_arrays'` (Boolean) - Whether to deep-merge hashes within arrays, by position. For example, `[ {a => high}, {b => high} ]` and `[ {c => low}, {d => low} ]` would be merged as `[ {c => low, a => high}, {d => low, b => high} ]`. Defaults to `false`.

Note: Unlike a hash merge, a deep merge can also accept arrays as the root values. It merges them with its normal array merging behavior, which differs from a unique merge as described above. This does not apply to the deprecated Hiera 3 `hiera_hash` function, which can be configured to do deep merges but can’t accept arrays.


# 3 Set merge behavior at lookup time

Use merge behaviour at lookup time to override preconfigured merge behavior for a key.

Use the `lookup` function or the `puppet lookup` command to provide a merge behavior as an argument or flag.

Function example:

```
# Merge several arrays of class names into one array:
lookup('classes', {merge => 'unique'})
```

Command line example:

```
$ puppet lookup classes --merge unique --environment production --explain
```

Note: Each of the deprecated `hiera_*` functions is locked to one particular merge behavior. (For example, Hiera only merges first-found, and `hiera_array` only performs a unique merge.)

# 4 Set `lookup_options` to refine the result of a lookup

You can set `lookup_options` to further refine the result of a lookup, including defining merge behavior and using the `convert_to` key to get automatic type conversion.

## 4.1 The `lookup_options` format

The value of `lookup_options` is a hash. It follows this format:

```
 lookup_options:
  <NAME or REGEXP>:
    merge: <MERGE BEHAVIOR>
```

Each key is either the full name of a lookup key (like `ntp::servers`) or a regular expression (like `'^profile::(.*)::users$'`). In a module’s data, you can configure lookup keys only within that module’s namespace: the ntp module can set options for `ntp::servers`, but the `apache` module can’t.

Each value is a hash with either a `merge` key, a `convert_to key`, or both. A merge behavior can be a string or a hash, and the type for type conversion is either a Puppet type, or an array with a type and additional arguments.

`lookup_options` is a reserved key in Hiera. You can’t put other kinds of data in it, and you can’t look it up directly.

## 4.2 Location for setting `lookup_options`

You can set `lookup_options` metadata keys in Hiera data sources, including module data, which controls the default merge behavior for other keys in your data. Hiera uses a key’s configured merge behavior in any lookup that doesn’t explicitly override it.

Note: Set `lookup_options` in the data sources of your backend; **don’t put it in the `hiera.yaml` file**. For example, you can set `lookup_options` in `common.yaml`.

## 4.3 Defining Merge Behavior with `lookup_options`

In your Hiera data source, set the `lookup_options` key to configure merge behavior:

```
# <ENVIRONMENT>/data/common.yaml
lookup_options:
  ntp::servers:     # Name of key
    merge: unique   # Merge behavior as a string
  "^profile::(.*)::users$": # Regexp: `$users` parameter of any profile class
    merge:          # Merge behavior as a hash
      strategy: deep
      merge_hash_arrays: true
```

Hiera uses the configured merge behaviors for these keys.

Note: The `lookup_options` settings have no effect if you are using the deprecated `hiera_*` functions, which define for themselves how they do the lookup. To take advantage of `lookup_options`, use the lookup function or Automatic Parameter Lookup (APL).

## 4.4 Overriding merge behavior

When Hiera is given lookup options, a hash merge is performed. Higher priority sources override lower priority lookup options for individual keys. You can configure a default merge behavior for a given key in a module and let users of that module specify overrides in the environment layer.

As an example, the following configuration defines `lookup_options` for several keys in a module. One of the keys is overridden at the environment level – the others retain their configuration:

```
# <MYMODULE>/data/common.yaml
lookup_options:
  mymodule::key1:
    merge:
      strategy: deep
      merge_hash_arrays: true
  mymodule::key2:
    merge: deep
  mymodule::key3:
    merge: deep

# <ENVIRONMENT>/data/common.yaml
lookup_options:
  mymodule::key1:
    merge: deep # this overrides the merge_hash_arrays true
```

## 4.5 Overriding merge behavior in a call to `lookup()`

When you specify a merge behavior as an argument to the lookup function, it overrides the configured merge behavior. For example, with the configuration above:

```
lookup('mymodule::key1', 'strategy' => 'first')
```

The lookup of `'mymodule::key1'` uses strategy `'first'` instead of strategy `'deep'` in the `lookup_options` configuration.

## 4.6 **Make Hiera return data by casting to a specific data type**

To convert values from Hiera backends to rich data values, not representable in YAML or JSON, use the `lookup_options` key `convert_to`, which accepts either a type name or an array of type name and arguments.

When you use `convert_to`, you get automatic type checking. For example, if you specify a `convert_to` using type `"Enum['red', 'blue', 'green']"` and the looked-up value is not one of those strings, it raises an error. You can use this to assert the type when there is not enough type checking in the Puppet code that is doing the lookup.

For types that have a single-value constructor, such as Integer, String, Sensitive, or Timestamp, specify the data type in string form.

For example, to turn a String value into an Integer:

```
mymodule::mykey: "42"
lookup_options:
  mymodule::mykey:
    convert_to: "Integer"
```

To make a value Sensitive:

```
mymodule::mykey: 42
lookup_options:
 mymodule::mykey:
   convert_to: "Sensitive"
```

If the constructor requires arguments, specify type and the arguments in an array. You can also specify it this way when a data type constructor takes optional arguments.

For example, to convert a string ("042") to an Integer with explicit decimal (base 10) interpretation of the string:

```
mymodule::mykey: "042"
lookup_options:
  mymodule::mykey:
    convert_to:
      - "Integer"
      - 10
```

The default would interpret the leading 0 to mean an octal value (octal 042 is decimal 34):

To turn a non-Array value into an Array:

```
mymodule::mykey: 42
lookup_options:
 mymodule::mykey:
   convert_to:
     - "Array"
     - true
```

**Related information**  

- [Automatic lookup of class parameters](https://www.puppet.com/docs/puppet/6/hiera_automatic#class_parameters "Puppet looks up the values for class parameters in Hiera, using the fully qualified name of the parameter (myclass::parameter_one) as a lookup key.")


# 5 Use a regular expression in `lookup_options`

You can use regular expressions in `lookup_options` to configure merge behavior for many lookup keys at the same time.

A regular expression key such as `'^profile::(.*)::users$'` sets the merge behavior for `profile::server::users, profile::postgresql::users`, `profile::jenkins::server::users`. Regular expression lookup options use Puppet’s regular expression support, which is based on Ruby’s regular expressions.

To use a regular expression in `lookup_options`:

1. Write the pattern as a quoted string. Do not use the Puppet language’s forward-slash `(/.../)` regular expression delimiters.
2. Begin the pattern with the start-of-line metacharacter (`^`, also called a carat). If ^ isn’t the first character, Hiera treats it as a literal key name instead of a regular expression.
3. If this data source is in a module, follow `^` with the module’s namespace: its full name, plus the `::` namespace separator. For example, all regular expression lookup options in the `ntp` module must start with `^ntp::`. Starting with anything else results in an error.

Results

The merge behavior you set for that pattern applies to all lookup keys that match it. In cases where multiple lookup options could apply to the same key, Hiera resolves the conflict. For example, if there’s a literal (not regular expression) option available, Hiera uses it. Otherwise, Hiera uses the first regular expression that matches the lookup key, using the order in which they appear in the module code.

Note: `lookup_options` are assembled with a hash merge, which puts keys from lower priority data sources before those from higher priority sources. To override a module’s regular expression configured merge behavior, use the exact same regular expression string in your environment data, so that it replaces the module’s value. A slightly different regular expression won’t work because the lower-priority regular expression goes first.


# 6 Interpolation

In Hiera you can insert, or interpolate, the value of a variable into a string, using the syntax `%{variable}`.

Hiera uses interpolation in two places:

- Hierarchies: you can interpolate variables into the `path`, `paths`, `glob`, `globs`, `uri`, `uris`, `datadir`, `mapped_paths`, and `options` of a hierarchy level. This lets each node get a customized version of the hierarchy.
- Data: you can use interpolation to avoid repetition. This takes one of two forms:
    - If some value always involves the value of a fact (for example, if you need to specify a mail server and you have one predictably-named mail server per domain), reference the fact directly instead of manually transcribing it.
    - If multiple keys need to share the same value, write it out for one of them and reuse it for the rest with the `lookup` or `alias` interpolation functions. This makes it easier to keep data up to date, as you only need to change a given value in one place.

## 6.1 Interpolation token syntax

Interpolation tokens consist of the following:

- A percent sign (`%`)
    
- An opening curly brace (`{`)
    
- One of:
    
    - A variable name, optionally using key.subkey notation to access a specific member of a hash or array.
        
    - An interpolation function and its argument.
        
- A closing curly brace (`}`).
    

For example, `%{trusted.certname}` or `%{alias("users")}`.

Hiera interpolates values of Puppet data types and converts them to strings. Note that the exception to this is when using an alias. If the alias is the only thing present, then its value is not converted.

In YAML files, any string containing an interpolation token must be enclosed in quotation marks.

Note: Unlike the Puppet interpolation tokens, you can’t interpolate an arbitrary expression.

Related topics: [Puppet’s data types](https://www.puppet.com/docs/puppet/6/lang_data "Most of the things you can do with the Puppet language involve some form of data. An individual piece of data is called a value, and every value has a data type, which determines what kind of information that value can contain and how you can interact with it."), [Puppet’s rules for interpolating non-string values](https://www.puppet.com/docs/puppet/6/lang_data_string#lang_data_string "Strings are unstructured text fragments of any length. They’re a common and useful data type.").

# 7 Interpolate a Puppet variable

The most common thing to interpolate is the value of a Puppet top scope variable.

The `facts` hash, `trusted` hash, and `server_facts` hash are the most useful variables to Hiera and behave predictably.

Note: If you have a hierarchy level that needs to reference the name of a node, get the node’s name by using `trusted.certname`. To reference a node’s environment, use `server_facts.environment`.

Avoid using local variables, namespaced variables from classes (unless the class has already been evaluated), and Hiera-specific pseudo-variables (pseudo-variables are not supported in Hiera 5).

If you are using Hiera 3 pseudo-variables, see Puppet variables passed to Hiera.

Puppet makes facts available in two ways: grouped together in the `facts` hash ( `$facts['networking']`), and individually as top-scope variables ( `$networking`).

When you use individual fact variables, specify the (empty) top-scope namespace for them, like this:

- `%{::networking}`

Not like this:

- `%{networking}`

Note: The individual fact names aren’t protected the way `$facts` is, and local scopes can set unrelated variables with the same names. In most of Puppet, you don’t have to worry about unknown scopes overriding your variables, but in Hiera you do.

To interpolate a Puppet variable:

Use the name of the variable, omitting the leading dollar sign (`$`). Use the Hiera key.subkey notation to access a member of a data structure. For example, to interpolate the value of `$facts['networking']['domain']` write: `smtpserver: "mail.%{facts.networking.domain}"`

Results

For more information, see [facts](https://www.puppet.com/docs/puppet/6/lang_facts_and_builtin_vars "Before requesting a catalog for a managed node, or compiling one with puppet apply, Puppet collects system information, called facts, by using the Facter tool. The facts are assigned as values to variables that you can use anywhere in your manifests. Puppet also sets some additional special variables, called built-in variables, which behave a lot like facts."), [environments](https://www.puppet.com/docs/puppet/6/environments_about#environments_about "An environment is a branch that gets turned into a directory on your primary server.").

**Related information**  

- [Access hash and array elements using a key.subkey notation](https://www.puppet.com/docs/puppet/6/hiera_automatic#access_hash_array-elements_keysubkey_notation "Access hash and array members in Hiera using a key.subkey notation.")

## 7.1 Interpolation functions

In Hiera data sources, you can use interpolation functions to insert non-variable values. These aren’t the same as Puppet functions; they’re only available in Hiera interpolation tokens.

Note: You cannot use interpolation functions in `hiera.yaml.` They’re only for use in data sources.

There are five interpolation functions:

- `lookup` - looks up a key using Hiera, and interpolates the value into a string
    
- `hiera` - a synonym for `lookup`
    
- `alias` - looks up a key using Hiera, and uses the value as a replacement for the enclosing string. The result has the same data type as what the aliased key has - no conversion to string takes place if the value is exactly one alias
    
- `literal` - a way to write a literal percent sign (`%`) without accidentally interpolating something
    
- `scope` - an alternate way to interpolate a variable. Not generally useful
    

## 7.2 The `lookup` and `hiera` function

The `lookup` and `hiera` interpolation functions look up a key and return the resulting value. The result of the lookup must be a string; any other result causes an error. The `hiera` interpolation functions look up a key and return the resulting value. The result of the lookup must be a string; any other result causes an error.

This is useful in the Hiera data sources. If you need to use the same value for multiple keys, you can assign the literal value to one key, then call lookup to reuse the value elsewhere. You can edit the value in one place to change it everywhere it’s used.

For example, suppose your WordPress profile needs a database server, and you’re already configuring that hostname in data because the MySQL profile needs it. You could write:

```
# in location/pdx.yaml:
profile::mysql::public_hostname: db-server-01.pdx.example.com

# in location/bfs.yaml:
profile::mysql::public_hostname: db-server-06.belfast.example.com

# in common.yaml:
profile::wordpress::database_server: "%{lookup('profile::mysql::public_hostname')}"
```

The value of `profile::wordpress::database_server` is always the same as `profile::mysql::public_hostname`. Even though you wrote the WordPress parameter in the `common.yaml` data, it’s location-specific, as the value it references was set in your per-location data files.

The value referenced by the lookup function can contain another call to lookup; if you accidentally make an infinite loop, Hiera detects it and fails.

Note: The `lookup` and `hiera` interpolation functions aren’t the same as the Puppet functions of the same names. They only take a single argument.

## 7.3 The `alias` function

The `alias` function lets you reuse Hash, Array, Boolean, Integer or String values.

When you interpolate `alias` in a string, Hiera replaces that entire string with the aliased value, using its original data type. For example:

```
original:
  - 'one'
  - 'two'
aliased: "%{alias('original')}"
```

A lookup of original and a lookup of aliased would both return the value `['one', 'two']`.

When you use the `alias` function, its interpolation token must be the only text in that string. For example, the following would be an error:

```
aliased: "%{alias('original')} - 'three'"
```

Note: A lookup resulting in an interpolation of `alias` referencing a non-existant key returns an empty string, not a Hiera "not found" condition.

## 7.4 The `literal` function

The `literal` interpolation function lets you escape a literal percent sign (`%`) in Hiera data, to avoid triggering interpolation where it isn’t wanted.

This is useful when dealing with Apache config files, for example, which might include text such as `%{SERVER_NAME}`. For example:

```
server_name_string: "%{literal('%')}{SERVER_NAME}"
```

The value of `server_name_string` would be `%{SERVER_NAME}`, and Hiera would not attempt to interpolate a variable named `SERVER_NAME`.

The only legal argument for `literal` is a single `%` sign.

## 7.5 The `scope` function

The `scope` interpolation function interpolates variables.

It works identically to variable interpolation. The functions argument is the name of a variable.

The following two values would be identical:

```
smtpserver: "mail.%{facts.domain}"
smtpserver: "mail.%{scope('facts.domain')}"
```

## 7.6 Using interpolation functions

To use an interpolation function to insert non-variable values, write:

1. The name of the function.
    
2. An opening parenthesis.
    
3. One argument to the function, enclosed in single or double quotation marks.
    
4. Use the opposite of what the enclosing string uses: if it uses single quotation marks, use double quotation marks.
    
5. A closing parenthesis.
    

For example:

```
wordpress::database_server: "%{lookup('instances::mysql::public_hostname')}"
```

Note: There must be no spaces between these elements.
