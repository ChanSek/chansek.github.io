---
layout: post
title: "Introducing Chanse Code"
date: 2018-23-11 19:37:20 +0530
description:  # Add post description (optional)
img:  # Add image post (optional)
tags: [Chanse Code]
---
In today's internet world, everybody learns from internet and I am not an exception. I have been learning from internet since last 6 - 7 years. So thought of giving back to the community by creating some content and share my learnings to the new comers.

I am a big fan of YouTube and always learns something from various tech channels. But all YouTube content around related topics are covering how to use it or tutorial kind of stuff. I am mostly interested to know how it works internally.

`Chanse Code` is born to build video content with all these APIs and mentioning how they work internally.

## Advantages:
It is a genuine question that anybody can ask is, "What are the advantages of exploring any API's internal work?". And to answer this, I would like to mention below points.
1. By knowing the internals of any API, you will able to write better code.
2. It makes you a better developer day by day.
3. You can build awesome libraries around those with ease.

This is the first video explaining the reason why `Chanse Code` is made. Please have a look at it and leave your honest feedback to improve the channel.

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
