---
layout: post
title: "Extension Functions + Schedulers in RxJava chain"
date: 2019-08-23 22:03:20 +0530
description: Leveraging Kotlin's extension function feature to apply schedulers in Rxjava chain
img: kotlin-heroes.png
tags: [Kotlin, RxJava, Extension Function, Scheduler]
---
It's quite funny that when Android developers are gradually migrating from `RxJava` to `Coroutines Flow`, I started learning `Rxjava`.

In most of the cases, our `Rxjava` code looks somewhat similar to below snippet:
```
@GET(...)
fun myServerCode(...): Observable<...>

fun myApi(): Observable<...> {
    return myServerCode(...)
        .subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread())
        . // Your other operators
}
```
For all APIs, I have to write the below scheduling logic:
```
.subscribeOn(Schedulers.io())
.observeOn(AndroidSchedulers.mainThread())
```
I wanted to avoid this, so asked to few of my friends here and all of them suggested to use `compose()` operator and shared me a link to `Dan Lew`'s blog [Don't break the chain: use RxJava's compose() operator](https://blog.danlew.net/2015/03/02/dont-break-the-chain/).

After using `compose()` operator, my code looked like this:
```
fun myApi(): Observable<...> {
    return myServerCode(...)
        .compose(applySchedulers())
        . // Your other operators
}

private fun <T> applySchedulers(): ObservableTransformer<T, T> {
    return ObservableTransformer { 
        it.subscribeOn(Schedulers.io())
            .observeOn(AndroidSchedulers.mainThread())
    }
}
```
This looks nice, but I thought of improving it a bit more by leveraging the Kotlin's Extension function feature. And here is my updated `applySchedulers()` method.
```
private fun <T> Observable<T>.applySchedulers(): Observable<T> {
    return this.subscribeOn(Schedulers.io())
            .observeOn(AndroidSchedulers.mainThread())
}
```
This made my API codes look like below:
```
fun myApi(): Observable<...> {
    return myServerCode(...)
        .applySchedulers()
        . // Your other operators
}
```
It looks much nicer now. However it forces you to use only `io` and `Android's main` thread. What if we want to use `computation` thread? So I created two methods instead of one generic one. And here is the final output:
```
private fun <T> Observable<T>.ioToMain(): Observable<T> {
    return this.subscribeOn(Schedulers.io())
            .observeOn(AndroidSchedulers.mainThread())
}

private fun <T> Observable<T>.computationToMain(): Observable<T> {
    return this.subscribeOn(Schedulers.computation())
            .observeOn(AndroidSchedulers.mainThread())
}
```
That's it... This is quite a simple solution. However I wrote this to document it so that it can be used by myself when required.