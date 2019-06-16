---
title: "GSoC with GNOME - Weeks 1 & 2 "
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
    - meson
---

May, 27, 2019 marked the official beginning of coding for GNOME. I had been reading the source codes of libgdata and GVfs for the better part of a month and I felt comfortable enough to get my hands dirty. I worked on two things this week:

### Meson port for `libgdata` ([MR](https://gitlab.gnome.org/GNOME/libgdata/merge_requests/6))

The meson port for libgdata has been long due, and it was direly needed this time since Autotools 1.16.1 breaks an older API in such a way that at configure time, you get [this issue](https://gitlab.gnome.org/GNOME/libgdata/issues/24). Moreover, even though you configure with `--disable-dependency-tracking`, at compile time you're gifted with yet another error - `Makefile:4517: *** missing separator.  Stop.`

The issue stems from the fact that `AX_CODE_COVERAGE` recently changed the way it embeds code coverage rules in its outputted Makefile, i.e. the older `@CODE_COVERAGE_RULES` has been removed completely in support of including `aminclude_static.am` in Makefile.am. This all finally paved the way for libgdata's meson port.

Now, libgdata uses the namespace `GData` instead of `Gdata` and this raises quite a lot of issues when trying to create the enum header and source files using `gnome.mkenums` in meson. In autotools, we were using `sed` passes to edit out the generated enum files at compile time, and those files were being placed in their respective source directories. These files are further being included by other sources, and we can't generate anything in the source directory because that's the whole philosophy of meson, i.e. never clutter source directory for anything pertaining to build.

On advice of Emannuele (@ebassi), we decided to create python wrapper scripts for `glib-mkenums` which would generate those build files in the build directory. This caused even more issues since the build directory where these sources would be compiled were missing necessary headers, so we had to copy those using `run_command` as well. I also got introduced to various other tools like `gtk-doc`, `g-ir-scanner`, `vapigen`, etc and their functionalities.

Hopefully, we'll be able to merge this within a couple of days and we'd totally switch to meson completely by the next release cycle. This wasn't mentioned in my proposal and I did totally out of will to learn something new. I'm absolutely in love with meson and how easy it makes to write build files. The ninja backend feels really fast compared to old-school `make` and I'd publish some benchmarks in a different post.

### Extending the libgdata API to use the [Drive v2 Properties API](https://developers.google.com/drive/api/v2/reference/properties/)

This was primarily my agenda for the first two weeks or until the first evaluation which is due on June, 24. Since, I had spent a lot of time going through code during the community bonding period, I was able to produce a WIP MR (working) which can get/set/remove any property on a file object.

I had to extend the `GDataParsable` type to accomodate parsing of JSON objects to a newly created `GDataProperty`. This gave me a good head-start and hence I was able to focus more on the meson port. I learned as much as I could about the GObject Memory Management, what ownership of objects is and how to transfer it, what all reference counting is about and a lot more. I do feel the lack of some good examples which use both `json-glib` and `libsoup` to call standard APIs, and I'd like to write a tutorial on how to use these two libs. 

Once, I'm done with writing the documentation, we'll be able to properly review the MR ([here](https://gitlab.gnome.org/GNOME/libgdata/merge_requests/7)) with Philip and have planned on removing the WIP badge before First Evaluation deadline.

### Conclusion

Before I started this, I had never written a single build file for any build system (apart from Makefiles) let alone porting a full project from one build system to another. This was really a great learning experience. I also learned a lot from Inigo Martinez (@inigomartinez) and I'm extremely thankful to him for patiently testing my private branches and reviewing the MR. A big thanks to Philip (@pwithnall) and my mentor Ondrej (@oholy) for their constant support. See you in the next week's post. :-)
