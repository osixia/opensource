# Values reference

## Top-level

| Key | Default | Description |
|---|---|---|
| `replicaCount` | `1` | Number of replicas |
| `image.repository` | `phpldapadmin/phpldapadmin` | Container image |
| `image.tag` | `2.3.11` | Image tag |
| `image.pullPolicy` | `IfNotPresent` | Pull policy |
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
| `podSecurityContext` | `{}` | Pod-level securityContext |
| `securityContext` | `{}` | Container-level securityContext |
| `resources` | `{}` | CPU / memory requests and limits |
| `nodeSelector` | `{}` | Node selector |
| `tolerations` | `[]` | Tolerations |
| `affinity` | `{}` | Affinity |

## Service

| Key | Default | Description |
|---|---|---|
| `service.type` | `ClusterIP` | Service type |
| `service.port` | `80` | Service port (forwards to container port `http`, 80) |

## Ingress

| Key | Default | Description |
|---|---|---|
| `ingress.enabled` | `false` | Create an Ingress resource |
| `ingress.className` | `""` | `ingressClassName` |
| `ingress.annotations` | `{}` | Ingress annotations |
| `ingress.hosts` | `[{host: phpldapadmin.local, paths: [{path: /, pathType: Prefix}]}]` | Hosts and paths |
| `ingress.tls` | `[]` | TLS configuration blocks |

## HTTPRoute (Gateway API)

| Key | Default | Description |
|---|---|---|
| `httpRoute.enabled` | `false` | Create an HTTPRoute resource |
| `httpRoute.annotations` | `{}` | HTTPRoute annotations |
| `httpRoute.parentRefs` | `[{name: ""}]` | Gateway parent references |
| `httpRoute.hostnames` | `[]` | Hostnames matched by the route |

## Probes

| Key | Default | Description |
|---|---|---|
| `livenessProbe.enabled` | `true` | Enable liveness probe |
| `livenessProbe.*` | `httpGet` on `/` (port `http`) | Standard Kubernetes probe fields |
| `readinessProbe.enabled` | `true` | Enable readiness probe |
| `readinessProbe.*` | `httpGet` on `/` (port `http`) | Standard Kubernetes probe fields |

## Application (app.*)

| Key | Default | Description |
|---|---|---|
| `app.appKey` | `""` | Laravel `APP_KEY`. Auto-generated and stable across upgrades if left empty; set explicitly for a predictable value |
| `app.existingSecret` | `""` | Existing secret name containing `APP_KEY`. If set, the chart does not generate its own Secret |
| `app.ldap.name` | `LDAP Server` | Display name shown in the UI |
| `app.ldap.host` | `""` | LDAP server host, e.g. `<openldap-release>.<namespace>.svc.cluster.local` |
| `app.ldap.port` | `389` | LDAP server port |
| `app.ldap.connection` | `ldap` | Connection profile: `ldap` \| `ldaps` |
| `app.ldap.ssl` | `false` | Use LDAPS (implicit TLS) |
| `app.ldap.tls` | `false` | Use StartTLS |
| `app.ldap.timeout` | `5` | Connection timeout (seconds) |
| `app.ldap.alertRootDn` | `true` | Warn in the UI when logged in as the root DN |
| `app.ldap.cache` | `false` | Enable LDAP query caching |

These values are rendered into a ConfigMap (`<release>-env`) consumed by the
container as environment variables (`LDAP_CONNECTION`, `LDAP_HOST`, `LDAP_PORT`, …),
alongside the generated/referenced `APP_KEY` Secret.
