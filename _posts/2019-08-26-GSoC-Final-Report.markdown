---
title: "GSoC'19 - A Final Report"
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

It has rightly been said - "All good things come to an end". Google Summer of Code too was one of the good experiences I've had, in the sense that I didn't know anything about the Open Source world. It provided the exact platform that I needed to kickstart my open source contributions and to let me feet as fantastic a community as GNOME.

For a detailed explanation of my project and the Google Backend for GVfs, please see [my last post](/gsoc%202019/GSoC-GVfs-and-Google-Backend-demystified/).

## What we achieved?

Over the past 3 months, I've been contributing in the capacity of a GSoC intern to [GNOME](gnome.org), specifically to the [GNOME Virtual File System (GVfs)](https://gitlab.gnome.org/GNOME/gvfs). For the most part, I've been able to achieve what me and my mentor had expected from my work, but there have been a few lapses as well.

Speaking of what happened during these 3 months, I've done a some extra work as well. The first thing I needed to do was to create a new libgdata API  which could support "Properties Resource" on file objects. At that time, libgdata was using autotools as its build system. Now, autotools is really an archaic system and quite prone to bugs, so that the latest version at that time (1.16.1) completely broke the build of libgdata.

Upon discussion with Philip Withnall (@pwithnall), I decided that I'd take the onus of porting libgdata to meson. My mentor asked me to contact Inigo Martinez who's had solid experience on porting GNOME projects to meson in the past as well. So, me and @inigo both forged our way to a meson port and eventually we were able to drop autotools completely. This was no-where mentioned in my proposal, and I didn't have the slightest clue that I'll be writing build files for meson as well. I had to learn everything from documentation and from earlier `meson.build` files written for other projects.

Then, upon discussion with my mentor I started testing the newly created libgdata API (`GDataDocumentsProperty`) in GVfs. The first problem we wanted to fix was copy operation which I was able to do within a couple of days. The approach we followed was that we'd solve the easier cases first and then move on to more and more complex cases. I started off with just 8 cases in my mind, but soon realized that the number of cases can be very high (owing to files having multiple parents, shared files, their behaviour while copying and moving, etc). Everyday, I'd hit some new test-case and the backend would crash abruptly. Moreover, debugging such a complex piece of code is really difficult and each time I executed google daemon for testing, I'd do it inside gdb (just to make sure I don't miss on anything).

At the moment of writing this post, the [MR for this](https://gitlab.gnome.org/GNOME/gvfs/merge_requests/58) fix is being reviewed and I'm happy to say that the copy/move functionality works as expected. We even support having files with multiple titles in the same folder, something like below:

<img src="/assets/images/multiple-files-same-titles.png" alt="Operation unsupported error" style="display: block; margin: auto;">

The above image may look quite shocking to people used to POSIX file systems, but it's all too common in a database-backed system like Google Drive. Moreover, we now support all operations (like rename, move, copy, delete, make_directory, replace, etc.) on such files as well. This is possible because under the hood, each file is uniquely identified by an ID and we use those IDs to create paths for these files in GVfs.

## But where's the code?

Below is a table which lists whatever I've done during GSoC'19. All the code that I've written is directly submitted to the Gitlab instance of GNOME and will eventually make way for GNOME's 3.36 stable release (we kinda missed 3.34 release because of a clashing feature freeze).

<table class="table table-bordered table-hover table-condensed">
<thead><tr><th title="Field #1">Sr. No</th>
<th title="Field #2">Project</th>
<th title="Field #3">What I did?</th>
<th title="Field #4">Merge Request or Code Link</th>
<th title="Field #5">Status</th>
</tr></thead>
<tbody><tr>
<td align="right">1</td>
<td>libgdata</td>
<td>Support Deletion for Shared files in Google Drive</td>
<td><a href="https://gitlab.gnome.org/GNOME/libgdata/issues/26">https://gitlab.gnome.org/GNOME/libgdata/issues/26</a></td>
<td>MERGED</td>
</tr>
<tr>
<td align="right">2</td>
<td>libgdata</td>
<td>Port to Meson</td>
<td><a href="https://gitlab.gnome.org/GNOME/libgdata/issues/27">https://gitlab.gnome.org/GNOME/libgdata/issues/27</a></td>
<td>MERGED</td>
</tr>
<tr>
<td align="right">3</td>
<td>libgdata</td>
<td>Drive v2 Properties API</td>
<td><a href="https://gitlab.gnome.org/GNOME/libgdata/merge_requests/7">https://gitlab.gnome.org/GNOME/libgdata/merge_requests/7</a></td>
<td>MERGED</td>
</tr>
<tr>
<td align="right">4</td>
<td>libgdata</td>
<td>meson: Fix G_LOG_DOMAIN for shared_library as well as demos</td>
<td><a href="https://gitlab.gnome.org/GNOME/libgdata/merge_requests/11">https://gitlab.gnome.org/GNOME/libgdata/merge_requests/11</a></td>
<td>MERGED</td>
</tr>
<tr>
<td align="right">5</td>
<td>gvfs</td>
<td>google: Support deleting shared Google Drive files</td>
<td><a href="https://gitlab.gnome.org/GNOME/gvfs/merge_requests/46">https://gitlab.gnome.org/GNOME/gvfs/merge_requests/46</a></td>
<td>MERGED</td>
</tr>
<tr>
<td align="right">6</td>
<td>gvfs</td>
<td>google: Check ownership in is_owner() without additional HTTP request</td>
<td><a href="https://gitlab.gnome.org/GNOME/gvfs/merge_requests/49">https://gitlab.gnome.org/GNOME/gvfs/merge_requests/49</a></td>
<td>MERGED</td>
</tr>
<tr>
<td align="right">7</td>
<td>gvfs</td>
<td>google: Fix issue with stale entries remaining after rename operation</td>
<td><a href="https://gitlab.gnome.org/GNOME/gvfs/merge_requests/52">https://gitlab.gnome.org/GNOME/gvfs/merge_requests/52</a></td>
<td>MERGED</td>
</tr>
<tr>
<td align="right">8</td>
<td>gvfs</td>
<td>google: Fix erroneous unreffing of resolved entry in out label</td>
<td><a href="https://gitlab.gnome.org/GNOME/gvfs/merge_requests/56">https://gitlab.gnome.org/GNOME/gvfs/merge_requests/56</a></td>
<td>MERGED</td>
</tr>
<tr>
<td align="right">9</td>
<td>gvfs</td>
<td>google: Fix move and copy operations</td>
<td><a href="https://gitlab.gnome.org/GNOME/gvfs/merge_requests/58">https://gitlab.gnome.org/GNOME/gvfs/merge_requests/58</a></td>
<td>Being Reviewed</td>
</tr>
</tbody></table>

## What we under-estimated?

One thing I wrote in my proposal was to write a complete test-suite for automatically testing the Google Backend in GVfs over an actual Google account. This was supposed to be done before the final evaluation but I did the mistake of under-estimating how much time it takes to produce efficient and re-useable software.

The 10k feet view of the testing framework is that we wish to perform actual GIO operations and check for crashes 
on a real world scenario. There's no other way to test the GVfs Google Backend because for any changes to the state of the cache, we must make a contact to Drive APIs. Mocking the Drive API isn't a solution because we'd then be spending time creating hundreds of fixtures, if not thousand. Moreover, the state of the Drive changes after each operation so we must take fetch a new list of updated files (which requires yet another fixture).
So far, I've been able to create a program in C which can perform testing automatically and fail in case of crashes or abnormal behaviour. The idea was to use an actual Google account and use GNOME Online Accounts (GOA) APIs to create a service for libgdata. We can then use this service to perform various operations.

Also, the number of corner cases that may arise in as highly parallel and dynamic systems as Google Drive, is unfathomable. Me and my mentor after some elaborate discussions decided that for the time being, we'd just be implementing some basic operations and add more as we go.

## What's next? (pertaining to OSS in general)

Time and again, I've written about my love for open source software and the freedom it brings. Since, I'm a proponent of it, I've decided that I'll continue contributing to open source in my free/spare time. Hopefully, I'll be able to get a job where I can contribute to GNOME full-time and that I'd get paid chatting up with my GNOMEies (kinda a dream job :-P). I've also planned on applying for a Foundation Membership and becoming a #FriendOfGNOME sometime later when I start earning. As for GVfs and libgdata, I'll keep contributing to these projects because the codebase is really interesting and that they've provided me the stepping stones into the OSS world.

## Final Note

All in all, this journey of GSoC has been one of the most beautiful ones when it comes to my career. It was a really wholesome experience with the whole community bonding with each other and helping. I'd like to extend my gratitude to each and every person involved with me over this journey, to GNOME GSoC admins who believed in my abilities and chose me to carry forward my project, to the founders of GNOME for having the vision of having a "complete free software solution for everyone" and to everyone else too.

A special thanks to my mentor Ondrej Holy. He's a fantastic guy to work with and has a great expertise in dealing with file systems. He's one of the best humans I've interacted online for such an extended period of time. He mentored me really well and was always ready to help. Thank you!

Also, a big thanks to Philip Withnall (@pwithnall), Inigo Martinez (@inigo), Emmanuelle Bassi (@ebassi) for being patient with me and clarifying my doubts. Thank you Google supporting open source software and for providing students like me with this opportunity. 
