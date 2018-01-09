---
layout: post
title: XML document structures must start and end within the same entity.
date: 2018-01-08 13:32:20 +0300
description:
img:
tags: [Errors, Android, XML]
---
Let's try to find out when this error can occur and then will try to fix this with a solution.

## Problem:
Here is an Android XML layout which will cause the error mentioned.
{% highlight xml %}
<TextView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="45dp"
    android:gravity="center_vertical"
    android:paddingEnd="16dp"
    android:paddingStart="16dp"
    android:textAppearance="@style/TextAppearance.AppCompat.Medium"
    tools:text="Pan india">
{% endhighlight %}
If you observe closely, the layout XMl is started with `<TextView>` tag, but never ended with it.

## Solution:
So to avoid this error we have to make sure the TextView has an end tag. THis we can achieve in two ways. Either we can end the `TextView` with an end tag like `</TextView>` or we can end like `/>`.
The later one only can work if your XML tag doesn't has any child tags. For example: the earlier one should be used in case of `LinearLayout` or some similar `ViewGroup`s where as the later one should be used in case of `TextView`, `Button` or some similar `View`s.
Below are the examples in both the ways t solve it:
### 1st Way
{% highlight xml %}
<TextView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="45dp"
    android:gravity="center_vertical"
    android:paddingEnd="16dp"
    android:paddingStart="16dp"
    android:textAppearance="@style/TextAppearance.AppCompat.Medium"
    tools:text="Pan india">

</TextView>
{% endhighlight %}
### 2nd Way
{% highlight xml %}
<TextView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="45dp"
    android:gravity="center_vertical"
    android:paddingEnd="16dp"
    android:paddingStart="16dp"
    android:textAppearance="@style/TextAppearance.AppCompat.Medium"
    tools:text="Pan india" />
{% endhighlight %}

## Conclusion:
If we have multiple XML tags in a single file, we should always close the XML tag before starting a new tag. If we have only a single XML tag in the file, we should close the tag at the end of the file.
