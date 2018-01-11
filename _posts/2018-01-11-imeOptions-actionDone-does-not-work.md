---
layout: post
title: "android:imeOptions=\"actionDone\" does not work"
date: 2018-01-11 13:32:20 +0300
description:  # Add post description (optional)
img:  # Add image post (optional)
tags: [Errors, Android, XML]
---
There might be many other reason to this. But here I am going to share what I faced and how I fixed it.

## Problem:
Let's consider you want to have an EditText for taking only alpha numeric values as input. So here is an Android XML layout which will cause the error mentioned.
{% highlight xml %}
<android.support.v7.widget.AppCompatEditText
        android:id="@+id/newDeliverable"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginEnd="8dp"
        android:layout_marginStart="8dp"
        android:digits="1234567890 abcdefghijklmnopqrstuvwxyz"
        android:hint="@string/hint_add_new_deliverable"
        android:imeOptions="actionDone"
        android:inputType="text" />
{% endhighlight %}
If you observe closely, the EditText has {% highlight %} android:digits {% endhighlight %} attribute for filtering the allowed characters for input. This attribute is the cause behind this problem.

## Solution:
So to avoid this problem, you have to add {% highlight %} android:singleLine {% endhighlight %} attribute. Below is the final working XML.
{% highlight xml %}
<android.support.v7.widget.AppCompatEditText
        android:id="@+id/newDeliverable"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginEnd="8dp"
        android:layout_marginStart="8dp"
        android:digits="1234567890 abcdefghijklmnopqrstuvwxyz"
        android:hint="@string/hint_add_new_deliverable"
        android:imeOptions="actionDone"
        android:inputType="text"
        android:singleLine="true" />
{% endhighlight %}

## Conclusion:
If we have {% highlight %} android:digits {% endhighlight %} attribute in {% highlight %} EditText {% endhighlight %}, then {% highlight %} android:imeOptions {% endhighlight %} will work ones we add {% highlight %} android:singleLine {% endhighlight %} attribute.
