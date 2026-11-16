# `photo_frame` role

Set up a digital photo frame using the **swayimg** image viewer running under **cage**.

Requires a target system without an active `systemd` graphical target (i.e. a text-mode console).

## Dependencies

- **ansible.posix** collection (if using NFS)
- **community.general** collection
- Python `jc` package

## Variables

#### `photo_frame_user` = *string*

The name of the passowrdless user that will be auto-logged on to run
`cage` and `swayimg`.

Default: photo-frame

#### `photo_frame_group` = *string*

The name of the group for the passowrdless user that will be auto-logged on to run
`cage` and `swayimg`.

Default: the value of `photo_frame_user`

#### `photo_frame_nfs_source` = *host*:*path*

An NFS export to be automounted (presumably containing the photos to use but this is not checked).

Default: `none`  (no NFS automount will be defined)

#### `photo_frame_nfs_mount` = *path*

The mount point for the NFS volume.

Default: `none` (no NFS automount will be defformed)

#### `photo_frame_nfs_options` = *string*

The options for the NFS mount.

Default: ro

#### `photo_frame_directory` = *path*

The directory containing the photos to show

Default: *`photo_frame_nfs_mount`* if *`photo_frame_nfs_source`* is defined,
otherwise `none` (which is not valid)

#### `photo_frame_scale` = optimal|width|height|fit|fill|real

How to scale photos to fit the screen.

The value of the [**scale**](https://manpages.debian.org/trixie/swayimg/swayimgrc.5.en.html#scale)
setting in the **viewer** sectiono f the `swayimg` configuration file (`/etc/xdg/swayimg/config`).

Default: optimal

#### `photo_frame_time` = *float*

The number of seconds before the photo changes

The value of the
[**slideshow_time**](https://manpages.debian.org/trixie/swayimg/swayimgrc.5.en.html#slideshow_time)
 setting in the **viewer** section of the `swayimg` configuration file (`/etc/xdg/swayimg/config`).

Default: 30

#### `photo_frame_restart_limit` = *int*
How many times the photo frame can repeatedly restart before it exits to the shell

Default: 5

#### `photo_frame_restart_window` = *int*
The time window during which consecutive restarts are regarded as "repeated".

Default: 300
