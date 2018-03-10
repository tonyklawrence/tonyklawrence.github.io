---
title: Backing up and restoring a ReadyNAS Ultra
date: 2012-09-16
comments: true
categories: [readynas, backup, unix]
keywords: "readynas, backup, unix"

aliases: 
  - /blog/2012/09/16/backing-up-readynas-for-a-factory-reset/
  - /2012/09/16/backing-up-readynas-for-a-factory-reset/
---

I've had my ReadyNAS drop of the network a couple of times, and lately has been warning me about errors on my disk 2 so I thought it was a good idea to replace that drive.  Due to the few network issues I've had I decided that taking the opportunity to restore the ReadyNAS was a good idea (I had messed with it quite a bit before.)

<!-- more -->

As I have just under 3TB used my new replacement drive would be 3TB so that I could use it for backup / restore and then as a replacement to the failing 2TB drive.

## Initialising the 3TB drive for use

My first big headache was that the ReadyNAS would not recognise the new drive (in an icebox usb enclosure) as 3TB and would only format it to be around 800MB, this was no use.  Also, from my Mac using Disk Utility I could create a 3TB partition but unable to format it to be ext3.  So I had to use my ReadyNAS bash skills.

Firstly I found out that `fdisk` does not support drives over 2TB so after much annoyance I gave up.  Apparently the solution is to use `parted` which does not come on the ReadyNAS.  However, a quick `apt-get` later and I'm away (I didn't care about installing stuff as I was going to restore.)

``` bash
$ apt-get install parted
```

## Parted

I followed Vivek Gites excellent guide here: [http://www.cyberciti.biz/tips/fdisk-unable-to-create-partition-greater-2tb.html](http://www.cyberciti.biz/tips/fdisk-unable-to-create-partition-greater-2tb.html)

I wanted a 3TB primary partition, this was to make the most of the drive.  My USB drive was available as /dev/sde.

``` bash
$ parted /dev/sde
GNU Parted 1.7.1
Using /dev/sde
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted)                                           
```

To create the partition

``` bash
(parted) mklabel gpt
(parted) unit TB
(parted) mkpart primary 0.00TB 3.00TB
```

You can check everything is ok using the `print` command.

``` bash
(parted) print

Disk /dev/sde: 3001GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt

Number  Start   End     Size    File system  Name     Flags
 1      17.4kB  3001GB  3001GB  ext3         primary       
```

To exit simple type

``` bash
(parted) quit
```

Now we can format the partition as `ext3` (you can use `ext4` if you prefer)

``` bash
$ mkfs.ext3 /dev/sde1 # for ext3
$ mkfs.ext4 /dev/sde1 # for ext4
```

## Doing the backup

I decided that the easiest way to backup was to use `rsync`.  This would allow me to do small chunks at a time and also selectively restore.  I was surprised that to backup around 3TB of data (using USB and Green 5400rpm drives) took most of a weekend! (at least I could monitor the progress with the `rsync` output and `df -h`.)

For reference, I used the following command line

``` bash
$ rsync --progress --recursive --times --perms --human-readable <source> <destination>
```

## The factory restore

There was two options I had when resetting my ReadyNAS firmware.

* OS Re-Install - reinstalls the firmware from the internal flash to the disks
* Factory Default - resets the unit to factory settings, erases all data, resets all defaults, and reformats the disk to X-RAID2

Since I had installed things I shouldn't the re-install would not remove this mess.  I thought if I'm going to do it, I might as well clean it properly (it does mean I need to setup my add-ons again.)

## Doing the restore

NetGear has a nice page for reference on how to enter the boot menu and what the options mean. [How do I use the Boot Menu](http://www.readynas.com/kb/faq/boot/how_do_i_use_the_boot_menu)

For my ReadyNAS Ultra 4 I had to do the following to access the boot menu:

* Power off the unit.
* Using a straightened paper clip, press and hold the Reset button.
* Press the Power button to power on the unit.
* Continue to press the Reset button until the status display screen shows an boot menu message.
* Press the Backup button to scroll through the boot mode options.
* Press and release Reset button to confirm your boot menu selection.

And then wait…

## Installing services

### Enabling SSH

First thing is to download and install the SSH access plugin.  This is done the same way as any add-on.  Once this has been installed we can `ssh` into the ReadyNAS and configure other things.  You can use the same username / password of any user you have created.

> A restart is required before you can log in

### Transmission

Download and install the Transmission add-on from the ReadyNAS community forums.  Once installed you will see it available in your add-ons tab.

> Transmission will not start until you have set the `download-dir` in `settings.json` to a valid location

### Automatic

Just like Transmission, download and install from the ReadyNAS comment forums and install in the same way.

## Customisation

If you don't want Mac OS X to write network stores then issue the following command on each Mac OS X client.

``` bash
$ defaults write com.apple.desktopservices DSDontWriteNetworkStores true
```

### Email

``` bash
$ apt-get install courier-imap-ssl courier-maildrop courier-doc
```

You can test with `telnet localhost 143` to see if you can connect, you should expect a response like this.

``` bash
* OK [CAPABILITY IMAP4rev1 UIDPLUS CHILDREN NAMESPACE THREAD=ORDEREDSUBJECT THREAD=REFERENCE SORT QUOTA IDLE ACL ACL2=UNION]
Courier-IMAP ready. Copyright 1998-2005 Double Precision, Inc. See COPYING for distribution information.
```

Creating a user, location for email and then testing.

``` bash
userdb tonylawrence.com set uid=1004 gid=50 home=/c/email mail=tonylawrence.com
userdbpw -md5 | userdb tonylawrence.com set imappw
makeuserdb
```

``` bash
host:/c/email# mkdir tonylawrence.com
host:/c/email# chown tony:staff tonylawrence.com/
host:/c/email# chmod 755 tonylawrence.com/
```

``` bash
$ telnet localhost 143
01 LOGIN <user> <password>
01 OK LOGIN Ok.
```

Install fetch mail and mail drop to fetch and move email into our new home.

``` bash
$ apt-get install fetchmail courier-maildrop
```

Mail Filter is used to place different emails into different locations.  Here I move any email to `tonylawrence.com` into a different folder `~/.mailfilter`.

``` bash
DEFAULT="/c/email"
TL="$DEFAULT/tonylawrence.com/"

if (/^(To|Cc|Bcc):.*@tonylawrence.com/) {
  to $TL
}

to $DEFAULT
```

Fetchmail configuration to pull email from google mail `.fetchmailrc`.

``` bash
set invisible
set bouncemail

poll "imap.gmail.com" protocol imap
  username "<user@gmail.com>"
  password "<password>"
  keep
  ssl
  mda "/usr/bin/maildrop -d <user>"
```

Then automate the fetching via cron

``` bash
$ export EDITOR=vi
$ crontab -e

m h  dom mon dow   command
*/2 * * * * fetchmail --pidfile /tmp/fetchmail.pid
#*/5 * * * * fetchmail >/dev/null 2>&1
```

### Avahi icons

``` bash
$ cd /etc/avahi/services/
$ cat afp.service 
<?xml version="1.0" standalone='no'?><!--*-nxml-*-->
<service-group>
  <name replace-wildcards="yes">%h</name>
  <service>
    <type>_afpovertcp._tcp</type>
    <port>548</port>
  </service>
  <service>
    <type>_device-info._tcp</type>
    <port>0</port>
    <txt-record>model=Macmini</txt-record>
  </service>
</service-group>
Patrician:/etc/avahi/services# cat timemachine.service 
<?xml version="1.0" standalone='no'?><!--*-nxml-*-->
<service-group>
  <name replace-wildcards="yes">Time Machine</name>
  <service>
    <type>_adisk._tcp</type>
    <port>9</port>
    <txt-record>sys=waMA=00:22:3F:AA:2C:E1,adVF=0x100</txt-record>
    <txt-record>dk0=adVF=0xa1,adVN=ReadyNAS,adVU=29d6becd-d614-4346-aa51-bb2f0c8fcbb2</txt-record>
  </service>
  <service>
    <type>_device-info._tcp</type>
    <port>0</port>
    <txt-record>model=TimeCapsule</txt-record>
  </service>
</service-group>
```

> Had to chown admin:admin on /etc/cron.d otherwise couldn't create backups!

## Links

[Root Access SSH](http://www.readynas.com/?p=4203)
[Automatic](http://www.readynas.com/forum/viewtopic.php?f=48&t=41667)
[Transmission](http://www.readynas.com/forum/viewtopic.php?f=48&t=24272)
[Courier IMAP](http://www.courier-mta.org/imap)
[Courier MailDrop](http://www.courier-mta.org/maildrop)
