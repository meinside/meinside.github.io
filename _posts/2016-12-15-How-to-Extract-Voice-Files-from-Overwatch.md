---
layout: post
title: How to extract voice files from Overwatch
tags:
- overwatch
- windows
published: true
---

Here are steps for extracting voice files from Overwatch, in any language.

# 0. Install Overwatch

Install Overwatch on your compuater, and make sure it is up-to-date.

----

# 1. Download tools for extracting files

Download `dist-toolchain-release.zip` file from [this site](https://owdev.wiki/User:Yukimono/Toolchain).

Unzip it in any folder you want.

----

# 2. Download tools for converting .wem files to .ogg

Follow the guide in [this page](http://forum.xentax.com/viewtopic.php?p=66311#p66311):

## A. ww2ogg

Download [ww2ogg019.zip](http://hcs64.com/files/ww2ogg019.zip) and unzip it.

You'll need **ww2ogg.exe** and **packed_codebooks_aoTuV_603.bin** from the unzipped files.

## B. revorb

Download [revorb.exe](http://yirkha.fud.cz/progs/foobar2000/revorb.exe).

For convenience, put **ww2ogg.exe**, **packed_codebooks_aoTuV_603.bin**, and **revorb.exe** together in a same folder.

----

# 3. Create a batch file

Create **convert.bat** file with your favorite text editor,

and fill it with following text:

```
:: Batch script for extracting voice files from Overwatch
:: created by meinside@gmail.com
:: on 2016.12.15.

::
:: (1) overtool
::    https://owdev.wiki/User:Yukimono/Toolchain
:: (2) wem2ogg
::    http://forum.xentax.com/viewtopic.php?p=66311#p66311

:: ==== TODO: change language ====
SET LANG=[which language]

:: ==== TODO: change folder locations below ====
SET OVERTOOL_FOLDER=[where (1) overtool is located]
SET WEM2OGG_FOLDER=[where (2) wem2ogg is located]
SET OW_FOLDER=[where Overwatch is installed]
SET EXTRACTED_FOLDER=[where extracted files will be placed]
SET PATH=%PATH%;%WEM2OGG_FOLDER%

:: ==== Extract ====
cd "%OVERTOOL_FOLDER%"
OverTool.exe -L%LANG% "%OW_FOLDER%" v "%EXTRACTED_FOLDER%"

:: ==== Convert ====
cd "%EXTRACTED_FOLDER%"
for /r %%f in (*.wem) do ww2ogg.exe "%%f" --pcb packed_codebooks_aoTuV_603.bin
for /r %%f in (*.ogg) do revorb.exe "%%f"

:: ==== Cleanup ====
for /r %%f in (*.wem) do del "%%f"
```

Change **LANG**, **OVERTOOL_FOLDER**, **WEM2OGG_FOLDER**, **OW_FOLDER**, and **EXTRACTED_FOLDER** values to your desired ones.

Here is an example:

```
SET LANG=koKR

SET OVERTOOL_FOLDER=D:\Downloads\ow-toolchain-release
SET WEM2OGG_FOLDER=D:\Downloads\wem2ogg
SET OW_FOLDER=D:\Games\Overwatch
SET EXTRACTED_FOLDER=D:\Documents\ow-extracted-%DATE%
SET PATH=%PATH%;%WEM2OGG_FOLDER%

cd "%OVERTOOL_FOLDER%"
OverTool.exe -L%LANG% "%OW_FOLDER%" v "%EXTRACTED_FOLDER%"

cd "%EXTRACTED_FOLDER%"
for /r %%f in (*.wem) do ww2ogg.exe "%%f" --pcb packed_codebooks_aoTuV_603.bin
for /r %%f in (*.ogg) do revorb.exe "%%f"

for /r %%f in (*.wem) do del "%%f"
```

----

# 4. Run the batch file

Double-click your **convert.bat** file.

If nothing goes wrong, it will create a directory and put extracted .ogg files in it.

----

# 99. Trouble-shooting

## A. Something went wrong

Check if folder locations are all set correctly in the batch file.

## B. Files for certain characters are not extracted

If the characters' names have special or broken characters in them, there could be errors related to the filepaths.

Try running each step (not all at once), and renaming folders with broken characters.

## C. Some extracted files are not correctly placed, or duplicated

`OverTool.exe` sometimes misplaces or extracts duplicated files.

Correct them by yourself, or wait for any fix/update for it.

