---
layout: post
title: E/RecyclerView: No adapter attached; skipping layout
date: 2018-01-08 13:32:20 +0300
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: i-rest.jpg # Add image post (optional)
tags: [Errors, Android, XML]
---
E/RecyclerView: No adapter attached; skipping layout

Solution - 
with(dialog.recyclerView) {
                layoutManager = LinearLayoutManager(this.context)
                setHasFixedSize(true)
                adapter = DeliverySettingsAdapter(items)
            }