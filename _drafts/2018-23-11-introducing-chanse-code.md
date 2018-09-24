---
layout: post
title: "Introducing Chanse Code"
date: 2018-23-11 19:37:20 +0530
description:  # Add post description (optional)
img:  # Add image post (optional)
tags: [Chanse Code]
---
In today's internet world, everybody learns from internet and I am not an exception. I have been learning from internet since last 6 - 7 years. So thought of giving back to the community by creating some content and share my learnings to the new comers.

I am a big fan of YouTube and always learns something from various tech channels. But all YouTube content around related topics are covering how to use it or tutorial kind of stuff. I am most interested in how it works internally.

`Chanse Code` is also becuase of this 

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
If you observe closely, the EditText has `android:digits` attribute for filtering the allowed characters for input. This attribute is the cause behind this problem.

## Solution:
So to avoid this problem, you have to add `android:singleLine` attribute. Below is the final working XML.
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
If we have `android:digits` attribute in `EditText`, then `android:imeOptions` will work ones we add `android:singleLine` attribute.
