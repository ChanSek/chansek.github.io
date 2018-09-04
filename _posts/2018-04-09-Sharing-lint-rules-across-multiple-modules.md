---
layout: post
title: "Sharing lint rules across modules"
date: 2018-04-09 15:05:20 +0530         # YYYY-MM-DD
description:  # Add post description (optional)
img:  # Add image post (optional)
tags: [Android, Lint, Static Analysis, Code Quality]
---
Recently I got a requirement where I wanted to modify the default lint rules applied to my application. There were many lint warnings, I wanted to supress. Of course there are many ways to achieve this and all those solutions are listed [in this documentation](https://developer.android.com/studio/write/lint).

But there are few problems with the solutions given in the documentation.
1. Supressing warnings in your IDE
...It only applies locally in your machine. CI server has no clue about the supress rules. So it still raises those as warnings or errors.
2. Supressing warnings in `build.gradle`'s `lintOptions`
...This can work, but we need to add the `lintOptions` separately to every module's gradle file.
3. Supressing warnings in `lint.xml`
...This is the best option I thought to use, but as per documentation, we should put it in our root folder. I didn't want to do it in this way as I do have multiple modules and all those modules might not share the same rules mentioned in `lint.xml` configuration.

This article is about how I solved this.

## Single lint.xml file for monorepo:
As I have already mentioned that out of above mentioned three solutions, I chose the 3rd one. So I created a `lint.xml` file and put it in the root directory of the project as suggested by Android's documentation. But to my surprise, it didn't work. I was still getting the same warnings what I have suppressed through the configuration file.

## Solution:
I created a separate directory structure under my project root directory and put my `lint.xml` file there. Now somehow I have to give this path to the **lint task** run by gradle and make sure it picks my configuration. So I created a task in my root gradle file.
{% highlight xml %}
allProjects {
    tasks.whenTaskAdded { task ->
        if (task.name.startsWith("lint")) {
            task.doFirst {
                android.lintOptions.lintConfig = file("some_path/lint.xml")
            }
        }
    }
}
{% endhighlight %}
Since the task is in the root gradle file, it will be executed for all the modules. Now my `lint.xml` is at one place and all the rules are applied to my entire repository.
## Conclusion:
Here for the task, we have checked if the task name starts with `lint` or not. This makes our lint rule to be applied for all tasks like lintDebug or similar.