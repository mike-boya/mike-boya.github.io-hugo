---
title: "Metasploit CTF 2020 - Queen Of Diamonds Write-Up"
date: "2020-02-04"
slug: "metasploitctf2020-queen-of-diamonds-write-up"
Categories:
- CTF
- Stego
---

## Intro

My team and I participated in the Metasploit CTF this past week and came in third place! I wanted to write up a solution for one of my favorite challenges. 

Special thanks to the Metasploit team for creating another great CTF and congrats to team pepega and [excusemewtf](https://twitter.com/excusemewtf_ctf) for taking the top spots!

## Challenge

First, I browsed to port 80 and was met with this screen:

![80](/images/metasploit-ctf_2020/port80.png)

Interesting, I kicked off a **gobuster** scan but returned to find no results. The icon in the top left looked suspicious, so I loaded it directly.

![ms logo](/images/metasploit-ctf_2020/ms_logo.png)

Wow, a QR code...

SPOILERS BELOW - The picture linked above is the exact image from the challenge, so feel free to give it a shot before reading the solution.

<!--more-->

## Solution

I cleaned up the QR code with **gimp** in order to scan it.

![cleaned QR](/images/metasploit-ctf_2020/cleaned_qr.png)

Now, time to scan. I used [zxing](https://zxing.org/w/decode.jspx):

![zxing](/images/metasploit-ctf_2020/qr_decode.png)

Well, that's no flag. However, this output reminded us of a similar message we encountered during the initial enumeration of the box.

During the initial enumeration we ran a full **nmap** scan:

![Nmap](/images/metasploit-ctf_2020/nmap_all_ports.png)

We saw port 1994 open, so we ran some quick tests against it:

![Port 1994](/images/metasploit-ctf_2020/1994.png)

See, this also mentions a cigar. I did attempt to submit the QR code and full ms_logo to the port with **nc** but no luck. I started analyzing the ms_logo image with a number of Stego tools, but wasn't getting any usable results. However, that band of color in the top left corner kept bothering me. I took a closer look with **gimp**:

![Color Band](/images/metasploit-ctf_2020/color-band.png)

Then, I wrote a small Python script to print out those unique pixels:

{{< highlight python >}}
from PIL import Image

im = Image.open('ms_logo.png')
rgb_im = im.convert('RGB')

#print('width: ', im.width)
#print('height:', im.height)

for i in range(im.height):
  r,g,b = im.getpixel((13,i))
  if r != b and b != g:
    print(r,g,b)
{{< /highlight >}}

Here are the results:

{{< highlight bash >}}
python3 color-column.py | head -n 3
244 243 246
248 246 255
246 255 247
{{< /highlight >}}

Now, I don't want anyone reading this post to think I instantly recognized this. I had a suspicion that Least Significant Bit (LSB) stego was a possibility, but I stared at these numbers for awhile and even jumped to other challenges. Finally, I came back and took a look with CyberChef:

![LSB Analysis](/images/metasploit-ctf_2020/cyberchef_lsb_analysis.png)

I see a pattern here. Let me illustrate it a little better:

![LSB](/images/metasploit-ctf_2020/lsb.png)

The least significant bits, four to be exact, are changing. I wrote a Python script to pull these out:

{{< highlight python >}}
#!/usr/bin/env python3

from PIL import Image

im = Image.open('ms_logo.png')

for i in range(13,65):
    pix = im.getpixel((13,i))
    for n in range(0,3):
      print(bin(pix[n])[-4:],end="")
{{< /highlight >}}

Here is the output after running the tool:

{{< highlight bash >}}
$ python3 colors.py
010000110110100001101111011011110111001101100101001000000100110001101001011001100110010100101110001000000100001001110101011101000010000001110111011010000111100100100000011101110110111101110101011011000110010000100000010010010010000001110111011000010110111001110100001000000111010001101111001000000110010001101111001000000110000100100000011101000110100001101001011011100110011100100000011011000110100101101011011001010010000001110100011010000110000101110100001000000111011101101000011001010110111000100000010010010010000001101000011000010111011001100101001000000110100001100001011000110110101101101001011011100110011100111111
{{< /highlight >}}

I collected the binary and ran it through CyberChef:

![CyberChef Binary Decode](/images/metasploit-ctf_2020/cyberchef_binary_decode.png)

That looks like what we have been searching for! Moving back to the port I mentioned earlier, I tried submitting the string:

![Phrase Submission](/images/metasploit-ctf_2020/1994_submission.png)

As you can see, it confirms we cracked the code and sends flag data. I redirected this information to a file:

{{< highlight bash >}}
$ echo -n "Choose Life. But why would I want to do a thing like that when I have hacking?" | nc 172.16.1.101 1994 > qr.txt
{{< /highlight >}}

I opened it in **vim** and deleted the top two lines. The image did open and display fine, but I ran a quick run with **pngcheck** to verify the file:

{{< highlight bash >}}
$ pngcheck qr.png
qr.png  additional data after IEND chunk
ERROR: qr.png
{{< /highlight >}}

Luckily, I have seen this before:

```bash
xxd qr.png | tail -n 5
0000a7e0: 3734 6dda 1484 481f 3db7 d4f9 8bd9 d851  74m...H.=......Q
0000a7f0: 5c5c 8cfd fbf7 e3f8 f1e3 2829 f12c fa05  \\........().,..
0000a800: 1144 10be 0741 1048 4d4d 45fb f6ed 45d9  .D...A.HMME...E.
0000a810: ce88 c1ff 0102 5199 95f0 4bd1 1e00 0000  ......Q...K.....
0000a820: 0049 454e 44ae 4260 820a                 .IEND.B`..
```

This is a quick fix. I opened the png in a hex editor and deleted the trailing 0a.

Now it should pass the check:

{{< highlight bash >}}
$ pngcheck queen_of_diamonds.png
OK: queen_of_diamonds.png (283x378, 32-bit RGB+alpha, non-interlaced, 89.9%).
{{< /highlight >}}

After making these modifications, we can view the final flag!

![Queen of Diamonds](/images/metasploit-ctf_2020/queen_of_diamonds.png)

{{< highlight bash >}}
$ queen_of_diamonds.png
MD5 (queen_of_diamonds.png) = 01880728bf92ae2570ce3c0cd4b01d8b
{{< /highlight >}}

Thanks for reading!
