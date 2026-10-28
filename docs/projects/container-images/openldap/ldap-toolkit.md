# 🛠️ LDAP Toolkit

The image ships with the full suite of OpenLDAP utilities and with `openldap-ctl`, a helper for common tasks. They can be used inside a running container or in a throwaway one.

``` bash
docker exec openldap container run -- <command>   # inside a running container
docker run --rm osixia/openldap:2.6.15-rc run -- <command>  # in a throwaway container
```

`container run --` loads the image environment variables before running the command.

## Database Tools

`slapcat`, `slapadd`, `slapindex` and `slaptest` operate directly on the local database files and must run inside the container.

Export the data database to LDIF:

``` bash
docker exec openldap container run -- slapcat -n 1
```

Export the configuration database (`cn=config`):

``` bash
docker exec openldap container run -- slapcat -n 0
```

!!! note

    `slapadd` and `slapindex` require OpenLDAP to be stopped first:

    ``` bash
    docker exec openldap container processes stop openldap
    docker exec openldap container processes start openldap
    ```

    For a full export and import, prefer [`openldap-ctl backup` and `restore`](backup-restore.md).

## LDAP Client Tools

`ldapsearch`, `ldapwhoami`, `ldapadd`, `ldapmodify`, `ldapdelete` and `ldappasswd` connect to any LDAP server over the network.

Search for all entries under a base DN:

``` bash
docker run --rm osixia/openldap:2.6.15-rc run -- \
  ldapsearch -H ldap://172.16.0.1 \
  -D "cn=admin,dc=example,dc=org" -w passw0rd \
  -b "dc=example,dc=org" "(objectClass=*)"
```

Verify bind credentials:

``` bash
docker run --rm osixia/openldap:2.6.15-rc run -- \
  ldapwhoami -H ldap://172.16.0.1 \
  -D "cn=admin,dc=example,dc=org" -w passw0rd
```

Add entries from a local LDIF file:

``` bash
docker run --rm -v /path/to/entries.ldif:/tmp/entries.ldif \
  osixia/openldap:2.6.15-rc run -- \
  ldapadd -H ldap://172.16.0.1 \
  -D "cn=admin,dc=example,dc=org" -w passw0rd \
  -f /tmp/entries.ldif
```

Change a user's password:

``` bash
docker run --rm osixia/openldap:2.6.15-rc run -- \
  ldappasswd -H ldap://172.16.0.1 \
  -D "cn=admin,dc=example,dc=org" -w passw0rd \
  -s newpassword "uid=jdoe,ou=people,dc=example,dc=org"
```

### Local Administration Without a Password

Inside the container, the `ldap` user has full access to the data database through the local socket:

``` bash
docker exec openldap container run -- \
  ldapsearch -Q -Y EXTERNAL -H ldapi:/// -b "dc=example,dc=org"
```

To pipe an LDIF file from the host, add `-i` to `docker exec`:

``` bash
docker exec -i openldap container run -- \
  ldapadd -Q -Y EXTERNAL -H ldapi:/// < entries.ldif
```

## openldap-ctl

``` bash
docker run --rm osixia/openldap:2.6.15-rc run -- openldap-ctl --help
```

| Command       | Description                                                            |
| ------------- | ---------------------------------------------------------------------- |
| `backup`      | Back up the configuration and data databases                           |
| `restore`     | Restore a backup                                                       |
| `validate-ldif` | Check data LDIF against the initialized configuration without importing it |
| `password`    | Generate or hash a password                                            |
| `schema2ldif` | Convert a `.schema` file to `cn=config` LDIF                           |

`backup` and `restore` are described in [Backup / Restore](backup-restore.md).

### Validate LDIF

Check entries against the instance's schemas without importing them or stopping OpenLDAP. First copy the file into the running container:

```bash
docker cp users.ldif openldap:/tmp/users.ldif
docker exec openldap container run -- openldap-ctl validate-ldif /tmp/users.ldif
```

You can pass several files or a directory. Directories are searched recursively for `*.ldif`, excluding hidden paths. Each file must contain complete data entries. The command checks syntax, attribute values and schemas, and returns a non-zero status if validation fails.

It requires initialized OpenLDAP configuration. It does not check existing entries, ACLs or overlay rules, and does not accept `ldapmodify` change records.

### Password Helpers

Generate a random password:

``` bash
docker run --rm osixia/openldap:2.6.15-rc run -- openldap-ctl password generate
```

Hash a password:

``` bash
docker run --rm osixia/openldap:2.6.15-rc run -- openldap-ctl password hash 'passw0rd'
```

The result is an Argon2 hash, ready to be used in a `*_PASSWORD_HASHED` bootstrap variable or in a `userPassword` attribute:

``` text
{ARGON2}$argon2id$v=19$m=7168,t=5,p=1$...
```

Quote the password with single quotes so the shell does not interpret special characters.

To read a password from a file or standard input:

```bash
docker run --rm --user "$UID" -v "$PWD/password.txt:/run/secrets/password:ro" \
  osixia/openldap:2.6.15-rc run -- openldap-ctl password hash --file /run/secrets/password
docker run --rm -i osixia/openldap:2.6.15-rc run -- \
  openldap-ctl password hash --stdin < password.txt
```

The contents are hashed exactly, including trailing newlines. Use `hash -- password` for a literal password named `--file`, `--stdin` or `--`.

Both commands can be changed with `OPENLDAP_CTL_PASSWORD_GENERATE_CMD` and `OPENLDAP_CTL_PASSWORD_HASH_CMD`.

### Convert Schema to LDIF

OpenLDAP loads schemas in `cn=config` LDIF format. `schema2ldif` converts a traditional `.schema` file:

``` bash
docker run --rm --user $UID -v $(pwd)/schema:/data \
  osixia/openldap:2.6.15-rc run -- openldap-ctl schema2ldif /data/test.schema
```

The LDIF file is written next to the source file, here `schema/test.ldif`, and printed. The command fails if the LDIF file already exists.

Usage:

``` bash
openldap-ctl schema2ldif "schema-files" ["dependencies"]
```

- Several schema files can be converted at once, separated by spaces in a single quoted argument.
- Schemas the converted file depends on are listed in the second argument. The default is `core.schema cosine.schema inetorgperson.schema` (`OPENLDAP_CTL_SCHEMA2LDIF_DEPENDENCIES`).

To load the result at bootstrap, see [Schemas](bootstrap/schemas.md#adding-your-own-schema).
