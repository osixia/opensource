# 🛠️ LDAP Toolkit

This image ships with the full suite of OpenLDAP utilities, plus the `openldap-ctl` helper.
Commands are run with the `osixia/baseimage` [`container run`](../baseimage/container-command/run.md) command, which loads the image environment variables first.

## Managing a running server

Inside the container, the `ldap` system user connected through the `ldapi:///` socket is authenticated with SASL `EXTERNAL` and has `manage` access on the data database. No password is needed:

``` bash
docker exec openldap container run -- \
  ldapsearch -Y EXTERNAL -H ldapi:/// -b "dc=example,dc=org" "(objectClass=*)"
```

Add entries from a local LDIF file:

``` bash
docker exec -i openldap container run -- \
  ldapadd -Y EXTERNAL -H ldapi:/// < entries.ldif
```

To change the configuration (`cn=config`), bind as the config root DN:

``` bash
docker exec -i openldap container run -- \
  ldapmodify -H ldapi:/// -D "cn=admin,cn=config" -w <config root password> <<EOF
dn: cn=config
changetype: modify
replace: olcSizeLimit
olcSizeLimit: 1000
EOF
```

## Database tools

`slapcat`, `slapadd`, `slapindex` and `slaptest` operate directly on the local database files and must run inside the container.

Export the data database to LDIF:

``` bash
docker exec openldap container run -- slapcat -n 1
```

Export the config database (`cn=config`):

``` bash
docker exec openldap container run -- slapcat -n 0
```

!!! note
    `slapadd` and `slapindex` require OpenLDAP to be stopped first:

    ``` bash
    docker exec openldap container processes stop openldap
    docker exec openldap container run -- slapindex -n 1
    docker exec openldap container processes start openldap
    ```

## LDAP client tools

`ldapsearch`, `ldapwhoami`, `ldapadd`, `ldapmodify`, `ldapdelete` and `ldappasswd` connect to any LDAP server over the network. The image can be used as a throwaway client:

Search for all entries under a base DN:

``` bash
docker run --rm osixia/openldap run -- \
  ldapsearch -H ldap://172.16.0.1 \
  -D "cn=admin,dc=example,dc=org" -w passw0rd \
  -b "dc=example,dc=org" "(objectClass=*)"
```

Verify bind credentials:

``` bash
docker run --rm osixia/openldap run -- \
  ldapwhoami -H ldap://172.16.0.1 \
  -D "cn=admin,dc=example,dc=org" -w passw0rd
```

Add entries from a local LDIF file:

``` bash
docker run --rm -v /path/to/entries.ldif:/tmp/entries.ldif \
  osixia/openldap run -- \
  ldapadd -H ldap://172.16.0.1 \
  -D "cn=admin,dc=example,dc=org" -w passw0rd \
  -f /tmp/entries.ldif
```

Change a user's password:

``` bash
docker run --rm osixia/openldap run -- \
  ldappasswd -H ldap://172.16.0.1 \
  -D "cn=admin,dc=example,dc=org" -w passw0rd \
  -s newpassword "uid=jdoe,ou=people,dc=example,dc=org"
```

## openldap-ctl

```
openldap-ctl command [args]...
```

| Command       | Description                                                  |
| ------------- | ------------------------------------------------------------ |
| `backup`      | Back up config and/or data, see [Backup / Restore](backup-restore.md) |
| `restore`     | Restore config and/or data, see [Backup / Restore](backup-restore.md) |
| `password`    | Generate or hash passwords                                   |
| `schema2ldif` | Convert `.schema` files to LDIF                              |

### Password helpers

Generate a random password:

``` bash
docker run --rm osixia/openldap run -- openldap-ctl password generate
```

Hash a password (default `{ARGON2}`, see `OPENLDAP_CTL_PASSWORD_HASH_CMD`):

``` bash
docker run --rm osixia/openldap run -- openldap-ctl password hash 'passw0rd'
```

The result can be used in `OPENLDAP_BOOTSTRAP_*_PASSWORD_HASHED` variables or as a `userPassword` value.

### Convert schema to LDIF

OpenLDAP loads schemas in LDIF format. Convert a legacy `.schema` file:

``` bash
docker run --rm -v $PWD/schema:/data \
  osixia/openldap run -- openldap-ctl schema2ldif /data/my-schema.schema
```

This writes `/data/my-schema.ldif` (the directory must be writable by uid `911`) and prints it.

Usage:

```
openldap-ctl schema2ldif "schema-files" ["dependencies"]
```

- `schema-files`: one or more `.schema` files, space-separated,
- `dependencies`: schemas required by the converted ones, taken from `OPENLDAP_SCHEMAS_DIR` (default `OPENLDAP_CTL_SCHEMA2LDIF_DEPENDENCIES`: `core.schema cosine.schema inetorgperson.schema`).

``` bash
docker run --rm -v $PWD/schema:/data \
  osixia/openldap run -- openldap-ctl schema2ldif /data/my-schema.schema "core.schema cosine.schema nis.schema"
```

Then load it at bootstrap, see [Custom schemas](bootstrap.md#custom-schemas).
