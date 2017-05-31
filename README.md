# Docker container for HandBrake
[![Docker Automated build](https://img.shields.io/docker/automated/jlesage/handbrake.svg)](https://hub.docker.com/r/jlesage/handbrake/) [![](https://images.microbadger.com/badges/image/jlesage/handbrake.svg)](http://microbadger.com/#/images/jlesage/handbrake "Get your own image badge on microbadger.com") [![Build Status](https://travis-ci.org/jlesage/docker-handbrake.svg?branch=master)](https://travis-ci.org/jlesage/docker-handbrake)

This is a Docker container for HandBrake.  The GUI of the application is
accessed through a modern web browser (no installation or configuration needed
on client side) or via any VNC client.

A fully automated mode is also available: drop files into a watch folder and let
HandBrake process them without any user interaction.

---

[![HandBrake logo](https://raw.githubusercontent.com/jlesage/docker-templates/master/jlesage/images/handbrake-icon.png)](https://handbrake.fr/)
[![HandBrake](https://dummyimage.com/400x110/ffffff/575757&text=HandBrake)](https://handbrake.fr/)

HandBrake is a tool for converting video from nearly any format to a selection
 of modern, widely supported codecs.

---

## Quick Start
First create the configuration directory for HandBrake.  In this example,
`/docker/appdata/handbrake` is used.  Other directories also need to be defined:
source of videos to be converted (`$HOME/Videos`), destination of converted
videos (`$HOME/HandBrake/output`) and optionally a watch folder
(`$HOME/HandBrake/watch`).  Launch the HandBrake docker container with the
following command:
```
docker run -d --rm \
    --name=handbrake \
    -p 5800:5800 \
    -p 5900:5900 \
    -v /var/docker/handbrake:/config \
    -v $HOME/Videos:/storage:ro \
    -v $HOME/HandBrake/watch:/watch:ro \
    -v $HOME/HandBrake/output:/output \
    jlesage/handbrake
```

Browse to `http://your-host-ip:5800` to access the HandBrake GUI.  Your video
files appear under the `/storage` folder in the container.

## Usage
```
docker run [-d] [--rm] \
    --name=handbrake \
    [-e <VARIABLE_NAME>=<VALUE>]... \
    [-v <HOST_DIR>:<CONTAINER_DIR>[:PERMISSIONS]]... \
    [-p <HOST_PORT>:<CONTAINER_PORT>]... \
    jlesage/handbrake
```
| Parameter | Description |
|-----------|-------------|
| -d        | Run the container in background.  If not set, the container runs in foreground. |
| --rm      | Automatically remove the container when it exits. |
| -e        | Pass an environment variable to the container.  See the [Environment Variables](#environment-variables) section for more details. |
| -v        | Set a volume mapping (allows to share a folder/file between the host and the container).  See the [Data Volumes](#data-volumes) section for more details. |
| -p        | Set a network port mapping (exposes an internal container port to the host).  See the [Ports](#ports) section for more details. |

### Environment Variables

To customize some properties of the container, the following environment
variables can be passed via the `-e` parameter (one for each variable).  Value
of this parameter has the format `<VARIABLE_NAME>=<VALUE>`.

| Variable       | Description                                  | Default |
|----------------|----------------------------------------------|---------|
|`USER_ID`       | ID of the user the application runs as.  See [User/Group IDs](#usergroup-ids) to better understand when this should be set. | 1000    |
|`GROUP_ID`      | ID of the group the application runs as.  See [User/Group IDs](#usergroup-ids) to better understand when this should be set. | 1000    |
|`TZ`            | [TimeZone] of the container.  Timezone can also be set by mapping `/etc/localtime` between the host and the container. | Etc/UTC |
|`DISPLAY_WIDTH` | Width (in pixels) of the display.             | 1280    |
|`DISPLAY_HEIGHT`| Height (in pixels) of the display.            | 768     |
|`VNC_PASSWORD`  | Password needed to connect to the application's GUI.  See the [VNC Pasword](#vnc-password) section for more details. | (unset) |
|`KEEP_GUIAPP_RUNNING`| When set to `1`, the application will be automatically restarted if it crashes or if user quits it. | (unset) |
|`APP_NICENESS`  | Priority at which the application should run.  A niceness value of −20 is the highest priority and 19 is the lowest priority.  By default, niceness is not set, meaning that the default niceness of 0 is used.  **NOTE**: A negative niceness (priority increase) requires additional permissions.  In this case, the container should be run with the docker option `--cap-add=SYS_NICE`. | (unset) |
|`AUTOMATED_CONVERSION_PRESET`| HandBrake preset used by the automatic video converter.  See the [Automatic Video Conversion](#automatic-video-conversion) section for more details. | "Very Fast 1080p30" |
|`AUTOMATED_CONVERSION_FORMAT`| Video container format used by the automatic video converter for output files.  This is typically the video filename extension.  See the [Automatic Video Conversion](#automatic-video-conversion) section for more details.  | "mp4" |
|`AUTOMATED_CONVERSION_KEEP_SOURCE`| When set to `0`, a video that has been successfully converted is removed from the watch folder. | 1 |
|`AUTOMATED_CONVERSION_SOURCE_STABLE_TIME`| Time during which properties (e.g. size, time, etc) of a video file in the watch folder need to remain the same.  This is to avoid processing a file that is being copied. | 5 |
|`HANDBRAKE_DEBUG`| Setting this to `1` enables HandBrake debug logging.  Log messages are sent to `/config/handbrake.debug.log` (container path).  **NOTE**: When enabled, a lot of information is generated and the log file will grow quickly.  Make sure to enable this temporarily and only when needed. | (unset) |

[TimeZone]: http://en.wikipedia.org/wiki/List_of_tz_database_time_zones

### Data Volumes

The following table describes data volumes used by the container.  The mappings
are set via the `-v` parameter.  Each mapping is specified with the following
format: `<HOST_DIR>:<CONTAINER_DIR>[:PERMISSIONS]`.

| Container path  | Permissions | Description |
|-----------------|-------------|-------------|
|`/config`        | rw          | This is where the application stores its configuration, log and any files needing persistency. |
|`/storage`       | ro          | This is where video files to be converted on your host are made available to the application. |
|`/watch`         | ro          | This is where videos to be automatically converted are located. |
|`/output`        | rw          | This is where automatically converted video files are written. |

### Ports

Here is the list of ports used by the container.  They can be mapped to the host
via the `-p` parameter (one per port mapping).  Each mapping is defined in the
following format: `<HOST_PORT>:<CONTAINER_PORT>`.  The port number inside the
container cannot be changed, but you are free to use any port on the host side.

| Port | Mapping to host | Description |
|------|-----------------|-------------|
| 5800 | Mandatory       | Port used to access the application's GUI via the web interface. |
| 5900 | Mandatory       | Port used to access the application's GUI via the VNC protocol.  |

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

Assuming the host is mapped to the same ports as the container, the graphical
interface of the application can be accessed via:

  * A web browser:
```
http://<HOST IP ADDR>:5800
```

  * Any VNC client:
```
<HOST IP ADDR>:5900
```

If different ports are mapped to the host, make sure they respect the
following formula:

    VNC_PORT = HTTP_PORT + 100

This is to make sure accessing the GUI with a web browser can be done without
specifying the VNC port manually.  If this is not possible, then specify
explicitly the VNC port like this:

    http://<HOST IP ADDR>:5800/?port=<VNC PORT>

## VNC Password

To restrict access to your application, a password can be specified.  This can
be done via two methods:
  * By using the `VNC_PASSWORD` environment variable.
  * By creating a `.vncpass_clear` file at the root of the `/config` volume.
  This file should contains the password (in clear).  During the container
  startup, content of the file is obfuscated and renamed to `.vncpass`.

**NOTE**: This is a very basic way to restrict access to the application and it
should not be considered as secure in any way.

## Automatic Video Conversion
This container has an automatic video converter built-in.  This is useful to
batch-convert videos without user interaction.

Basically, files copied to the `/watch` container folder are automatically
converted by HandBrake to a pre-defined video format according to a pre-defined
preset.  Both the format and the preset are specified via environment variables:

| Variable       | Default |
|----------------|---------|
|`AUTOMATED_CONVERSION_PRESET` | "Very Fast 1080p30" |
|`AUTOMATED_CONVERSION_FORMAT` | "mp4" |

See the [Environment Variables](#environment-variables) section for details
about setting environment variables.

**NOTE**: Converted videos are stored to the `/output` folder of the container.

**NOTE**: All default presets, along with personalized/custom ones, can be seen
with the HandBrake GUI.

### Hooks

Custom actions can be performed using hooks.  Hooks are shell scripts executed
by the automatic video converter.

**NOTE**: Hooks are always invoked via `/bin/sh`, ignoring any shebang the
script may have.

Hooks are optional and by default, no one is defined.  A hook is defined and
executed when the script is found at a specific location.

The following table describe available hooks:

| Container location | Description | Parameter(s) |
|--------------------|-------------|--------------|
| `/config/hooks/post_conversion.sh` | Hook executed when the conversion of a video file is terminated. | The first parameter is the status of the conversion.  A value of `0` indicates that the conversion terminated successfuly.  Any other value represent a failure.  The second argument is the path to the converted video (the output). |

During the first start of the container, example hooks are installed in
`/config/hooks/`.  Example scripts have the suffix `.example`.  For example,
you can use `/config/hooks/post_conversion.sh.example` as a starting point.

**NOTE**: Keep in mind that this container has the minimal set of packages
required to run HandBrake.  This may limit actions that can be performed in
hooks.
