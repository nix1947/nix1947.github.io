---
title: "samba4 server configuration in debian linux"
date: 2016-12-23 
description: samba server configuration in debian 8
categories: samba
---

Samba is file and print service, which is used to share files between different platforms such as windows, linux and unix environment.
In this post, I am going to show you, how to install and configure the *SAMBA* server as a file sharing server.
First, we will create a different users, which belongs to  different groups, and we will also create a directory Engineering and Administration, such that Administration directory consists of extra three directories named as HR, Logistic, and Finance as shown in the tree structure below.

```
└───Administration
    ├───Finance
    ├───HR
    └───Logistic
```

{% highlight bash %}
 magautam@manoj1947:~$ cd /home
 magautam@manoj1947:~$ sudo mkdir Engineering Administration
 magautam@manoj1947:~$ sudo cd Administration && mkdir Finance HR Logistic
{% endhighlight %}

So, according to our directory structure, let us create a different group, which will have different types of permission level to the different directories as shown below.

*Finance_Group*: will have, read, write and execute permission to Finance directory
*HR_Group*: will have,  read, write and execute permission to HR directory.
*Logistic_Group*: will have read, write and execute permission to Logistic directory
*Engineering_Group*: will have read, write and execute permission to Engineering directory
*Finance_Group*: will have read and execute permission to Logistic directory.

{% highlight bash %}
 root@manoj1947:~# groupadd Engineering_Group
 root@manoj1947:~# groupadd Logistic_Group
 root@manoj1947:~# groupadd Finance_Group
 root@manoj1947:~# groupadd HR_Group
{% endhighlight %}


so, we are done with the permission policy, now let us create four users, which belongs to different groups

I have identified the following user which belongs to the following groups.

*Manoj Gautam* belongs to *Engineering_Group*
*Kevin Khadka* belongs to *Finance_Group*
*Aliza Shrestha* belongs to *HR_Group*
*Pratish Shrestha* belongs to *Logistic_Group*

So, now let us add these users to our Linux server.

{% highlight bash %}
 magautam@manoj1947:~$ sudo useradd -m -d /home/Personal\ Folders/manoj  -s /bin/bash -c "Manoj Gautam" -g Engineering_Group -G Manoj1947_Group pshakya
{% endhighlight %}

Let us add user aliza, who belongs to HR_Group.

{% highlight bash %}
 magautam@manoj1947:~$ sudo useradd ashrestha -m -d /home/Personal\ Folders/aliza -g HR_Group -G Manoj1947_Group -c "Aliza Shrestha" -s /sbin/nologin
{% endhighlight %}

Let us add user pratish, who belongs to Logistic_Group.

{% highlight bash %}
magautam@manoj1947:~$ sudo useradd pshrestha -m -d /home/Personal\ Folders/pratish -g Logistic_Group -G Manoj1947_Group -c "Pratish Shrestha" -s /sbin/nologin
{% endhighlight %}

Let us add user Kevin, who belongs to Finance Group.
{% highlight bash %}
magautam@manoj1947:~$ sudo useradd kkhadka -m -d /home/Personal\ Folders/kevin -g Finance_Group -G Manoj1947_Group -c "Kevin Khadka" -s /sbin/nologin
{% endhighlight %}

So, the above series of command will add the Linux user to our Linux Samba system with respective groups.

We are done with the user and groups creation, now it is time to install samba server in our  ubuntu server.

{% highlight bash %}
sudo apt-get upgrade && sudo apt-get install samba
{% endhighlight %}

The configuration file of the samba services is located under `/etc/samba/smb.conf`, so we will share our directories,
using this file to the outside world.

Ok, before sharing our directories, let us set the permission to our directories, we can set the permission using UNIX file permission methods, but for the complex permission strategies  we can use ACL also called Access control list,  by default ACL packages is not installed, so we need to install it.

{% highlight bash %}
 sudo apt-get install acl
{% endhighlight %}

ACL has been installed, so we got the setfacl command to set the permission in our directories.

Before playing with setfacl,  ACL must be supported by our file system. To check whether the acl has been supported or not, just issue the following commands against your file system.

{% highlight bash %}
 sudo tune2fs -l /dev/sda1 | grep acl
{% endhighlight %}

If you see the output like  Default mount options: user_xattr acl  that's good news, our file system support acl.
To enable ACL in our file system, just edit fstab file and append acl in mount option as shown in the snapshot below.

<img src="/images/acl.png" alt="Access control list">

Enabling the ACL has been done, now its time to apply the ACL policies in our directories.
As from our previous policy, we want, only Engineering_Group can access the Engineering directory, to set this permission, use  setfacl command as shown below.

{% highlight bash %}
 sudo setfacl -Rm g:Engineering_Group:rwx,o:--- Engineering/
 sudo setfacl -Rm d:g:Engineering_Group:rwx,o:--- Engineering/
{% endhighlight %}

The first command provides the read, write and execute permission in Engineering directory to all those users who belongs to the *Engineering_Group* , but for others, no read, write and execute permission.

The second command will set the default read, write and execute permission to the file and directories which will be created in the future.

Similarly, we can create the permission for other directories as well.

{% highlight bash%}
 #only logistic_group can have read, write and excecute permission to Logistic directoy.
 sudo setfacl -Rm g:Logistic_Group:rwx,o:--- Logistic/
 sudo setfacl -Rm d:g:Logistic_Group:rwx,o:--- Logistic/

 #only HR_Group can have read, write and execute permission to HR directory.
 sudo setfacl -Rm g:HR_Group:rwx,o:--- Hr/
 sudo setfacl -Rm d:g:HR_Group:rwx,o:--- Hr/
{% endhighlight %}

We also want to have the read only permission to Logistic directory for Finance_Group, so let's set the permission as well.

{% highlight bash %}
 sudo setfacl g:Finance_Group:r-x Logistic/
 sudo setfacl d:Finance_Group:r-x Logistic/
{% endhighlight %}

Also, the Administration directory must have read and execute permission to others as well, which is r-x mode by default.

So, we are done with the permission, now it is time to share the directories. To share the directory, we need to edit the smb.conf file and share the directories as shown below

{% highlight bash %}
 [global]
        server string = %h server (Samba, Ubuntu)
        server role = standalone server
        map to guest = Bad User
        obey pam restrictions = Yes
        pam password change = Yes
        passwd program = /usr/bin/passwd %u
        passwd chat = *Enter\snew\s*\spassword:* %n\n *Retype\snew\s*\spassword:* %n\n *password\supdated\ssuccessfully* .
        unix password sync = Yes
        syslog = 0
        log file = /var/log/samba/log.%m
        max log size = 1000
        unix extensions = No
        dns proxy = No
        usershare allow guests = Yes
        panic action = /usr/share/samba/panic-action %d
        idmap config * : backend = tdb
        wide links = Yes

[Engineering]
        comment = "Share Engineering directory"
        path = /home/Engineering
        read only = No

[Administration]
        path = /home/Administration/
        read only = No

{% endhighlight %}

So, now you are done with Sharing and setting the permission, it's time to test our setup.
Open your window machine and access the Samba server as shown below, as my server IP is 192.168.10.4 as shown below

<img src="/images/share.png" alt="File Sharing">
<img src="/images/share11.png" alt="Shared directories">

So, let us click to Administration directory, When I clicked to Administration directory, it prompt me to enter the network credentials.  Let us try to login to our server using kkhadka username and it's password., as kkhadka has the permission to access the Finance folder with read and write permission and Logistic with only read permission.

<img src="/images/snap1.png" alt="Login dialog">

After entering the valid credentials, I was able to login and access the Finance folder as shown below.

<img src="/images/snap2.png" alt="Shared directories">
<img src="/images/snap3.png" alt="shared directories">

But when I try to access the HR folder, with same credentials, as window OS remember our credentials, I was denied to access the HR Folder.
<img src="/images/snapr.png" width="120%" alt="Authentication problem">

So, that's it on setting up the Samba server as a standalone File server, We can do much more with samba server, we can set up a samba server as a PDC(Primary Domain Controller) and much more, to explore more about samba server please visit samba [documentation](https://www.samba.org/samba/docs/).
