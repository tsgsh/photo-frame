# Quick Guide to Setting up you Raspberry Pi(s) before running photo-frame

## Creating the Micro-SD card

The easiest way to create a card is to use the
[Raspberry Pi Imager](https://www.raspberrypi.com/software/) to write the card. 8GiB is sufficient
for the normal desktop version of RasPiOS Trixie (although it may not be for the future "Forky"
release).
For I/O performance, choose a Class 10 card (indicated by 'C10' on the card), but UHS
(e.g., 'U1') and Video (e.g. 'V30') Speed classes should not matter.

Place the SD card in a card reader and run the Imager executable.
The following choices are recommended when running the imager (these refer to v2.0)
1.  Select the correct Raspberry Pi device type
2.  Select either the first option ("Rasberry Pi OS (64-bit)") or, if you know you do not want to use
    the desktop environment, select "Raspberry Pi OS (Other)" then "Rasberry Pi OS Lite (64-bit)" 
3.  Select the SD card to write to
4.  Set the hostname to the name you have chosen for your Raspberry Pi
5.  Set the appropriate capital city, time zone and keyboard layout
6.  Add a user whose name matches the user you use on the control host (if you change this, you will
    have to include it in your **Ansible** configuration) and a password
7.  Add your Wi-Fi SSID and password; leave these blank if you are connecting with an Ethernet cable
8.  Enable SSH and add the SSH public key for your user on the control host; it will be in the file
    `~/.ssh/id_rsa.pub`; if you don't have one, use the `ssh-keygen` command.
    Without this step you will have to provide a password when you connect to the device with SSH
    and a password option (`-k <password>`) on each Ansible command.
    Using SSH is more secure than using passwords to log on.
9.  Optionally, enable the
   [Raspberry Pi Connect service](https://www.raspberrypi.com/documentation/services/connect.html)
10. Press "WRITE" to create the SD card.

After the card is written and verified
1.  Remvoe the card and install it in your Raspberry Pi's Micro SD card slot.
2.  You may want to attach a keyboard, mouse and monitor to it, depending on your approach to 
    finding an IP address (see below)
3.  Connect it to the Ethernet network if you are not using Wi-Fi
4.  Attach a USB-C (Pi 4 or 5) or Micro-USB-B power supply.
    Make sure you use a good quality supply (you can't go wrong with a Raspberry-Pi branded one).
    However, Pi Zero W's and Pi Zero 2 W's have
    [quite low power demands](https://raspberry.tips/en/raspberrypi-tutorials/raspberry-pi-power-consumption-update-2026-all-models-compared)
    and can usuanlly be powered by a USB port on a monitor.

## Finding and Retaining the IP Address of your Raspberry Pi

If you are using your ISP's router as a DHCP server then this section probably applies to you.  If
you are using IPv6 only, or running your own DHCP and/or DNS service, then I assume you know what
you are doing and this section doesn't apply.

### Finding the IP address

When you start the Raspberry Pi it uses DHCP to find an IP address from your router.  You (a) need
to know this address and (b) want it to stay the same.  There is a second kind of address, the MAC
address that you may also need, but this one is guaranteed to stay the same.  The MAC address is how
your router distinguishes its different DHCP clients.

The first way to find out the addresses is to log on to the device with an attached keyboard, mouse
and HDMI-monitor (e.g., the one you will use as a picture frame).
For a Raspberry Pi 4 or 5 or any Zero model you will need a specific
[HDMI connector](https://en.wikipedia.org/wiki/HDMI#Connectors) or adapter: micro-HDMI for the 4 or
5, or mini-HDMI for the Zero.

Open the Terninal application and issue the command `ip a` and the result will be something like
```
steve@photo1:~ $ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether b8:27:eb:a3:71:84 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.14/24 brd 192.168.1.255 scope global dynamic noprefixroute eth0
       valid_lft 1809sec preferred_lft 1809sec
    inet6 fe80::2c00:e730:96f1:8cc6/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
3: wlan0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether b8:27:eb:f6:24:d1 brd ff:ff:ff:ff:ff:ff
steve@photo1:~ $ 
```
Ignore the first (loopback) entry, the second or third one &ndash; whichever has `state UP` &ndash;
contains the two addresses of interest.  `eth0` is the Ethernet interface and `wlan0` is the Wi-Fi
interface (unsurpisingly).
- The MAC address is on the second line of the entry after `link\ether` and looks like
  `b8:27:eb:a3:71:84`
- The IP address is on the third line of the entry after `inet` and before the slash and looks like
  `192.168.1.14`

Another approach is to run the `nmap` tool from your **Ansible** control host (you may need to
install it first).  Find the IP address and subnet using the `ip a` command as above and this time
don't discard the bit that begins with a slash and use it as the argument for the command.  You
should also add the options `--open -p22` to limit the search to computers that support inbound
`ssh` connections (again, you enabled this in the Raspberry Pi imager).  The output looks like this:
```
steve@linux:~$ nmap --open -p22 192.168.1.0/24
Starting Nmap 7.92 ( https://nmap.org ) at 2026-06-26 16:54 BST
Nmap scan report for 192.168.1.14
Host is up (0.0018s latency).

Nmap done: 256 IP addresses (1 hosts up) scanned in 17.91 seconds

steve@linux:~$
```

Alternatively, your WiFi router should show a list of assigned IP addresses.
RasPiOS provides the hostname (which you set in the Raspberry Pi imager) on the DHCP request and
your router *may* show this.
Mobile wifi scanning tools may also be able to find it for you

### Saving the Hostname and IP address in `/etc/hosts` and `~/.ssh/known_hosts`

Now that we know the IP address, we need to add it to the `/etc/hosts` file: edit that as `root`,
```
root@linux:~# cat /etc/hosts
127.0.0.1      localhost
127.0.1.1      linux.example.com	linux

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
root@linux:~# vi /etc/hosts  # add rows as shown in the output below
root@linux:~# cat /etc/hosts
127.0.0.1      localhost
127.0.1.1      linux.example.com	linux

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

# Photo-frame entries
192.168.1.14   photo1
```
### Inital SSH login

Now we logon using the command `ssh photo1`.  This works because we installed our public SSH key
as part of the Raspberry Pi Imager process.  This puts an entry in the `~/.ssh/known_hosts` file,
which will be needed when we connect with **Ansible**.  Press `Ctrl-D` to logout again.
```
steve@linux:~$ ssh photo1
The authenticity of host 'photo1 (192.168.1.14)' can't be established.
ED25519 key fingerprint is: SHA256:RKWCB7fLon8UIxC1WwaMXINV6bOhnxxfCmjyRIE48ec
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'photo1' (ED25519) to the list of known hosts.
Linux photo1.example.com 6.18.34+rpt-rpi-v8 #1 SMP PREEMPT Debian 1:6.18.34-1+rpt1 (2026-06-09) aarch64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
steve@photo1:~ $ 
logout
Connection to photo1 closed.
steve@jabberwock:~$ 
```
### Setting a Fixed IP Address
The 'D' in 'DHCP' is for 'Dynamic' implying that IP addresses configured with DHCP can change.
Generally they don't and the DHCP client will ask for the same address and the DHCP protocol 
requires the server to remember addresses it has issued until it runs out of free adresses after
which it will start to recycle ones that haven't been used recently.  Typically a router has
253 addresses it can use.  So there are four possible approaches you can take.

**Note:** if you like, you can also do this for your **Ansible** control host but there is nothing
in **photo-frame** that requires you to do this.

1.  #### Don't do anything
    This is the simplest approach but the least reliable.
    If you have your Raspberry Pi on frequently or all of the time, its IP address should stay the
    same even if you have lots of devices that can connect to your WiFi, provided that your WiFi
    router obeys the DHCP specification.
    It may even survive a replacement of or a factory reset of your router.
    If it fails though, you will have to discover the new address and update `/etc/hosts` on your
    **Ansible** control host and your **photo-frame** devices.

2.  #### Permanently assign the current address in DHCP
    Your router may support assigning anny address as a fixed addresses in DHCP.
    In that case you can just configure the address it allocated initially to be permanently
    assigned.
    You will need to repeat this if you factory reset or change your router.

3.  #### Permanently assign a different address in DHCP
    Your router may support assigning fixed addresses in DHCP but may not allow this to be done with
    an address in the *pool* of dynamic addresses.
    You will need to assign a different address outside the DHCP address pool (and you may have to
    reconfigure the pool to free up some addresses outside it).

    In this case you will need to update `/etc/hosts` on your **Ansible** control host and any other
    **photo-frame** devices with the new address.
    Then power the Raspberry Pi off and on and it will be given the new fixed address.
4.  #### Statically assign a different address (no DHCP)
    Your router may not support assigning fixed addresses in DHCP.
    You can assign a static IP address and not use DHCP.
    In most cases, this should be a last resort.
    You will need to reconfigure the DHCP pool on the router to free up some space for static
    addresses, then you allocate those addresses to your **photo-frame** devices yourself.  

    For example, assume that the WiFi router was configured the address `192.168.1.1` and a DHCP
    pool of `192.168.1.2-254` that corresponds to 253 addresses and leaves 
    [no free addresses](https://en.wikipedia.org/wiki/IPv4#First_and_last_subnet_addresses).
    Reconfigure the DHCP pool to `192.168.1.2-224`, leaving 30 addresses that are not controlled by
    DHCP.
    Then allocate the first of these (`192.168.1.225`) to your first Raspberry Pi.
    ```
    steve@photo1:~ $ sudo -i
    root@photo1:~ $ nmcli conn show
    NAME                        UUID                                  TYPE      DEVICE 
    netplan-wlan0-my_wifi_ssid  01be8a50-b9b8-372d-8054-250836aa0b5d  wifi      wlan0  
    lo                          44c95f2b-3f72-4eca-8213-862841816d08  loopback  lo     
    netplan-eth0                75a1216a-9d1a-30cd-8aca-ace5526ec021  ethernet  --     
    root@photo1:~ $ nmcli conn show netplan-wlan0-my_wifi_ssid | grep -E 'IP4\.GATEWAY|IP4\.DNS'
    IP4.GATEWAY:                            192.168.2.1
    IP4.DNS[1]:                             9.9.9.9
    IP4.DNS[2]:                             8.8.8.8
    IP4.DNS[3]:                             1.1.1.1
    root@photo1:~ $ nmcli con mod netplan-wlan0-my_wifi_ssid ipv4.method manual ipv4.addr 192.168.1.200/24 ipv4.gateway 192.168.1.1 ipv4.dns 9.9.9.9,8.8.8.8,1.1.1.1 && systemctl restart NetworkManager
    ```
    The first command shows the network connections on the system.
    You need either the `eth0` or the `wlan0` device, depending on whether you are connecting with
    Ethernet or Wifi.
    If you are using `wifi`, the name of the connection will include your Wifi SSID. 

    The second command shows details of the interface, filtered to specific IP addressing information
    that you need for the next command: the "gateway" and the DNS servers.

    The final command converts the IP configuration to "manual" with the selected IP address, the
    gateway and DNS servers (which need to be separated by commas if there is more than one
    `IP4.DNS` line in the output of the second dommand).
    If, and only if, that completes successfully  (i.e., the operator `&&`) it restarts the
    `NetworkManager` service.
    
    If you have connected using SSH this will cause the session to hang because you have changed the
    IP address you are connecting to.  
    
    Edit `/etc/hosts` and update the IP address and then you will be able to connect using SSH again.

Your Raspberry Pi is now setup for control using **Ansible** and you can install and configure the
software you need to dsiplay photos.