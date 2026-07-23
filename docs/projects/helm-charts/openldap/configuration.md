# Configuration

- [Protocols](#protocols)
- [TLS](#tls)
- [Persistence](#persistence)
- [Replication](#replication)
- [Overlays](#overlays)
- [Custom volumes](#custom-volumes)
- [Extra environment variables and secrets](#extra-environment-variables-and-secrets)
- [Backup CronJob](#backup-cronjob)
- [Upgrading](#upgrading)

## Protocols

By default both LDAP (port 3890 internal / 389 service) and LDAPS (port 6360 / 636)
are enabled, along with `ldapi:///`.

``` yaml
service:
  ldapPort: 389      # service port → container port 3890
  ldapsPort: 636     # service port → container port 6360

openldap:
  protocols:
    ldap:
      enabled: true
      port: 3890
    ldaps:
      enabled: true
      port: 6360
    ldapi:
      enabled: true
```

Disable LDAPS (plain LDAP only):

``` yaml
openldap:
  protocols:
    ldaps:
      enabled: false
```

## TLS

1. Create a Kubernetes secret with your certificates:

``` bash
kubectl create secret generic my-ldap-tls \
  --from-file=tls.crt=server.crt \
  --from-file=tls.key=server.key \
  --from-file=ca.crt=ca.crt
```

2. Enable TLS in values:

``` yaml
tlsSecret:
  existingSecret: my-ldap-tls
  keys:
    cert: tls.crt   # key name inside the secret
    key: tls.key
    ca: ca.crt

openldap:
  bootstrap:
    tls:
      enabled: true
      required: false        # true = reject non-TLS connections
      verifyClient: allow    # allow | demand | never
      protocolMin: "3.4"     # TLS 1.3 = "3.4"
```

The secret is mounted at `/container/services/openldap/assets/certs/` in every
container (init, pre-upgrade, main).

## Persistence

Persistence is enabled by default. Three PVCs are created per pod:

| PVC prefix | Mount path | Default size |
|---|---|---|
| `conf-<release>-0` | `/etc/openldap/slapd.d` | 1 Gi |
| `data-<release>-0` | `/var/lib/openldap/openldap-data` | 8 Gi |
| `backups-<release>-0` | `/var/lib/openldap/openldap-backups` | 8 Gi |

``` yaml
persistence:
  enabled: true
  storageClass: ""       # leave empty for default StorageClass
  accessModes:
    - ReadWriteOnce
  conf:
    size: 1Gi
  data:
    size: 8Gi
  backups:
    size: 8Gi
```

Disable persistence (all data lost on restart — for testing only):

``` yaml
persistence:
  enabled: false
```

## Replication

Replication is **automatic** when `replicaCount > 1` (mode `auto`).

Minimal multi-replica setup:

``` yaml
replicaCount: 2

openldap:
  bootstrap:
    replication:
      mode: auto          # auto | enabled | disabled
      protocol: ldap      # ldap | ldaps
      dataReadonlyDnPrefix: cn=replicator
      dataReadonlyPassword: "changeme"   # plain text, used internally only
```

The chart builds the replication host list automatically from pod FQDNs:

```
ldap://<release>-0.<release>-headless.<namespace>.svc.cluster.local:3890
ldap://<release>-1.<release>-headless.<namespace>.svc.cluster.local:3890
```

With persistence enabled, the bootstrap replication topology is written into the
OpenLDAP config database on disk. Changing `replicaCount` later can leave old
pods with the previous peer list while new pods are bootstrapped with the new
one. For that reason, upgrades fail by default when `replicaCount` changes with
auto-generated replication hosts.

If you have manually migrated the OpenLDAP replication config on all existing
pods, you can opt out of this guard:

``` yaml
openldap:
  bootstrap:
    replication:
      allowReplicaCountChange: true
```

Override the host list if your cluster domain differs or you need custom endpoints:

``` yaml
openldap:
  bootstrap:
    replication:
      hosts:
        - ldap://ldap1.example.com:3890
        - ldap://ldap2.example.com:3890

clusterDomain: cluster.local   # used for auto-generated FQDNs
```

**Replication with TLS:**

``` yaml
openldap:
  bootstrap:
    replication:
      protocol: ldaps
      tls:
        enabled: true
        syncRepl: "starttls=critical tls_reqcert=demand"
```

**Pod scheduling:** with `replicaCount > 1`, `podAntiAffinity` spreads pods across
nodes by default:

``` yaml
podAntiAffinity:
  enabled: true
  type: preferred   # preferred | required
  weight: 100
  topologyKey: kubernetes.io/hostname
```

## Overlays

All overlays are disabled by default. Enable them under `openldap.bootstrap`.

### Password Policy (ppolicy)

``` yaml
openldap:
  bootstrap:
    ppolicy:
      enabled: true
      hashClearText: true
      useLockout: true
      groupDnPrefix: "ou=Policies"
      default:
        policyDnPrefix: "cn=default"
        minLength: 12
        maxAge: 7776000        # seconds (~90 days)
        maxFailure: 5
        lockoutDuration: 900   # seconds (15 min)
```

### Referential Integrity (refint)

``` yaml
openldap:
  bootstrap:
    refint:
      enabled: true
      attributes: "member uniqueMember manager owner"
```

### MemberOf

``` yaml
openldap:
  bootstrap:
    memberof:
      enabled: true
      groupObjectClass: groupOfNames
      memberAttribute: member
      memberOfAttribute: memberOf
```

### Unique attribute enforcement

``` yaml
openldap:
  bootstrap:
    unique:
      enabled: true
      uris: "ldap:///?uid?sub ldap:///?mail?sub"
```

### Monitor backend

``` yaml
openldap:
  bootstrap:
    monitor:
      enabled: true
      readonly:
        enabled: true
        dnPrefix: "cn=readonly-monitor"
        passwordHashed: "{ARGON2}..."
```

## Custom volumes

All custom volume values default to `{}` (disabled). Define any valid Kubernetes
volume spec to enable mounting.

### Custom modules

Mounted at `/container/services/openldap/assets/module` in **all** containers
(init, pre-upgrade, main).

``` yaml
customModulesVolume:
  configMap:
    name: my-ldap-modules
```

### Custom schemas

Mounted at `/container/services/openldap/assets/schema` in **all** containers.

``` yaml
customSchemasVolume:
  configMap:
    name: my-ldap-schemas
```

### Bootstrap LDIF

Bootstrap LDIF volumes are mounted only in the **bootstrap init container**:

| Value | Mount path |
|---|---|
| `bootstrapCustomConfigLdifVolume` | `/container/services/openldap-bootstrap/assets/ldif/config/custom` |
| `bootstrapCustomDataLdifVolume` | `/container/services/openldap-bootstrap/assets/ldif/data/custom` |

Create ConfigMaps from local directories containing LDIF files:

``` bash
kubectl create configmap my-ldap-config-ldif \
  --from-file=./ldif/config \
  -n <namespace>

kubectl create configmap my-ldap-data-ldif \
  --from-file=./ldif/data \
  -n <namespace>
```

Then mount them during bootstrap:

``` yaml
bootstrapCustomConfigLdifVolume:
  configMap:
    name: my-ldap-config-ldif

bootstrapCustomDataLdifVolume:
  configMap:
    name: my-ldap-data-ldif
```

Each file directly inside the source directory becomes one ConfigMap key and is
mounted as a file. `kubectl create configmap --from-file=./dir` does not load
files recursively from nested directories.

Bootstrap LDIF files are applied only when the OpenLDAP config/data volumes are
empty. With persistence enabled, updating these ConfigMaps after the first
successful bootstrap does not automatically reapply the LDIF files.

### Extra volumes

`extraVolumes` / `extraVolumeMounts` are added to **all** containers.

``` yaml
extraVolumes:
  - name: my-volume
    configMap:
      name: my-config

extraVolumeMounts:
  - name: my-volume
    mountPath: /my/path
    readOnly: true
```

## Extra environment variables and secrets

### Bootstrap secret

The chart always generates a `<release>-bootstrap-secret` Secret containing all
`OPENLDAP_BOOTSTRAP_*` variables. It is loaded only by the bootstrap init container —
the main container never sees it.

To override specific bootstrap variables, use `bootstrapEnvSecret.env` to set them
directly in values (the chart generates a second secret from them):

``` yaml
bootstrapEnvSecret:
  env:
    OPENLDAP_BOOTSTRAP_DATA_ROOT_PASSWORD_HASHED: "{ARGON2}..."
```

Or reference an existing secret (e.g. from External Secrets Operator, Vault, or
Sealed Secrets) with `bootstrapEnvSecret.existingSecret`. `env` and `existingSecret`
are mutually exclusive — `existingSecret` takes priority. Either way, the override
secret is loaded **after** the auto-generated one so its values win:

``` yaml
bootstrapEnvSecret:
  existingSecret: my-ldap-bootstrap-overrides
```

### Plain environment variables

``` yaml
extraEnv:
  - name: MY_VAR
    value: "my-value"
```

### Secret environment variables

`extraEnvSecret` is loaded into **all** containers (init, pre-upgrade, main). Use it
for runtime secrets unrelated to bootstrap (e.g. custom application credentials).

Create a secret:

``` bash
kubectl create secret generic my-ldap-env \
  --from-literal=MY_VAR=my-value
```

Reference it:

``` yaml
extraEnvSecret:
  existingSecret: my-ldap-env
```

If `existingSecret` is empty no secret is mounted. The secret is loaded with
`optional: true` so a missing secret does not block startup.

## Backup CronJob

The backup CronJob is disabled by default. It mounts the same PVCs as the
StatefulSet (requires `persistence.enabled: true`) and runs `openldap-ctl backup`
on a schedule.

``` yaml
backupCronJob:
  enabled: true
  schedule: "15 2 * * *"    # 02:15 UTC daily
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 3
  command:
    - /bin/sh
    - -ec
    - container run -- openldap-ctl backup "$(date -I)-${OPENLDAP_VERSION}-openldap-cron" --clean 15
  resources: {}
```

`--clean 15` removes backup files older than 15 days. Adjust or remove as needed.

Restore a backup manually:

``` bash
kubectl exec -it <pod> -- container run -- openldap-ctl restore <backup-name> --force
```

## Upgrading

### Patch / minor upgrades (v2.x → v2.y)

The chart runs an **openldap-pre-upgrade** init container before the main container
starts. It uses the *previous* image version to perform any required migration.

You **must** set `previousImage.tag` during upgrades:

``` yaml
image:
  tag: 2.6.11

previousImage:
  tag: 2.6.10   # the version currently running
```

Control how aggressively migrations are applied:

``` yaml
openldap:
  upgrade:
    migrationLevel: minor   # patch | minor | major
    force: false
```

Remove `previousImage.tag` after the upgrade completes.

### v1 → v2 (breaking change)

!!! warning
    v2 is a **breaking release**. There is no automatic migration from v1. A manual
    migration is required. See the
    [container-openldap documentation](https://github.com/osixia/container-openldap)
    for the migration procedure.
