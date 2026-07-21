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

Photo-frame uses the `swayimg` package to display software.
The package should work on other Linux distribution other than RasPiOS Trixie, with the following
provisos.
- The distribution must have a Wayland-based graphical environment; X11 is not supported
- The distribution must be systemd-based
- The distribution must meet the requirements of the ansible installation on your control host
- The system should be configured to log a user into the graphical user environment automatically
- That user should not have passwordless `sudo` access
- The system should have the `swayimg` package available in a repository using a package manager
  supported by the Ansible built-in `package` module (if it does not, then it is necessary to
  install the package manually and add `--skip-tags package` to the `ansible-playbook`
  command)


See [the variables used by the photo_frame role](./roles/photo_frame/README.md#Variables) 

## Security Considerations

Because the system uses a logged-in graphical desktop anyone with physical access to the server to
insert a keyboard and/or mouse has access to the system.

Therefore, you should not store any data on the system (or any network devices that it has access
to).

Furthermore, passwordless `sudo` should be disabled for the login user.
This is done by default for Raspberry Pi's, where it is normally safe to remove any such entries
because the login user is also a member of the `sudo` group which has full password-authenticated
sudo access.
If you have a different 



## Pre-requisites

At a minimum you need fixed (or, at least, known) IP addresses for all devices for the installation
of **aardsound** and fixed addresses for any Snapcast servers.
If you have that, but no DNS service, then it is probably a good idea to set up `/etc/hosts` entries
on each device, including the Ansible control node, to map each IP address to a hostname.

### Ansible Control Node

You need a computer that can serve the Ansible control node, which includes Linux and macOS and
Windows provided that a Windows Subsystem for Linux is installed.
See the [Ansible Installation Guide](https://docs.ansible.com/projects/ansible/latest/installation_guide/intro_installation.html#installation-guide).
The control node could, in theory, be one of the target systems, but that has not been tested.

On the Ansible control node you need to install a
[current version of Ansible](https://docs.ansible.com/projects/ansible/latest/reference_appendices/release_and_maintenance.html).
At the time of writing, that was Ansible 13 (based on **ansible-core** 2.20).
Older versions are not supported but will probably work back to **ansible-core** 2.14
(Ansible 7).
Alternatively, if your Linux distribution has a packaged **ansible-core** version of 2.14 or higher,
that may work, but you will need supported versions of the following Ansible collections to be
installed:
- ansible.posix
- ansible.utils
- community.general

If the **ansible-core** version is 2.18 or earlier, native Jinja2 templating operations must be set
(using the environment variable `ANSIBLE_JINJA2_NATIVE=true` or the Ansible configuration option
`DEFAULT_JINJA2_NATIVE=true`).

In addtion, you will need an Ansible inventory file to define the target systems and the
inventory variables that will define what is to be deployed to to each target.
See the [**Inventory**](#Ansible-Inventory) section below.

### Target systems

Aardsound is intended to run on a Linux "control host" against one or more newly
installed Raspberry Pis booting from Micro-SD cards, each with a known IP address that is
resolvable to a hostname via DNS or `/etc/hosts`.
MacOS may work as a control host operating system, but this has not been tested.
Your id on your Ansible control host must be authorised to connect to each target system using
SSH and to run the `sudo` command with **no** limitations on which privlieged commands it can run
(this is an Ansible [requirement](
https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_privilege_escalation.html#privilege-escalation-must-be-general)).

If your userid on the Ansible control host is not the same as the userid on the target servers
you will need to set the Ansible variable `ansible_user` to the target userid.
This can be done by:
- Setting `ansible_user` as an inventory variable (the most straightforward way)
- Setting the environment variable [`ANSIBLE_REMOTE_USER`](
  https://docs.ansible.com/projects/ansible/latest/reference_appendices/config.html#default-remote-user)
- Setting `remote_user` in the `[defaults]` section of the Ansible [configuration file](
  https://docs.ansible.com/projects/ansible/latest/reference_appendices/config.html#generating-a-sample-ansible-cfg-file)
- Adding the variable to each `ansible` or `ansible-playbook` command with
  `-e ansible_user=<my_user>`

If your user does not have an SSH authorized_key on the target, then fix this!
Failing that, if password login is allowed over SSH, add the option `-k` to the
`ansible-playbook` command.

You will need to use the `-K` option on the `ansible-playbook` command if:
- the user does not have the privileges to run the `sudo` command on the target host without
  entering a password; or
- the user has the privileges to run the `sudo` command on the target host without
  entering a password but:
  - the target host is a Raspberry Pi;
  - the Ansible variable `photo_frame_disable_nopasswd` is true (the default); and
  - the Ansible tag `nopasswd` is not being skipped

Alternative;y, you can create a separate user on the target, such as `ansible`, with passwordless
`sudo` and set the Ansible variable `ansible_user` to that as described above.
Details on how to do this are outside the scope of this guide.

For a quick guide to setting up your Raspberry Pi(s) like this, see the
[Quick Guide to Setting up you Raspberry Pi(s) for Aardsound](../PiSetup.md).

## Ansible Inventory
There are many ways to build an [Ansible inventory](
https://docs.ansible.com/projects/ansible/latest/inventory_guide/intro_inventory.html).
If you already use Ansible, then add the *photo-frame* "hosts" (Ansible refers to a target server as
a *host*), and at least the `photo_frame` group to match your current inventory.
If you are new to Ansible, then the inventory approach described here is a simple one but it isn't
suitable for a complex Ansible environment (which is not an issue for *photo-frame*).

### A basic YAML inventory
Although Ansible supports many inventory formats, we will use one of the built-in ones: YAML.
This should be saved in the file `/etc/ansible/hosts`, although if you already have an Ansible
iventory that you don't want to or can't modify, you can save the inventory somewhere else and
pass the path to the file to the `ansible` or `ansible-playbook` command with the `-i` or
`--inventory` option.

A very simple inventory, for one host that uses NFS as a data source, might be:
```YAML
photo_frame:
  hosts:
    photo1:
  vars:
    photo_frame_nfs_source: 'my-nas:/photos'
    ansible_timeout: 30
```
The `ansible-timeout` is recommended for the Raspberry Pi Zero/Zero W and for the original
Model A/A+ and Model B/B+ boards: when `swayimg` is running, the default timeout value of
12 is sometimes too low.
It is not required for other models of Raspberry Pi, which have quad-core processors.

An inventory for a host that stores photos locally in the login user's `Pictures` directory might
be:
```YAML
photo_frame:
  hosts:
    photo1:
  vars:
    photo_frame_directory: /home/steve/Pictures
    ansible_timeout: 30
```

An inventory for two hosts that use different sub-directories on the NAS server might
be:
```YAML
photo_frame:
  hosts:
    photo1:
      photo_frame_directory: /mnt/photos/landscape
    photo1:
      photo_frame_directory: /mnt/photos/portrait
  vars:
    photo_frame_nfs_source: 'my-nas:/photos'
    ansible_timeout: 30
```
