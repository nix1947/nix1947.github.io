---
layout: post
title: "Sftp server configuration in debian"
date: 2016-09-6 
description: sftp configuration
tags: [system-administration]
comments: true

---
SFTP stands for SSH file transfer protocol, It is not the actual FTP server, rather the extension of SSH, which uses SSH tunnel to transfer the files between server and client.

SFTP is more than SCP, as it supports more features than SCP with an integration of remote SCP client. If you want to learn more about SFTP check out this [wiki](https://en.wikipedia.org/wiki/SFTP).

OK, to configure SFTP we don't need to install extra packages, as I have already told you, it's just an extension of ssh, you might be guessing,  you need to make some tweaks to your `sshd_config` file, you are right.

OK, Open your `sshd_config` file, which resides under `/etc/` directory, feel free to use any text editor you want. I love VIM.

```
vim /etc/ssh/sshd_config
```
Inside `sshd_config` file on line no `77` change, `#Subsystem sftp /usr/lib/openssh/sftp-server` to
{% highlight python %}
 Subsystem sftp internal-sftp
{% endhighlight %}

Next, we need to define a policy,  based on user or a group. At the last of a `sshd_config` file, put the following code given below.

{% highlight python %}
 Match Group Sftp_Group
 ChrootDirectory %h
 ForceCommand internal-sftp
 AllowTcpForwarding no
 PermitTunnel no
 X11Forwarding no 
{% endhighlight %}

The above policy tells sftp server to authenticate all the users which belong to `Sftp_Group` and Chroot them only into their home directory,
so that they can't browse and access the parent directories and other user directories.

One important thing to keep in mind while configuring sftp server is, the parent directory must be always owned by a `root` user. Let's say, We want to export the home directory of a user test and we want her to confine into her home directory only. To do that first we need to create a test user with a home directory `test`(of course, you can create a home directory with any name) and make her a member of `Sftp_Group`, such that the parent directory of a test user must own by the root user. Let's create a test user.

{% highlight python %}
 useradd -m -d /home/test -s /usr/sbin/nologin -G Sftp_Group -g Sftp_Group 
{% endhighlight %}

The above command adds a user `test` with a home directory `test` and a secondary group `Sftp_Group`.  As we know parent directory should be owned by the root user. The user `test` has been created. Let's set the password for a test user.

{% highlight python %}
  passwd test
{% endhighlight %}

The above command will let you set a password for user `test`. Using password command we have also set a password for user test. 
Now it's time to create a **public_html** directory under the /test directory, such that it is owned by the user test and the parent directory of this **public_html** is owned by a user root.

{% highlight python %}
 mkdir /home/test/public_html
 chown test:test /home/test/public_html
 chown root:root /home/test
{% endhighlight %}

With the above series of commands, we can now access the sftp server via sftp client.

To test using SFTP protocol, You can use any SFTP client, in market there are many sftp clients some of them are advance with a feature of a window explorer integrations. For testing purpose. *Filezilla* would be a perfect choice for us.

One final thing, Before connecting to our SFTP server, we need to restart our SSH server.

{% highlight python %}
 systemctl restart ssh
{% endhighlight %}

To connect to the SFTP server, we need to prefix the server name with a a sftp prefix as shown below.
<img src="/images/sftp_and_filezilla.png" alt="sftp server testing" width="130%">
