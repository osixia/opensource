# Schemas

Schemas are loaded at bootstrap from the list in `OPENLDAP_BOOTSTRAP_SCHEMAS`. Each name is a file in `/etc/openldap/schema`, in `cn=config` LDIF format.

The default list is:

``` bash
OPENLDAP_BOOTSTRAP_SCHEMAS="core.ldif cosine.ldif inetorgperson.ldif rfc2307bis.ldif"
```

## Loading More Schemas

Add the schema file names to the list, after the schemas they depend on:

``` bash
docker run --name openldap \
  -e OPENLDAP_BOOTSTRAP_SCHEMAS="core.ldif cosine.ldif inetorgperson.ldif rfc2307bis.ldif openssh-lpk.ldif sudo.ldif" \
  osixia/openldap
```

!!! warning

    The [default access rules](access-control.md#default-rules) reference attributes from `cosine.ldif` and `rfc2307bis.ldif`.
    Keep these two schemas in the list, or replace the default rules.

## Schemas Shipped With the Image

In addition to the schemas provided by OpenLDAP (`core`, `cosine`, `inetorgperson`, `nis`, `dyngroup`, `misc` and others), the image ships these:

| Schema               | Purpose                                                                    |
| -------------------- | -------------------------------------------------------------------------- |
| `rfc2307bis.ldif`    | UNIX accounts and groups, with better group modeling than `nis`            |
| `eduperson.ldif`     | Academic identities used in higher-education federations                   |
| `kerberos.ldif`      | Kerberos principals for LDAP-backed KDC setups                             |
| `openssh-lpk.ldif`   | SSH public keys (`sshPublicKey`)                                           |
| `postfix-book.ldif`  | Mail accounts and forwards, as used by docker-mailserver                   |
| `virtualMail.ldif`   | Virtual mailbox deployments                                                |
| `samba.ldif`         | Samba domain and account attributes                                        |
| `sudo.ldif`          | Sudo rules (`sudoRole`)                                                    |
| `totp.ldif`          | TOTP secrets for LDAP-based two-factor authentication                      |

!!! warning "Incompatible schemas"

    - Do not load `postfix-book.ldif` together with `virtualMail.ldif`: they define attributes with the same names but different OIDs.
    - `rfc2307bis.ldif` replaces `nis.ldif`: load one or the other.

Sources are listed in the [schema directory](https://github.com/osixia/container-openldap/tree/develop/services/openldap/assets/schema) of the repository.

## Adding Your Own Schema

Custom schemas are read from `/container/services/openldap/assets/schema` and copied to the OpenLDAP schema directory at startup.

**1. Convert the schema to LDIF** if it is in the traditional `.schema` format:

``` bash
docker run --rm --user $UID -v $(pwd)/schema:/data \
  osixia/openldap run -- openldap-ctl schema2ldif /data/my-app.schema
```

This writes `my-app.ldif` next to the source file. See [LDAP Toolkit](../ldap-toolkit.md#convert-schema-to-ldif).

**2. Mount the LDIF file and add it to the list:**

``` bash
docker run --name openldap \
  -v $(pwd)/schema/my-app.ldif:/container/services/openldap/assets/schema/my-app.ldif \
  -e OPENLDAP_BOOTSTRAP_SCHEMAS="core.ldif cosine.ldif inetorgperson.ldif rfc2307bis.ldif my-app.ldif" \
  osixia/openldap
```

To add a schema to an existing directory, load the LDIF file with `ldapadd` bound as the configuration root DN.

## Modules

Modules loaded at bootstrap are listed in `OPENLDAP_BOOTSTRAP_MODULES`:

``` bash
OPENLDAP_BOOTSTRAP_MODULES="back_mdb.so argon2.so ppolicy.so unique.so refint.so memberof.so syncprov.so"
```

The image includes all OpenLDAP backends and overlays, plus the Argon2, PBKDF2 and SHA-2 password modules. Add a module name to the list to load it.

To use a module that is not part of the image, mount it in `/container/services/openldap/assets/module`: it is copied to the OpenLDAP modules directory at startup.
