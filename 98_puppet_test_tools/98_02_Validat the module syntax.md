https://www.puppet.com/docs/pdk/3.x/pdk_testing

- Validate the `metadata.json` file.
- Validate Puppet syntax.
- Validate Puppet code style.
- Validate Ruby code style.

# 1 pdk validate

By default, the `validate` command runs metadata validation first, then Puppet validation, then Ruby validation. Optionally, you can validate only certain files or directories, run a specific type of validations, such as metadata or Puppet validation, or run all validations simultaneously. Additionally, you can send your validation output to a file in either JUnit or text format.

1. From the command line, change into the module's directory with `cd <MODULE_NAME>`
    
2. Run all validations by running `pdk validate` .
    
    To change validation behavior, add option flags to the command. For example, to run all validations simultaneously on multiple threads, run:
    
    ```
    pdk validate --parallel
    ```
    
    To validate against a specific version of Puppet or PE, add the `--puppet-version` option flag.
    
    For example. To validate against Puppet 5.5.12, run:
    
    ```
    pdk validate --puppet-version 5.5.12
    ```
    
    For a complete list of command options and usage information, see the PDK command [reference](https://www.puppet.com/docs/pdk/3.x/pdk_reference#).
    

# 2 Ignoring files during module validation

There are times when certain files can be ignored when validating the contents of a Puppet module. PDK provides a configuration option to list sets of files to ignore when running any of the validators.

To configure PDK to ignore a file, use `pdk set config project.validate.ignore`. The `project.validate.ignore` setting accepts multiple files. To add one or more files, run the command for each file or pattern you want to add.

For example, to ignore a file called `example.yaml` in the folder called `config`, you would run the following command:

```
pdk set config project.validate.ignore "config/example.yaml"
```

To add a wildcard, use a valid [Git ignore pattern](http://git-scm.com/docs/gitignore):

```
pdk set config project.validate.ignore "config/*.yaml"
```