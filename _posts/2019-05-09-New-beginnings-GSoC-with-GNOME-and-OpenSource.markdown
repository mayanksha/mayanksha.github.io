---
title: "A New Beginning - GSoC'19 with GNOME and my thoughts on Open Source"
toc: true
toc_label: "Table of Contents"
toc_icon: "file-alt"
toc_sticky: true
categories: 
    - GSoC 2019
tags: 
    - GSoC
    - Open Source
    - GNOME
    - GVfs
    - libgdata
---


<img src="/assets/images/gnomeandgsoc.png" alt="GSoC + GNOME" style="display: block; margin: auto;">

## Prologue

***"Well begun is half done!"***

Or, rather the way my mom puts it - ***"Getting something to begin is half the work done, and the biggest hurdle overcome"***. This sentence has been really influential to me, for it makes me do things which otherwise I'd procrastinate and put off for later. Right from my freshmen year, I have used various opensource tools and distros, and hence, I consider myself a proponent of the whole ideology of free software. Like everyone else, I too started my \*nix journey with Ubuntu and pretty soon, I got annoyed by the bloatware that Canonical had stuffed in - Amazon App, that too on Sidebar? Are you really kidding me? That's when I searched for other Desktop Managers (with floating windows) and found that GNOME had one. I made a switch to GNOME then, but soon shifted over to Arch and that's when I felt how much I loved GDM. 

GDM worked well out of the box, and since Wayland is default these days, initially I didn't trouble myself with changing it. Thanks to Arch wiki, I had everything working, including the Nvidia Graphics. I loved how snappy GNOME felt whenever I pressed the Super key, its notification system, nautilus (File Manager), evince (Document Viewer) and the support for plugins was a cherry on the cake. I had some issues with WiFi Hotspot in Network Manager, but I eventually fixed them. 

## Introduction to GVfs

In 2018, I had gotten a VM for myself (GitHub Student Developer's pack) and I had to `scp` a lot for any file transfer. Now, `scp` is good for files being transferred from the same directory but for files spread across different folders, it is painful and cumbersome. I searched for some GUI tool and found that our beloved `nautilus` had integration with GVfs which supported the SFTP backend, along with a whole lot of other backends - MTP, AFP, FTP, SMB, etc. I was hooked! 

## My Journey - Newcomer Issues and Proposal

I always wanted to contribute to open source but call it for a lack of enthusiasm, or the innate ability to procrastinate, I never initiated anything. So, this year around February, I thought I should take my chance with GSoC given its prestige and my desire to contribute. 

During my mid-semester vacations (around March 18), I started looking for Orgs to apply to and resolve some low-hanging fruits. Since, I was doing a security course, I made a PR to The Honey Net Org (Cowrie Honeypot), but python was never my language of choice and hence I moved away from it. I had experience with Google Drive API, and FUSE file systems so I thought I'd give GVfs a shot. The GVfs ecosystem is HUGE, since it has so many different backends and each backend supports a different protocol, with each protocol having its own standard (from RFC or Google or Apple), and everything written in C. But I was determined to contribute to it, and hence I made the first contact over GNOME's IRC. The people there were very helpful, and I talked to my mentor over there. 

He pointed me to some Newcomer friendly bugs (mostly translation fixes), and I picked some others on my own. Around 29th March, I started working on my proposal and was in constant contact with my mentor. Everyday, he'd make some comments over my proposal and we'd talk and sort it out. With his green flag, I submitted my proposal on 7th April, two days before the deadline. Since, I hadn't contributed much (honestly, the issues I resolved were quite trivial), I told my mentor that I'd make some noteworthy contributions during the time of waiting for results. Alas! I got busy with course projects and assignments and couldn't pay much attention to it. I started with my end-semester exams around 23rd April (feeling all happy about my progress), but on 26th, I received a message from my mentor that I need to make some non-trivial contribution to GVfs or libgdata. The deadline to select projects was around 1st May, and I had my next exam on 28th and then final one of 30th. I was screwed!

So, I asked my mentor to point me to some issue (during the examinations), and set out on this herculean task. We narrowed down upon ["Support deleting shared Google Drive files"](https://gitlab.gnome.org/GNOME/libgdata/issues/26) and I started working on it. I had to pull an all nighter on 26th to identify where the issue exactly was, and how it might be fixed, along with studying for an exam on 28th. But, the coding was yet to be done.

One more all nighter on 28th and by 29th morning and I had written code which worked well functionally, albeit it crashed with SEGFAULT. My mentor tested out the code on 30th and since it was working partly, he told me to keep working on it. Over the next couple of days, I worked anxiously on it (the anxiety was due to my project being selected or not) and with help from my mentor (Ondrej Holy) and Philip Withnall, I was able to produce the fix. Those fixes have now been merged and shall make their way in the next GNOME release (3.34). 

## My GSoC Project

My project is titled - "Improve Google-Drive Support in GVfs". The current issue with Drive backend is that Google-Drive is a database backed system, meaning that files stored don't have a path per se, but are rather stored by their IDs with access privileges on each file/folder object. GIO (the library which performs IO) natively supports POSIX based file systems, but doesn't have know anything about any other protocols. Cometh GVfs. It's a separate package. It provides implementations for various backends, with help from different libraries and moulds the view of other protocols into the one GIO accepts. There's also a GVfs daemon which allows runtime  backends to be loaded as per needed at runtime. The Architecture is present here: [https://developer.gnome.org/gio/stable/ch01.html#gvfs-overview](https://developer.gnome.org/gio/stable/ch01.html#gvfs-overview).

Now, the major problem is that the files are not identified by their paths, but instead by their IDs. The filename which is shown in Google Drive Web App is nothing but a separate property which can't identify a file uniquely. This allows for two different files to have same name in the same folder (but different IDs), which is not possible with POSIX world. The GVfsGoogleBackend maintains different HashTables to account for the tree-like structure of file system, and uses `display-name` property of GIO to show the file names. Fortunately, nautilus also shows this `display-name` as the actual name of file. But as I said, this property is merely useful for any file operation. To actually copy/move/delete any file, you have to use the `id` of a file. 

From the actual [issue](https://gitlab.gnome.org/GNOME/gvfs/issues/8): Let's say that we want to copy some file, e.g. with the title `title1` and with `/id1` path to some folder with `/idfolder`. In this case we can call `g_file_copy ("/id1", "/idfolder/title1")`. This will create the copy of the file which will have `/idfolder/id2` path. The problem is that you don't know what is `id2` as `g_file_copy` doesn't return the new path, because it is not needed in POSIX world. Thus volatile file is created under `/idfolder/title1` path, which points to `/idfolder/id2`. This works nicely as the backend automatically creates those volatile files from the titles.

However, Nautilus calls `g_file_copy ("/id1", "/idfolder/id1")` in this case (the destination is not `/idfolder/title1`, but `/idfolder/id1`) and the backend returns Operation unsupported error. The problem here is that volatile files are automatically created from the titles. The backend returns the error to prevent loss of title because it would have to create a file with `/idfolder/id2` and title `id1`.

So far, we have thought about providing a new API in libgdata to support the `AppProperties` property over a file object in Google Drive, but it's too soon to state anything concretely. I'm positive that I'll be able to resolve this and a lot of other issues with GVfs and libgdata by the time I finish with GSoC.

## Ending Notes

I'd be updating this blog fortnightly, and I've set my milestones according to that. If you'd like to have a look at my proposal, [here it is](https://drive.google.com/open?id=1ORZRsImr0WOr8Ur-6KCxXxXW__jHTQbK). 

Finally, a big thanks to GNOME for believing in me and giving me this opportunity to contribute. I'd also like to thank my mentor Ondrej Holy for sitting patiently with me, for testing every change I make, and for guiding me. A big thanks to Philip Withnall (libgdata Maintainer, for his review of my code, and patiently answering my questions) and Debarshi Ray (for his excellent [post on volatile paths](https://debarshiray.wordpress.com/2015/09/13/google-drive-and-gnome-what-is-a-volatile-path/)). 

There's a lot to be done here, and I sure want to stick with GNOME in the long run. To every aspriring GSoC'er, keep Coding and keep spreading the love for Open Source! <3

