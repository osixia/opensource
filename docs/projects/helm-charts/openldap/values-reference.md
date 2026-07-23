# Values reference

## Top-level

| Key | Default | Description |
|---|---|---|
| `replicaCount` | `1` | Number of replicas |
| `clusterDomain` | `cluster.local` | Kubernetes cluster domain, used to build pod FQDNs |
| `image.repository` | `osixia/openldap` | Container image |
| `image.tag` | `2.6.10-alpha` | Image tag |
| `image.pullPolicy` | `IfNotPresent` | Pull policy |
| `previousImage.repository` | `""` | Previous image repository (defaults to `image.repository`) |
| `previousImage.tag` | `""` | Previous image tag — **required during upgrades** |
| `previousImage.pullPolicy` | `IfNotPresent` | Previous image pull policy |
| `imagePullSecrets` | `[]` | Image pull secrets |
| `nameOverride` | `""` | Override chart name |
| `fullnameOverride` | `""` | Override full release name |

## Service account

| Key | Default | Description |
|---|---|---|
| `serviceAccount.create` | `true` | Create a ServiceAccount |
| `serviceAccount.annotations` | `{}` | Annotations for the ServiceAccount |
| `serviceAccount.name` | `""` | Name override (defaults to release fullname) |

## Pod

| Key | Default | Description |
|---|---|---|
| `podAnnotations` | `{}` | Pod annotations |
| `podLabels` | `{}` | Extra pod labels |
| `podSecurityContext.fsGroup` | `911` | Pod fsGroup |
| `securityContext.runAsUser` | `911` | Container user |
| `securityContext.runAsGroup` | `911` | Container group |
| `securityContext.runAsNonRoot` | `true` | Enforce non-root |
| `resources` | `{}` | CPU / memory requests and limits |
| `nodeSelector` | `{}` | Node selector |
| `tolerations` | `[]` | Tolerations |
| `affinity` | `{}` | Affinity (overrides podAntiAffinity) |
| `terminationGracePeriodSeconds` | `60` | Grace period |
| `updateStrategy.type` | `RollingUpdate` | StatefulSet update strategy |

## Services

| Key | Default | Description |
|---|---|---|
| `service.type` | `ClusterIP` | Service type |
| `service.annotations` | `{}` | Service annotations |
| `service.ldapPort` | `389` | Service LDAP port |
| `service.ldapsPort` | `636` | Service LDAPS port |
| `headlessService.annotations` | `{}` | Headless service annotations |

## Pod anti-affinity

| Key | Default | Description |
|---|---|---|
| `podAntiAffinity.enabled` | `true` | Enable automatic anti-affinity (multi-replica) |
| `podAntiAffinity.type` | `preferred` | `preferred` or `required` |
| `podAntiAffinity.weight` | `100` | Weight for preferred anti-affinity |
| `podAntiAffinity.topologyKey` | `kubernetes.io/hostname` | Topology key |

## TLS secret

| Key | Default | Description |
|---|---|---|
| `tlsSecret.existingSecret` | `""` | Secret name containing TLS certificates |
| `tlsSecret.keys.cert` | `tls.crt` | Key name for the certificate |
| `tlsSecret.keys.key` | `tls.key` | Key name for the private key |
| `tlsSecret.keys.ca` | `ca.crt` | Key name for the CA certificate |

## Bootstrap secret

| Key | Default | Description |
|---|---|---|
| `bootstrapEnvSecret.existingSecret` | `""` | Existing secret loaded after the auto-generated bootstrap secret (overrides its values) |
| `bootstrapEnvSecret.env` | `{}` | Env vars rendered into a generated override secret (ignored if `existingSecret` is set) |

## Extra env / secrets

| Key | Default | Description |
|---|---|---|
| `extraEnv` | `[]` | Extra environment variables (all containers) |
| `extraEnvSecret.existingSecret` | `""` | Secret name loaded as `envFrom` (all containers) |
| `extraEnvSecret.env` | `{}` | Env vars written into a generated secret (not used if `existingSecret` is set) |

## Probes

| Key | Default | Description |
|---|---|---|
| `livenessProbe.enabled` | `true` | Enable liveness probe |
| `livenessProbe.*` | tcpSocket on `ldap` port | Standard Kubernetes probe fields |
| `readinessProbe.enabled` | `true` | Enable readiness probe |
| `readinessProbe.*` | tcpSocket on `ldap` port | Standard Kubernetes probe fields |

## Persistence

| Key | Default | Description |
|---|---|---|
| `persistence.enabled` | `true` | Enable PVC-backed storage |
| `persistence.storageClass` | `""` | StorageClass (empty = default) |
| `persistence.accessModes` | `[ReadWriteOnce]` | Access modes |
| `persistence.conf.size` | `1Gi` | Config PVC size |
| `persistence.data.size` | `8Gi` | Data PVC size |
| `persistence.backups.size` | `8Gi` | Backups PVC size |

## OpenLDAP

| Key | Default | Description |
|---|---|---|
| `openldap.nofile` | `8192` | `ulimit -n` for slapd |
| `openldap.debugLevel` | `256` | slapd debug level |
| `openldap.protocols.ldap.enabled` | `true` | Enable LDAP listener |
| `openldap.protocols.ldap.port` | `3890` | Internal LDAP port |
| `openldap.protocols.ldaps.enabled` | `true` | Enable LDAPS listener |
| `openldap.protocols.ldaps.port` | `6360` | Internal LDAPS port |
| `openldap.protocols.ldapi.enabled` | `true` | Enable LDAPI socket |
| `openldap.preUpgrade.enabled` | `true` | Run pre-upgrade init container |
| `openldap.upgrade.force` | `false` | Force upgrade (allow downgrades) |
| `openldap.upgrade.migrationLevel` | `minor` | `patch` / `minor` / `major` |

## Bootstrap

| Key | Default | Description |
|---|---|---|
| `openldap.bootstrap.organization` | `Example Org` | Organization name |
| `openldap.bootstrap.suffix` | `dc=example,dc=org` | Base DN suffix |
| `openldap.bootstrap.modules` | see values.yaml | OpenLDAP modules loaded |
| `openldap.bootstrap.schemas` | see values.yaml | LDIF schemas imported |
| `openldap.bootstrap.global.sizeLimit` | `500` | Global size limit |
| `openldap.bootstrap.global.timeLimit` | `120` | Global time limit |
| `openldap.bootstrap.global.idleTimeout` | `3600` | Global idle timeout |
| `openldap.bootstrap.frontend.passwordHash` | `{ARGON2}` | Default password hash scheme |
| `openldap.bootstrap.config.rootDnPrefix` | `cn=admin` | Config root DN prefix (assembled: `<prefix>,cn=config`) |
| `openldap.bootstrap.config.rootPasswordHashed` | `""` | Config root password hash |
| `openldap.bootstrap.database.maxSize` | `10737418240` | MDB max size (bytes) |
| `openldap.bootstrap.database.rootDnPrefix` | `cn=admin` | Data root DN prefix |
| `openldap.bootstrap.database.rootPasswordHashed` | `""` | Data root password hash |
| `openldap.bootstrap.database.readonly.enabled` | `false` | Create read-only data account |
| `openldap.bootstrap.database.readonly.dnPrefix` | `cn=readonly` | Read-only account prefix |
| `openldap.bootstrap.database.readonly.passwordHashed` | `""` | Read-only account password hash |
| `openldap.bootstrap.monitor.enabled` | `false` | Enable monitor backend |
| `openldap.bootstrap.monitor.readonly.*` | — | Monitor read-only account |
| `openldap.bootstrap.tls.enabled` | `false` | Configure TLS during bootstrap |
| `openldap.bootstrap.tls.required` | `false` | Enforce TLS-only access |
| `openldap.bootstrap.tls.verifyClient` | `allow` | `olcTLSVerifyClient` value |
| `openldap.bootstrap.tls.protocolMin` | `3.4` | `olcTLSProtocolMin` value |
| `openldap.bootstrap.replication.mode` | `auto` | `auto` / `enabled` / `disabled` |
| `openldap.bootstrap.replication.allowReplicaCountChange` | `false` | Allow unsafe `replicaCount` changes after bootstrap when hosts are auto-generated |
| `openldap.bootstrap.replication.protocol` | `ldap` | Replication protocol (`ldap` / `ldaps`) |
| `openldap.bootstrap.replication.hosts` | `[]` | Override replication host list |
| `openldap.bootstrap.replication.syncprovCheckpoint` | `100 10` | syncprov checkpoint |
| `openldap.bootstrap.replication.dataReadonlyDnPrefix` | `cn=replicator` | Replication account prefix |
| `openldap.bootstrap.replication.dataReadonlyPassword` | `""` | Replication account password (plain) |
| `openldap.bootstrap.replication.tls.enabled` | `false` | Enable TLS in replication config |
| `openldap.bootstrap.replication.tls.syncRepl` | `starttls=critical tls_reqcert=demand` | Extra syncrepl TLS options |

## Custom volumes

| Key | Default | Description |
|---|---|---|
| `customModulesVolume` | `{}` | Volume spec for custom modules (all containers) |
| `customSchemasVolume` | `{}` | Volume spec for custom schemas (all containers) |
| `bootstrapCustomConfigLdifVolume` | `{}` | Volume spec for bootstrap config LDIF (init only) |
| `bootstrapCustomDataLdifVolume` | `{}` | Volume spec for bootstrap data LDIF (init only) |
| `extraVolumes` | `[]` | Extra volumes (all containers) |
| `extraVolumeMounts` | `[]` | Extra volume mounts (all containers) |

## Backup CronJob

| Key | Default | Description |
|---|---|---|
| `backupCronJob.enabled` | `false` | Enable backup CronJob |
| `backupCronJob.schedule` | `15 2 * * *` | Cron schedule |
| `backupCronJob.successfulJobsHistoryLimit` | `3` | Successful jobs to keep |
| `backupCronJob.failedJobsHistoryLimit` | `3` | Failed jobs to keep |
| `backupCronJob.command` | see values.yaml | Command run by the backup job |
| `backupCronJob.resources` | `{}` | CPU / memory for backup job |
