---
layout: post
title: "Kotlin Heroes: Practice"
date: 2019-05-29 10:51:20 +0530
description: A Kotlin contest by JetBrains in collaboration with CodeForces
img: kotlin-heroes.png
tags: [Kotlin, Contest, Algorithm, Kotlin Heroes, JetBrains, CodeForces]
---
This post answers the questions of **Kotlin Heroes: Practice** contest.

## Questions
In this contest, there are six questions. Problem statements of all of these questions along with my answers are mentioned below.

### [Question A (Dice Rolling)](https://codeforces.com/contest/1171/problem/A)
Problem statment can be found in the mentioned URL: https://codeforces.com/contest/1171/problem/A

### Answer A (Dice Rolling)
My answer to the above question is this:
```
import java.util.*

fun main() {
    val reader = Scanner(System.`in`)
    val queries = reader.nextInt()
    val output = IntArray(queries)
    for (i in 0 until queries) {
        val point = reader.nextInt()
        output[i] = getCount(point)
    }
    output.forEach {
        println(it)
    }
}

fun getCount(point: Int) = if (point % 7 == 0) point / 7 else (point / 7) + 1
```

### [Question B (New Year and the Christmas Ornament)](https://codeforces.com/contest/1171/problem/B)
Problem statment can be found in the mentioned URL: https://codeforces.com/contest/1171/problem/B

### Answer B (New Year and the Christmas Ornament)
My answer to the above question is this:
```
import java.util.*

fun main() {
    val reader = Scanner(System.`in`)
    val yellow = reader.nextInt()
    val blue = reader.nextInt() - 1
    val red = reader.nextInt() - 2
    val temp = if (yellow > blue) blue else yellow
    val smallest = if (temp > red) red else temp
    println(smallest * 3 + 3)
}
```

### [Question C (Letters Rearranging)](https://codeforces.com/contest/1171/problem/C)
Problem statment can be found in the mentioned URL: https://codeforces.com/contest/1171/problem/C

### Answer C (Letters Rearranging)
My answer to the above question is this:
```
import java.lang.StringBuilder
import java.util.*

fun main() {
    val reader = Scanner(System.`in`)
    val queries = reader.nextInt()
    val output = arrayOfNulls<String>(queries)
    for (i in 0 until queries) {
        val str = reader.next()
        when {
            str.allCharactersSame() -> output[i] = "-1"
            str.isPalindrome().not() -> output[i] = str
            else -> output[i] = str.rearrange()
        }
    }
    output.forEach { println(it) }
}

fun String.isPalindrome(): Boolean {
    val size = length
    for (i in 0 until size / 2) {
        if (this[i] != this[size - i - 1]) return false
    }
    return true
}

fun String.allCharactersSame(): Boolean {
    for (i in 0 until length - 1) {
        if (this[i] != this[i + 1]) return false
    }
    return true
}

fun String.rearrange(): String {
    val middle = length / 2
    for (i in middle until length - 1) {
        if (this[i] != this[i + 1]) {
            val builder = StringBuilder(this)
            val temp = builder[i]
            builder[i] = builder[i + 1]
            builder[i + 1] = temp
            return builder.toString()
        }
    }
    return this
}
```

### [Question D (Got Any Grapes?)](https://codeforces.com/contest/1171/problem/D)
Problem statment can be found in the mentioned URL: https://codeforces.com/contest/1171/problem/D

### Answer D (Got Any Grapes?)
My answer to the above question is this:
```
import java.util.*

fun main() {
    val reader = Scanner(System.`in`)
    val andrewWants = reader.nextInt()
    val dmitryWants = reader.nextInt()
    val michalWants = reader.nextInt()

    val greenGrapes = reader.nextInt()
    val purpleGrapes = reader.nextInt()
    val blackGrapes = reader.nextInt()

    val remainingGreenGrapes = greenGrapes - andrewWants
    val remainingPurpleGrapes = (remainingGreenGrapes + purpleGrapes) - dmitryWants
    when {
        greenGrapes < andrewWants -> println("NO")
        (remainingGreenGrapes + purpleGrapes) < dmitryWants -> println("NO")
        (remainingPurpleGrapes + blackGrapes) < michalWants -> println("NO")
        else -> println("YES")
    }
}
```

### [Question E (Doggo Recoloring)](https://codeforces.com/contest/1171/problem/E)
Problem statment can be found in the mentioned URL: https://codeforces.com/contest/1171/problem/E

### Answer E (Doggo Recoloring)
My answer to the above question is this:
```
import java.util.*

fun main() {
    val reader = Scanner(System.`in`)
    val puppies = reader.nextInt()
    if (puppies == 1) println("Yes")
    else {
        val colors = reader.next()
        val counts = IntArray(26)
        colors.forEach {
            val index = it - 'a'
            ++counts[index]
            if (counts[index] == 2) {
                println("Yes")
                return
            }
        }
        println("No")
    }
}
```

### [Question F (Division and Union)](https://codeforces.com/contest/1171/problem/F)
Problem statment can be found in the mentioned URL: https://codeforces.com/contest/1171/problem/F

### Answer F (Division and Union)
Unfortunately I failed to submit the answer for this before the contest gets closed. Now I have a half baked code which I am not sure whether it will work or not as CodeForces doesn't allow to submit the code if the contest is closed.

If you have an answer to this, please mention it in the comment section.

## Contribution:
Please feel free to discuss if you have any concerns or improvements on my answers. I would love to discuss about more optimized answers.
