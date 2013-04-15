---
layout: post
title: "Android − My life in gray!"
date: 2011-10-21 12:06:00
comments: true
categories: [Android, Old Enki Blog]
---
After some existencial questions on how the gray level colors should be deﬁned in our android projects, I've decided to check the predeﬁned colors on the android platform:

    $ find . -type f -name "colors.xml" -print0 | xargs -0 grep grey
    $ find . -type f -name "colors.xml" -print0 | xargs -0 grep gray
    ./platforms/android-4/data/res/values/colors.xml:    <color name="lighter_gray">#ddd</color>
    ./platforms/android-4/data/res/values/colors.xml:    <color name="darker_gray">#aaa</color>
    […]
    ./platforms/android-10/data/res/values/colors.xml:    <color name="lighter_gray">#ddd</color>
    ./platforms/android-10/data/res/values/colors.xml:    <color name="darker_gray">#aaa</color>
    […]

Ok, the ﬁrst conclusion is that the android platform deﬁnetely used a US convention… No surprises there. Two gray level colors are deﬁned in platforms. Now, I should check on the android developers website: [here](http://developer.android.com/reference/android/graphics/Color.html) and [there](http://developer.android.com/reference/android/R.color.html) and search for gray level colors. According to this documentation, only `darker_gray` seems to be public, although `lighter_gray` is deﬁned within `data/res/values/colors.xml`…

Ok, make a sample stupid project:

    $ android create project --name Grayish \
     --target android-10 --path /foo/bar/Grayish \
     --package org.summo.grayish --activity MyLifeInGray

Let's deﬁne some colors in the project `res/values/colors.xml` ﬁle:

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
  <color name="lighter_gray">#ddd</color>
  <color name="darker_gray">#aaa</color>
  <color name="gray_40">#696969</color>
</resources>
```

Use them (`res/layout/main.xml` file):

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:orientation="vertical"
              android:layout_width="fill_parent"
              android:layout_height="fill_parent" >
  <TextView
      android:background="@android:color/darker_gray"
      android:layout_width="fill_parent"
      android:layout_height="wrap_content"
      android:text="@android:color/darker_gray"
      android:textColor="@android:color/black"
      />
  <TextView
      android:background="@color/lighter_gray"
      android:layout_width="fill_parent"
      android:layout_height="wrap_content"
      android:text="@color/lighter_gray"
      android:textColor="@android:color/black"
      />
  <TextView
      android:background="@color/darker_gray"
      android:layout_width="fill_parent"
      android:layout_height="wrap_content"
      android:text="@color/darker_gray"
      android:textColor="@android:color/black"
      />
  <TextView
      android:background="@color/gray_40"
      android:layout_width="fill_parent"
      android:layout_height="wrap_content"
      android:text="@color/gray_40"
      android:textColor="@android:color/black"
      />
</LinearLayout>
```

Compile and install the project on your device:

    $ ant install

A ddms screenshot and a cropping later:

{% img center http://blog.renaud.io/images/mylifeingray.png %}

That's weird but `lighter_gray` is not public… We have access to only one gray system color! To conclude: although we could use the system colors, and since one and only one color is just useless, we would rather deﬁne `lighter_gray` and `darker_gray` in our own `res/values/colors.xml` ﬁle and if we need more grayish colors, we would use a `gray_xx`, `xx` corresponding to the percentage of white color.

Et voilà !
