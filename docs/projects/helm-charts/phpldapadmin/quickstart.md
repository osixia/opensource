# ⚡ Quickstart

## Point it at your LDAP server

The only value you **must** set is the LDAP host — typically the Service name of an
existing `osixia/openldap` chart release:

``` yaml
app:
  ldap:
    name: "LDAP Server"                                          # display name in the UI
    host: "my-ldap.<namespace>.svc.cluster.local"
    port: 389
    connection: ldap    # ldap | ldaps
    ssl: false
    tls: false
```

Install:

``` bash
helm install my-phpldapadmin ./phpldapadmin -f my-values.yaml
```

## Access the UI

By default no Ingress or HTTPRoute is created. Reach the UI with a port-forward:

``` bash
kubectl port-forward svc/my-phpldapadmin 8080:80
```

Then open `http://localhost:8080`.

The chart prints these instructions (and the configured LDAP endpoint) as the Helm
release notes after every install/upgrade.

## Expose it with Ingress

``` yaml
ingress:
  enabled: true
  className: "nginx"
  hosts:
    - host: phpldapadmin.example.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: phpldapadmin-tls
      hosts:
        - phpldapadmin.example.com
```

## Expose it with Gateway API (HTTPRoute)

`ingress` and `httpRoute` are independent — enable whichever matches your cluster's
ingress controller.

``` yaml
httpRoute:
  enabled: true
  parentRefs:
    - name: my-gateway
      namespace: gateway-system
  hostnames:
    - phpldapadmin.example.com
```

## Connect over LDAPS / StartTLS

``` yaml
app:
  ldap:
    host: "my-ldap.<namespace>.svc.cluster.local"
    port: 636
    connection: ldaps   # ldap | ldaps
    ssl: true
    tls: false          # set true instead of ssl for StartTLS on port 389
```

## Application key (APP_KEY)

phpLDAPadmin is a Laravel application and needs a stable encryption key (`APP_KEY`)
for sessions and cookies to survive pod restarts/rescheduling.

- Leave `app.appKey` empty (default) and the chart generates one automatically. It stays
  stable across upgrades as long as the release is not deleted and reinstalled.
- Set `app.appKey` explicitly if you need a predictable, portable value.
- Or reference an existing secret with `app.existingSecret` (the chart then skips
  generating its own Secret — the referenced secret must contain an `APP_KEY` key).

``` yaml
app:
  existingSecret: my-phpldapadmin-appkey
```
