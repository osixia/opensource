# 🪪 OpenLDAP

This chart runs [osixia/openldap](../../container-images/openldap/index.md) as a StatefulSet. The quickstart below installs a test directory with one replica and persistent storage. For network access, prepare [TLS](#tls-for-a-new-installation) before installing.

You need Helm 3, `kubectl`, and a cluster with dynamic volume provisioning, or your own volume configuration.

## 1. Prepare passwords

Generate two different passwords and hashes, one for the data root and one for the configuration root:

```bash
docker run --rm osixia/openldap:2.6.15-rc run -- openldap-ctl password generate
```

Run the command twice. Keep the passwords somewhere safe and put their `{ARGON2}...` hashes in `my-values.yaml`:

```yaml
openldap:
  bootstrap:
    organization: My Org
    suffix: dc=example,dc=org
    database:
      rootPasswordHashed: "{ARGON2}..."
    config:
      rootPasswordHashed: "{ARGON2}..."
```

These values initialize an empty directory. Changing them later does not rotate passwords in an existing database.

## 2. Install and connect

After [adding the chart repository](index.md#add-the-repository):

```bash
helm install my-ldap osixia/openldap \
  --namespace directory --create-namespace -f my-values.yaml --wait
kubectl -n directory get pods,pvc
```

The LDAP address inside the cluster is `my-ldap-openldap.directory.svc.cluster.local:389` with the default cluster domain. For a local check:

```bash
kubectl -n directory port-forward svc/my-ldap-openldap 1389:389
ldapsearch -x -H ldap://127.0.0.1:1389 \
  -D 'cn=admin,dc=example,dc=org' -W -b 'dc=example,dc=org'
```

The initial setup has TLS disabled. Enable it before sending credentials over the network.

## TLS for a new installation

Before your first installation, create the `directory` namespace if it does not exist (`kubectl create namespace directory`). Then create a Secret containing your server certificate, private key and CA certificate. The certificate must cover the DNS names clients use:

```bash
kubectl -n directory create secret generic ldap-tls \
  --from-file=tls.crt=certs/tls.crt \
  --from-file=tls.key=certs/tls.key \
  --from-file=ca.crt=certs/ca.crt
```

Add these values **before the first installation**:

```yaml
tlsSecret:
  existingSecret: ldap-tls

openldap:
  tlsReload:
    enabled: true
  bootstrap:
    tls:
      enabled: true
      required: true
```

Merge them into your existing values under the same `openldap` key. Clients can use StartTLS on service port `389` or LDAPS on `636`, and must trust your CA. The TLS reload sidecar restarts the OpenLDAP process when certificate contents change; the pod stays running, but LDAP connections can be interrupted briefly.

On an existing directory, bootstrap values do not reconfigure TLS. Update the live `cn=config` settings first; see the [container TLS guide](../../container-images/openldap/tls.md).

## Storage and backups

Persistence is enabled by default, with separate volumes for configuration (`1Gi`), data (`8Gi`) and backups (`8Gi`). Set `persistence.storageClass` if the cluster has no default StorageClass. A custom `persistence.<name>.volume` overrides generated storage for that volume.

To enable the daily backup job:

```yaml
backupCronJob:
  enabled: true
  schedule: "15 2 * * *"
```

The job uses pod `0`'s volumes and runs on the same node. Its default command keeps scheduled backups for 15 days and preserves manual and upgrade backups. Copy backups to separate storage as part of your backup process.

For manual backup and restore commands, see [Backup / Restore](../../container-images/openldap/backup-restore.md).

## More than one replica

The chart enables replication automatically when `replicaCount` is greater than one, unless you override the replication mode. Supply the same plaintext replication password to every replica, preferably through `bootstrapEnvSecret.existingSecret` using `OPENLDAP_BOOTSTRAP_REPLICATION_DATA_READONLY_PASSWORD`.

Choose the replica count at installation. Changing it later is guarded because the generated replication hosts are stored in the directory; changing a value alone does not safely reconfigure replication.

## Upgrade

For an upgrade with the default pre-upgrade step enabled, set `previousImage.tag` to the image currently running:

```yaml
previousImage:
  tag: 2.6.15-beta  # example: replace with your currently running image tag
image:
  tag: 2.6.15-rc
```

```bash
helm upgrade my-ldap osixia/openldap -n directory -f my-values.yaml --wait
```

The old image creates a migration backup before the new image starts. Remove `previousImage.tag` from your values after the upgrade. A fresh install does not need it. Moving from the v1 image requires a [manual migration](../../container-images/openldap/upgrade.md).

For scoped accounts, replication settings and the full values reference, see the [chart README](https://github.com/osixia/helm-charts/tree/main/openldap).
