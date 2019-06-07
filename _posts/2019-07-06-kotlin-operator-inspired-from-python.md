---
layout: post
title: "Kotlin Operator inspired from Python"
date: 2019-06-07 11:18:20 +0530
description: How I used Kotiln's operator overloading feature to mimic a Python operator.
img: kotlin-python.png
tags: [Kotlin, Contest, Python, HackerRank, Algorithm]
---
Since last week, I am practicing *HackerRank* problems in *Python* to learn it. While solving probelms, yesterday I found an interesting operator in Python. I still don't know what it is called though.

In this post, we are going to see how I used Kotlin's operator overloading feature to mimic this Python operator.


## Question: [Highest Hourglass Value (2D Array - DS)](https://www.hackerrank.com/challenges/2d-array/problem?h_l=interview&playlist_slugs%5B%5D=interview-preparation-kit&playlist_slugs%5B%5D=arrays)

The question, I was solving can be found in the mentioned URL: https://www.hackerrank.com/challenges/2d-array/problem?h_l=interview&playlist_slugs%5B%5D=interview-preparation-kit&playlist_slugs%5B%5D=arrays


### Answer: Highest Hourglass Value
My answer to the above question was this:
```
fun hourglassSum(arr: Array<Array<Int>>): Int {
    var result = -63
    for (row in 0 until 4) {
        for (col in 0 until 4) {
            val temp = arr[row][col] +
                    arr[row][col + 1] +
                    arr[row][col + 2] +
                    arr[row + 1][col + 1] +
                    arr[row + 2][col] +
                    arr[row + 2][col + 1] +
                    arr[row + 2][col + 2]
            if (temp > result) result = temp
        }
    }
    return result
}
```
As I already said that I solve these problems to learn Python. So I tried the solution with Python too. And here is my Python equivalent code.
```
def hourglass_sum(arr):
    result = -63
    for row in range(4):
        for col in range(4):
            temp = arr[row][col] +\
                   arr[row][col + 1] +\
                   arr[row][col + 2] +\
                   arr[row + 1][col + 1] +\
                   arr[row + 2][col] +\
                   arr[row + 2][col + 1] +\
                   arr[row + 2][col + 2]
            if temp > result:
                result = temp
    return result
```
### Improvement
After digging a bit more into variou Python syntax, I found a better way to write the above answer. Here it is:
```
def hourglass_sum(arr):
    result = -63
    for row in range(4):
        for col in range(4):
            temp = sum(arr[row][col:col + 3]) +\
                   arr[row + 1][col + 1] +\
                   sum(arr[row + 2][col:col + 3])
            if temp > result:
                result = temp
    return result
```
### Inspiration
The above code in Python looks more readable. This inspired me to look into my Kotlin code and refactor it into something similar in readability.

#### Problem Statement
I have to find two below things:
1. An operator in Kotlin, which looks similar to `a:b`.
2. An equivalent method to `sum(\*args)`

#### Solutions
1. As Kotlin has reserved `:` operator to be used while variable declaration `val x: Int = 0`, I had to find some other operator as a replacement.
Guess what!!! Kotlin alrady has thought about it and has given us an operator `..` by rotating `:` to 90&deg;.
2. We already have a `sum()` extension function in Kotlin.

Now let's modify our Kotlin code to make it better.
```
fun hourglassSum(arr: Array<Array<Int>>): Int {
    var result = -63
    for (row in 0 until 4) {
        for (col in 0 until 4) {
            val temp = arr[row][col..col + 2].sum() +
                    arr[row + 1][col + 1] +
                    arr[row + 2][col..col + 2].sum()
            if (temp > result) result = temp
        }
    }
    return result
}
```
The above code looks similar to our Python equivalent. but unfortunately the code doesn't compile.
[![Required Int found IntRange](https://chansek.github.io/assets/img/kotlin-operator-inspired-from-python/error.png)]

And to solve this, we need to overload the corresponding operator we just created. Here it is,
```
operator fun <T> Array<T>.get(range: IntRange) = sliceArray(range)
```
Now the code compiles and works as expected with better readability.

## Contribution:
Please feel free to discuss if you have any further improvements on my answer. I would love to discuss about more optimized answer.
