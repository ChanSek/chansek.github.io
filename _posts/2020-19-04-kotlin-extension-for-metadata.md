---
layout: post
title: "Extension Functions + AndroidManifest's <meta-data> Tag"
date: 2020-04-19 07:30:20 +0530
description: Leveraging Kotlin's extension function feature to read <meta-data> tags from AndroidManifest
img:
tags: [Kotlin, Extension Function, AndroidManifest]
---
If you are building an Android library, then you might need to read `<meta-data>` tag of your user. I came across with similar situation, where I need to do the same.

My initial code looked something like this:
```
fun getVersion(): Int {
    val appInfo = packageManager.getApplicationInfo(packageName, PackageManager.GET_META_DATA)
    val metaData = appInfo.metaData
    return metaData.getInt(META_KEY_VERSION)
}
```

However, this didn't scale well. As my library grew, I wanted to give more control to library users and hence more `<meta-data>` tags were being introduced. And to read them, I have to write more methods like `getVersion()` scattered in many `Activity` classes.

So I have refactored it to create few extension functions like this:
```
fun AppCompatActivity.getMetaData(): Bundle =
    packageManager.getApplicationInfo(packageName, PackageManager.GET_META_DATA).metaData
```

Now, I can simple call `getMetaData().getString(...)` or `getMetaData().getInt(...)`, etc. And to make it even more intuitive, we can also create few more extensions like this:
```
fun AppCompatActivity.getMetaString(key: String) = getMetaData().getString(key)

fun AppCompatActivity.getMetaInt(key: String) = getMetaData().getInt(key)
```

Now, we can simply call `getMetaString(...)` from any Activity to get the `<meta-data>` declared by user.
