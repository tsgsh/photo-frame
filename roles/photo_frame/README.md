# `photo_frame` role

Set up a digital photo frame using the **swayimg** image vieweer.
Creates a systemd service to view all files in a specified directory in fullscreen mode.

Can use the **sway** Wayland compositor or another Wayland-based graphical desktop.
**swayimg** is optimised for **sway** (the clue being in the name) and **sway** is recommended for
low-end Raspberry Pi's (Pi Zero, Zero W, Zero WH and the original Models A and B) if your photos
directory has hundreds of photos, particularly over NFS.

If the host is a Raspberry Pi host running the `labwc` compositor, the `wf-panel-pi` panel will be
disabled by default (making the desktop largely useless unless the panel is restarted manually).
This is because `wf-panel-pi` currently doesn't support auto-hide for the taksk bar, which, 
therefore, covers part of the screen.

## Dependencies

- **community.general** collection
- **Wayland** on the target host

## Variables

#### `photo_frame_sway` = true | false
Use the **sway** Wayland compositor.

This can only be done on a distribution without a grapical installation (specifically where the
`systemd` `graphical.target` has not been defined).

The system will be automatically rebooted to start **sway** for the "tyy1" user if it is not already
running.

This will install **sway** if required and enable it for the user defined by `photo_frame_user`.
**sway** will run at the `systemd` `multi-user.target`: no `graphical.target` will be created.

Default: `true` if `graphical.target` does not exist in `systemd` otherwise `false`

#### `photo_frame_sway_output` = *string*
The physical video output to for `photo_frame_display`

Default: HDMI-A-1, ignored if `photo_frame_sway` is false

#### `photo_frame_sway_transform` = normal | '90' | '180' | '270' | flipped | flipped-90 | flipped-180 | flipped-270

The transformation to be applied to the the sway video output.

Normally, this would be used for a portrait display, specifying either 90&deg; or 270&deg; rotation.
The angle is measured in a clockwise direction, so for a physical 90&deg; clockwise rotation of the
display, the value should be set to `'270'` to compensate

The values 90, 180 and 270 must be expressed as strings not as integers.

Default: normal, ignored if `photo_frame_sway` is false

#### `photo_frame_nfs_source` = *host*:*path*

An NFS export to be mounted (presumably containing the photos to use but this is not checked).
This will be mounted using a `systemd` mount unit that is bound to the the systemd unit that runs
`swayimg` to avoid sequencing issues with the mount and the service startup.

Default: `none`  (no NFS mount will be performed)

#### `photo_frame_nfs_mount` = *path*
The mount point for the NFS volume.

Default: `none` (no NFS mount will be performed)

#### `photo_frame_nfs_optopns` = *string*
The options for the NFS mount.

Default: ro

#### `photo_frame_directory` = *path*
The directory containing the photos to show

Default: *`pnoto_frame_nfs_mount`* if *`pnoto_frame_nfs_source`* is defined,
otherwise `none` (which is not valid)

#### `photo_frame_scale` = optimal|width|height|fit|fill|real
How to scale photos to fit the screen.

The value of the [**scale**](https://manpages.debian.org/trixie/swayimg/swayimgrc.5.en.html#scale)
setting in the **viewer** sectionof the `swayimg` configuration file (`/etc/xdg/swayimg/config`).

Default: optimal

#### `photo_frame_time` = *float*
The number of seconds before the photo changes

The value of the
[**slideshow_time**](https://manpages.debian.org/trixie/swayimg/swayimgrc.5.en.html#slideshow_time)
 setting in the **viewer** section of the `swayimg` configuration file (`/etc/xdg/swayimg/config`).

Default: 30.0

#### `photo_frame_user` = *string*
The name of the user running the desktop display

Default: user with UID 1000

#### `photo_frame_group` = *string*
The group for the user running the desktop display 

Default: primary group of the user with UID 1000

#### `photo_frame_display` = *string*
The name of the Wayland display to use.

Default: wayland-0

#### `photo_frame_disable_nopasswd` = false|true
Disable passwordless `sudo` for the user displaying the photos by commenting out any lines in files
in `/etc/sudoers.d` that apply to that user and have `NOPASSWD:` in them.

By default, on RasPiOS, the default login user (*uid* = 1000) is a member of the `sudo` group, which
has `sudo` access to all commands (with a password prompt) in `/etc/sudoers`, so this isn't
dangerous.

Default: true for Rasberry Pi systems, false otherwise

#### `photo_frame_disable_wf_panel_pi` = false|tru
Disable `wf-panel-pi` if using the `labwc` Wayland compositor (RasPiOS).

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

This has no effect if the target host is not running RasPiOS or the installed OS is RasPiOS
'Bookworm' or earlier.

Default: true