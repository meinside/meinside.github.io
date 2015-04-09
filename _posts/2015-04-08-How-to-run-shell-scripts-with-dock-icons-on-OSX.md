---
layout: post
title: How to run shell scripts with dock icons on OSX (especially for Atom)
tags:
- raspberry pi
- apple script
- atom
published: true
---

Sometimes you'll need to run shell scripts through dock icons on your OSX boxes.

In my case, it was for running [Atom](https://atom.io/) editor from the shell, to fix the [go-plus](https://atom.io/packages/go-plus) package's complaints about missing environment variables like **GOPATH**.

I could simply launch Atom from the terminal, but I didn't want to do that every time.

So I added that shortcut to the dock like this:

----

## 1. Open Apple Script Editor

![apple_script_for_atom1](https://cloud.githubusercontent.com/assets/185988/7042850/c6447082-de1f-11e4-962a-0033b4d75c40.png)


## 2. Drop a line there

```
do shell script "PATH_TO_THE_SCRIPT"
```

Replace **PATH\_TO\_THE\_SCRIPT** with the path of your script or executable file.

As you see in the image below, mine is ``/usr/local/bin/atom``, for launching Atom.

![apple_script_for_atom2](https://cloud.githubusercontent.com/assets/185988/7042848/c6412db4-de1f-11e4-8134-385386e85809.png)

If you want Atom to be opened in your home directory, do like this:

```
do shell script "/usr/local/bin/atom" & " $HOME"
```

I don't know what's the reason, but in Apple Script, you have to append shell parameters with '&'.

Trailing **" $HOME"** will make Atom editor launched in your home directory, so you can change it to any location you want.


## 3. Save it as an Application

Now save it as an **Application**. You can select it in the **File Format** option.

![apple_script_for_atom3](https://cloud.githubusercontent.com/assets/185988/7042849/c6424000-de1f-11e4-8ebb-cd3cc8e89647.png)


## 4. Drag the application file to your dock and launch it.

![apple_script_for_atom4](https://cloud.githubusercontent.com/assets/185988/7042847/c63f206e-de1f-11e4-8e01-e03d8b7b5876.png)

It now launches with shell environment variables, eliminating the problems in go-plus!

----

All done :-) Too short, isn't it?

