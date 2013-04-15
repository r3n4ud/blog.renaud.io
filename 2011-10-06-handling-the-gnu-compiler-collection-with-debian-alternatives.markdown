---
layout: post
title: "Handling the GNU Compiler Collection with debian alternatives"
date: 2011-10-06 15:06:00
comments: true
categories: [Debian GNU/Linux, Old Enki Blog]
---
The story so far: I have several versions of gcc and g++ on my debian distro and during some external project compilation I went to version-related compilation issues. OK, gcc should be managed by the alternatives debian stuﬀ…

Just after a `$ sudo update-alternative --config gcc[tab][tab]`, I discovered that gcc, g++, gcov, *etc.* are NOT managed the alternatives way… OK, there are some alternatives deﬁned for `c++`, `cc` and `cpp`, but these alternatives are symbolic links to… symbolic links:

    foo@bar:/etc/alternatives$ ls -al c++ cc cpp
    lrwxrwxrwx 1 root root 12  2 avril  2011 c++ -> /usr/bin/g++
    lrwxrwxrwx 1 root root 12  2 avril  2011 cc -> /usr/bin/gcc
    lrwxrwxrwx 1 root root 12  1 août  23:41 cpp -> /usr/bin/cpp
    foo@bar:/etc/alternatives$ ls -al /usr/bin/g++ /usr/bin/gcc /usr/bin/cpp
    lrwxrwxrwx 1 root root 7  1 août  23:32 /usr/bin/cpp -> cpp-4.6
    lrwxrwxrwx 1 root root 7  9 juil. 19:39 /usr/bin/g++ -> g++-4.6
    lrwxrwxrwx 1 root root 7  9 juil. 19:39 /usr/bin/gcc -> gcc-4.6

It's really really strange since `c++` and `cc` are generic but not `cpp`. Here is my proposal to have a better management for `cpp`, `gcc`, `g++` and `gcov`, assuming that you have 4.4 and 4.6 versions on your system:

    foo@bar:/etc/alternatives$ sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.6 10
    foo@bar:/etc/alternatives$ sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.4 1
    foo@bar:/etc/alternatives$ sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-4.6 10
    foo@bar:/etc/alternatives$ sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-4.4 1
    foo@bar:/etc/alternatives$ sudo update-alternatives --install /usr/bin/gcov gcov /usr/bin/gcov-4.6 10
    foo@bar:/etc/alternatives$ sudo update-alternatives --install /usr/bin/gcov gcov /usr/bin/gcov-4.4 1
    foo@bar:/etc/alternatives$ sudo rm /usr/bin/cpp
    foo@bar:/etc/alternatives$ sudo update-alternatives --install /usr/bin/cpp cpp /usr/bin/cpp-4.6 10
    foo@bar:/etc/alternatives$ sudo update-alternatives --install /usr/bin/cpp cpp /usr/bin/cpp-4.4 1

Obviously, you should adapt the previous commands to your system (_i.e._ which versions are present).

Now, when you need `gcc-4.4` for a speciﬁc compilation and if `$ export CC=/usr/bin/gcc-4.4` is not sufficient, you just have to do `$ sudo update-alternative --config gcc` and manually select `gcc-4.4`… to reset to `gcc-4.6` if you nominally need it.

That's it!
