[![logo](https://raw.githubusercontent.com/larstobi/samba/master/logo.jpg)](https://www.samba.org)

# Samba

Samba Docker container.

# What is Samba?

Since 1992, Samba has provided secure, stable, and fast file and print services
for all clients using the SMB/CIFS protocol, such as all versions of DOS and
Windows, OS/2, Linux, and many others.

# How to use this image

No shares are configured by default. Add shares with the options described
below.

## Version tags

The image is tagged with semantic version aliases. Use `larstobi/samba:1` to
track the newest build for major version 1, `larstobi/samba:1.0` to track the
newest build for minor version 1.0, or `larstobi/samba:1.0.0` for the current
patch version. These tags are rebuilt periodically so base image and package
updates are picked up.

Each build also gets a unique traceable tag in the form
`1.0.0-YYYYMMDD.RUN_NUMBER`. Docker image tags do not support SemVer build
metadata with `+`, so the date and workflow run number are encoded with Docker
compatible characters.

## Hosting a Samba instance

```console
sudo docker run -it --net host -d larstobi/samba -p
```

To use local storage:

```console
sudo docker run -it --name samba --net host \
            -v /path/to/directory:/mount \
            -d larstobi/samba -p
```

## Configuration

```console
sudo docker run -it --rm larstobi/samba -h
```

```text
Usage: samba.sh [-opt] [command]
Options (fields in '[]' are optional, '<>' are required):
    -h          Show this help
    -c "<from:to>" Configure character mappings for file and directory names
                required arg: "<from:to>" character mappings separated by ','
    -G "<section;parameter>" Configure a generic section option for smb.conf
                required arg: "<section>" - for example: "share"
                required arg: "<parameter>" - for example: "log level = 2"
    -g "<parameter>" Configure a global option for smb.conf
                required arg: "<parameter>" - for example: "log level = 2"
    -i "<path>" Import an smbpasswd file
                required arg: "<path>" - full file path inside the container
    -n          Start the 'nmbd' daemon to advertise shares
    -d "<args>" Configure wsdd for WS-Discovery
                arg: "false" disables wsdd; any other value is passed to wsdd
                example: "-i eth0 --no-http"
    -p          Set ownership and permissions on shares
    -r          Disable the recycle bin for shares
    -S          Disable the SMB2 minimum version
    -s "<name;/path>[;browse;readonly;guest;users;admins;writelist;comment]"
                Configure a share
                required arg: "<name>;</path>"
                <name> is the client-visible share name
                <path> is the path to share
                NOTE: leave optional fields blank to use default values
                [browsable] default: 'yes' or 'no'
                [readonly] default: 'yes' or 'no'
                [guest] default: 'yes' or 'no'
                NOTE: usernames in the lists below are separated by ','
                [users] default: 'all' or list of allowed users
                [admins] default: 'none' or list of admin users
                [writelist] list of users that can write to a read-only share
                [comment] share description
    -u "<username;password>[;ID;group;GID]"       Add a user
                required arg: "<username>;<passwd>"
                <username> user name
                <password> user password
                [ID] user ID
                [group] user group
                [GID] group ID
    -w "<workgroup>"       Configure the Samba workgroup or domain
                required arg: "<workgroup>"
                <workgroup> Samba workgroup
    -W          Allow wide symbolic links
    -I          Add an include option at the end of smb.conf
                required arg: "<include file path>"
                <include file path> inside the container, such as a bind mount

The 'command' argument, when provided and valid, runs instead of Samba.
```

## Environment variables

 * `CHARMAP` - Configure character mappings
 * `GENERIC` - Configure a generic section option. See note 4 below.
 * `GLOBAL` - Configure a global option. See note 4 below.
 * `IMPORT` - Import an smbpasswd file
 * `NMBD` - Enable nmbd
 * `AVAHI` - Enable Avahi/mDNS discovery. Defaults to `yes`; set to `false` to disable.
 * `PERMISSIONS` - Set file permissions on all shares
 * `RECYCLE` - Disable the recycle bin
 * `SHARE` - Set up a share. See note 4 below.
 * `SMB` - Disable the SMB2 minimum version
 * `TZ` - Set a timezone, for example `EST5EDT`
 * `USER` - Set up a user. See note 4 below.
 * `WIDELINKS` - Allow wide symbolic links
 * `WORKGROUP` - Set the workgroup
 * `WSDD` - Enable wsdd/WS-Discovery for Windows Network discovery. Defaults to `yes`; set to `false` to disable.
 * `WSDD_ARGS` - Extra flags passed to wsdd, for example `-i eth0 --no-http`. Set to `false` to disable wsdd.
 * `WSDD_OPTS` - Backward-compatible alias for `WSDD_ARGS`.
 * `USERID` - Set the UID for the default Samba user, `smbuser`
 * `GROUPID` - Set the GID for the default Samba group, `smb`
 * `INCLUDE` - Add an smb.conf include
 * `VETO` - Configure predefined veto files. Defaults to `yes`; set to `no` to disable.

**NOTE**: If you enable nmbd with `-n` or the `NMBD` environment variable, also
expose ports 137 and 138 with `-p 137:137/udp -p 138:138/udp`.

**NOTE2**: There are reports that `-n` and `NMBD` only work when the container
uses the host's network stack.

**NOTE3**: Avahi and wsdd use multicast discovery. For the server to appear
automatically in Finder or Windows File Explorer's Network view, use the host
network stack with `--net host` or an equivalent network setup that passes
multicast traffic. If you do not use host networking, publish the service ports
you need, such as:

```console
-p 139:139 -p 445:445 -p 3702:3702/udp -p 5353:5353/udp -p 5357:5357
```

**NOTE4**: Some environment variables support numbered variants. For example,
`SHARE` also works as `SHARE2`, `SHARE3`, and so on.

## Examples

All commands can be passed at container creation with `docker run`, or run later
with `docker exec -it samba samba.sh`.

### Setting the Timezone

```console
sudo docker run -it -e TZ=EST5EDT --net host -d larstobi/samba -p
```

### Start an instance with users and shares

```console
sudo docker run -it --net host -d larstobi/samba -p \
            -u "example1;badpass" \
            -u "example2;badpass" \
            -s "public;/share" \
            -s "users;/srv;no;no;no;example1,example2" \
            -s "example1 private share;/example1;no;no;no;example1" \
            -s "example2 private share;/example2;no;no;no;example2"
```

# User Feedback

## Troubleshooting

* The client reports `Access is denied` or a similar error, or the container
logs show `change_to_user_internal: chdir_current_service() failed!`.

Add the `-p` option to the end of the container options, or set the
`PERMISSIONS` environment variable.

```console
sudo docker run -it --name samba -p 139:139 -p 445:445 \
            -v /path/to/directory:/mount \
            -d larstobi/samba -p
```

If changing file permissions is not possible in your setup, set `USERID` and
`GROUPID` to the owner values for your files.

* Samba uses a high amount of memory. Multiple people have reported memory use
that is never released by the Samba processes. The recommended workaround is to
set a container memory limit.

Add `-m 512m` to the `docker run` command, or set `mem_limit:` in
`docker-compose.yml`.

```console
sudo docker run -it --name samba -m 512m -p 139:139 -p 445:445 \
            -v /path/to/directory:/mount \
            -d larstobi/samba -p
```

* Connections with the `smbclient` command-line tool fail. By default,
`smbclient` still tries to use SMB1, which is deprecated and has known security
issues. This container defaults to SMB2 or newer, so run `smbclient -m SMB3`
followed by any other options you need.

## Issues

If you have any problems with or questions about this image, please contact me
through a [GitHub issue](https://github.com/larstobi/samba/issues).
