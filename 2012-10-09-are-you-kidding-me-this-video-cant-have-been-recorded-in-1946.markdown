---
layout: post
title: "Are you kidding me? This video can't have been recorded in 1946!"
date: 2012-10-09 01:16
comments: true
external-url:
categories: [Android, FFmpeg]
---

{% img right http://blog.renaud.io/images/ffmpeg_android_timestamp_workaround.png %}

<span itemprop="description">Today, I made some experiment with avconv on videos made with some android devices and found out that the `creation_time` of a video of mine recorded back in june seems to be crippled since the value returned by ffprobe is `1946-06-26 12:30:42`! That can't be possible…</span>

Googling time
-------------

That's strange but there are only few relevant results returned by Google on that crippled timestamp. After some digging, I found out that [FFmpeg issue](https://ffmpeg.org/trac/ffmpeg/ticket/1471). In fact, that's the fourth result of a Google search on `android video creation_time` but it's like always, you just need to ﬁnd the good keywords.

Matter ﬁxed
------------
The ticket's status is `closed defect: fixed`. OK but the issue tracker doesn't contain any reference to the ﬁx.

    $ git clone git://source.ffmpeg.org/ffmpeg.git
    $ git log --grep=1471
    […]
    commit 23eeffcd48a15e73fb2649b712870b6d101c5471
    Author: Michael Niedermayer <michaelni@gmx.at>
    Date:   Sun Jul 1 21:41:06 2012 +0200

        mov: add workaround for incorrect 0 time point.

        Fixes Ticket1471

        Signed-off-by: Michael Niedermayer <michaelni@gmx.at>
    […]

    $ git show 23eeffcd48a15e73fb2649b712870b6d101c5471
    commit 23eeffcd48a15e73fb2649b712870b6d101c5471
    Author: Michael Niedermayer <michaelni@gmx.at>
    Date:   Sun Jul 1 21:41:06 2012 +0200

        mov: add workaround for incorrect 0 time point.

        Fixes Ticket1471

        Signed-off-by: Michael Niedermayer <michaelni@gmx.at>

    diff --git a/libavformat/mov.c b/libavformat/mov.c
    index af5b126..faa8c65 100644
    --- a/libavformat/mov.c
    +++ b/libavformat/mov.c
    @@ -780,7 +780,8 @@ static void mov_metadata_creation_time(AVDictionary **metadata, time_t time)
         char buffer[32];
         if (time) {
             struct tm *ptm;
    -        time -= 2082844800;  /* seconds between 1904-01-01 and Epoch */
    +        if(time >= 2082844800)
    +            time -= 2082844800;  /* seconds between 1904-01-01 and Epoch */
             ptm = gmtime(&time);
             if (!ptm) return;
             strftime(buffer, sizeof(buffer), "%Y-%m-%d %H:%M:%S", ptm);

Uh!

Compile time
------------

ATTOW, the debian package version of `libav-tools` is `0.8.xx` and a `git log --tags --simplify-by-decoration --pretty="format:%ai %d"` shows that the `0.9` tag has been bumped in the late 2011… Too bad… I'll need to compile that myself if I really wanted that `creation_date`.

Update
------

The wording of the *Compile time* is unclear at least, so:

- libav and FFmpeg has forked.

- libav is now the default on Debian GNU/Linux.

- `libavformat/mov.c` has evolved diﬀerently but FFmpeg seems to provide a ﬁx (that must be
  veriﬁed) but that doesn't mean that the issue has not been ﬁxed in the libav edge, only that
  I've found only clue that the issue has been adressed by the FFmpeg project.

Update 2
--------

I had ﬁled [a bug on libav](https://bugzilla.libav.org/show_bug.cgi?id=381) just to found out that
the issue had been ﬁxed in Android 4.2.

Matter ﬁxed!
