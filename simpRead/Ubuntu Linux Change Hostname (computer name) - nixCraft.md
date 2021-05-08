> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cyberciti.biz](https://www.cyberciti.biz/faq/ubuntu-change-hostname-command/)

[![](https://www.cyberciti.biz/media/new/category/old/ubuntu-logo.jpg)](https://www.cyberciti.biz/faq/category/ubuntu-linux/ "See all Ubuntu Linux related FAQ")

I am a new Ubuntu Linux laptop user. I setup my computer name to ‘tom’ during installation but now I would like to change the computer name to ‘jerry’. Can you tell me how do I remove tom and set it to jerry on Ubuntu Linux? How do I change the Ubuntu computer name from ‘ubuntu’ to ‘AvlinStar’? Can you tell me more about Ubuntu Linux change hostname command?  
  
You can use the hostname command or [nixmd name=”hostnamectl”] to see or set the system’s host name. The host name or computer name is usually at system startup in /etc/hostname file. Open the terminal application and type the following commands to set or change hostname or computer name on Ubuntu Linux.

Ubuntu change hostname command
------------------------------

The procedure to change the computer name on Ubuntu Linux:

1.  Type the following command to edit /etc/hostname using nano or vi text editor:  
    **sudo nano /etc/hostname**  
    Delete the old name and setup new name.
2.  Next Edit the /etc/hosts file:  
    **sudo nano /etc/hosts**  
    Replace any occurrence of the existing computer name with your new one.
3.  Reboot the system to changes take effect:  
    **sudo reboot**

Sample outputs:  

![](https://www.cyberciti.biz/media/new/faq/2016/01/ubuntu-change-computer-name-demo-copy.gif)

Gif 01: Ubuntu change the computer name demo

Display the current Ubuntu hostname
-----------------------------------

Simply type the following command:  
`$ hostname`  
Sample outputs:  

![](https://www.cyberciti.biz/media/new/faq/2016/01/ubuntu-show-hostname.jpg)

Fig.01: Ubuntu Linux Show the hostname/computer name command

How to change the Ubuntu server hostname without a system restart?
------------------------------------------------------------------

Type the following commands:  
`$ sudo hostname new-server-name-here`  
Next edit the /etc/hostname file and update hostname:  
`$ sudo nano /etc/hostname`  
Finally, edit the /etc/hosts file and update the lines that reads your old-host-name:  
`$ sudo nano /etc/hosts`  
From:  
`127.0.1.1 old-host-name`  
To:  
`127.0.1.1 new-server-name-here`  
Save and close the file.

Ubuntu Linux Change Hostname Using hostnamectl
----------------------------------------------

Systemd based Linux distro such as Ubuntu Linux 16.04 LTS and above can simply use the hostnamectl command to change hostname. To see current setting just type the following command:  
`$ hostnamectl`  
Sample outputs:

```
Static hostname: nixcraft
         Icon name: computer-laptop
           Chassis: laptop
        Machine ID: 291893e6499e4d99891c3cf4b70a138b
           Boot ID: 9fda2365b77841649e40a141fde46537
  Operating System: Ubuntu 17.10
            Kernel: Linux 4.13.0-21-generic
      Architecture: x86-64
```

To change hostname from nixcraft to viveks-laptop, enter:  
`$ hostnamectl set-hostname viveks-laptop  
$ hostnamectl`

Conclusion
----------

In this tutorial, you learned how to change hostname on Ubuntu Linux. For more information see [this page here](https://manpages.ubuntu.com/manpages/focal/en/man1/hostname.1.html).  

<table><thead><tr><th>Category</th><th>List of Unix and Linux commands</th></tr></thead><tbody><tr><td><abbr title="DS (Disk space analyzers) command category">Disk space analyzers</abbr></td><td><a title="ncdu Linux/Unix examples and syntax" href="https://www.cyberciti.biz/open-source/install-ncdu-on-linux-unix-ncurses-disk-usage/">ncdu</a> • <a title="pydf Unix/Linux examples and syntax" href="https://www.cyberciti.biz/tips/unix-linux-bsd-pydf-command-in-colours.html">pydf</a></td></tr><tr><td><abbr title="FS (File Management) command category">File Management</abbr></td><td><a title="cat command examples and syntax" href="https://www.cyberciti.biz/faq/linux-unix-appleosx-bsd-cat-command-examples/">cat</a></td></tr><tr><td><abbr title="FW (Firewall) command category">Firewall</abbr></td><td><a title="Awall firewall on Alpine Linux examples and syntax" href="https://www.cyberciti.biz/faq/how-to-set-up-a-firewall-with-awall-on-alpine-linux/">Alpine Awall</a> • <a title="firewalld on CentOS 8 examples and syntax" href="https://www.cyberciti.biz/faq/how-to-set-up-a-firewall-using-firewalld-on-centos-8/">CentOS 8</a> • <a title="firewalld on OpenSUSE Linux examples and syntax" href="https://www.cyberciti.biz/faq/set-up-a-firewall-using-firewalld-on-opensuse-linux/">OpenSUSE</a> • <a title="firewalld on RHEL (Red Hat Enterprise Linux) 8 examples and syntax" href="https://www.cyberciti.biz/faq/configure-set-up-a-firewall-using-firewalld-on-rhel-8/">RHEL 8 </a>• <a title="ufw on Ubuntu 16.04 LTS examples and syntax" href="https://www.cyberciti.biz/faq/howto-configure-setup-firewall-with-ufw-on-ubuntu-linux/">Ubuntu 16.04</a> • <a title="ufw on Ubuntu 18.04 LTS examples and syntax" href="https://www.cyberciti.biz/faq/how-to-setup-a-ufw-firewall-on-ubuntu-18-04-lts-server/">Ubuntu 18.04</a> • <a title="ufw on Ubuntu 20.04 LTS examples and syntax" href="https://www.cyberciti.biz/faq/how-to-configure-firewall-with-ufw-on-ubuntu-20-04-lts/">Ubuntu 20.04</a></td></tr><tr><td><abbr title="NU (Network Utilities) command category">Network Utilities</abbr></td><td><a title="NetHogs - Monitor per process network bandwidth usage under Linux examples and syntax" href="https://www.cyberciti.biz/faq/linux-find-out-what-process-is-using-bandwidth/">NetHogs</a> • <a title="dig command examples and syntax" href="https://www.cyberciti.biz/faq/linux-unix-dig-command-examples-usage-syntax/">dig</a> • <a title="host command examples and syntax" href="https://www.cyberciti.biz/faq/linux-unix-host-command-examples-usage-syntax/">host</a> • <a title="ip command in Linux examples and syntax" href="https://www.cyberciti.biz/faq/linux-ip-command-examples-usage-syntax/">ip</a> • <a title="nmap command in Linux examples and syntax" href="https://www.cyberciti.biz/security/nmap-command-examples-tutorials/">nmap</a></td></tr><tr><td><abbr title="OVPN (OpenVPN) command category">OpenVPN</abbr></td><td><a title="CentOS 7 Set Up OpenVPN Server In 5 Minutes examples and syntax" href="https://www.cyberciti.biz/faq/centos-7-0-set-up-openvpn-server-in-5-minutes/">CentOS 7</a> • <a title="CentOS 8 OpenVPN server in 5 mintues examples and syntax" href="https://www.cyberciti.biz/faq/centos-8-set-up-openvpn-server-in-5-minutes/">CentOS 8</a> • <a title="Debian 10 Set Up OpenVPN Server In 5 Minutes examples and syntax" href="https://www.cyberciti.biz/faq/debian-10-set-up-openvpn-server-in-5-minutes/">Debian 10</a> • <a title="OpenVPN server on Debian 9/8 examples and syntax" href="https://www.cyberciti.biz/faq/install-configure-openvpn-server-on-debian-9-linux/">Debian 8/9</a> • <a title="Ubuntu 18.04 LTS Set Up OpenVPN Server In 5 Minutes examples and syntax" href="https://www.cyberciti.biz/faq/ubuntu-18-04-lts-set-up-openvpn-server-in-5-minutes/">Ubuntu 18.04</a> • <a title="Ubuntu 20.04 LTS OpenVPN server in 5 mintues examples and syntax" href="https://www.cyberciti.biz/faq/ubuntu-20-04-lts-set-up-openvpn-server-in-5-minutes/">Ubuntu 20.04</a></td></tr><tr><td><abbr title="PM (Package Manager) command category">Package Manager</abbr></td><td><a title="apk command in Apline Linux examples and syntax" href="https://www.cyberciti.biz/faq/10-alpine-linux-apk-command-examples/">apk</a> • <a title="apt command in Debian/Ubuntu Linux examples and syntax" href="https://www.cyberciti.biz/faq/ubuntu-lts-debian-linux-apt-command-examples/">apt</a></td></tr><tr><td><abbr title="PRM (Processes Management) command category">Processes Management</abbr></td><td><a title="bg command examples and syntax" href="https://www.cyberciti.biz/faq/unix-linux-bg-command-examples-usage-syntax/">bg</a> • <a title="chroot command examples and syntax" href="https://www.cyberciti.biz/faq/unix-linux-chroot-command-examples-usage-syntax/">chroot</a> • <a title="cron and crontab command examples and syntax" href="https://www.cyberciti.biz/faq/how-do-i-add-jobs-to-cron-under-linux-or-unix-oses/">cron</a> • <a title="disown command examples and syntax" href="https://www.cyberciti.biz/faq/unix-linux-disown-command-examples-usage-syntax/">disown</a> • <a title="fg command examples and syntax" href="https://www.cyberciti.biz/faq/unix-linux-fg-command-examples-usage-syntax/">fg</a> • <a title="jobs command examples and syntax" href="https://www.cyberciti.biz/faq/unix-linux-jobs-command-examples-usage-syntax/">jobs</a> • <a title="killall command examples and syntax" href="https://www.cyberciti.biz/faq/unix-linux-killall-command-examples-usage-syntax/">killall</a> • <a title="kill command examples and syntax" href="https://www.cyberciti.biz/faq/unix-kill-command-examples/">kill</a> • <a title="pidof command examples and syntax" href="https://www.cyberciti.biz/faq/linux-pidof-command-examples-find-pid-of-program/">pidof</a> • <a title="pstree command examples and syntax" href="https://www.cyberciti.biz/faq/unix-linux-pstree-command-examples-shows-running-processestree/">pstree</a> • <a title="pwdx command examples and syntax" href="https://www.cyberciti.biz/faq/unix-linux-pwdx-command-examples-usage-syntax/">pwdx</a> • <a title="time command examples and syntax" href="https://www.cyberciti.biz/faq/unix-linux-time-command-examples-usage-syntax/">time</a></td></tr><tr><td><abbr title="SER (Searching) command category">Searching</abbr></td><td><a title="grep command examples and syntax" href="https://www.cyberciti.biz/faq/howto-use-grep-command-in-linux-unix/">grep</a> • <a title="whereis command examples and syntax" href="https://www.cyberciti.biz/faq/unix-linux-whereis-command-examples-to-locate-binary/">whereis</a> • <a title="which command examples and syntax" href="https://www.cyberciti.biz/faq/unix-linux-which-command-examples-syntax-to-locate-programs/">which</a></td></tr><tr><td><abbr title="UI (User Information) command category">User Information</abbr></td><td><a title="groups command examples and syntax" href="https://www.cyberciti.biz/faq/unix-linux-groups-command-examples-syntax-usage/">groups</a> • <a title="id command examples and syntax" href="https://www.cyberciti.biz/faq/unix-linux-id-command-examples-usage-syntax/">id</a> • <a title="lastcomm command examples and syntax" href="https://www.cyberciti.biz/faq/linux-unix-lastcomm-command-examples-usage-syntax/">lastcomm</a> • <a title="last command examples and syntax" href="https://www.cyberciti.biz/faq/linux-unix-last-command-examples/">last</a> • <a title="lid/libuser-lid command examples and syntax" href="https://www.cyberciti.biz/faq/linux-lid-command-examples-syntax-usage/">lid/libuser-lid</a> • <a title="longname command examples and syntax" href="https://www.cyberciti.biz/faq/unix-linux-logname-command-examples-syntax-usage/">logname</a> • <a title="members command examples and syntax" href="https://www.cyberciti.biz/faq/linux-members-command-examples-usage-syntax/">members</a> • <a title="users command examples and syntax" href="https://www.cyberciti.biz/faq/unix-linux-users-command-examples-syntax-usage/">users</a> • <a title="whoami command examples and syntax" href="https://www.cyberciti.biz/faq/unix-linux-whoami-command-examples-syntax-usage/">whoami</a> • <a title="who command examples and syntax" href="https://www.cyberciti.biz/faq/unix-linux-w-command-examples-syntax-usage-2/">who</a> • <a title="w command examples and syntax" href="https://www.cyberciti.biz/faq/unix-linux-w-command-examples-syntax-usage-2/">w</a></td></tr><tr><td><abbr title="WG (WireGuard VPN) command category">WireGuard VPN</abbr></td><td><a title="Alpine Linux set up WireGuard VPN server examples and syntax" href="https://www.cyberciti.biz/faq/how-to-set-up-wireguard-vpn-server-on-alpine-linux/">Alpine</a> • <a title="CentOS 8 set up WireGuard VPN server examples and syntax" href="https://www.cyberciti.biz/faq/centos-8-set-up-wireguard-vpn-server/">CentOS 8</a> • <a title="Debian 10 set up WireGuard VPN server examples and syntax" href="https://www.cyberciti.biz/faq/debian-10-set-up-wireguard-vpn-server/">Debian 10</a> • <a title="WireGuard Firewall Rules in Linux examples and syntax" href="https://www.cyberciti.biz/faq/how-to-set-up-wireguard-firewall-rules-in-linux/">Firewall</a> • <a title="Ubuntu 20.04 set up WireGuard VPN server examples and syntax" href="https://www.cyberciti.biz/faq/ubuntu-20-04-set-up-wireguard-vpn-server/">Ubuntu 20.04</a></td></tr></tbody></table>