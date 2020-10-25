---
title: "How to become admin during exams"
description: "Poking around the University exam interface to do unexpected things"
keywords: [writeup, security, university, windows, pdrive]
date: 2019-05-25
---

This is not a CVE writeup or Privilege escalation for Windows, its just a badly configure exam environment and my experience by poking it.

## Context
---

At The University of Manchester some exams you take may be *online*. This means that the exam will take place in one of the PC clusters, you’ll be assigned a seat, log in with your username and password in what seems to be a web-login screensaver. After this a Firefox window will open and you’ll be asked to navigate to the [Blackboard page](http://online.manchester.ac.uk/) and select the course. There you’ll find the exam paper locked behind a password (5 alpha-numerical long). When the exam starts you get the password and start answering the questions.

![Blackboard Input Field](/assets/images/blackboard.png)

## Restrictions
---

* Have a single browser window open (you can "Open in New Window" at any time tho)
* Only access University domains (if only we could proxy or cross site)
* Can only open: Firefox, Internet Explorer, Calculator (if allowed for the current exam)

The above are the only things somebody expects you to be able to do. Staff wouldn’t want you to let’s say print stuff, or download foreign files, or execute foreign programs … but you can.

How? It all started with P-drive.

## P-Drive
---

P-Drive is a nice web interface provided by the university. It allows students to access (upload, delete, download, rename, etc) files from their Windows file system.

![P-Drive](/assets/images/pdrive.png)

When I realised I have access to this, I went on and tried to upload files from the local exam machine. **It worked**. No only could I upload files, but this made me realise I don’t need **File Explorer**, I have a browser. These days browsers are like Swiss Multitools, they can do anything. So now, I can browse the filesystem (discovered that the exam machines have the Tor Browser installed … interesting?), users have read permissions pretty much everywhere but can’t write anywhere. Fair is fair, *but oh yeah*, you can still write/download files to the browser installation directory. Sweet.

An extra “feature” for CS students, is the shared directory between the Linux and Windows file systems. `public_html`, not only is it shared between the two, it also has a PHP web server accessing it. Any files you place there are server on the University CS domain (sadly not wishing the `manchester.ac.uk` domain, if somebody can find a way to load the content from `public_html` from a `manchester.ac.uk`, would be nice). Anyway you don’t have to load web-pages to get content on your exam machine, you can simply upload whatever you want on the P-Drive and then download it or view it in the browser.

## Run anything
---

I said you can only run Firefox, IE and the Calculator. So did the staff think, in reality you can open any app as long as you use Firefox to open it. You have to love Windows with all its *features*. I managed to open: Outlook (and connect to my email as it is within `manchester.ac.uk` domain), Notepad and the Control Panel.

> I wanted to run Paint, but they removed it :(

Any executable you upload to P-Drive can then be downloaded and executed. Glorious.

## Small stuff
---

Don’t have calculator access? Or the calculator doesn’t have the *computational power* you demand? Don’t worry, just use **JavaScript**. Pop open the browser developer console and type in whatever you want.

Do you want to change that ugly purple wallpaper that burns your retina? Simple,  download your favourite image from P-drive and set it as wallpaper.

![Wallpaper Change](/assets/images/exam_wp.jpeg)

Do you want a copy of the exam questions but nobody will give it to you? Print them. You can do this in two ways, start the printing service using the browser or save the exam page as `.html` then upload it to your P-Drive.

## Conclusion
P-Drive access and the weird exam user permissions can easily lead to many issues and exploits. I haven’t done anything malicious but somebody with that intent could easily do it:

* You can upload HTML files to P-Drive with the course content, the page being HTML can be made to look like the blackboard exam theme, making it very hard for anyone else to notice.
* P-Drive could be used as a communication medium between you and somebody on the outside. You could provide the question and they can provide the answers.
* Access to the JavaScript console is also quite exploitable by CS students. Some problems may require you to provide algorithms, with JavaScript available you could test your algorithms before submitting.

There are endless possibilities with these exploits available. If somebody had the malicious intent, something quite complex could be setup using these.
