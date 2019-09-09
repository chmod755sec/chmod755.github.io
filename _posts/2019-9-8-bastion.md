---
layout: post
title:  "Bastion writeup by godzillabg"
---

![Image](/assets/bastion/Writeup_for_Bastion-000.jpg)

# Enumeration
We start with Enumeration on the box with nmap, scanning all ports

## Nmap Scan
![Image](/assets/bastion/Writeup_for_Bastion-001.jpg)  
We notice 445 is open. Port 445 is a popular point of attack. Lets start there first, going
for some low hanging fruit and see if old school NULL Sessions work on this port.

## Smbmap and smbclient
For that, we will use smbmap. smbmap allows users to enumerate samba share drives
across an entire domain:
```
Smbmap -H 10.10.10.134 -u ' '
```  
![Image](/assets/bastion/Writeup_for_Bastion-002.jpg)

So looks like we can do something with IPC$ and Backups. Switching to smbclient, will
explore those two shares. IPC$ did not have anything, but Backups had info:
```
smbclient \\\\10.10.10.134\\Backups -U ' ' -N
```
![Image](/assets/bastion/Writeup_for_Bastion-003.jpg)  
Even though we can only read, let’s explore some more. There is a note in this share,
lets see what it says:
```
Sysadmins: please don't transfer the entire backup file locally, the VPN to the subsidiary
office is too slow.
```
Backup file? Hmm, further enumerating here did not yield any other info. Lets see if we
can mount the file.

## Mount and Guestmount
We can mount the SMB share to explore more. Let’s make a new folder in /mnt called
Bastion_mount. We will use this to mount the Backup share from 10.10.10.134.
```
mount -t cifs //10.10.10.134/BACKUPS -o username=" ",password=" " ./Bastion_mount/
```
![Image](/assets/bastion/Writeup_for_Bastion-004.jpg)

Since the note mentioned backups, let’s explore the WindowsImageBackup directory.
There, we find some juicy stuff  
![Image](/assets/bastion/Writeup_for_Bastion-005.jpg)

In particular are two .vhd files. These files are virtual hard disks and is the format used
to represent a virtual hard disk drive. They may have stored information on the windows
machine and looks like they were used for the backup the note referred to.  

However, these are huge files:  
![Image](/assets/bastion/Writeup_for_Bastion-006.jpg)

Using the information here:
"https://medium.com/@klockw3rk/mounting-vhd-file-on-kali-linux-through-remote-share-f2f9542c1f25" , we can mount vhd files without having to download the entire vhd.
From /mnt, lets create a new mount point for the vhd. Once that is done, we will run the
guestmount command.
```
guestmount -a /mnt/Bastion_mount/WindowsImageBackup/L4mpje-PC/Backup\
2019-02-22\ 124351/9b9cfbc4-369e-11e9-a17c-806e6f6e6963.vhd -m /dev/sda1 -r
/mnt/vhd1/ -o allow_other -v
```

There is alot going on here, but what we are doing is making a mount of the .vhd file to
another mount point named vhd1.
It will take a minute or so to complete, but once done, you will have the vhd mounted:  
![Image](/assets/bastion/Writeup_for_Bastion-007.jpg)

Samdump, Initial Entry, and the User Flag
Since this is a Windows virtual disk, lets try and dump the hashes of users with
samdump2. Move to windows/system32/config and run samdump2.
```
samdump2 SYSTEM SAM > /mnt/samdump_bastion.txt
```
This will get the hashes and output it to a text file in /mnt.
Once it is complete, you will have the following:  
![Image](/assets/bastion/Writeup_for_Bastion-008.jpg)

Great, we now have at least three users and their hashes.
Windows hashes are NTLM. Using: "https://hashkiller.co.uk/Cracker/NTLM" , able to find
the plaintext of one of the hashes:
```
31d6cfe0d16ae931b73c59d7e0c089c0 [No Match]
26112010952d963c8dc4217daec986d9 NTLM bureaulampje
```
Now we have a password for one of the users: L4mpje
Remember port 22 was open from our nmap scan. Lets test these credentials:
```
ssh L4mpje@10.10.10.134
```
Success!  
![Image](/assets/bastion/Writeup_for_Bastion-09.jpg)

Doing a quick check on our user using net user, we see that L4mpje is part of Users:  
![Image](/assets/bastion/Writeup_for_Bastion-010.jpg)

This may be enough to grab the flag:  
![Image](/assets/bastion/Writeup_for_Bastion-011.jpg)

We now have the flag from user.text

# Privesc
Windows Enumeration
Still using L4mpje account, we enumerate the Windows box some more. Powershell
usually is installed on Windows and with it, we can grab a list of programs on the box.
First, go into powershell with
```
powershell
```

Once you are dropped into powershell (the command prompt will turn to PS), run the
following:
```
Get-ItemProperty
HKLM:\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\* |
Select-Object DisplayName, DisplayVersion, Publisher, InstallDate | Format-Table
–AutoSize
```
We get a list of programs:  
![Image](/assets/bastion/Writeup_for_Bastion-012.jpg)

We see some Visual C++, but one other program stands out. mRemoteNG is an open
source program used to manage remote connections. A quick search on DuckDuckGo
or another search engine, reveals there are quite a number of exploits on mRemoteNG.

### mRemoteNG Exploitation
mRemoteNG has a particular vulnerability that we are interested in:
https://github.com/haseebT/mRemoteNG-Decrypt​ . Download the contents from this git
and save it for later.
However, you need to have a set of credentials to make use of this, so let’s see if we
can find something to use as the NTLM hashes and the current password we know are
not enough.
WIndows sometimes has data created by programs in a folder called AppData and is
usually hidden. Maybe the mRemoteNG program left some data behind in these folders.
To get to them from CLI, we will go through \Users\L4mpje. After some searching, we
find mRemoteNG and it still has some data:  
![Image](/assets/bastion/Writeup_for_Bastion-013.jpg)

So let’s find out what the most recent file has.
```
type confCons.xml.20190222-1403486580.backup
```
![Image](/assets/bastion/Writeup_for_Bastion-014.jpg)

From this backup, we can see that there is an Administrator user and an encrypted
password. Now, remembering the python script from mRemoteNG-Decrypt...  
![Image](/assets/bastion/Writeup_for_Bastion-015.pg)

We have a new password, this time for Administrator. Let’s use this to see if we can ssh
back in as this user.  
![Image](/assets/bastion/Writeup_for_Bastion-016.jpg)


Boom, we get root flag.

_Thank you for reading my writeup for Bastion. Shout out to Chmod755 crew and
hackthbox.eu._
