# `photo_frame` role

Set up a digital photo frame using the **swayimg** image vieweer.
Creates a systemd service to view all files in a specified directory in fullscreen mode.

Requires a Wayland-based graphical display.

If the host is a Raspberry Pi host running the `labwc` compositor, the `wf-panel-pi` panel will be
disabled by default (making the desktop largely useless unless the panel is restarted manually).
This is because `wf-panel-pi` currently doesn't support auto-hide for the taksk bar, which, 
therefore, covers part of the screen.

## Dependencies

- **community.general** collection
- **Wayland** on the target host

## Variables

#### `photo_frame_display`=*string*

The name of the Wayland display to use.

Default: wayland-0

#### `photo_frame_nfs_source`=*host*:*path*

An NFS export to be mounted (presumably containing the photos to use but this is not checked).
This will be mounted using a systemd mount unit that is bound to the the systemd unit that
runs feh
sequencing issues with the mount and the service startup.

Default: `none`  (no NFS mount will be performed)

#### `photo_frame_nfs_mount`=*path*

The mount point for the NFS volume.

Default: `none` (no NFS mount will be performed)

#### `photo_frame_nfs_options`=*string*

The options for the NFS mount.

Default: ro

#### `photo_frame_directory`=*path*

The directory containing the photos to show

Default: *`pnoto_frame_nfs_mount`* if *`pnoto_frame_nfs_source`* is defined,
otherwise `none` (which is not valid)

#### `photo_frame_user`=*string*

The name of the user running the desktop display

Default: user with UID 1000

#### `photo_frame_group`=*string*

The group for the user running the desktop display 

Default: primary group of the user with UID 1000

<!-- #### `photo_frame_scale`=none|shrink|full|crop

How to scale photos to fit the screen

The value of the `imv` scaling mode
([`-s` option](https://manpages.debian.org/trixie/imv/imv-wayland.1.en.html))

Default: crop
 -->
#### `photo_frame_scale`=optimal|width|height|fit|fill|real

How to scale photos to fit the screen.

The value of the [**scale**](https://manpages.debian.org/trixie/swayimg/swayimgrc.5.en.html#scale)
setting in the **viewer** sectiono f the `swayimg` configuration file (`/etc/xdg/swayimg/config`).

Default: optimal

<!-- #### `photo_frame_time`*integer**

The value of the `imv` slideshow duration
([`-t` option](https://manpages.debian.org/trixie/imv/imv-wayland.1.en.html))

Default: 30
 -->
#### `photo_frame_time`**float*

The number of seconds before the photo changes

The value of the
[**slideshow_time**](https://manpages.debian.org/trixie/swayimg/swayimgrc.5.en.html#slideshow_time)
 setting in the **viewer** section of the `swayimg` configuration file (`/etc/xdg/swayimg/config`).

Default: 30

#### `photo_frame_disable_nopasswd`=false|true

Disable passwordless `sudo` for the user displaying the photos by commenting out any lines in files
in `/etc/sudoers.d` that apply to that user and have `NOPASSWD:` in them.

By default, on RasPiOS, the default login user (*uid*=1000) is a member of the `sudo` group, which
has `sudo` access to all commands (with a password prompt) in `/etc/sudoers`, so this isn't
dangerous.

Default: true for Rasberry Pi systems, false otherwise

#### `photo_frame_disable_wf_panel_pi`=false|true

Disable `wf-panel-pi` by modifying the file `/etc/config/labwc/autostart` modifying the line
```
/usr/bin/lwrespawn /usr/bin/wf-panel-pi &
```
to 
```
echo Disabled: /usr/bin/lwrespawn /usr/bin/wf-panel-pi &
```
The mouse cursor is hidden by creating a key-binding to do this in `~/.config/labwc/rc.xml` and
simulating that key press in `~/.config/labwc/autostart`.

This has no effect if the target host is not a Raspberry Pi or the installed OS is RasPiOS
'Bookworm' or earlier.

Default: true