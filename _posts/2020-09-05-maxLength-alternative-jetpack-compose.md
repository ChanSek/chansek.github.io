---
layout: post
title: "android:maxLength Alternative in Jetpack Compose"
date: 2020-05-09 07:30:20 +0530
description: How to restrict a limited number of characters in a TextField
img:
tags: [Jetpack Compose, Android XML, TextField, EditText]
---
To take user's input, we usually use `EditText` in our XML layouts by applying some rules as per our need. One of such rule is to limit the number of characters or digits the input field should take. For the saake of an example, let's say we want to take the postal pincode from user which is of leangth 6 (in India). If we have such requirement, we usually define our `EditText` as below:

```
<EditText
    ...
    android:maxLength="6" />
```

Now let's see how should we achive this using `TextField` or `OutlinedTextField` composables in Jetpack Compose.

The initial wild guess would be to assign the value to required composaable parameter. So let's try this:
```
@Composable
fun PinCodeField() {
    var text by remember { mutableStateOf("") }
    OutlinedTextField(
        value = text,
        onValueChange = { text = it },
        maxLength = 6
    )
}
```

However, the above code will not work. Becaause, neither `TextField` nor `OutlinedTextField` accepts `maxLength` as an argument. So the only solution is to write validaation code for this.

And to do that, we neeed to have the length check everytime we get the input text in the `onValueChange` callback. Here is the new code looks like:
```
@Composable
fun PinCodeField() {
    var text by remember { mutableStateOf("") }
    OutlinedTextField(
        value = text,
        onValueChange = { text = manageLength(it) },
        maxLength = 6
    )
}

private fun manageLength(input: String) = if (input.length > 6) input.substring(0..5) else input
```

Notice that, instead of simply assigning the input value to text in `onValueChange` callback, we do have a validation check and the asssign the output of `manageLength` function to text.

If you are new to Jetpack Compose, thhiss might be helpful. Thank you for reading!!!