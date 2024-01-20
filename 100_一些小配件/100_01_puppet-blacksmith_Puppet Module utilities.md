

https://confluence.ivu.de/display/SYS/%28needs+reworking%29+Proposal%3A+Automatic+Version+Increments+for+Puppet+Modules


https://github.com/voxpupuli/puppet-blacksmith

The Ruby Gem [puppet-blacksmith](https://github.com/voxpupuli/puppet-blacksmith) offers programmatic control of Puppet module version increments via `pdk bundle exec rake module:bump:{major,minor,patch}` and an extended operation `bump_commit` that also handles git commit creation for said increments.

The operation should be triggered during a pull request in Bitbucket. Version increments and their respective commits should happen inside of Pull Requests. This **prevents polluting the history of the main branch** with _Version Bump_ commits that have little informational value.

