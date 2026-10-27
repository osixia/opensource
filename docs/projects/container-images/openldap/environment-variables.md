# 🔤 Environment Variables

Defaults come from the `.env` files of the image. Variables passed to the container take precedence.

`OPENLDAP_BOOTSTRAP_*` variables are only read on first start, when the directory is created. See [Bootstrap](bootstrap/index.md).

## Core

| Variable                      | Description                                           | Default                                      |
| ----------------------------- | ----------------------------------------------------- | -------------------------------------------- |
| `OPENLDAP_CONF_DIR`           | OpenLDAP configuration directory (`slapd.d`)          | `/etc/openldap/slapd.d`                      |
| `OPENLDAP_DATA_DIR`           | OpenLDAP data directory                               | `/var/lib/openldap/openldap-data`            |
| `OPENLDAP_BACKUP_DIR`         | Backup directory used by `openldap-ctl backup`        | `/var/lib/openldap/openldap-backups`         |
| `OPENLDAP_MODULES_DIR`        | Directory used by OpenLDAP dynamic modules            | `/usr/lib/openldap`                          |
| `OPENLDAP_SCHEMAS_DIR`        | Directory containing LDIF schemas loaded at bootstrap | `/etc/openldap/schema`                       |
| `OPENLDAP_CUSTOM_MODULES_DIR` | Custom modules copied at startup                      | `/container/services/openldap/assets/module` |
| `OPENLDAP_CUSTOM_SCHEMAS_DIR` | Custom schemas copied at startup                      | `/container/services/openldap/assets/schema` |
| `OPENLDAP_NOFILE`             | `ulimit -n` value before starting `slapd`             | `8192`                                       |
| `OPENLDAP_DEBUG_LEVEL`        | Default `slapd` debug level                           | `256`                                        |
| `OPENLDAP_URLS`               | Listener URLs passed to `slapd -h`                    | `ldap://:3890 ldaps://:6360 ldapi:///`      |

## Bootstrap

| Variable                                              | Description                                                                       | Default                                                     |
| ----------------------------------------------------- | --------------------------------------------------------------------------------- | ----------------------------------------------------------- |
| `OPENLDAP_BOOTSTRAP_ORGANIZATION`                     | Organization name used in bootstrap data                                          | `Example Org`                                               |
| `OPENLDAP_BOOTSTRAP_SUFFIX`                           | Base LDAP suffix; must start with `dc=`                                           | `dc=example,dc=org`                                         |
| `OPENLDAP_BOOTSTRAP_MODULES`                          | OpenLDAP modules loaded during bootstrap                                          | `back_mdb.so argon2.so ppolicy.so unique.so refint.so memberof.so syncprov.so` |
| `OPENLDAP_BOOTSTRAP_SCHEMAS`                          | LDIF schemas imported during bootstrap                                            | `core.ldif cosine.ldif inetorgperson.ldif rfc2307bis.ldif`  |
| `OPENLDAP_BOOTSTRAP_GLOBAL_SIZE_LIMIT`                         | Global LDAP size limit                                                            | `500`                                                       |
| `OPENLDAP_BOOTSTRAP_GLOBAL_TIME_LIMIT`                         | Global LDAP time limit                                                            | `120`                                                       |
| `OPENLDAP_BOOTSTRAP_GLOBAL_IDLE_TIMEOUT`                       | Global LDAP idle timeout                                                          | `3600`                                                      |
| `OPENLDAP_BOOTSTRAP_FRONTEND_PASSWORD_HASH`                      | Password hash scheme configured in `olcPasswordHash`                              | `{ARGON2}`                                                  |
| `OPENLDAP_BOOTSTRAP_CONFIG_ROOT_DN`                   | Root DN for `cn=config`                                                           | `cn=admin,cn=config`                                        |
| `OPENLDAP_BOOTSTRAP_CONFIG_ROOT_PASSWORD_HASHED`      | Hashed password for `cn=config`; generated if empty                               | empty                                                        |
| `OPENLDAP_BOOTSTRAP_DATA_DATABASE_MAX_SIZE`           | MDB max size for main database                                                    | `10737418240`                                               |
| `OPENLDAP_BOOTSTRAP_DATA_ROOT_DN`                     | Root DN for main database                                                         | `cn=admin,${OPENLDAP_BOOTSTRAP_SUFFIX}`                     |
| `OPENLDAP_BOOTSTRAP_DATA_ROOT_PASSWORD_HASHED`        | Hashed password for main database root DN; generated if empty                     | empty                                                        |
| `OPENLDAP_BOOTSTRAP_DATA_READONLY`                    | Enable read-only data account creation                                            | `false`                                                     |
| `OPENLDAP_BOOTSTRAP_DATA_READONLY_DN`                 | DN of the read-only data account                                                  | `cn=readonly,${OPENLDAP_BOOTSTRAP_SUFFIX}`                  |
| `OPENLDAP_BOOTSTRAP_DATA_READONLY_PASSWORD_HASHED`    | Hashed password of the read-only data account; generated if empty when enabled    | empty                                                        |
| `OPENLDAP_BOOTSTRAP_MONITOR`                  | Enable monitor backend                                                            | `false`                                                     |
| `OPENLDAP_BOOTSTRAP_MONITOR_READONLY`                 | Enable read-only monitor account creation                                         | `false`                                                     |
| `OPENLDAP_BOOTSTRAP_MONITOR_READONLY_DN`              | DN of the read-only monitor account                                               | `cn=readonly-monitor,${OPENLDAP_BOOTSTRAP_SUFFIX}`          |
| `OPENLDAP_BOOTSTRAP_MONITOR_READONLY_PASSWORD_HASHED` | Hashed password of the read-only monitor account; generated if empty when enabled | empty                                                        |
| `OPENLDAP_BOOTSTRAP_AUDITLOG` | Enable file-based auditing on the data database | `false` |
| `OPENLDAP_BOOTSTRAP_AUDITLOG_FILE` | Audit LDIF file path | `/var/log/openldap/audit.ldif` |
| `OPENLDAP_BOOTSTRAP_LDIF_CONFIG_DIR`                  | Source directory for config LDIF bootstrap files                                  | `/container/services/openldap-bootstrap/assets/ldif/config` |
| `OPENLDAP_BOOTSTRAP_LDIF_DATA_DIR`                    | Source directory for data LDIF bootstrap files                                    | `/container/services/openldap-bootstrap/assets/ldif/data`   |
| `OPENLDAP_BOOTSTRAP_SCRIPTS_DIR`                      | Source directory for bootstrap scripts                                            | `/container/services/openldap-bootstrap/assets/scripts`     |

## TLS

See [TLS](tls.md).

| Variable                               | Description                                    | Default                                              |
| -------------------------------------- | ---------------------------------------------- | ---------------------------------------------------- |
| `OPENLDAP_BOOTSTRAP_TLS`               | Enable TLS configuration during bootstrap      | `false`                                              |
| `OPENLDAP_BOOTSTRAP_TLS_CERT`          | Server certificate path                        | `/container/services/openldap/assets/certs/tls.crt`  |
| `OPENLDAP_BOOTSTRAP_TLS_CERT_KEY`      | Server private key path                        | `/container/services/openldap/assets/certs/tls.key`  |
| `OPENLDAP_BOOTSTRAP_TLS_CA_CERT`       | CA certificate path                            | `/container/services/openldap/assets/certs/ca.crt`   |
| `OPENLDAP_BOOTSTRAP_TLS_VERIFY_CLIENT` | Value for `olcTLSVerifyClient`                 | `allow`                                              |
| `OPENLDAP_BOOTSTRAP_TLS_PROTOCOL_MIN`  | Value for `olcTLSProtocolMin`                  | `3.4`                                                |
| `OPENLDAP_BOOTSTRAP_TLS_REQUIRED`      | Enforce TLS-only access rules during bootstrap | `false`                                              |

## RefInt

See [Overlays](bootstrap/overlays.md#referential-integrity).

| Variable                               | Description                                   | Default                             |
| -------------------------------------- | --------------------------------------------- | ----------------------------------- |
| `OPENLDAP_BOOTSTRAP_REFINT`            | Enable the `refint` overlay during bootstrap  | `false`                             |
| `OPENLDAP_BOOTSTRAP_REFINT_ATTRIBUTES` | Attributes maintained by the `refint` overlay | `member uniqueMember manager owner` |

## PPolicy

See [Overlays](bootstrap/overlays.md#password-policy).

| Variable                                                    | Description                                             | Default                                            |
| ----------------------------------------------------------- | ------------------------------------------------------- | -------------------------------------------------- |
| `OPENLDAP_BOOTSTRAP_PPOLICY`                                | Enable the password policy overlay during bootstrap     | `false`                                            |
| `OPENLDAP_BOOTSTRAP_PPOLICY_HASH_CLEAR_TEXT`                | Value for `olcPPolicyHashCleartext`                     | `true`                                             |
| `OPENLDAP_BOOTSTRAP_PPOLICY_USE_LOCKOUT`                    | Value for `olcPPolicyUseLockout`                        | `true`                                             |
| `OPENLDAP_BOOTSTRAP_PPOLICY_CHECK_MODULE`                   | Optional value for `olcPPolicyCheckModule`              | empty                                               |
| `OPENLDAP_BOOTSTRAP_PPOLICY_GROUP_DN`                       | DN of the policy container created in the main database | `ou=policies,${OPENLDAP_BOOTSTRAP_SUFFIX}`         |
| `OPENLDAP_BOOTSTRAP_PPOLICY_DEFAULT_POLICY_DN`              | DN of the default password policy                       | `cn=default,${OPENLDAP_BOOTSTRAP_PPOLICY_GROUP_DN}` |
| `OPENLDAP_BOOTSTRAP_PPOLICY_DEFAULT_MIN_LENGTH`             | Value for `pwdMinLength`                                | `12`                                               |
| `OPENLDAP_BOOTSTRAP_PPOLICY_DEFAULT_CHECK_QUALITY`          | Value for `pwdCheckQuality`                             | `0`                                                |
| `OPENLDAP_BOOTSTRAP_PPOLICY_DEFAULT_IN_HISTORY`             | Value for `pwdInHistory`                                | `12`                                               |
| `OPENLDAP_BOOTSTRAP_PPOLICY_DEFAULT_ALLOW_USER_CHANGE`      | Value for `pwdAllowUserChange`                          | `true`                                             |
| `OPENLDAP_BOOTSTRAP_PPOLICY_DEFAULT_SAFE_MODIFY`            | Value for `pwdSafeModify`                               | `true`                                             |
| `OPENLDAP_BOOTSTRAP_PPOLICY_DEFAULT_MUST_CHANGE`            | Value for `pwdMustChange`                               | `false`                                            |
| `OPENLDAP_BOOTSTRAP_PPOLICY_DEFAULT_MIN_AGE`                | Value for `pwdMinAge`                                   | `0`                                                |
| `OPENLDAP_BOOTSTRAP_PPOLICY_DEFAULT_MAX_FAILURE`            | Value for `pwdMaxFailure`                               | `5`                                                |
| `OPENLDAP_BOOTSTRAP_PPOLICY_DEFAULT_FAILURE_COUNT_INTERVAL` | Value for `pwdFailureCountInterval`                     | `300`                                              |
| `OPENLDAP_BOOTSTRAP_PPOLICY_DEFAULT_LOCKOUT`                | Value for `pwdLockout`                                  | `true`                                             |
| `OPENLDAP_BOOTSTRAP_PPOLICY_DEFAULT_LOCKOUT_DURATION`       | Value for `pwdLockoutDuration`                          | `900`                                              |
| `OPENLDAP_BOOTSTRAP_PPOLICY_DEFAULT_MAX_AGE`                | Value for `pwdMaxAge`                                   | `7776000`                                          |
| `OPENLDAP_BOOTSTRAP_PPOLICY_DEFAULT_EXPIRE_WARNING`         | Value for `pwdExpireWarning`                            | `1209600`                                          |
| `OPENLDAP_BOOTSTRAP_PPOLICY_DEFAULT_GRACE_AUTH_NLIMIT`      | Value for `pwdGraceAuthNLimit`                          | `3`                                                |

## Unique

See [Overlays](bootstrap/overlays.md#unique-attributes).

| Variable                         | Description                                   | Default                              |
| -------------------------------- | --------------------------------------------- | ------------------------------------ |
| `OPENLDAP_BOOTSTRAP_UNIQUE`      | Enable the `unique` overlay during bootstrap  | `false`                              |
| `OPENLDAP_BOOTSTRAP_UNIQUE_URIS` | URIs enforced by the `unique` overlay         | `ldap:///?uid?sub ldap:///?mail?sub` |

## MemberOf

See [Overlays](bootstrap/overlays.md#memberof).

| Variable                                  | Description                                           | Default        |
| ----------------------------------------- | ----------------------------------------------------- | -------------- |
| `OPENLDAP_BOOTSTRAP_MEMBEROF`             | Enable the `memberof` overlay during bootstrap        | `false`        |
| `OPENLDAP_BOOTSTRAP_MEMBEROF_GROUP_OC`    | ObjectClass treated as a group by `memberof`          | `groupOfNames` |
| `OPENLDAP_BOOTSTRAP_MEMBEROF_MEMBER_AD`   | Group membership attribute watched by `memberof`      | `member`       |
| `OPENLDAP_BOOTSTRAP_MEMBEROF_MEMBEROF_AD` | Reverse membership attribute maintained by `memberof` | `memberOf`     |
| `OPENLDAP_BOOTSTRAP_MEMBEROF_ADD_CHECK`   | Check new entries against existing groups on Add      | `true`         |
| `OPENLDAP_BOOTSTRAP_MEMBEROF_DANGLING`    | Handling of group members that do not exist yet       | `ignore`       |

## Replication

See [Replication](replication.md).

| Variable                                                   | Description                                                                | Default                                                               |
| ---------------------------------------------------------- | -------------------------------------------------------------------------- | --------------------------------------------------------------------- |
| `OPENLDAP_BOOTSTRAP_REPLICATION`                           | Enable replication bootstrap                                               | `false`                                                               |
| `OPENLDAP_BOOTSTRAP_REPLICATION_HOSTS`                     | Replication endpoints list; two to nine, same order on every server        | `ldap://ldap1.example.org:3890 ldap://ldap2.example.org:3890`         |
| `OPENLDAP_BOOTSTRAP_REPLICATION_SYNCPROV_CHECKPOINT`       | `syncprov` checkpoint configuration                                        | `100 10`                                                              |
| `OPENLDAP_BOOTSTRAP_REPLICATION_DATA_READONLY_DN`          | Replication account DN for the main database                               | `cn=replicator,${OPENLDAP_BOOTSTRAP_SUFFIX}`                          |
| `OPENLDAP_BOOTSTRAP_REPLICATION_DATA_READONLY_PASSWORD`    | Plain password for the data replication account; generated if empty        | empty                                                                  |
| `OPENLDAP_BOOTSTRAP_REPLICATION_DATA_SYNC_REPL_TEMPLATE`   | Template used to generate data `olcSyncRepl`                               | `rid=\${OPENLDAP_BOOTSTRAP_REPLICATION_DATA_SYNC_REPL_RID} ...`       |
| `OPENLDAP_BOOTSTRAP_REPLICATION_DATA_LIMITS`               | Limits ACL for the data replication account                                | `dn.exact="${OPENLDAP_BOOTSTRAP_REPLICATION_DATA_READONLY_DN}" ...`   |
| `OPENLDAP_BOOTSTRAP_REPLICATION_TLS`                       | Enable TLS settings in generated replication config                        | `${OPENLDAP_BOOTSTRAP_TLS_REQUIRED}`                                  |
| `OPENLDAP_BOOTSTRAP_REPLICATION_TLS_SYNC_REPL`             | Extra `syncrepl` TLS options                                               | `starttls=critical tls_reqcert=demand`                                |
| `OPENLDAP_BOOTSTRAP_REPLICATION_MEMBEROF`                  | Enable memberof settings in generated replication config               | `${OPENLDAP_BOOTSTRAP_MEMBEROF}`                                      |
| `OPENLDAP_BOOTSTRAP_REPLICATION_MEMBEROF_SYNC_REPL`        | Extra `syncrepl` option used when `memberof` replication handling is enabled | `exattrs=${OPENLDAP_BOOTSTRAP_MEMBEROF_MEMBEROF_AD}`                  |

## Helpers and Maintenance

See [Backup / Restore](backup-restore.md), [Upgrade](upgrade.md) and [LDAP Toolkit](ldap-toolkit.md).

| Variable                                 | Description                                                 | Default                                                                                                                                                                          |
| ---------------------------------------- | ----------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `OPENLDAP_CTL_BACKUP_CONFIG_FILE_SUFFIX` | Suffix used for config backups                              | `-config.gz`                                                                                                                                                                     |
| `OPENLDAP_CTL_BACKUP_DATA_FILE_SUFFIX`   | Suffix used for data backups                                | `-data.gz`                                                                                                                                                                       |
| `OPENLDAP_CTL_PASSWORD_GENERATE_CMD`     | Command used by `openldap-ctl password generate`            | `slappasswd -n -g; slappasswd -g`                                                                                                                                                |
| `OPENLDAP_CTL_PASSWORD_HASH_CMD`         | Command used by `openldap-ctl password hash`                | `slappasswd -h {ARGON2} -o module-path=${OPENLDAP_MODULES_DIR} -o module-load=argon2`                                                                                            |
| `OPENLDAP_CTL_SCHEMA2LDIF_DEPENDENCIES`  | Default schema dependencies for `schema2ldif`               | `core.schema cosine.schema inetorgperson.schema`                                                                                                                                 |
| `OPENLDAP_CRON_JOB`                      | Cron job used by the backup service                         | `15 2 * * * PATH=${PATH} container run -- openldap-ctl backup "$(date -I)-${OPENLDAP_VERSION}-openldap-cron" --clean 15` |
| `OPENLDAP_UPGRADE_FORCE`                 | Allow upgrade bypasses, including forced downgrade handling | `false`                                                                                                                                                                          |
| `OPENLDAP_UPGRADE_MIGRATION_LEVEL`       | Allowed migration level                                     | `minor`                                                                                                                                                                          |
| `OPENLDAP_UPGRADE_BACKUP_FILES_PREFIX`   | Prefix for upgrade backup files                             | `openldap-upgrade`                                                                                                                                                               |
| `OPENLDAP_UPGRADE_CONF_VERSION_FILE`     | File storing the current config version                     | `${OPENLDAP_CONF_DIR}/.version`                                                                                                                                                  |
