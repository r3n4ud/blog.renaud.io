---
layout: post
title: "QR codes − From strings to a full-ﬂedged pdf ﬁle"
date: 2011-11-19 00:10:00
comments: true
categories: [Bash, Debian GNU/Linux, ImageMagick, PDF, Old Enki Blog]
---

{% img right http://blog.renaud.io/images/qr.png %}

<span itemprop="description">Last days at work, we integrate some QR codes recognition development on android using [ZXing](http://code.google.com/p/zxing/). Although we have a special printer, we need some test cases and a standalone solution to generate and print out several QR codes on a standard printer. At ﬁrst, a colleague of mine has used some online QR generator but it does not scale very well. Fortunately, a C library and a derived command line tool exists in the [libqrencode project](http://fukuchi.org/works/qrencode/index.en.html), thanks to Kentaro Fukuchi. Moreover, that's packaged on debian…</span>

qrencode is the right tool, the unix way. However, our use case requires that we get an organized output for testing. To be more speciﬁc, we need to sequentially recognize several sets of QR codes.

Black box description
---------------------

The input is a sequence of strings. As for the output, I need to obtain a standalone printed document. As a consequence, I choose to use the pdf format to not be bothered with any printing issue, even if the content is pure raster.

Input data sample
-----------------

I will not provide there my real data. The sample data are:

1. Lorem ipsum dolor sit amet duis mattis ullamcorper ultricies aliquam sit amet vestibulum augue
2. Aenean ut orci eget nisi tincidunt pulvinar
3. Aenean eleifend tellus ac lacus convallis fermentum


Be prepared…
------------

I have already used ImageMagick in the past to do composition and conversion operations and my feelings at the beginning were that ImageMagick is all I need to get my formatted output.

Obviously, we need to install qrencode and ImageMagick (and the associated verbose documentation):

    $ sudo apt-get install qrencode imagemagick imagemagick-doc

But, wait, I need a multipage pdf in the end. I certainly need some pdf merging tool. Go with the installation of pdfsam then:

    $ sudo apt-get install pdfsam


QR codes generation with human-readable label
---------------------------------------------

Pretty straightforward. An example on the ﬁrst dataset:

{% codeblock lang:bash %}
for i in lorem ipsum dolor sit amet duis mattis ullamcorper
do
    qrencode -l Q -s 20 -m 10 -o $i.png $i
    composite -geometry +0+40 -font Lucida-Sans-Typewriter-Regular -pointsize 40 \
        -gravity north label:"${i}" $i.png $i.png
done
{% endcodeblock %}

The `qrencode` call is to generate each png from a single string. The `-l Q` option is to set the error correction level to Q (25% of codewords can be restored), `-s 20` is to specify the size of dot and `-m 10` is to set the margin width.

ImageMagick `composite` is used to add a human-readable label to each png. The `Lucida-Sans-Typewriter-Regular` font is provided by the `sun-java6-fonts` debian package. You can use whatever font you like as listed by `identify -list font`.


Generate a multipage pdf ﬁle for each dataset
---------------------------------------------

From the generated png ﬁles, we use ImageMagick `montage` to arrange a multipage pdf (2 rows / 3 columns) with the following command:

    $ montage -page A4+20-20 -tile 2x3 -geometry +1+1 \
      -transparent white 'input files' \
      -font Lucida-Sans-Typewriter-Regular -pointsize 40 \
      -title "A title" output.pdf

with `input files` the list of the png input ﬁles. The `-transparent white` option is really important to enable the correct insertion of the title. Without that option, the title would be overridden by the white pixels of the input ﬁles.

Putting it all together and merge the result to a standalone pdf ﬁle
--------------------------------------------------------------------

Once we have the intermediate pdf ﬁles, it is simple to merge them into one using `pdfsam-console`. The ﬁnal quick-and-dirty bash script is:

{% codeblock lang:bash %}
#!/bin/bash

LABEL=("first" "second" "third")
DATA=("Lorem ipsum dolor sit amet duis mattis ullamcorper ultricies aliquam sit amet vestibulum augue" \
    "Aenean ut orci eget nisi tincidunt pulvinar" \
    "Aene eleifend tellus ac lacus convallis fermentum")

dataLen=${#DATA[@]}

FILE=()

for (( j=0; j<${dataLen}; j++ ));
do
    for i in ${DATA[$j]}
    do
        # Generate the QR codes
        qrencode -l Q -s 5 -m 50 -o $i.png $i
        # Add a human-readable label
        composite -geometry +0+40 -font Lucida-Sans-Typewriter-Regular -pointsize 40 \
            -gravity north label:"${i}" $i.png $i.png
        FILE[$j]="${FILE[$j]} ${i}.png"
    done
    # Assemble them into a pdf
    montage -page A4+20-20 -tile 2x3 -geometry +1+1 -transparent white ${FILE[$j]} \
        -font Lucida-Sans-Typewriter-Regular -pointsize 40 \
        -title "${LABEL[$j]}" ${LABEL[$j]}.pdf
done

# Create the final pdf
if [ -f output.pdf ]; then rm -rf $PWD/output.pdf; fi;
pdfsam-console -o $PWD/output.pdf -d $PWD/ concat

# Delete all the intermediate files
for i in ${DATA[@]}; do rm -rf ${i}.png ;done;
for (( i=0; i<${dataLen}; i++ )); do rm -rf ${LABEL[$i]}.pdf; done;
{% endcodeblock %}

Check out [the resulting pdf ﬁle](http://blog.renaud.io/data/qr.pdf)… Et voilà !
