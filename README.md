# photo-frame

Uses Ansible to creates a Digital Photo Frame based on a Raspberry Pi running RasPiOS Trixie with
the Raspberry Pi Desktop, optionally using an NFS directory as a source of photos.

Typical invocation:
```
ansible-playbook -K photo_frame.yml
```

Note that, by design, the desktop of the Raspberry Pi is not usable once the photo frame service
is running and any tasks you need to perform to maintain it (notably patching) needs to be done
using SSH.

**photo-frame** should work on any model of Raspberry Pi that can be connected to a network.
It has been tested on a Raspberry Pi Zero W, using the on-board WiFi connection using an NFS
export to hold the photos.

Photo-frame will install and configure `sway` as a Wayland compositor if the target system does not
have a graphical installation (specifically a `systemd` definition of `graphical.target`).
The system will be rebooted to start `sway` on the `tty1` interface for the automatically logged
in user if `sway` is not already running when it is configured.

Rotation of the Wayland display to match the orientation of the physical display is supported
with `sway`; for any other Wayland compositor, this must be performed manually.

Photo-frame uses the `swayimg` package to display software.
The package should work on other Linux distribution other than RasPiOS Trixie, with the following
provisos.
- The distribution must have a Wayland-based graphical environment; X11 is not supported
- The distribution must be systemd-based
- The distribution must meet the requirements of the ansible installation on your control host
- The system should be configured to log a user into the graphical user environment automatically
- That user should not have passwordless `sudo` access
- The system should have the `sway*` packages available in a repository using a package manager
  supported by the Ansible built-in `package` module (if it does not, then it is necessary to
  install the package manually and add `--skip-tags package` to the `ansible-playbook`
  command)

See [the variables used by the photo_frame role](./roles/photo_frame/README.md#Variables) 

## Limitations



## Release Notes

### v0.2.0
- Optional feature to install sway Window manager.
- Add meta for parameter checking

### v0.1.0
Initial release


## Security Considerations

Because the system uses a logged-in graphical desktop anyone with physical access to the server to
insert a keyboard and/or mouse has access to the system.

Therefore, you should not store any data on the system (or any network devices that it has access
to).

Furthermore, passwordless `sudo` should be disabled for the login user.
This is done by default for Raspberry Pi's, where it is normally safe to remove any such entries
because the login user is also a member of the `sudo` group which has full password-authenticated
sudo access.


