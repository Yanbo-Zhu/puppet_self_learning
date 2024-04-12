
[https://puppet.com/docs/puppet/7/configuration.html](https://puppet.com/docs/puppet/7/configuration.html)

- Each of these settings can be specified in puppet.conf or on the command line.
- Puppet Enterprise (PE) and open source Puppet share the configuration settings documented here. However, PE defaults differ from open source defaults for some settings, such as node_terminus, storeconfigs, always_retry_plugins, disable18n, environment_timeout (when Code Manager is enabled), and the Puppet Server JRuby max-active-instances setting. To verify PE configuration defaults, check the puppet.conf or pe-puppet-server.conf file after installation.
- When using boolean settings on the command line, use --setting and --no-setting instead of --setting (true|false). (Using --setting false results in "Error: Could not parse application options: needless argument".)
- Settings can be interpolated as $variables in other settings; $environment is special, in that puppet master will interpolate each agent node's environment instead of its own.
- Multiple values should be specified as comma-separated lists; multiple directories should be separated with the system path separator (usually a colon).
- Settings that represent time intervals should be specified in duration format: an integer immediately followed by one of the units 'y' (years of 365 days), 'd' (days), 'h' (hours), 'm' (minutes), or 's' (seconds). The unit cannot be combined with other units, and defaults to seconds when omitted. Examples are '3600' which is equivalent to '1h' (one hour), and '1825d' which is equivalent to '5y' (5 years).
- If you use the splay setting, note that the period that it waits changes each time the Puppet agent is restarted.
- Settings that take a single file or directory can optionally set the owner, group, and mode for their value: rundir = $vardir/run { owner = puppet, group = puppet, mode = 644 }
- The Puppet executables ignores any setting that isn't relevant to their function.

agent_catalog_run_lockfile

agent_disabled_lockfile

allow_duplicate_certs

always_retry_plugins

autoflush

autosign

basemodulepath

binder_config

bucketdir

ca_fingerprint

ca_name

ca_port

ca_server

ca_ttl

cacert

cacrl

cadir

cakey

capub

catalog_cache_terminus

catalog_terminus

cert_inventory

certdir

certificate_revocation

certname

ciphers

classfile

client_datadir

clientbucketdir

clientyamldir

code

codedir

color

confdir

config

config_file_name

config_version

configprint

crl_refresh_interval

csr_attributes

csrdir

daemonize

data_binding_terminus

default_file_terminus

default_manifest

default_schedules

deviceconfdir

deviceconfig

devicedir

diff

diff_args

digest_algorithm

disable_i18n

disable_per_environment_manifest

disable_warnings

dns_alt_names

document_all

environment

environment_data_provider

environment_timeout

environmentpath

evaltrace

external_nodes

fact_name_length_soft_limit

fact_value_length_soft_limit

factpath

facts_terminus

fileserverconfig

filetimeout

forge_authorization

freeze_main

genconfig

genmanifest

graph

graphdir

group

hiera_config

hostcert

hostcrl

hostcsr

hostprivkey

hostpubkey

http_connect_timeout

http_debug

http_extra_headers

http_keepalive_timeout

http_proxy_host

http_proxy_password

http_proxy_port

http_proxy_user

http_read_timeout

http_user_agent

ignore_plugin_errors

ignoremissingtypes

ignoreschedules

key_type

keylength

lastrunfile

lastrunreport

ldapattrs

ldapbase

ldapclassattrs

ldapparentattr

ldappassword

ldapport

ldapserver

ldapssl

ldapstackedattrs

ldapstring

ldaptls

ldapuser

libdir

localcacert

localedest

localesource

location_trusted

log_level

logdest

logdir

manage_internal_file_permissions

manifest

masterport

max_deprecations

max_errors

max_warnings

maximum_uid

maxwaitforcert

maxwaitforlock

merge_dependency_warnings

mkusers

module_groups

module_repository

module_working_dir

modulepath

name

named_curve

no_proxy

node_cache_terminus

node_name_fact

node_name_value

node_terminus

noop

number_of_facts_soft_limit

onetime

passfile

path

payload_soft_limit

pidfile

plugindest

pluginfactdest

pluginfactsource

pluginsignore

pluginsource

pluginsync

postrun_command

preferred_serialization_format

preprocess_deferred

prerun_command

preview_outputdir

priority

privatedir

privatekeydir

profile

publicdir

publickeydir

puppet_trace

puppetdlog

report

report_include_system_store

report_port

report_server

reportdir

reports

reporturl

requestdir

resourcefile

resubmit_facts

rich_data

route_file

rundir

runinterval

runtimeout

serial

server

server_datadir

server_list

serverport

settings_catalog

show_diff

signeddir

skip_tags

sourceaddress

splay

splaylimit

srv_domain

ssl_client_header

ssl_client_verify_header

ssl_lockfile

ssl_trust_store

ssldir

statedir

statefile

statettl

static_catalogs

storeconfigs

storeconfigs_backend

strict

strict_environment_mode

strict_variables

summarize

supported_checksum_types

syslogfacility

tags

tasks

top_level_facts_soft_limit

trace

transactionstorefile

trusted_external_command

trusted_oid_mapping_file

use_cached_catalog

use_last_environment

use_srv_records

usecacheonfailure

user

vardir

vendormoduledir

versioned_environment_dirs

waitforcert

waitforlock

write_catalog_summary

yamldir

