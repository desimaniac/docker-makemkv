# Docker container for MakeMKV
[![Docker Automated build](https://img.shields.io/docker/automated/jlesage/makemkv.svg)](https://hub.docker.com/r/jlesage/makemkv/) [![](https://images.microbadger.com/badges/image/jlesage/makemkv.svg)](http://microbadger.com/#/images/jlesage/makemkv "Get your own image badge on microbadger.com") [![Build Status](https://travis-ci.org/jlesage/docker-makemkv.svg?branch=master)](https://travis-ci.org/jlesage/docker-makemkv) [![Donate](https://img.shields.io/badge/Donate-PayPal-green.svg)](https://paypal.me/JocelynLeSage)

This is a Docker container for MakeMKV.

The GUI of the application is accessed through a modern web browser (no installation or configuration needed on client side) or via any VNC client.

A fully automated mode is also available: insert a DVD or Blu-ray disc into an optical drive and let MakeMKV rips it without any user interaction.

---

[![MakeMKV logo](https://images.weserv.nl/?url=raw.githubusercontent.com/jlesage/docker-templates/master/jlesage/images/makemkv-icon.png&w=200)](http://www.makemkv.com/)[![MakeMKV](https://dummyimage.com/400x110/ffffff/575757&text=MakeMKV)](http://www.makemkv.com/)

MakeMKV is your one-click solution to convert video that you own into free and
patents-unencumbered format that can be played everywhere. MakeMKV is a format
converter, otherwise called "transcoder". It converts the video clips from
proprietary (and usually encrypted) disc into a set of MKV files, preserving
most information but not changing it in any way. The MKV format can store
multiple video/audio tracks with all meta-information and preserve chapters.

---

## Quick Start

Launch the MakeMKV docker container with the following command:
```
docker run -d \
    --name=makemkv \
    -p 5800:5800 \
    -v /docker/appdata/makemkv:/config:rw \
    -v $HOME:/storage:ro \
    -v $HOME/MakeMKV/output:/output:rw \
    --device /dev/sr0 \
    jlesage/makemkv
```

Where:
  - `/docker/appdata/makemkv`: This is where the application stores its configuration, log and any files needing persistency.
  - `$HOME`: This location contains files from your host that need to be accessible by the application.
  - `$HOME/MakeMKV/output`: This is where extracted videos are written.
  - `/dev/sr0`: This is the optical drive.

Browse to `http://your-host-ip:5800` to access the MakeMKV GUI.  Files from
the host appear under the `/storage` folder in the container.

## Usage

```
docker run [-d] \
    --name=makemkv \
    [-e <VARIABLE_NAME>=<VALUE>]... \
    [-v <HOST_DIR>:<CONTAINER_DIR>[:PERMISSIONS]]... \
    [-p <HOST_PORT>:<CONTAINER_PORT>]... \
    jlesage/makemkv
```
| Parameter | Description |
|-----------|-------------|
| -d        | Run the container in background.  If not set, the container runs in foreground. |
| -e        | Pass an environment variable to the container.  See the [Environment Variables](#environment-variables) section for more details. |
| -v        | Set a volume mapping (allows to share a folder/file between the host and the container).  See the [Data Volumes](#data-volumes) section for more details. |
| -p        | Set a network port mapping (exposes an internal container port to the host).  See the [Ports](#ports) section for more details. |

### Environment Variables

To customize some properties of the container, the following environment
variables can be passed via the `-e` parameter (one for each variable).  Value
of this parameter has the format `<VARIABLE_NAME>=<VALUE>`.

| Variable       | Description                                  | Default |
|----------------|----------------------------------------------|---------|
|`USER_ID`| ID of the user the application runs as.  See [User/Group IDs](#usergroup-ids) to better understand when this should be set. | `1000` |
|`GROUP_ID`| ID of the group the application runs as.  See [User/Group IDs](#usergroup-ids) to better understand when this should be set. | `1000` |
|`SUP_GROUP_IDS`| Comma-separated list of supplementary group IDs of the application. | (unset) |
|`UMASK`| Mask that controls how file permissions are set for newly created files. The value of the mask is in octal notation.  By default, this variable is not set and the default umask of `022` is used, meaning that newly created files are readable by everyone, but only writable by the owner. See the following online umask calculator: http://wintelguy.com/umask-calc.pl | (unset) |
|`TZ`| [TimeZone] of the container.  Timezone can also be set by mapping `/etc/localtime` between the host and the container. | `Etc/UTC` |
|`KEEP_APP_RUNNING`| When set to `1`, the application will be automatically restarted if it crashes or if user quits it. | `0` |
|`APP_NICENESS`| Priority at which the application should run.  A niceness value of -20 is the highest priority and 19 is the lowest priority.  By default, niceness is not set, meaning that the default niceness of 0 is used.  **NOTE**: A negative niceness (priority increase) requires additional permissions.  In this case, the container should be run with the docker option `--cap-add=SYS_NICE`. | (unset) |
|`CLEAN_TMP_DIR`| When set to `1`, all files in the `/tmp` directory are delete during the container startup. | `1` |
|`DISPLAY_WIDTH`| Width (in pixels) of the application's window. | `1280` |
|`DISPLAY_HEIGHT`| Height (in pixels) of the application's window. | `768` |
|`SECURE_CONNECTION`| When set to `1`, an encrypted connection is used to access the application's GUI (either via web browser or VNC client).  See the [Security](#security) section for more details. | `0` |
|`VNC_PASSWORD`| Password needed to connect to the application's GUI.  See the [VNC Password](#vnc-password) section for more details. | (unset) |
|`X11VNC_EXTRA_OPTS`| Extra options to pass to the x11vnc server running in the Docker container.  **WARNING**: For advanced users. Do not use unless you know what you are doing. | (unset) |
|`MAKEMKV_KEY`| MakeMKV registration key to use.  The key is written to the configuration file during container startup.  When set to `BETA`, the latest beta key is automatically used.  When set to `UNSET`, no key is automatically written to the configuration file. | `BETA` |
|`AUTO_DISC_RIPPER`| When set to `1`, the automatic disc ripper is enabled. | `0` |
|`AUTO_DISC_RIPPER_EJECT`| When set to `1`, disc is ejected from the drive when ripping is terminated. | `0` |
|`AUTO_DISC_RIPPER_INTERVAL`| Interval, in seconds, the automatic disc ripper checks for the presence of a DVD/Blu-ray discs. | `5` |
|`AUTO_DISC_RIPPER_MIN_TITLE_LENGTH`| Titles with a length less than this value are ignored.  Length is in seconds.  By default, no value is set, meaning that value from MakeMKV's configuration file is taken. | (unset) |
|`AUTO_DISC_RIPPER_BD_MODE`| Rip mode of Blu-ray discs.  `mkv` is the default mode, where a set of MKV files are produced.  When set to `backup`, a copy of the (decrypted) file system is created instead. **NOTE**: This applies to Blu-ray discs only.  For DVD discs, MKV files are always produced. | `mkv` |

### Data Volumes

The following table describes data volumes used by the container.  The mappings
are set via the `-v` parameter.  Each mapping is specified with the following
format: `<HOST_DIR>:<CONTAINER_DIR>[:PERMISSIONS]`.

| Container path  | Permissions | Description |
|-----------------|-------------|-------------|
|`/config`| rw | This is where the application stores its configuration, log and any files needing persistency. |
|`/storage`| ro | This location contains files from your host that need to be accessible by the application. |
|`/output`| rw | This is where extracted videos are written. |

### Ports

Here is the list of ports used by the container.  They can be mapped to the host
via the `-p` parameter (one per port mapping).  Each mapping is defined in the
following format: `<HOST_PORT>:<CONTAINER_PORT>`.  The port number inside the
container cannot be changed, but you are free to use any port on the host side.

| Port | Mapping to host | Description |
|------|-----------------|-------------|
| 5800 | Mandatory | Port used to access the application's GUI via the web interface. |
| 5900 | Optional | Port used to access the application's GUI via the VNC protocol.  Optional if no VNC client is used. |
| 51000 | Optional | Port used by the streaming service. |

## Docker Compose File
Here is an example of a `docker-compose.yml` file that can be used with
[Docker Compose](https://docs.docker.com/compose/overview/).

Make sure to adjust according to your needs.  Note that only mandatory network
ports are part of the example.

```yaml
version: '3'
services:
  makemkv:
    build: .
    ports:
      - "5800:5800"
    volumes:
      - "/docker/appdata/makemkv:/config:rw"
      - "$HOME:/storage:ro"
      - "$HOME/MakeMKV/output:/output:rw"
    devices:
      - "/dev/sr0:/dev/sr0"
```

## Docker Image Update

If the system on which the container runs doesn't provide a way to easily update
the Docker image, the following steps can be followed:

  1. Fetch the latest image:
```
docker pull jlesage/makemkv
```
  2. Stop the container:
```
docker stop makemkv
```
  3. Remove the container:
```
docker rm makemkv
```
  4. Start the container using the `docker run` command.

## User/Group IDs

When using data volumes (`-v` flags), permissions issues can occur between the
host and the container.  For example, the user within the container may not
exists on the host.  This could prevent the host from properly accessing files
and folders on the shared volume.

To avoid any problem, you can specify the user the application should run as.

This is done by passing the user ID and group ID to the container via the
`USER_ID` and `GROUP_ID` environment variables.

To find the right IDs to use, issue the following command on the host, with the
user owning the data volume on the host:

    id <username>

Which gives an output like this one:
```
uid=1000(myuser) gid=1000(myuser) groups=1000(myuser),4(adm),24(cdrom),27(sudo),46(plugdev),113(lpadmin)
```

The value of `uid` (user ID) and `gid` (group ID) are the ones that you should
be given the container.

## Accessing the GUI

Assuming that container's ports are mapped to the same host's ports, the
graphical interface of the application can be accessed via:

  * A web browser:
```
http://<HOST IP ADDR>:5800
```

  * Any VNC client:
```
<HOST IP ADDR>:5900
```

## Security

By default, access to the application's GUI is done over an unencrypted
connection (HTTP or VNC).

Secure connection can be enabled via the `SECURE_CONNECTION` environment
variable.  See the [Environment Variables](#environment-variables) section for
more details on how to set an environment variable.

When enabled, application's GUI is performed over an HTTPs connection when
accessed with a browser.  All HTTP accesses are automatically redirected to
HTTPs.

When using a VNC client, the VNC connection is performed over SSL.  Note that
few VNC clients support this method.  [SSVNC] is one of them.

### Certificates

Here are the certificate files needed by the container.  By default, when they
are missing, self-signed certificates are generated and used.  All files have
PEM encoded, x509 certificates.

| Container Path                  | Purpose                    | Content |
|---------------------------------|----------------------------|---------|
|`/config/certs/vnc-server.pem`   |VNC connection encryption.  |VNC server's private key and certificate, bundled with any root and intermediate certificates.|
|`/config/certs/web-privkey.pem`  |HTTPs connection encryption.|Web server's private key.|
|`/config/certs/web-fullchain.pem`|HTTPs connection encryption.|Web server's certificate, bundled with any root and intermediate certificates.|

**NOTE**: To prevent any certificate validity warnings/errors from the browser
or VNC client, make sure to supply your own valid certificates.

**NOTE**: Certificate files are monitored and relevant daemons are automatically
restarted when changes are detected.

### VNC Password

To restrict access to your application, a password can be specified.  This can
be done via two methods:
  * By using the `VNC_PASSWORD` environment variable.
  * By creating a `.vncpass_clear` file at the root of the `/config` volume.
    This file should contains the password in clear-text.  During the container
    startup, content of the file is obfuscated and moved to `.vncpass`.

The level of security provided by the VNC password depends on two things:
  * The type of communication channel (encrypted/unencrypted).
  * How secure access to the host is.

When using a VNC password, it is highly desirable to enable the secure
connection to prevent sending the password in clear over an unencrypted channel.

**ATTENTION**: Password is limited to 8 characters.  This limitation comes from
the Remote Framebuffer Protocol [RFC](https://tools.ietf.org/html/rfc6143) (see
section [7.2.2](https://tools.ietf.org/html/rfc6143#section-7.2.2)).  Any
characters beyhond the limit are ignored.

## Access to Optical Drive(s)

By default, a Docker container doesn't have access to host's devices.  However,
access to one or more device can be granted with the `--device DEV` parameter.

Optical drives usually have `/dev/srX` as device.  For example, the first drive
is `/dev/sr0`, the second `/dev/sr1`, and so on.  To allow MakeMKV to access
the first drive, this parameter is needed:
```
--device /dev/sr0
```

## Automatic Disc Ripper

This container has an automatic disc ripper built-in.  When enabled, any DVD or
Blu-ray video disc inserted into an optical drive is automatically ripped.  In
other words, MakeMKV decrypts and extracts all titles (such as the main movie,
bonus features, etc) from the disc to MKV files.

To enable the automatic disc ripper, set the environment variable
`AUTO_DISC_RIPPER` to `1`.

To eject the disc from the drive when ripping is terminated, set the environment
variable `AUTO_DISC_RIPPER_EJECT` to `1`.

See the [Environment Variables](#environment-variables) section for details
about setting environment variables.

**NOTE**: All titles, audio tracks, chapters, subtitles, etc are
        extracted/preserved.

**NOTE**: Titles and audio tracks are kept in their original format.  They are
        not transcoded or converted to other formats or into smaller sizes.

**MOTE**: Ripped Blu-ray discs can take a large amount of disc space (~40GB).

**NOTE**: MKV Files are written to the `/output` folder of the container.

**NOTE**: The automatic disc ripper processes all available optical drives.

## Troubleshooting

### Expired Beta Key

If the beta key is expired, just restart the container to automatically fetch
and install the latest one.

**NOTE**: For this solution to work, the `MAKEMKV_KEY` environment variable must
be set to `BETA`.  See the [Environment Variables](#environment-variables)
section for more details.

[TimeZone]: http://en.wikipedia.org/wiki/List_of_tz_database_time_zones

## Support or Contact

Having troubles with the container or have questions?  Please
[create a new issue].

[create a new issue]: https://github.com/jlesage/docker-makemkv/issues
