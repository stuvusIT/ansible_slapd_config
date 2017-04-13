# slapd-config

This role allows configuring the entire OLC of an OpenLDAP server.
It requires you to have already created an OLC and have slapd running.
Check out the [slapd-base](https://github.com/stuvusIT/slapd-base) for this purpose.

slapd-config will create the global slapd configuration directly in `cn=config`, and load global modules like mdb if they are not compiled as static backends.
Afterwards, the monitoring backend and an MDB are created and configured.
The MDB supports specifying overlays.

## Requirements

A dpkg- or pacman-based Linux distribution.

## Role Variables

**All variables must be quoted when they contain spaces!**"

This role requires the variables that are exported by `slapd-base`, so run it first or check out the variables to set there.
There are some role-global variables:

| Name                            | Required                 | Default         | Description                                                                     |
|---------------------------------|:------------------------:|-----------------|---------------------------------------------------------------------------------|
| `slapd_enable_monitor`          | :heavy_check_mark:       | `true`          | Load the monitor module (if required) and configure the monitor database.       |
| `slapd_modules_path`            | :heavy_check_mark:       | `/usr/lib/ldap` | Path to dynamic modules, if backends are required but need to be loaded.        |
| `slapd_additional_modules`      | :heavy_multiplication_x: |                 | Also load these modules (apart from MDB and monitor) if not compiled in.        |
| `slapd_olc_rootdn_password`     | :heavy_check_mark:       |                 | Password to access the OLC. Not automatically exported by `slapd-base`.         |
| `slapd_monitor_rootdn_password` | :heavy_check_mark:       |                 | The rootdn password for the monitor database. Will automatically be hashed.     |
| `slapd_mdb_rootdn_password`     | :heavy_check_mark:       |                 | The rootdn password for the MDB database. Will automatically be hashed.         |
| `slapd_schemas`                 | :heavy_multiplication_x: |                 | Array of paths to schema files to load.                                         |
| `slapd_global_config`           | :heavy_check_mark:       | See description | Every single global slapd configuration option. See below for each description. |
| `slapd_olc_config`              | :heavy_multiplication_x: | See description | Every single global configuration value for the OLC database.                   |
| `slapd_monitor_config`          | :heavy_multiplication_x: | See description | Every single global configuration value for the monitor database.               |
| `slapd_mdb_config`              | :heavy_check_mark:       | See description | Every single global configuration value for the MDB database.                   |
| `slapd_mdb_overlays`            | :heavy_multiplication_x: | See description | Every module for the MDB. See example for an example.                           |

#### slapd-config variables

You need to set these variables if `slapd-base` was not run in a previous step in this playbook.
All variables are required.

| Name                 | Required           | Default | Description                                                |
|----------------------|:------------------:|---------|------------------------------------------------------------|
| `slapd_run_dir`      | :heavy_check_mark: |         | Runtime directory for args file, pid file and ldapi socket |
| `slapd_ldapi_socket` | :heavy_check_mark: |         | ldapi unix socket for local slapd administration           |
| `slapd_olc_dir`      | :heavy_check_mark: |         | Path where the LDIF files of the OLC reside                |
| `slapd_olc_rootdn`   | :heavy_check_mark: |         | Rootdn of the OLC                                          |


#### Global configuration options

The global slapd OLC configuration is separated into different sections.

###### General configuration

| Name            | Required                 | Default                        | Description                                                                                             |
|-----------------|:------------------------:|--------------------------------|---------------------------------------------------------------------------------------------------------|
| `olcConfigFile` | :heavy_multiplication_x: |                                | Path to a configuration file to load. Superseded by the OLC.                                            |
| `olcConfigDir`  | :heavy_check_mark:       | `{{slapd_olc_dir}}`            | Path to the OLC database files.                                                                         |
| `olcArgsFile`   | :heavy_multiplication_x: | `{{slapd_run_dir}}/slapd.args` | slapd will write its arguments to this file.                                                            |
| `olcPidFile`    | :heavy_check_mark:       | `{{slapd_run_dir}}/slapd.pid`  | slapd will write its PID to this file.                                                                  |
| `olcGentleHUP`  | :heavy_multiplication_x: | `FALSE`                        | When `TRUE`, slapd will not kill existing connections on `SIGHUP`, but will wait for them to terminate. |
| `olcServerID`   | :heavy_multiplication_x: | `0`                            | ID of this server. Only required with multi-master replication.                                         |

###### Security-related configuration

| Name                         | Required                 | Default                 | Description                                                                                                                                         |
|------------------------------|:------------------------:|-------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------|
| `olcAllows`                  | :heavy_multiplication_x: |                         | A set of features to allow.                                                                                                                         |
| `olcDisallows`               | :heavy_multiplication_x: |                         | A set of features to disallow.                                                                                                                      |
| `olcRequires`                | :heavy_multiplication_x: | `bind`                  | A set of conditions to require.                                                                                                                     |
| `olcRestrict`                | :heavy_multiplication_x: |                         | A list of operations that are restructed.                                                                                                           |
| `olcSecurity`                | :heavy_multiplication_x: | `ssf=1 simple_bind=128` | Specify a set of security strength factors to require.                                                                                              |
| `olcAuthIDRewrite`           | :heavy_multiplication_x: |                         | Used to convert simple user names to an LDAP DN used for auth purposes.                                                                             |
| `olcAuthzRegexp`             | :heavy_multiplication_x: |                         | Used to convert simple user names to an LDAP DN used for auth purposes. Can be specified multiple times and requires server restart to take effect. |
| `olcAuthzPolicy`             | :heavy_multiplication_x: | `none`                  | Which rules to use for Proxy Authorization.                                                                                                         |
| `olcLocalSSF`                | :heavy_multiplication_x: | `300`                   | Assumed SSF for LDAPI connections.                                                                                                                  |
| `olcPasswordHash`            | :heavy_multiplication_x: | `{SSHA}`                | One or more hashing algorithms to use for password change extended modifications.                                                                   |
| `olcPasswordCryptSaltFormat` | :heavy_multiplication_x: | `%s`                    | The format of the salt when hashing passwords with `crypt()`.                                                                                       |

###### TLS configuration

| Name                       | Required                 | Default | Description                                                                    |
|----------------------------|:------------------------:|---------|--------------------------------------------------------------------------------|
| `olcTLSCertificateFile`    | :heavy_multiplication_x: |         | Public key file for slapd.                                                     |
| `olcTLSCertificateKeyFile` | :heavy_multiplication_x: |         | Private key for slapd.                                                         |
| `olcTLSRandFile`           | :heavy_multiplication_x: |         | The file to obtain random bits from when urandom is not available.             |
| `olcTLSDHParamFile`        | :heavy_multiplication_x: |         | File containting primes for the Diffie-Hellman ephermal key exchange.          |
| `olcTLSCipherSuite`        | :heavy_multiplication_x: |         | TLS cipher suites to use.                                                      |
| `olcTLSProtocolMin`        | :heavy_multiplication_x: |         | Minimum TLS version to require. Default is the highest possible level.         |
| `olcTLSCACertificateFile`  | :heavy_multiplication_x: |         | Path to a file containing all trusted certificate authorities.                 |
| `olcTLSCACertificatePath`  | :heavy_multiplication_x: |         | Path to a directory containing files with all trusted certificate authorities. |
| `olcTLSCRLCheck`           | :heavy_multiplication_x: | `none`  | If the CRL of the CA should be checked on connections.                         |
| `olcTLSVerifyClient`       | :heavy_multiplication_x: | `never` | When to verify the identity of the client.                                     |

###### Logging configuration

| Name               | Required                 | Default | Description                                        |
|--------------------|:------------------------:|---------|----------------------------------------------------|
| `olcLogLevel`      | :heavy_multiplication_x: | `stats` | Logging level configuration for each subsystem.    |
| `olcLogFile`       | :heavy_multiplication_x: |         | File to log to. slapd will always log to stderr.   |
| `olcPluginLogFile` | :heavy_multiplication_x: |         | File to log slapi plugin output to.                |
| `olcReplogFile`    | :heavy_multiplication_x: |         | File for the replog which may be read by `slurpd`. |

###### Threading configuration

| Name                 | Required                 | Default | Description                                                                                       |
|----------------------|:------------------------:|---------|---------------------------------------------------------------------------------------------------|
| `olcConcurrency`     | :heavy_multiplication_x: |         | Threading hint for the operating system. Not used under Linux.                                    |
| `olcListenerThreads` | :heavy_check_mark:       | `1`     | Amount of threads for listening for connections. 1 is enough for up to 16 cores.                  |
| `olcThreads`         | :heavy_check_mark:       | `16`    | Amount of CPU threads for request processing.                                                     |
| `olcToolThreads`     | :heavy_check_mark:       | `1`     | Amount of CPU threads when running in tool mode. Should not exceed amount of cores in the system. |

###### Timeouts and limits

| Name              | Required                 | Default | Description                                                                                    |
|-------------------|:------------------------:|---------|------------------------------------------------------------------------------------------------|
| `olcIdleTimeout`  | :heavy_multiplication_x: | `0`     | Amount of seconds a client can do nothing before getting disconnected.                         |
| `olcWriteTimeout` | :heavy_multiplication_x: | `0`     | Amount of seconds a client with outstanding writes can do nothing before getting disconnected. |
| `olcTimeLimit`    | :heavy_multiplication_x: | `3600`  | Maximum number of seconds slapd will spend answering a requirest. Allows an `unlimited` value. |
| `olcSizeLimit`    | :heavy_multiplication_x: | `500`   | Maximmum number of entries to return from a search.                                            |

###### Connections

| Name                        | Required                 | Default   | Description                                                           |
|-----------------------------|:------------------------:|-----------|-----------------------------------------------------------------------|
| `olcConnMaxPending`         | :heavy_multiplication_x: | `50`      | Maximum number of pending requests in anonymous sessions.             |
| `olcConnMaxPendingAuth`     | :heavy_multiplication_x: | `1000`    | Maximum number of pending requests in authenticated sessions.         |
| `olcTCPBuffer`              | :heavy_multiplication_x: |           | Size of the TCP buffer. Operating system may automatically tune this. |
| `olcSockbufMaxIncoming`     | :heavy_multiplication_x: | `262143`  | Maximum size of the LDAP PDU for anonymous sessions.                  |
| `olcSockbufMaxIncomingAuth` | :heavy_multiplication_x: | `4194303` | Maximum size of the LDAP PDU for authenticated sessions.              |

###### SASL

| Name              | Required                 | Default | Description                                              |
|-------------------|:------------------------:|---------|----------------------------------------------------------|
| `olcSaslHost`     | :heavy_multiplication_x: |         | Fully qualified domain name used for SASL processing.    |
| `olcSaslRealm`    | :heavy_multiplication_x: |         | The SASL realm for SASL processing.                      |
| `olcSaslSecProps` | :heavy_multiplication_x: |         | Specify Cyrus SASL security properties.                  |
| `olcSaslAuxprops` | :heavy_multiplication_x: |         | Which auxprop plugins to use for authentication lookups. |

###### Indexing

| Name                     | Required                 | Default | Description                                                                                                |
|--------------------------|:------------------------:|---------|------------------------------------------------------------------------------------------------------------|
| `olcIndexSubstrIfMinLen` | :heavy_multiplication_x: | `2`     | The minimum length of subinitial and subfinal indices.                                                     |
| `olcIndexSubstrIfMaxLen` | :heavy_multiplication_x: | `4`     | The maximum length of subinitial and subfinal indices.                                                     |
| `olcIndexSubstrAnyLen`   | :heavy_multiplication_x: | `4`     | Length for subany indices. Attributes longer than this length are processed in segments.                   |
| `olcIndexSubstrAnyStep`  | :heavy_multiplication_x: | `2`     | Steps used in subany lookups. This is the offset for the segments of the filter string that are processed. |
| `olcIndexIntLen`         | :heavy_multiplication_x: | `4`     | Key length for ordered integer idices.                                                                     |

###### Miscellaneous

| Name                  | Required                 | Default          | Description                                                                                                                                                                    |
|-----------------------|:------------------------:|------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `olcAttributeOptions` | :heavy_check_mark:       | `x-hidden lang-` | Tagging attribute options or option tag/range prefixes.                                                                                                                        |
| `olcReferral`         | :heavy_multiplication_x: |                  | A referral URL to pass back when slapd cannot find a local database.                                                                                                           |
| `olcReverseLookup`    | :heavy_multiplication_x: | `FALSE`          | Enable client name unverified reverse lookups.                                                                                                                                 |
| `olcRootDSE`          | :heavy_multiplication_x: |                  | Name of an LDIF file containing user defined attributes for the root DSE.                                                                                                      |
| `olcReadOnly`         | :heavy_check_mark:       | `FALSE`          | Set the entire server into read-only mode. **Warning: Once set to true, this value cannot be changed back without modifying the database files on disk and restarting slapd!** |
| `olcLdapSyntaxes`     | :heavy_multiplication_x: |                  | I have seriously no idea why this attribute is here. There is no documentation on the internet.                                                                                |

#### Database configuration values

These values apply to every database (OLC, monitor, and MDB).
This section gives an overview about all of them.
The default values for each database are specified below.
The MDB database also has some more attributes which are only supported on this database.

###### General

| Name             | Required                 | Description                                                                                                                                  |
|------------------|:------------------------:|----------------------------------------------------------------------------------------------------------------------------------------------|
| `olcSuffix`      | :heavy_check_mark:       | The DN suffix of queries that will be passed to the database backend. It is not required for the OLC and the monitor.                        |
| `olcReadOnly`    | :heavy_check_mark:       | Puts this database into read-only mode. No modifications are possible.                                                                       |
| `olcHidden`      | :heavy_multiplication_x: | Do not answer any queries to this database. slapd will deny the existence of this database.                                                  |
| `olcLastMod`     | :heavy_check_mark:       | Whether slapd will automatically maintain `modfiersName`, `modifyTimestamp`, `creatorsName`, `createTimestamp`, `entryCSN`, and `entryUUID`. |
| `olcSubordinate` | :heavy_multiplication_x: | Whether this database is a subordinate of another database.                                                                                  |

###### Security

| Name               | Required                 | Description                                                                        |
|--------------------|:------------------------:|------------------------------------------------------------------------------------|
| `olcSecurity`      | :heavy_multiplication_x: | Specify a set of security strength factors to require.                             |
| `olcRootDN`        | :heavy_check_mark:       | Name of the RootDN of this database.                                               |
| `olcRootPW`        | :heavy_check_mark:       | Hashed password of the RootDN of this database.                                    |
| `olcRequires`      | :heavy_multiplication_x: | A set of conditions to require.                                                    |
| `olcRestrict`      | :heavy_multiplication_x: | A list of operations that are restructed.                                          |
| `olcAddContentAcl` | :heavy_check_mark:       | Whether operations will perform ACL check on the content of the entry being added. |
| `olcAccess`        | :heavy_check_mark:       | Array of ACL rules for this database.                                              |

###### Timeouts and limits

| Name           | Required                 | Description                                                                                    |
|----------------|:------------------------:|------------------------------------------------------------------------------------------------|
| `olcTimeLimit` | :heavy_multiplication_x: | Maximum number of seconds slapd will spend answering a requirest. Allows an `unlimited` value. |
| `olcSizeLimit` | :heavy_multiplication_x: | Maximmum number of entries to return from a search.                                            |
| `olcLimits`    | :heavy_multiplication_x: | Time and size limits based on the operation's initiator or base DN.|

###### Syncrepl

| Name                 | Required                 | Description                                                                    |
|----------------------|:------------------------:|--------------------------------------------------------------------------------|
| `olcSyncrepl`        | :heavy_multiplication_x: | Syncrepl main configuration.                                                   |
| `olcUpdateDN`        | :heavy_multiplication_x: | DN permitted to update the replica. Should not be the rootDN.                  |
| `olcSyncUseSubentry` | :heavy_multiplication_x: | Store the syncrepl contextCSN in a subentry instead of the context entry.      |
| `olcUpdateRef`       | :heavy_multiplication_x: | The referral to bass back when slapd is asked to modify a replicated database. |
| `olcMirrorMode`      | :heavy_multiplication_x: | Puts this database into mirror mode.                                           |

###### slurpd

| Name                     | Required                 | Description |
|--------------------------|:------------------------:|-------------|
| `olcReplica`             | :heavy_multiplication_x: |             |
| `olcReplicaArgsFile`     | :heavy_multiplication_x: |             |
| `olcReplicaPidFile`      | :heavy_multiplication_x: |             |
| `olcReplicationInterval` | :heavy_multiplication_x: |             |
| `olcReplogFile`          | :heavy_multiplication_x: |             |

###### Miscellaneous

| Name               | Required                 | Description                                              |
|--------------------|:------------------------:|----------------------------------------------------------|
| `olcSchemaDN`      | :heavy_multiplication_x: | DN for the subschema subentry for the entries.           |
| `olcMaxDerefDepth` | :heavy_multiplication_x: | Maximum amount of aliases to follow.                     |
| `olcPlugin`        | :heavy_multiplication_x: | Load slapi plugins.                                      |
| `olcMonitoring`    | :heavy_multiplication_x: | Collect monitoring data for this database.               |
| `olcExtraAttrs`    | :heavy_multiplication_x: | Specify attributes to return even when not searched for. |

###### MDB settings

**These settings only apply to the MDB database!**

| Name               | Required                 | Default      | Description                                                    |
|--------------------|:------------------------:|--------------|----------------------------------------------------------------|
| `olcDbDirectory`   | :heavy_check_mark:       | `{{db_dir}}` | Path to the database directory on disk.                        |
| `olcDbNoSync`      | :heavy_multiplication_x: | `TRUE`       | Do not sync immediately after data was received.               |
| `olcDbCheckpoint`  | :heavy_multiplication_x: | `8192 15`    | How often (KB/minutes) to flush the database to disk.          |
| `olcDbMaxReaders`  | :heavy_multiplication_x: |              | Maximum number of threads that may access the DB concurrently. |
| `olcDbMaxSize`     | :heavy_multiplication_x: |              | Maximum size of DB in bytes.                                   |
| `olcDbMode`        | :heavy_multiplication_x: | `0600`       | File mode of database files.                                   |
| `olcDbSearchStack` | :heavy_multiplication_x: | `16`         | Depth of the stack during search filter evaulations.           |
| `olcDbRtxnSize`    | :heavy_multiplication_x: |              | Number of entries to process in one read transaction.          |
| `olcDbIndex`       | :heavy_multiplication_x: |              | Indices to create on this database.                            |

#### Default values for each database

| Name                     | Frontend default   | OLC default            | Monitor default      | MDB default             |
|:------------------------:|:------------------:|:----------------------:|:--------------------:|:-----------------------:|
| `olcSuffix`              |                    |                        |                      | :exclamation:           |
| `olcReadOnly`            | `FALSE`            | `FALSE`                |                      |                         |
| `olcHidden`              | `FALSE`            |                        |                      |                         |
| `olcLastMod`             | `TRUE`             | `TRUE`                 |                      |                         |
| `olcSubordinate`         |                    |                        |                      |                         |
| `olcSecurity`            |                    |                        |                      |                         |
| `olcRootDN`              |                    | `{{slapd_olc_rootdn}}` | `cn=root,cn=monitor` | `cn=root,{{olcSuffix}}` |
| `olcRootPW`              |                    | `[Hashed password]`    | `[Hashed password]`  | `[Hashed password]`     |
| `olcRequires`            |                    |                        |                      |                         |
| `olcRestrict`            |                    |                        |                      |                         |
| `olcAddContentAcl`       | `TRUE`             | `TRUE`                 |                      |                         |
| `olcAccess`              | `'to * by * read'` | `'to * by * none'`     | `'to * by * none'`   | `'to * by * none'`      |
| `olcTimeLimit`           |                    |                        |                      |                         |
| `olcSizeLimit`           |                    |                        |                      |                         |
| `olcLimits`              |                    |                        |                      |                         |
| `olcSyncrepl`            |                    |                        |                      |                         |
| `olcUpdateDN`            |                    |                        |                      |                         |
| `olcSyncUseSubentry`     | `FALSE`            | `FALSE`                |                      |                         |
| `olcUpdateRef`           |                    |                        |                      |                         |
| `olcMirrorMode`          | `FALSE`            |                        |                      |                         |
| `olcReplica`             |                    |                        |                      |                         |
| `olcReplicaArgsFile`     |                    |                        |                      |                         |
| `olcReplicaPidFile`      |                    |                        |                      |                         |
| `olcReplicationInterval` |                    |                        |                      |                         |
| `olcReplogFile`          |                    |                        |                      |                         |
| `olcSchemaDN`            | `cn=Subschema`     |                        |                      |                         |
| `olcMaxDerefDepth`       | `15`               | `15`                   |                      |                         |
| `olcPlugin`              |                    |                        |                      |                         |
| `olcMonitoring`          | `FALSE`            | `FALSE`                |                      |                         |
| `olcExtraAttrs`          |                    |                        |                      |                         |

## Dependencies

`schema2ldif` must be installed.

## Example Playbook

```yml
- hosts: ldap
  roles:
  - slapd-config
    slapd_modules_path: /usr/lib/openldap
    slapd_olc_rootdn_password: water
    slapd_mdb_rootdn_password: water
    slapd_monitor_rootdn_password: water
    slapd_additional_modules: [ 'memberof' ]
    slapd_mdb_config:
      olcSuffix: "dc=example,dc=com"
    slapd_mdb_overlays:
      memberof:
        olcOverlay: memberof
        objectClass: olcMemberOf
        olcMemberOfDangling: ignore
```

## License

This work is licensed under a [Creative Commons Attribution-ShareAlike 4.0 International License](http://creativecommons.org/licenses/by-sa/4.0/).

## Author Information

- [Janne Heß](https://github.com/dasJ)
