---
title: "Offensive Security Certified Professional (OSCP): My Experience"
date: "2015-07-07"
slug: "offensive-security-certified-professional-oscp-my-experience"
aliases: 
- /blog/2015/07/07/offensive-security-certified-professional-oscp-my-experience/
Categories:
- OSCP
- Offensive Security
---

Introduction
-------------

It was a long ride, but I finally finished my OSCP certification by completing the lab portion and passing the practical exam. I learned so much during the course and earned what I feel is a cert worth its weight in gold. As I have mentioned in previous blog posts, I take pride in guiding my professional development and I felt that taking a hands-on penetration testing course would be a great challenge and learning experience. This post summarizes my thoughts on the entire course and process.

<!-- more -->

Course Registration
--------------------

Offensive Security (offsec) offers 30, 60, or 90 days of lab time — I selected the 90 day option. After reading a plethora of reviews, I felt that it would help me to take my time and approach the course in a way that would allow me to learn the most information possible. Just a heads up - registration does require the use of a non-free email address (no gmail, yahoo, etc). I could expand on how the process works, but other reviews do an amazing job and the official FAQ page summarizes some additional info about registration here: http://www.offensive-security.com/faq/. 

The Course Materials
---------------------

The materials are provided to help guide your learning and development. Offsec supplies the student with ~300 page PDF lab guide and a number of video segments. The lab guide contains a number of exercises that will assist with learning by having the student perform the task at hand. I found myself very familiar with some topics (SSH tunneling) and completely in the dark on the others (buffer overflows). I made sure that I took the time to perform extra research and practice the topics I was not as familiar with. I do recommend completing all the exercises, because it pays off in the long run and will even assist you with some of the boxes in the lab.


The Lab
--------

The lab is by far the best part about this course. If you have ever spun up a vulnerable VM (metasploitable, De-ICE, etc.) to practice attacking techniques, you know that you need to configure and setup your "lab" -- which can result in you learning hints about the machine. The Offsec lab gives you access to ~50 machines spread out across four networks that range in difficulty which was quite a change from attacking a single vulnerable VM. As you progress in the lab, you will uncover juicy details about the users and ultimately start to refer to the machines by name - aka "last night I rooted pain."

Personally, I tried to own as much of the lab as possible. It taught me so much and gaining SYSTEM/root on some of the tougher systems definitely prepared me for the practical exam. At the end of my lab time, I got all the systems with the exception of two, I decided to start preparing for the exam instead of completing these boxes.

In terms of documentation, it really comes down to the student. The course guide recommends KeepNote, but I was more comfortable creating a directory structure and working with flat files that aligned with my daily process at work and my IR experience. My advice is to find what style works for you and document as much as possible. Enumeration is key and it is worth keeping all of your enumeration results so you can rotate to different machines in the lab and not re-scan every time. 

If I could change one thing about my lab time, I would have drafted the final report as I went along. I read reviews that mentioned the same tactic but it is difficult to type up your step-by-step guide when you are running around your house celebrating because you got root! I did, however, use this tactic for the exam, which I describe later. 

When I was concentrating on the lab, I spent around four hours a day during the week and up to eight hours on the weekends. Just a quick warning - as soon as you sign up for this course and commit to spending all your time doing it, life will get in the way. Be prepared to isolate yourself to get it done. You need to put in the proper time to truly get the most out of this course.

The Exam
---------

As many other OSCP reviews have mentioned, there are limitations on the use of Metasploit as well as automated vulnerability scanners such as Nessus or OpenVAS. I wouldn't lean on these tools too heavily in the lab but do not avoid them either. The lab is there for you to try different tools and techniques and there are different ways into each box, use that practice time to your advantage.  

Offsec allots you 24 hours to complete the exam and 24 hours to draft your accompanying report. I scheduled the exam for Saturday morning and ensured that I had enough coffee, snacks, energy drinks, music, etc. for the full 48-hour period. I received the email right on time, dove into the exam guide, connected to the VPN and buckled down on achieving my goal. After about eight hours, I had enough points to pass and I focused on gathering all the information and forming a rough draft of screenshots and evidence for my report to simply the report generation. I spent another two hours drafting a complete report. The next morning, I woke up and proofread the report, and finished tweaking any formatting. After I felt comfortable with my documentation, I submitted my report and received the notification that Offensive Security had received my deliverables. I should have felt confident at this point, but I couldn't fight the doubt that I had forgotten something. All that was eradicated when I received the notice that I had successfully completed the certification challenge early Tuesday morning.

As far as recommendations for the exam, make sure that you are prepared for highs and lows. There will be times during your 24 hours that you will doubt your skills and think you were not ready for the test -- use that time to take a break and go for a walk or grab some grub. The other tips I have for the exam are: 1.) Enumerate until you understand the box like the SysAdmin would -- what's running, any special configurations, any vulnerable versions? 2.) Organize your time and don't become hyper-focused on one machine, multitasking helps. 3.) Compile a very rough draft of your report as you go. It was so helpful to have everything prepared and laid out for my report before my VPN access was clipped. It reduced my stress about missing something important. Don't get too excited when you root a box and forget to collect the associated information and screenshots.

Conclusion
-----------

This was the most challenging and rewarding course that I have ever experienced. You truly get what you put in with the OSCP. I spent hours (cough... cough... days) on some of these machines and learned so much by trying different attacks in the immense "playground" that Offsec generously assembled. My recommendation for those interested in taking the course is to spend as much time in the labs as possible. The motto "Try Harder" and the more difficult boxes will frustrate you but I promise when you finish the course you will miss the lab environment. The course was well worth the time and money and I have increased not only my attacking skills, but my overall security skills in the process.

Feel free to comment or message me on twitter if you have any questions!

-Mike (<a href="http://twitter.com/mikeboya">@mikeboya</a>)
