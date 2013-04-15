---
layout: post
title: "Build and setup GNU Emacs 24 from git the alternatives way on Debian GNU/Linux"
date: 2011-10-31 23:17:00
comments: true
categories: [Debian GNU/Linux, GNU Emacs, Old Enki Blog]
---

Get the source from the Savannah git repository
-----------------------------------------------

I am just interested in the recent history and would like to save some precious bandwidth, so I just use `--depth 1` to get _only_ what is needed:

    $ git clone --depth 1 git://git.savannah.gnu.org/emacs.git

Build
-----

An `./autogen.sh` is needed to generate the `configure` script:

    $ cd emacs
    $ ./autogen.sh

I have some emacs binary packages installed, so I have chosen to install my new emacs to `/usr/local`. Moreover, I have the gtk3 dev packages. As a consequence, my `configure` call is:

    $ ./configure --prefix=/usr/local --with-x-toolkit=gtk3

At this step, you will have to retrieve the required and wanted `libWhatTheF**k-dev` dependencies by yourself…
For me, the `configure` step returns with:

    Configured for `x86_64-unknown-linux-gnu'.

      Where should the build process find the source code?    /foo/bar/git/emacs
      What operating system and machine description files should Emacs use?
            `s/gnu-linux.h' and `m/amdx86-64.h'
      What compiler should emacs be built with?               gcc -std=gnu99 -g -O2
      Should Emacs use the GNU version of malloc?             yes
          (Using Doug Lea's new malloc from the GNU C Library.)
      Should Emacs use a relocating allocator for buffers?    no
      Should Emacs use mmap(2) for buffer allocation?         no
      What window system should Emacs use?                    x11
      What toolkit should Emacs use?                          GTK
      Where do we find X Windows header files?                Standard dirs
      Where do we find X Windows libraries?                   Standard dirs
      Does Emacs use -lXaw3d?                                 no
      Does Emacs use -lXpm?                                   yes
      Does Emacs use -ljpeg?                                  yes
      Does Emacs use -ltiff?                                  yes
      Does Emacs use a gif library?                           yes -lgif
      Does Emacs use -lpng?                                   yes
      Does Emacs use -lrsvg-2?                                yes
      Does Emacs use imagemagick?                             yes
      Does Emacs use -lgpm?                                   yes
      Does Emacs use -ldbus?                                  yes
      Does Emacs use -lgconf?                                 yes
      Does Emacs use GSettings?                               yes
      Does Emacs use -lselinux?                               yes
      Does Emacs use -lgnutls (2.6.x or higher)?              yes
      Does Emacs use -lxml2?                                  yes
      Does Emacs use -lfreetype?                              yes
      Does Emacs use -lm17n-flt?                              yes
      Does Emacs use -lotf?                                   yes
      Does Emacs use -lxft?                                   yes
      Does Emacs use toolkit scroll bars?                     yes

Now, just compile, install and fix `blessmail` as required using:

    $ make all && make install
    $ sudo lib-src/blessmail /usr/local/libexec/emacs/24.0.91/x86_64-unknown-linux-gnu/movemail

Configure Debian GNU/Linux to use your brand new emacs
------------------------------------------------------

Rename your new binaries with the following bash script (reusable after each recompilation):

{% codeblock lang:bash %}
for i in etags ctags emacsclient ebrowse rcs-checkin grep-changelog emacs
do
    if [ -f /usr/local/bin/$i ]; then
        if [ -f /usr/local/bin/$i.emacs24 ]; then
            rm /usr/local/bin/$i.emacs24
        fi
        mv /usr/local/bin/$i /usr/local/bin/$i.emacs24
    fi
done

if [ -f /usr/local/bin/emacs.emacs24 ]; then
    if [ -f /usr/local/bin/emacs24-x ]; then
        rm /usr/local/bin/emacs24-x
    fi
    ln -s /usr/local/bin/emacs.emacs24 /usr/local/bin/emacs24-x
fi
{% endcodeblock %}

Install the _ad hoc_ new alternatives (needed just once, even if you recompile and reinstall emacs after a `git pull origin`):

    sudo update-alternatives --install /usr/bin/emacs emacs \
      /usr/local/bin/emacs24-x 30 \
      --slave /usr/share/icons/hicolor/128x128/apps/emacs.png emacs-128x128.png \
      /usr/local/share/icons/hicolor/128x128/apps/emacs.png \
      --slave /usr/share/icons/hicolor/16x16/apps/emacs.png emacs-16x16.png \
      /usr/local/share/icons/hicolor/16x16/apps/emacs.png \
      --slave /usr/share/icons/hicolor/24x24/apps/emacs.png emacs-24x24.png \
      /usr/local/share/icons/hicolor/24x24/apps/emacs.png \
      --slave /usr/share/icons/hicolor/32x32/apps/emacs.png emacs-32x32.png \
      /usr/local/share/icons/hicolor/32x32/apps/emacs.png \
      --slave /usr/share/icons/hicolor/48x48/apps/emacs.png emacs-48x48.png \
      /usr/local/share/icons/hicolor/48x48/apps/emacs.png \
      --slave /usr/share/icons/hicolor/scalable/mimetypes/emacs-document.svg emacs-document.svg \
      /usr/local/share/icons/hicolor/scalable/mimetypes/emacs-document.svg \
      --slave /usr/share/man/man1/emacs.1.gz emacs.1.gz \
      /usr/local/share/man/man1/emacs.1.gz \
      --slave /usr/share/icons/hicolor/scalable/apps/emacs.png emacs.svg \
      /usr/local/share/icons/hicolor/scalable/apps/emacs.svg

    sudo update-alternatives --install /usr/bin/etags etags \
      /usr/local/bin/etags.emacs24 35 \
      --slave /usr/share/man/man1/etags.1.gz etags.1.gz \
      /usr/local/share/man/man1/etags.1.gz

    sudo update-alternatives --install /usr/bin/ctags ctags \
      /usr/local/bin/ctags.emacs24 35 \
      --slave /usr/share/man/man1/ctags.1.gz ctags.1.gz \
      /usr/local/share/man/man1/ctags.1.gz

    sudo update-alternatives --install /usr/bin/emacsclient emacsclient \
      /usr/local/bin/emacsclient.emacs24 30 \
      --slave /usr/share/man/man1/emacsclient.1.gz emacsclient.1.gz \
      /usr/local/share/man/man1/emacsclient.1.gz

    sudo update-alternatives --install /usr/bin/ebrowse ebrowse \
      /usr/local/bin/ebrowse.emacs24 30 \
      --slave /usr/share/man/man1/ebrowse.1.gz ebrowse.1.gz \
      /usr/local/share/man/man1/ebrowse.1.gz

    sudo update-alternatives --install /usr/bin/rcs-checkin rcs-checkin \
      /usr/local/bin/rcs-checkin.emacs24 30 \
      --slave /usr/share/man/man1/rcs-checkin.1.gz rcs-checkin.1.gz \
      /usr/local/share/man/man1/rcs-checkin.1.gz

    sudo update-alternatives --install /usr/bin/grep-changelog \
      grep-changelog /usr/local/bin/grep-changelog.emacs24 30 \
      --slave /usr/share/man/man1/grep-changelog.1.gz grep-changelog.1.gz \
      /usr/local/share/man/man1/grep-changelog.1.gz

The given priorities are higher than the other alternatives. To finish, I set all those alternatives to the automatic mode:

    for i in ctags etags emacs emacsclient ebrowse rcs-checkin grep-changelog
    do
        sudo update-alternatives --auto $i
    done

Now, my new emacs will always be the default choice, even if I upgrade my distro and the emacs binary packages!

Bonus point?
------------

The default editor is ruled by two alternatives on my system: `editor` and `gnome-text-editor`. I set them up to use my new emacs by default.

Et voilà !
