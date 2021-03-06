---
file: /OpenSim-Grid-Docs/HAP-OpenSim-Region-Server-Build-Config-All.md
author: Alisha Awen, siren1@disroot.org
tags: 2021, drafts, unpublished, not-in-ed-cal, HAP-how-to, tutorials, video-games, VR, OpenSim, NeverWorld-grid, SysAdmin, droplet, VPS, hosting
created: 2020-010-31
updated: 2020-011-01
editorial calendar: No EdCal Yet
published: Not Publised
---

<!-- #2021 #drafts #unpublished #not-in-ed-cal #HAP-how-to #tutorials #video-games #VR #OpenSim #NeverWorld-grid #SysAdmin #droplet #VPS #hosting -->


# HAP OpenSim Region Server Build Config Guide

**[\[Table of Contents\]](#table-of-contents)**

## Introduction:

**Happy Halloween 2020!**  This is a first-draft of a complete tutorial _(from start to finish)_ about **_"How to self host an OpenSim region server connected to the NeverWorld grid"_**, running on a [DigitalOcean VPS droplet](https://m.do.co/c/449bba2e8a2d)... All sub tasks included, and hopefully written for folks with no prior DevOps experience to follow and successfully self-host their own regions in the same fashion for not too much money per month!  

This configuration will provide a much better in-world experience for Avitars visiting your region than any setup hosted from a residential home computer/network... Starting out with the minimal resources _(mostly RAM because bandwidth and disk space on smaller VPS droplets is more than sufficient)_ to see how performance goes for a small region starting out... 

Once this proof-of-concept is proven and working well, we will expand to a larger VPS droplet to see how performance compares with dedicated server hardware _(also running within a fast data center)_ 

From a system architecture point of view, whether the system used is virtual or bare metal hardware... performance will be the same at the end of the day given both systems are configured with similar resources/build specs...  Performace and scale are dependent on resources allocated... A powerful VPS should perform equally well with a single dedicated co-located hardware machine configured to the same requirements...

My experience with Virtual Computers goes back to IBM S390 days where I did DevOps on many virtual AIX machines all hosted within a single but super powerful IBM S390 multiprocessor/hypervisor server machine.  One of those machines alone ran all of General Motors Business operations back in the day!

:link: Link to: **[My OpenSim Journey - Project](https://github.com/users/harmonicalchemy/projects/2)**

:recycle: Last updated: `2020-011-01`

_____________________________________________


## Create new VPS droplet on DigitalOcean

### Step 01 - New VPS Creation:

These are my new revised for 2020 steps to spin up, and provision a **[DigitalOcean](https://m.do.co/c/449bba2e8a2d)** droplet VPS running the latest Ubuntu 20.04 LTS...

For this new OpenSim server I will also try a smaller package to see if it can still manage a 6x6 block of regions. i.e., 1536x1536x512 space on the map.

For this the following package will be created:

- **OS:** `Ubuntu 20.04 LTS`
- **Plan:** `basic`
- **Tier:** `$20/mo $0.030/hour`
- **Memory:** `4GB / 2 CPUs`
- **HD:** `80 GB`
- **Transfer:** `4TB`
- **Data Center:** `SF03`
- **IPv6** `YES`
- **Monitoring** `YES`
- **SSH keys:** `YES` _(use your existing id_ed25519 key)_
- **Host Name:** `vps1.example.com`


Call the new server My-VPS1@My-Domain.com or something like that...

### Step 02 - SSH Log In as root First Time:

Upon First Spinning up of a VPS perform the following steps:

Set up new HOST Records in your local `~/.ssh/config` file for this new SSH connection first... Create host record for `root` _(to use one time only)_ and another host record for `magos` the sudo system admin user... Both users use the same SSH key (ED25519).

Once SSH is configured locally, perform the first time log-in tasks as root user...

```yaml

$> ssh root@My-VPS.com       # root@vps1.example.com

Host key fingerprint is SHA256:
xoz4yKfKNICrdX8qamV205kENouJBWmLkR6tmXkz1yt
+--[ED25519 256]--+
|  ..=o=.         |
| + = . o         |
|o + . . o        |
|.+ B o o = .     |
|+.B E + S o      |
|o+ . * o         |
|. o + o o        |
|   = * *         |
|  .o=.O.         |
+----[SHA256]-----+
Welcome to Ubuntu 20.04.1 LTS (GNU/Linux 5.4.0-45-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Tue Oct 20 17:25:01 UTC 2020

  System load:           0.0
  Usage of /:            1.8% of 77.36GB
  Memory usage:          5%
  Swap usage:            0%
  Processes:             120
  Users logged in:       0
  IPv4 address for eth0: 143.110.159.141
  IPv4 address for eth0: 10.48.0.6
  IPv6 address for eth0: 2604:a880:4:1d0::63:6000
  IPv4 address for eth1: 10.124.0.3

73 updates can be installed immediately.
36 of these updates are security updates.
To see these additional updates run: apt list --upgradable

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

root@vps1:~#
```

**[\[Table of Contents\]](#table-of-contents)**

### Step 03 - Perform Initial Tasks as Root User:

```yaml
root@vps1:~#  apt update
root@vps1:~#  apt dist-upgrade

## Install mosh (mobile shell):

root@vps1:~#  apt install mosh

## Set up initial Firewall Rules:

root@vps1:~#  ufw status
Status: inactive

root@vps1:~#  ufw allow 22/tcp   # Enable normal TCP
Rules updated
Rules updated (v6)

root@vps1:~#  ufw allow 80/tcp   # Enable normal WEB
Rules updated
Rules updated (v6)

root@vps1:~#  ufw allow 443/tcp  # Enable TLS WEB (HTTPS)
Rules updated
Rules updated (v6)

root@vps1:~#  ufw allow 68/udp   # Enable normal UDP
Rules updated
Rules updated (v6)

## NOW Turn Firewall ON - And check Status:

root@vps1:~#  ufw enable
root@vps1:~#  ufw status
Status: active

To                         Action      From 
--                         ------      ---- 
Anywhere                   ALLOW       1.2.3.4 
22/tcp                     ALLOW       Anywhere 
80/tcp                     ALLOW       Anywhere 
443/tcp                    ALLOW       Anywhere
68/udp                     ALLOW       Anywhere
22/tcp (v6)                ALLOW       Anywhere (v6)
80/tcp (v6)                ALLOW       Anywhere (v6)
443/tcp (v6)               ALLOW       Anywhere (v6)
68/udp (v6)                ALLOW       Anywhere (v6)

root@vps1:~#

## Enable Firewall Ports for mosh (mobile Shell)

root@vps1:~#  ufw allow 60000:60010/udp
Rule added
Rule added (v6)
```

### Step 04 - Do Sanity Check of Installed Packages:

```yaml

## Check for Mosh Installed first:

root@vps1:~# dpkg -l mosh

####          List ALL installed packages:
##
## Make your terminal Screen W I D E display for listing)
## This command pipes to the more utility which allows
## you to search through the list using "/" character...
##
####

root@vps1:~# dpkg -l "*" | more
```

**[\[Table of Contents\]](#table-of-contents)**

## Provision your new VPS droplet

Now that you got your server up.. What next?

### Step 05 - Reboot the Server and Log back in with Mosh:

```yaml
root@vps1:~# reboot now

# Server goes offline and you are returned to local shell:

$> 

## Log back as root user using mosh now:

$> mosh root@vps1.example.com
root@vps1:~# whoami
root
root@vps1:~# 
```

### Step 06 - Perform First Step VPS Security Provisions:

#### Create sysAdmin sudo User:

Open your local KeePassXC password vault and create a new password entry record to hold your new sysAdmin sudo user's password and credentials...  Once you have all that saved use that info _(with strong generated password)_ to create your new user below...

> **Note:** Type the command showing at the first prompt at the top of the following code screen example    
i.e., `adduser <your-chosen-username>`     
The server's response to that will be an interactive session... Answer all questions as printed below in this example...  Change generic username "alice" to your chosen user of course. %^)


```yaml
root@vps1:~#  adduser alice
Adding user `alice' ...
Adding new group `alice' (1000) ...
Adding new user `alice' (1000) with group `alice' ...
Creating home directory `/home/alice' ...
Copying files from `/etc/skel' ...
New password:        # (paste password from KeePassXC entry)
Retype new password: # (paste password a second time to confirm)
passwd: password updated successfully
Changing the user information for alice
Enter the new value, or press ENTER for the default
        Full Name []: sys$Admin
        Room Number []: 
        Work Phone []: 
        Home Phone []: 
        Other []: System Admin sudo user for this VPS    
Is the information correct? [Y/n] y
root@vps1:~# 
```

Once the above is done, you do not need to enter any further information. Note: Room Number, Work Phone, Home Phone are unneccessary, (unless you are System Admin of a university or big company that is. %^) I simply put sys$Admin as the full name. That is a personal mark of mine, (an echo from past VAX/VMS and unix perl scripting days ;-) that lets me easily tell, after a year or so of many server set ups, that I did in fact create that system user. The obscure username also helps me remember that I was the one who created the account. In the next step we will grant “alice” root capabilities by proxy.

#### Grant sysAdmin User Root Privileges _(sudo)_:

Now we are ready to grant our sys admin user rootly superpower! Unix provides a means to temporarily grant rootly superpower through the ‘sudo’ (Super-USER-DO) or “mother may I be root?” command. The word sudo is pronounced like we pronounce the word “Pseudo”, and our sudo is actually more phonetically correct.. _(unless you speak Greek that is! %^)_

This command has to be prefaced before any system command that requires root level access. I always forget! What a bitch! But do you want to get hacked?

In order to perform root level tasks with your system admin user, you will need to use the command ‘sudo' before the command. This is a helpful command for 2 reasons:

1. It makes it harder for the system admin user break something by accident!
1. It stores all the commands run with sudo to a file: ‘ /var/log/secure ’ where they can be audited later if needed.


Keep in mind however, that your sys admin user is as powerful as the root user and can do just as much damage! If you only need a user for a limited number of tasks on the VPS, don’t give them rootly powers! They can build and run apps in their own home directory fine. You only need ONE System Admin user.

- First View all available users on your VPS (all fields) using `less`:

```yaml
root@vps1:~#  less /etc/passwd

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
... # more listing output etc...
messagebus:x:103:106::/nonexistent:/usr/sbin/nologin
syslog:x:104:110::/home/syslog:/usr/sbin/nologin
_apt:x:105:65534::/nonexistent:/usr/sbin/nologin
... # more listing output etc...
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
lxd:x:998:100::/var/snap/lxd/common/lxd:/bin/false
do-agent:x:997:997::/home/do-agent:/bin/false
alice:x:1000:1000:sysAdmin,,,,System Admin sudo user for this VPS:/home/alice:/bin/bash
```

Each line above represents a user account, and has seven fields delimited by a colon `:` as follows:

**`account:password:UID:GID:GECOS:directory:shell`**


- UID = user ID

- GID = group ID

- GECOS is an optional field used for informational purposes. Usually it contains the full user name, but it can also be used by services such as finger and managed with the chfn command. This field is optional and may be left blank. The others are kind of self explanatory. ;-)


If you only wish to see all usernames and nothing else, (i.e., the first field only in the list) use this command instead:

```yaml
root@vps2:~# cut -d : -f 1 /etc/passwd

root
daemon
bin
sys
sync
games
man
lp
mail
news
uucp
proxy
www-data
backup
list
irc
gnats
nobody
systemd-network
systemd-resolve
systemd-timesync
messagebus
syslog
_apt
tss
uuidd
tcpdump
sshd
landscape
pollinate
systemd-coredump
lxd
do-agent
alice

root@vps1:~#
```

You will see your admin user at the end of the list, _(because that was the last user created - by you: root)_

- Now Add your System Admin User to the sudo group:

```yaml
root@vps1:~#  gpasswd --add alice sudo

Adding user alice to group sudo
```
Your admin user now may obtain rootly powers with `sudo`.


Groups are stored on your VPS in a similar way inside: `/etc/group`

The `less` , `more` , & `cut` commands work great with `/etc/group` as well.

> **Warning:** Never edit these files by hand! There are utilities that properly handle locking of these files. If you edit them by hand you mess it all up and possibly hose the format of the database as well! Also by doing so, you are circumvating and messing with the integrity and security of your VPS!

Right now you need to create the `ssh-user` group and add your Sys$Admin user to it so that she can later log in remotely via SSH. _(see example `sshd_config` file under **Step 10 - Configure OpenSSH**, where you see that this group is allowed to use SSH)_

- **Create ssh-user group with** `groupadd` **:**     
`root@vps2:~# groupadd ssh-user`    


- **Add Sys$Admin user** `alice` **to** `ssh-user` **Group:**    
`root@vps2:~# gpasswd --add alice ssh-user`

Your Sys$Admin user is all set up and ready now... There are several more things to do below however...

Here are some more things for future reference about routine maintenance of users and groups...

- **Find out who is logged in with `who` & `w`:**     
I have used these commands in the past to see if my
boss was around and/logged in... No bosses anymore, but clients do lurk... ;-)     


- **List files owned by a user or group with the `find` utility**:     
`root@vps2:~# find / -group <group-name>`   
`find: ‘/path/to/file/that/matches`   
`find: ‘/path/to/file/that/matches`   
`find:  etc...`


- list all existing user accounts including their properties stored in the user database with `passwd -Sa` as root:     
`root@vps2:~# passwd -Sa`     
`root L 04/16/2014 0 99999 7 -1`     
`daemon L 01/26/2018 0 99999 7 -1`     
`bin L 01/26/2018 0 99999 7 -1`     
`Etc. . .`


- **Show group membership of a user:**    
`root@vps2:~# groups <username>`    
`user : user ssh-user`    


- **Add a user to one or more groups:**     
`root@vps2:~# usermod -a -G <group(s)> <username>`     
_(group(s) can be a comma-separated list of valid group names)_


> **Warning:** Make sure you use the -a option (as in above example)! If it is left out, the user will be removed from all previous groups that are not listed in <group(s)> This may be your intention though... ;-)

- **If you only need to add a user to one group use this command instead:**    
`root@vps2:~# gpasswd --add <username> <group>`


There are a lot more basic command tricks you should learn to make your life easier! Use the unix man pages, and/or read online docs...


**[\[Table of Contents\]](#table-of-contents)**

### Step 07 - Copy authorized_keys file from `root`:

- **Copy:** `/root/.ssh/authorized_keys` **to** `alice` **:**     
`root@vps2:~# mkdir /home/alice/.ssh`     
`root@vps2:~# cp /root/.ssh/authorized_keys /home/alice/.ssh/.`


- **Change ownership from** `root` **to** `alice` **:**     
`root@vps2:~# chown alice:alice /home/alice/.ssh`     
`root@vps2:~# chown alice:alice /home/alice/.ssh/authorized_keys`     
`root@vps2:~# chmod 700 /home/alice/.ssh`     
`root@vps2:~# chmod 600 /home/alice/.ssh/authorized_keys`     

The above allows `alice` to log in remotely via SSH, and also protects Alice's $HOME/.ssh directory from everyone except Alice...

Completing the above steps... You are all done... Now Alice, _(your new sys$Admin user is ready for action and can log in remotely)_

### Step 08 - Log In with your new Sys§Admin User:

Don't log off your present root terminal session yet!  Instead, check to see if you can log in with your new sys admin user via SSH within another terminal window. 

To do this open another terminal window or tab with your Terminal App.  This is usually done with: `SHIFT CTRL T` or something similar... _(check the docs for your Terminal App)_


> **Warning!** DO NOT log root out just yet... You are still not totally safe yet!!!

NOW Log into your system admin user's account via SSH from another terminal window:     

```yaml
$> mosh alice@web-host-name.com

Welcome to Ubuntu 20.04.1 LTS (GNU/Linux 5.4.0-52-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Wed Oct 21 21:36:22 UTC 2020

  System load:           0.0
  Usage of /:            2.2% of 77.36GB
  Memory usage:          5%
  Swap usage:            0%
  Processes:             120
  Users logged in:       1
  IPv4 address for eth0: 143.110.159.141
  IPv4 address for eth0: 10.48.0.6
  IPv6 address for eth0: 2604:a880:4:1d0::63:6000
  IPv4 address for eth1: 10.124.0.3

0 updates can be installed immediately.
0 of these updates are security updates.

To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

alice@vps2:~$
```

Login will be immediate with no prompt... This may surprise you at first... It may feel like your server is all wide open but no... You were just too smart about getting these keys set up and talking to each other! Pat yourself on the back for a job well done... You are not done yet though. ;-)

Notice your prompt ends with a Dollar Sign `$` now... instead of hash `#`...  This shows that you are a regular user... Not root... But you can run rootly commands using the `sudo` prefix... Then you will be asked for your password... You got that in your password vault right? No? omg! You did not follow an important step above! Go do that now! %^)

Test some commands that you know require rootly privileges. First without sudo just to see the `access denied` message. Then preface the command with `sudo` and type in the system admin user’s password when prompted. If you are granted access you are golden. 

A good test is to try to issue the ufw status command. First without sudo, you will get an error. Try again with sudo and it will display the list of ports as normal for the root user.

```bash
alice@vps2:~$ ufw status
ERROR: You need to be root to run this script
alice@vps2:~$
```

Now try it with `sudo`:

```bash
alice@vps2:~$ sudo ufw status
[sudo] password for alice:    # You will need to type your password here...
                              # You won't have to keep typing your password
                              # for more system commands as long as you do the
                              # within the designated time out period...
Status: active

To                         Action      From
--                         ------      ----
Anywhere                   ALLOW       1.2.3.4                   
22/tcp                     ALLOW       Anywhere                  
80/tcp                     ALLOW       Anywhere                  
443/tcp                    ALLOW       Anywhere                  
68/udp                     ALLOW       Anywhere                  
60000:60010/udp            ALLOW       Anywhere                  
22/tcp (v6)                ALLOW       Anywhere (v6)             
80/tcp (v6)                ALLOW       Anywhere (v6)             
443/tcp (v6)               ALLOW       Anywhere (v6)             
68/udp (v6)                ALLOW       Anywhere (v6)             
60000:60010/udp (v6)       ALLOW       Anywhere (v6)

alice@vps2:~$
```

Now you are done setting up your sys§Admin account for doing rootly things, and you have
tested to make sure it all works. Continue to stay logged in as the sys§Admin (system user with rootly privileges). You are now ready to finish configuration using your new sys§Admin account.


**[\[Table of Contents\]](#table-of-contents)**

### Step 09 - Install Emacs Editor:

If you never used a text editor before, install emacs using the guide below on both your local and server machines... There are plenty of cheat-sheets around and after a couple weeks of using it you will be hooked! Emacs is going to be your best app friend! I guarantee! (That is, your editor-of-choice-religious-attitude has not been poisoned yet by using another text editor... Of course... Right?) END: emacs recruitment pitch!

If you don't care to get involved with emacs _(I cannot hardly blame you. I had that attitude once until I saw someone going fast as crazy doing all kinds of stuff with it. Then I was ready to bite the bullet)_.  Just the same... there is a simple code editor installed by default on most Linux distributions clled **`nano`** This bare-bones editor works fine for doing the tasks below as well.  In that case, skip Step 9 alltogether and simply use **`nano`** instead... Replace the word "**`emacs`**" in the examples below with "**`nano`**" and you will be all set...


Emacs is available on almost everything! I usually install it from the package manager's right away first thing asap so I can get to editing files and managing the file system easier... This is especially important in the beginning stages of server provisioning without the help of yucky GUI apps like CPanel.  OMG don't get me started on that! 

You will need to edit a lot of files! And you can zoom much faster with dried-mode than you can in the shell alone!

- Update/Upgrade the system first:     
`alice@vps2:~$ sudo apt update`      
`alice@vps2:~$ sudo apt upgrade`     


- Now Install Emacs:    
`alice@vps2:~$ sudo apt install emacs`     
`alice@vps1:~$ emacs --version`     
`GNU Emacs 26.3`        
`Copyright (C) 2019 Free Software Foundation, Inc.`        
`GNU Emacs comes with ABSOLUTELY NO WARRANTY.`          
`You may redistribute copies of GNU Emacs`     
`under the terms of the GNU General Public License.`     
`For more information about these matters, see the file named COPYING.`     

That's an older version of Emacs than I use on my local machines, with fancy publishing tools installed etc.  But it works fine for editing files and getting around on the server...  We installed it first so we can edit `/etc/ssh/sshd_config` below using `sudo`:


### Step 10 - Configure OpenSSH:

> **Note to Self:** _(remove this later)_ For detailed information about SSH Key Configuration and more OpenSSH security management as a whole see this document: **HAP OpenSSH Configuration & Management** . If you are also setting up SSH for Mac OS, follow this guide for MacOS specifics: **HAP OS-X OpenSSH Best Practice Guide**

The following steps below will configure your VPS Droplet with tight and narrow SSH security that blocks SSH access to ALL except your Sys$Admin user AND ONLY your Sys$Admin user "alice" as we created her above, to connect remotely via SSH. **Note:** We are talking about **SSH** not **TLS** which is another story alltogether... You (or your designated system admin person) will be the only ones using SSH to connect to your VPS server. Keeping your server secure is an on going diligent task!  You must learn these skills _(octopus girl)_ in order to swim with the sharks!

> **Key Management Best Practice:** Store all access credentials in a KeePassXC vault, _(with Master Key Pairs stored as attachments)_... Backup a clone of your KeePassXC vault on a secure un-compromised USB stick and keep that safe in a place where no one can find it (except you and only you)!!! Your USB stick with encrypted KeePassXC database clone on it functions as your hardware backup safety-net so-to-speak, while your original master KeePassXC database vault lives on your office computer/laptop configured to interface with SSH Agent, loading keys as needed for connecting to HOSTs...  Closing your Database removes keys from SSH-Agent automatically, and opening it up loads them back in. (this can be configured on a key-by-key basis to keep your secret keys locked up always until you need to use one or more of them.  Then when done they automatically get locked away again reducing your risk of theft).  You can also use two-factor authentication for opening your KeePassXC database using a Yubikey. **[Read More about Setting That Up Here](https://example.com/comming-soon.html)**  

> Don’t loose your credentials otherwise you could get locked out of your VPS, and your only recourse would be to destroy and rebuild a completely new machine (a repeat raw build of the machine you were locked out of) and start all over (hope you made backups of configurations etc.). Any data stored on the old server that was not backed up in a place where you have access will be lost forever! Sorry Charlie! Consider yourself warned!  Be diligent and careful with security...

#### Edit SSH daemon Configuration file: `/etc/ssh/sshd_config`

The example below is my **_Security Hardened_** version of the default `sshd_config` file that came with **Ubuntu V20.04 LTS** out of the box... Many of the settings I explicitly set below are the actual defaults now...  But setting them explicitly here lets me know for sure they are the settings I am using on the server. _(sometimes relying on defaults can be dangerous)_ In addition to the default settings already supplied out-of-box, I have added extremely restrictive cipher and key exchange rules!  Much more strict than open-community SSH servers like GitHub.com etc...  It's your private machine, and only you need to SSH connect to it... Lets make that stronger...


```yaml
alice@vps2:~$ sudo emacs /etc/ssh/sshd_config

#### ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
#       $OpenBSD: sshd_config,v 1.103 2018/04/09 20:41:22 tj Exp $
#
# This is the sshd server system-wide configuration file.
# See sshd_config(5) for more information.
#
# This sshd was compiled with PATH=/usr/bin:/bin:/usr/sbin:/sbin
#
# The strategy used for options in the default sshd_config shipped
# with OpenSSH is to specify options with their default value where
# possible, but leave them commented.  Uncommented options override
# the default value.
#
# Created: 2020-010-18 - Alisha Awen, siren1@disroot.org
#           Modified the default /etc/ssh/sshd_config...
#
# Updated: 2020-010-22 - Alisha Awen, siren1@disroot.org
#           Added more restrictive directives to harden SSH
#           to single set of conditions, single SSH user...
#           All other access blocked, all other methods removed.
#### ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Include /etc/ssh/sshd_config.d/*.conf

### Set Protocol 2 explicitly here:

Protocol 2

### Use default port 22 (I may change this later)

Port 22

#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::


### Use only the single ed25519 host key:

#HostKey /etc/ssh/ssh_host_rsa_key
#HostKey /etc/ssh/ssh_host_ecdsa_key

HostKey /etc/ssh/ssh_host_ed25519_key


### Key Exchange Algorithms: (use only the single strongest)

KexAlgorithms curve25519-sha256@libssh.org


### Ciphers and keying: (limit cipher to single strongest one)

Ciphers chacha20-poly1305@openssh.com

#RekeyLimit default none


### Message Authentication Codes: 
### (limit MACs to single strongst choice)

MACs hmac-sha2-512-etm@openssh.com


### Logging:

#SyslogFacility AUTH
#LogLevel INFO

### Authentication:

#LoginGraceTime 2m

## Disallow root logins!

PermitRootLogin no

### Use AllowGroups instead of AllowUsers
### (much better practice for management)

AllowGroups ssh-user

#StrictModes yes

## Limit the number of password attempts:

MaxAuthTries 6

#MaxSessions 10

### Enable Public Key Authentication (your only way)

PubkeyAuthentication yes

### Expect .ssh/authorized_keys2 to be disregarded by 
### default in future.

#AuthorizedKeysFile     .ssh/authorized_keys .ssh/authorized_keys2

#AuthorizedPrincipalsFile none

#AuthorizedKeysCommand none
#AuthorizedKeysCommandUser nobody

### For this to work you will also need host keys in:
### /etc/ssh/ssh_known_hosts

#HostbasedAuthentication no

### Change to yes if you don't trust 
### ~/.ssh/known_hosts
### for Host Based Authentication...

#IgnoreUserKnownHosts no

### Don't read the user's ~/.rhosts and ~/.shosts files

#IgnoreRhosts yes

### To disable tunneled clear text passwords, change to no here!
### And, although redundant, disallow empty passwords as well...

PasswordAuthentication no
PermitEmptyPasswords no

### Do NOT Change to yes as it will allow logins via password,
### even though you set that above to NO already!!!
### (also, beware issues with some PAM modules and threads)

ChallengeResponseAuthentication no

### Kerberos options

#KerberosAuthentication no
#KerberosOrLocalPasswd yes
#KerberosTicketCleanup yes
#KerberosGetAFSToken no

### GSSAPI options

#GSSAPIAuthentication no
#GSSAPICleanupCredentials yes
#GSSAPIStrictAcceptorCheck yes
#GSSAPIKeyExchange no

### Set this to 'yes' to enable PAM authentication, account 
### processing, and session processing. If this is enabled, PAM 
### authentication will be allowed through the 
### ChallengeResponseAuthentication and PasswordAuthentication.  
### Depending on your PAM configuration, PAM authentication via 
### ChallengeResponseAuthentication may bypass the setting of 
### "PermitRootLogin yes... If you just want the PAM account and 
### session checks to run without PAM authentication, then enable
### this but set PasswordAuthentication and 
### ChallengeResponseAuthentication to 'no'.

UsePAM yes

#AllowAgentForwarding yes
#AllowTcpForwarding yes
#GatewayPorts no

X11Forwarding yes

#X11DisplayOffset 10
#X11UseLocalhost yes
#PermitTTY yes

PrintMotd no

#PrintLastLog yes
#TCPKeepAlive yes
#PermitUserEnvironment no
#Compression delayed
#ClientAliveInterval 0
#ClientAliveCountMax 3

UseDNS no

#PidFile /var/run/sshd.pid
#MaxStartups 10:30:100
#PermitTunnel no
#ChrootDirectory none
#VersionAddendum none

### no default banner path

#Banner none

### Allow client to pass locale environment variables

AcceptEnv LANG LC_*

### override default of no subsystems

Subsystem       sftp    /usr/lib/openssh/sftp-server

### Example of overriding settings on a per-user basis

### Match User anoncvs

# X11Forwarding no
# AllowTcpForwarding no
# PermitTTY no
# ForceCommand cvs server

### END: /etc/ssh/sshd_config
#### ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
```


#### Checking your Configuration Results:


After making any changes to the servers SSH configuration you must reload the daemon:

- **Reload the OpenSSH daemon:**        
`alice@vps2:~$ sudo systemctl reload sshd`    


- Verify/View current active address:ports with `netstat` command:      
`root@vps2:~# netstat -ntulp`     

> **Note:** You may need to install `net-tools` _(`apt install net-tools`)_ first to make netstat command available on your system...

You can test the most important setting above `Protocol 2` by attempting to log in using `Protocol 1` as follows:

```yaml
$> ssh -1 user@your-vps-host-IP
```
You should get the error: `SSH protocol v.1 is no longer suported`

#### Check SSH Server Logs:

```yaml
alice@vps2:~$ sudo journalctl -u ssh
```
Press `J` to scroll down, and `K` to scroll up.  Press `F` to scroll down one full screen, and `B` to scroll up one full screen.  Press `Q` to quit...

If you only want to see the end of the log, run:

```yaml
alice@vps2:~$ sudo journalctl -eu ssh
```
You will be surprised at all the bad guys trying to access your machine with those "Failed password" lines!!!


**[\[Table of Contents\]](#table-of-contents)**

### Step 11 - Install Fail2Ban:

Log back into your server if not already logged in...

```yaml
$> mosh alice@web-host-name.com

`alice@vps2:~$
```

Update/Upgrade the system first:

```yaml
alice@vps2:~$ sudo apt update
alice@vps2:~$ sudo apt upgrade
```

Install Fail2Ban:

```yaml
alice@vps2:~$ sudo apt install fail2ban
```

After it’s installed, it will be automatically started, as can be seen with:

```yaml
alice@vps2:~$ sudo systemctl status fail2ban
```

The fail2ban-server program included in fail2ban monitors log files and issues ban/unban command. By default, it would ban a client’s IP address for 10 minutes if the client failed password 5 times. The ban is done by adding iptables firewall rules. You can check iptables rules by running the following command.


```yaml
alice@vps2:~$ sudo iptbles -L
```
The fail2ban rules file is **`/etc/fail2ban/jail.conf`**. You don’t have to edit it as the default settings should suffice to prevent SSH brute force attack. If you want to modify the default settings, you should add your modifications in **`/etc/fail2ban/jail.local`** file.

if you change or add new rules to: **`/etc/fail2ban/jail.local`** you need to reload fail2ban afterwords to get those changes to take effect:

```yaml
alice@vps2:~$ sudo systemctl reload fail2ban
```
For more information consult the man pages for `jail.conf`

### Step 12 - Make Initial Snapshot of your Default Base VPS Server Now:

Shutdown your Server from the Terminal:

```yaml
sudo shutdown now
```

- Log into your **[DigitalOcean](https://m.do.co/c/449bba2e8a2d)** account in your web browser

- Click on this droplet's name

- Choose Turn Off from the control panel.     
(the green slider)

- Go to Snapshots section... 

- Give snapshot a name:  e.g., My-VPS1-Initial-Build

- Click Take Snapshot button...

When done you can start the droplet back up and log in...









**[\[Table of Contents\]](#table-of-contents)**

## Install / Provision Web Server

Our Base Server Operations Software is installed and provisioned...  Web server tasks are next...

### Step 13 - Install Nginx Web Server

Installing Nginx is easy... Log in as your normal `sudo` user, do `apt update` and `apt upgrade`:


```yaml
$> mosh alice@vps1.example.com
alice@vps1:~$ 
alice@vps1:~$ sudo apt update
alice@vps1:~$ sudo apt upgrade
```

Now install Nginx Web Server and enale it to auto-start at boot:

```yaml
alice@vps1:~$ sudo apt install nginx
alice@vps1:~$ sudo systemctl start nginx

## Now check Nginx status & version:

alice@vps1:~$ sudo systemctl status nginx

alice@vps1:~$ nginx -v
```
Now type the public IP of your server into a web browser and you should get the Welcome to nginx! page...

All Done!


### Step 14 - Install MariaDB Server

Log in as your normal `sudo` user, do `apt update` and `apt upgrade`:

```yaml
$> mosh alice@vps1.example.com
alice@vps1:~$ 
alice@vps1:~$ sudo apt update
alice@vps1:~$ sudo apt upgrade

alice@vps1:~$ sudo apt install software-properties-common 
alice@vps1:~$ sudo apt install mariadb-server
alice@vps1:~$ sudo apt install mariadb-client

## Check Status of installed service:

alice@vps1:~$ systemctl status mariadb
```

Create a root password for your MariaDB server, (by default is is not set yet).  Make sure you have created a KeePassXC data base item for this password!  Then have it ready to enter below after invoking the interactive session with the following command:

```yaml
alice@vps1:~$ sudo mysql_secure_installation
```

When it asks you to enter MariaDB root password, press Enter key as the root password isn’t set yet. Then enter y to set the root password for MariaDB server (copy the one you created in your KeePassXC database for MariaDB Root User.

Press Enter for all remaining questions which remove anonymous user, disables remote root logins, etc...

By default, the MaraiDB package on Ubuntu uses unix_socket to authenticate user login, which basically means you can use username and password of the OS to log into MariaDB console. So you can run the following command to login without providing MariaDB root password.

```yaml
alice@vps1:~$ sudo mariadb -u root

Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 55
Server version: 10.3.22-MariaDB-1ubuntu1 Ubuntu 20.04

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> exit;
Bye
alice@vps1:~$
# Check MariaDB server version information.
alice@vps1:~$ mariadb --version
mariadb  Ver 15.1 Distrib 10.3.22-MariaDB, 
for debian-linux-gnu (x86_64) using readline 5.2
alice@vps1:~$
```


**[\[Table of Contents\]](#table-of-contents)**

### Step 15 - Install PHP7.4

PHP7.4 is included in Ubuntu 20.04 repository and has a minor performance improvement over PHP7.3. Enter the following command to install PHP7.4 and some common extensions.

#### Install PHP7.7 with Extra Extensions:

```yaml
alice@vps1:~$ sudo apt install php7.4 php7.4-fpm php7.4-mysql php-common php7.4-cli php7.4-common php7.4-json php7.4-opcache php7.4-readline php7.4-mbstring php7.4-xml php7.4-gd php7.4-curl
```
Installing the extra PHP extensions ensure any CMS (like WordPress) run smoothly.

#### Start php7.4-fpm & Enable for System Boot:

```yaml
alice@vps1:~$ sudo systemctl start php7.4-fpm
# Enable auto-start at boot time.
alice@vps1:~$ sudo systemctl enable php7.4-fpm
# Check status:
alice@vps1:~$ systemctl status php7.4-fpm
```
### Step 16 - Create Nginx Server Block

Nginx server blocks are similar to virtual hosts in Apache. For our purposes we will not be using the default server block...  It is not set up to run PHP well... Therefore
remove the default symlink in the sites-enabled directory by running the following command: _(It’s still available as /etc/nginx/sites-available/default)_

```yaml
alice@vps1:~$ sudo rm /etc/nginx/sites-enabled/default
```

#### Create new Nginx Server Block file in `/etc/nginx/conf.d`

```yaml
alice@vps1:~$ sudo Emacs /etc/nginx/conf.d/default.conf
```

Paste the following text into the file. The following snippet will make Nginx listen on IPv4 port 80 and IPv6 port 80 with a catch-all server name.

```php
server {
  listen 80;
  listen [::]:80;
  server_name _;
  root /usr/share/nginx/html/;
  index index.php index.html index.htm index.nginx-debian.html;

  location / {
    try_files $uri $uri/ /index.php;
  }

  location ~ \.php$ {
    fastcgi_pass unix:/run/php/php7.4-fpm.sock;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
    include snippets/fastcgi-php.conf;
  }

 # A long browser cache lifetime can speed up repeat visits to your page
  location ~* \.(jpg|jpeg|gif|png|webp|svg|woff|woff2|ttf|css|js|ico|xml)$ {
       access_log        off;
       log_not_found     off;
       expires           360d;
  }

  # disable access to hidden files
  location ~ /\.ht {
      access_log off;
      log_not_found off;
      deny all;
  }
}
```
Save and Close the file after adding the above code...


#### Test for Nginx configuration:

```yaml
alice@vps1:~$ sudo nginx -t
```

If the test is successful, reload Nginx:

```yaml
alice@vps1:~$ sudo systemctl reload nginx
```


**[\[Table of Contents\]](#table-of-contents)**

### Step 17 - Test PHP

> **Warning!** The file you are about to create and view over the web is a dangerous thing! If you are hosting a live server _(even just for test purposes, i.e. you have not announced or promoted it yet)_ there are bot scripts out there probing IP addresses and these bot scripts look for that file as a web page! If they see the resulting page they will gain a hole lot of information about your machine and may try launching a penetration attack. That is, if they find anything interesting and worth pirateing.   Therefore, do this test quickly... _(i.e. within one minute)_ and then delete that file so it can no longer be probed by hackers...

To test PHP-FPM with Nginx Web server, you need to create a temporary file: **`info.php`** in the webroot directory. _(You need to create this file, view the web-page, and then delete the file immediately, ALL WITHIN ONE MINUTE or less!)_


```yaml
alice@vps1:~$ sudo emacs /usr/share/nginx/html/info.php
```

Paste the following PHP code into the file.

```php
<?php phpinfo(); ?>
```

Save and close the file... 

Now in the browser address bar, enter: `your-server-IP-or-hostname/info.php`

You should see your server’s PHP information. This means PHP scripts can run properly with Nginx web server...

Congrats! You have successfully installed Nginx, MariaDB and PHP7.4 on Ubuntu 20.04. For your server’s security, you should delete info.php file now to prevent hacker seeing it.

NOW DELETE THAT FILE ASAP!!!

```yaml
alice@vps1:~$ sudo rm /usr/share/nginx/html/info.php
```

### Step 18 - Nginx Troubleshooting Tips

- If you encounter errors, you can check the **Nginx error log** **(`/var/log/nginx/error.log`)** to find out what’s wrong.

- **Nginx Automatic Restart:**      
If for any reason your Nginx process is killed, you need to run the following command to restart it:      
`sudo systemctl restart nginx`     

Instead of manually typing the above command, we can make **Nginx** automatically restart by editing the `nginx.service` systemd service unit. 

To override the default systemd service configuration, we create a separate directory:

```bash
sudo mkdir -p /etc/systemd/system/nginx.service.d/
```

Then create a file under this directory:

```bash
sudo emacs /etc/systemd/system/nginx.service.d/restart.conf
```

Add the following lines in the file, which will make **Nginx** automatically restart 5 seconds after a failure is detected:

```conf
[Service]
Restart=always
RestartSec=5s
```

Save and close the file. Then reload systemd.

```bash
sudo systemctl daemon-reload
```

To check if this would work, kill Nginx with:

```bash
sudo pkill nginx
```

Then check Nginx status:

```bash
systemctl status nginx
```

You will find Nginx automatically restarted...



**[\[Table of Contents\]](#table-of-contents)**

## Install Wordpress Website

Nginx Web Server all set up now... Time to install our first WordPress website...

### Step 22 - Download & Install WordPress .zip archive:

From your Sys$Admin's home directory issue the following commands to download, unzip, and install the latest WordPress **`.zip`** archive...

```bash
wget https://wordpress.org/latest.zip

## Install unzip if you don't already have it:

sudo apt install unzip

## Unzip the archive into /usr/share/nginx: 

sudo unzip latest.zip -d /usr/share/nginx/
```

The archive will be extracted to the: `/usr/share/nginx/` directory. A new directory named **wordpress** will be created (`/usr/share/nginx/wordpress`). 


Rename `wordpress` to an abbreviation name of your website address similar to this:

```bash
sudo mv /usr/share/nginx/wordpress /usr/share/nginx/my-www
```

_(my-www is short for: www.example.com)_

## Install OpenSim Region Server

All of the basics are out of the way and we even have a WordPress site installed to makd a blog...  Now it is finally time to get to the meat of our project and install the OpenSim Server...

### Step 27 - Create MariaDB `opensim` Database & User:

- Go to your **phpMyAdmin** panel to do this... _(You set up and hosted phpMyAdmin in previous steps)..._  Simply enter the URL for your phpMyAdmin site instance within a browser window, and log in as the **MariaDB Admin User** who has privileges to create databases and users..  **phpMyAdmin** creates databases with `utf8mb4`  by default... This OpenSim db needs to be utf8mb3.  This is why we need to create the database first and then assign `opensim` as the DB Admin.

- Once logged in, click the **Databases** tab...

- Type `opensim` in the field under "**Create database**" for the DB name... 

- Choose `utf8_general_ci` for Collation type (this is `utf8mb3`) according to the Maria DB docs...

- Click **Create**

A new opensim database is created with no tables in it...

- Click on **Check Privileges** for **`opensim`** db in the database list...

- Now, Click **Edit privileges** for username **`opensim`**...

- Give DBA user **`opensim`** all privileges except `GRANT`

- Click the **Go** button...




### Step 28 - Use Squid as a reverse proxy to the asset server:

[Good Squid Tutorial Here:](https://www.cyberciti.biz/faq/ubuntu-squid-proxy-server-installation-configuration/)


- **Install `squid`:**     

```yaml
## Update/upgrade the system first:

sudo apt update
sudo apt upgrade

## Show Package Details for squid:

apt show swuid

## Install Squid:

sudo apt install squid
```

- **Create your squid.conf configuration file:**     
The default squid configuration file is located at: **`/etc/squid/squid.conf`**     
Make a backup of the original file first:      
`alice@vps1:~$ sudo cp -v /etc/squid/squid.conf{,.factory}`     
`'/etc/squid/squid.conf' -> '/etc/squid/squid.conf.factory'`

- **Replace contents of `etc/squid/squid.conf` with the following:**

```conf
### ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
### squid.conf
### ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

acl all src 0.0.0.0/0.0.0.0
acl manager proto cache_object
acl localhost src 127.0.0.1/255.255.255.255
acl to_localhost dst 127.0.0.0/8
acl SSL_ports port 443          # https
acl SSL_ports port 563          # snews
acl SSL_ports port 873          # rsync
acl Safe_ports port 80          # http
acl Safe_ports port 21          # ftp
acl Safe_ports port 443         # https
acl Safe_ports port 70          # gopher
acl Safe_ports port 210         # wais
acl Safe_ports port 1025-65535  # unregistered ports
acl Safe_ports port 280         # http-mgmt
acl Safe_ports port 488         # gss-http
acl Safe_ports port 591         # filemaker
acl Safe_ports port 777         # multiling http
acl Safe_ports port 631         # cups
acl Safe_ports port 873         # rsync
acl Safe_ports port 901         # SWAT
acl purge method PURGE
acl CONNECT method CONNECT
http_access allow manager localhost
http_access deny manager
http_access allow purge localhost
http_access deny purge
http_access deny !Safe_ports
http_access deny CONNECT !SSL_ports
http_access allow localhost
http_access deny all
icp_access allow all
http_port 127.0.0.1:3128 vhost vport

## This is configured for osgrid on port 8003, 
## change to your grid asset server:

cache_peer osgrid.org parent 8003 0 originserver default
hierarchy_stoplist cgi-bin ?
cache_dir ufs /var/spool/squid 1024 16 256
access_log /var/log/squid/access.log squid
acl QUERY urlpath_regex cgi-bin \?
cache deny QUERY
refresh_pattern ^ftp:           1440    20%     10080
refresh_pattern ^gopher:        1440    0%      1440

## The next line gives you about a week in cache before expiration.
## Change 100000 and 100800 to however long you want.

refresh_pattern . 100000  20% 100800 override-expire
acl apache rep_header Server ^Apache
broken_vary_encoding allow apache
extension_methods REPORT MERGE MKACTIVITY CHECKOUT
hosts_file /etc/hosts
coredump_dir /var/spool/squid

### ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
### END squid.conf
### ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
```

- Change your asset_server configuration in your OpenSim.ini to point to: http://localhost:3128/


Once you get OpenSim up, and regions are hooked up... assets will be cached in the squid cache on your region server, and will be served up much faster, especially on region restart.


**[\[Table of Contents\]](#table-of-contents)**

### Step 29 - Enable Ports on Sever for OpenSim:

Before starting the process, Open up some ports on your Ubuntu VPS using the default firewall utility: **`ufw`**

- **List Current Firewall Status:**

```yaml
alice@vps2:~$ sudo ufw status
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere                  
80/tcp                     ALLOW       Anywhere                  
443/tcp                    ALLOW       Anywhere                  
68/udp                     ALLOW       Anywhere                  
60000:60010/udp            ALLOW       Anywhere                  
22/tcp (v6)                ALLOW       Anywhere (v6)             
80/tcp (v6)                ALLOW       Anywhere (v6)             
443/tcp (v6)               ALLOW       Anywhere (v6)             
68/udp (v6)                ALLOW       Anywhere (v6)             
60000:60010/udp (v6)       ALLOW       Anywhere (v6)

alice@vps2:~$ _
```
You should see a screen just as the above from your previous configurations...

- **Open Ports 8000 to 8010 for udp:** These ports will be for your regions. _(up to 10 which is more than enough starting out!)_... Your region ports only need UDP access... The HTTP listener _(see below)_ uses TCP.


```yaml
## Allow 8000 to 8010 for UDP traffic:

alice@vps2:~$ sudo ufw allow 8000:8010/udp
Rule added
Rule added (v6)
alice@vps2:~$ _

## List Current Firewall Status:

alice@vps2:~$ sudo ufw status
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere                  
80/tcp                     ALLOW       Anywhere                  
443/tcp                    ALLOW       Anywhere                  
68/udp                     ALLOW       Anywhere                  
60000:60010/udp            ALLOW       Anywhere                  
8000:8010/udp              ALLOW       Anywhere                  
22/tcp (v6)                ALLOW       Anywhere (v6)             
80/tcp (v6)                ALLOW       Anywhere (v6)             
443/tcp (v6)               ALLOW       Anywhere (v6)             
68/udp (v6)                ALLOW       Anywhere (v6)             
60000:60010/udp (v6)       ALLOW       Anywhere (v6)             
8000:8010/udp (v6)         ALLOW       Anywhere (v6)             

alice@vps2:~$ _
```

Now you will see the your new UDP ports added to the list...


- **Open Port 9000 for TCP:** This is the Simulator HTTP port. This is not the region port, but the port the entire simulator listens on. This port uses the TCP protocol, whilst the region ports use UDP. This port must be unique to the simulator. It has been set within OpenSim.ini as follows:      
`http_listener_port = 9000`


```yaml
## Allow Port 9000 for TCP traffic:

alice@vps2:~$ sudo ufw allow 9000/tcp
Rule added
Rule added (v6)
alice@vps2:~$ _

## List Current Firewall Status:

alice@vps2:~$ sudo ufw status
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere                  
80/tcp                     ALLOW       Anywhere                  
443/tcp                    ALLOW       Anywhere                  
68/udp                     ALLOW       Anywhere                  
60000:60010/udp            ALLOW       Anywhere                  
8000:8010/udp              ALLOW       Anywhere                  
9000/tcp                   ALLOW       Anywhere                  
22/tcp (v6)                ALLOW       Anywhere (v6)             
80/tcp (v6)                ALLOW       Anywhere (v6)             
443/tcp (v6)               ALLOW       Anywhere (v6)             
68/udp (v6)                ALLOW       Anywhere (v6)             
60000:60010/udp (v6)       ALLOW       Anywhere (v6)             
8000:8010/udp (v6)         ALLOW       Anywhere (v6)             
9000/tcp (v6)              ALLOW       Anywhere (v6)

alice@vps2:~$ _
```


**[\[Table of Contents\]](#table-of-contents)**

### Step 30 - Install & Configure NeverWorld Region Server:

#### Get NeverWorld Region S/W V0.9.1.1:

While logged in to your server as your Sys$Admin user, use wget to put the .zip file into your home directory:

```yaml
wget https://www.dropbox.com/s/vsjjjtrkdhz991n/neverworld%20region%20server%200.9.1.1.zip?dl=0
```

Unzip this software in place within your System Admin's home directory... A new directory will be created named:  **`opensim-0.9.1.1`**

#### Configure OpenSim.ini - Add your ports:

- Go to your OpenSim `bin` directory:  `~/opensim-0.9.1.1/bin`     
`alice@vps2:~$ cd ~/opensim-0.9.1.1/bin`

Look for a file within this directory called **`opensim.ini`**.  In this MS Windows init `.ini` file, you will see the following settings close to the beginning of the file:

```ini
[const]

   ; this setcion defines constants for grid services
   ; etc. . .
   ; . . .

   PublicPort = "8001"

   ; A few more comments and settings follow . . .

   PrivatePort = "8003"

   ; Many comments and settings follow . . .

[Network]

   ; Simulator HTTP port.

   http_listener_port = "9000"
```

Make sure your `OpenSim.ini` Public and Private port settings are set as in the above example.  The listener port (TCP) must be a unique from anything else running on your machine. I simply kept it to the default `9000` which was already set that way.  

> **Note:** We enabled our firewall to accept TCP traffic on port 9000 above and also enabled ports 8000 to 8010 for UDP traffic... after confirming OpenSim.ini has those same ports set, you can save and close the file... Ports are all set up now...


#### OpenSim Database Settings

For our OpenSim server we will use **MySQL** _(i.e., **MariaDB** which we installed)_  We also already created a database named **`opensim`** with db admin user of the same name: `opensim`





## Table Of Contents:

<!-- markdown-toc start - Don't edit this section. Run M-x markdown-toc-refresh-toc -->

- [HAP OpenSim Region Server Build Config Guide](#hap-opensim-region-server-build-config-guide)
    - [Introduction:](#introduction)
    - [Create new VPS droplet on DigitalOcean](#create-new-vps-droplet-on-digitalocean)
        - [Step 01 - New VPS Creation:](#step-01---new-vps-creation)
        - [Step 02 - SSH Log In as root First Time:](#step-02---ssh-log-in-as-root-first-time)
        - [Step 03 - Perform Initial Tasks as Root User:](#step-03---perform-initial-tasks-as-root-user)
        - [Step 04 - Do Sanity Check of Installed Packages:](#step-04---do-sanity-check-of-installed-packages)
    - [Provision your new VPS droplet](#provision-your-new-vps-droplet)
        - [Step 05 - Reboot the Server and Log back in with Mosh:](#step-05---reboot-the-server-and-log-back-in-with-mosh)
        - [Step 06 - Perform First Step VPS Security Provisions:](#step-06---perform-first-step-vps-security-provisions)
            - [Create sysAdmin sudo User:](#create-sysadmin-sudo-user)
            - [Grant sysAdmin User Root Privileges _(sudo)_:](#grant-sysadmin-user-root-privileges-_sudo_)
        - [Step 07 - Copy authorized_keys file from `root`:](#step-07---copy-authorized_keys-file-from-root)
        - [Step 08 - Log In with your new Sys§Admin User:](#step-08---log-in-with-your-new-sysadmin-user)
        - [Step 09 - Install Emacs Editor:](#step-09---install-emacs-editor)
        - [Step 10 - Configure OpenSSH:](#step-10---configure-openssh)
            - [Edit SSH daemon Configuration file: `/etc/ssh/sshd_config`](#edit-ssh-daemon-configuration-file-etcsshsshd_config)
            - [Checking your Configuration Results:](#checking-your-configuration-results)
            - [Check SSH Server Logs:](#check-ssh-server-logs)
        - [Step 11 - Install Fail2Ban:](#step-11---install-fail2ban)
        - [Step 12 - Make Initial Snapshot of your Default Base VPS Server Now:](#step-12---make-initial-snapshot-of-your-default-base-vps-server-now)
    - [Install / Provision Web Server](#install--provision-web-server)
        - [Step 13 - Install Nginx Web Server](#step-13---install-nginx-web-server)
        - [Step 14 - Install MariaDB Server](#step-14---install-mariadb-server)
        - [Step 15 - Install PHP7.4](#step-15---install-php74)
            - [Install PHP7.7 with Extra Extensions:](#install-php77-with-extra-extensions)
            - [Start php7.4-fpm & Enable for System Boot:](#start-php74-fpm--enable-for-system-boot)
        - [Step 16 - Create Nginx Server Block](#step-16---create-nginx-server-block)
            - [Create new Nginx Server Block file in `/etc/nginx/conf.d`](#create-new-nginx-server-block-file-in-etcnginxconfd)
            - [Test for Nginx configuration:](#test-for-nginx-configuration)
        - [Step 17 - Test PHP](#step-17---test-php)
        - [Step 18 - Nginx Troubleshooting Tips](#step-18---nginx-troubleshooting-tips)
    - [Install Wordpress Website](#install-wordpress-website)
        - [Step 22 - Download & Install WordPress .zip archive:](#step-22---download--install-wordpress-zip-archive)
        - [Step 23 - Create Database & User for WordPress website:](#step-23---create-database--user-for-wordpress-website)
        - [Step 24 - Configure WordPress:](#step-24---configure-wordpress)
            - [Create wp-config.php:](#create-wp-configphp)
            - [Edit wp-config.php:](#edit-wp-configphp)
        - [Step 25 - Create Nginx Server Block for `my-www` WordPress site:](#step-25---create-nginx-server-block-for-my-www-wordpress-site)
        - [Step 26 - Start up you site's WordPress Install Wizard:](#step-26---start-up-you-sites-wordpress-install-wizard)
    - [Install OpenSim Region Server](#install-opensim-region-server)
        - [Step 27 - Create MariaDB Database `opensim`:](#step-27---create-mariadb-database-opensim)
        - [Step 28 - Use Squid as a reverse proxy to the asset server:](#step-28---use-squid-as-a-reverse-proxy-to-the-asset-server)
        - [Step 29 - Enable Ports on Sever for OpenSim:](#step-29---enable-ports-on-sever-for-opensim)
        - [Step 30 - Install & Configure NeverWorld Region Server:](#step-30---install--configure-neverworld-region-server)
            - [Get NeverWorld Region S/W V0.9.1.1:](#get-neverworld-region-sw-v0911)
            - [Configure OpenSim.ini - Add your ports:](#configure-opensimini---add-your-ports)
    - [Table Of Contents:](#table-of-contents)

<!-- markdown-toc end -->
