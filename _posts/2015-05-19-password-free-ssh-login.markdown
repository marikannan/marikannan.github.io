---
layout: post
title: "Password free SSH login"
date: 2015-05-19 21:32:38 +0530
comments: true
categories: [ Linux ]
---
This post is about setting up password-free secure shell login ( ssh ) in linux machine. This is specific to SSH/SSH2 protocol. For this , I am considering two linux systems with hostnames kaveri (hostname1 -  source) and vaigai (hostname2 - client). If you wish, you could also use the ip addresses instead of hostname.

## Generate rsa keys
Linux utility `key-gen` can be used to generate public/private keys ( based on [rsa asymetric algorithm](https://en.wikipedia.org/wiki/RSA_(cryptosystem))). When it asks for passphrase, just enter.

``` bash 
user@kaveri:~$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/user/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/user/.ssh/id_rsa.
Your public key has been saved in /home/user/.ssh/id_rsa.pub.
The key fingerprint is:
e143:fa:e2:fc:ab:1d:ff:d8:3a:e9:3e:55:95:fd:51:04 user@kaveri
The key's randomart image is:
+--[ RSA 2048]----+
|             Do.o|
|               oo|
|   o    .       =|
|       . .     .o|
|        S      . |
|       .      .  |
|      .   .  o   |
|     o.o . o+o   |
|     ..o=..o*=o  |
+-----------------+
user@kaveri:~$
```

## Files generated by key-gen
`key-gen` will create three files `known_hosts`, public key (`id_rsa.pub`) and private key (`id_rsa`). By the name, public key is something that has to be distributed in other linux boxes to which you want to ssh/scp/sftp without password and private key should be kept (kinda secured, thats why it is in hidden directory .ssh ) in source machine, should not be distributed.

``` bash 
user@kaveri:~$ cd .ssh
user@kaveri:~/.ssh$ ls -rlt
total 12
-rw-r--r-- 1 user user 1990 Apr 25 21:53 known_hosts
-rw-r--r-- 1 user user  399 May 19 21:38 id_rsa.pub
-rw------- 1 user user 1675 May 19 21:38 id_rsa
user@kaveri:~/.ssh$
```

## Distribute the public key to client machines
For this, ensure the directory .ssh does exist and append `id_rsa.pub` file with the `authorized_keys` file by executing the below commands in source machine.

``` bash 
user@kaveri:~/.ssh$ ssh user@vaigai mkdir -p .ssh
user@kaveri:~/.ssh$ cat id_rsa.pub | ssh user@vaigai \
				'cat >> .ssh/authorized_keys'
```

> In case of SSH2 protocol version, authorized_keys2 has to be used instead of authorized_keys. Also modify the permission of .ssh to 700 and authorized_keys2 to 640

## Check ssh (password-free) working
This can be done by secure-shell-ing to vaigai and execute the `uname` command in the client machine. If everything goes fine without prompting for password, you are fine to go.

``` bash 
user@kaveri:~$ ssh user@vaigai "hostname"
vaigai
```

