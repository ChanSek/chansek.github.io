---
layout: post
title: RecyclerView: No adapter attached; skipping layout
date: 2018-01-09 13:32:20 +0300
description:  # Add post description (optional)
img:  # Add image post (optional)
tags: [Errors, Android, XML]
---
This error is a common error many developers might find if they do a silly mistake. Let's discuss about the problem:
## Problem:
After inflating the RecyclerView into your view class (Activity or Fragment), you should mention the LayoutManager of it. If by mistake you miss it, Android will skip drawing the layout. Lets see the below example with mistake in it:
{% highlight java %}
recyclerView = findViewById(R.id.recycler_view);
adapter = new MyAdapter();
recyclerView.setAdapter(adapter);
{$ endhighlight %}

## Solution:
{% highlight java %}
recyclerView = findViewById(R.id.recycler_view);
LinearLayoutManager manager = new LinearLayoutManager(this);
recyclerView.setLayoutManager(manager);
recyclerView.setHasFixedSize(true);
adapter = new MyAdapter();
recyclerView.setAdapter(adapter);
{$ endhighlight %}
Since we have set the layout manager for RecyclerView, it should now be able to draw the layout properly.
> NOTE: setHasFixedSize(...) has nothing to do with current problem. It is just added in the example as a good practice to make our layout drawing faster.