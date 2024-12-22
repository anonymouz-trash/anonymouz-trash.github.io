---
layout: post
title:  "Batch-operations with 'find' and 'for'"
categories: [Linux Scripting]
tags: [linux,scripting,loop,for,batch]
image:
  path: /assets/img/2024-12-22-looping-through-files.jpg
last_modified_at: 2024-10-14 08:52:00 +0100
---
Here I want to share some quick and dirty small scripts for different use cases. These in most cases are one-liners.

## ... in a single FOR-loop
This example batch-decrypt password-protected PDF files. You'll need `pdftk` for this.
```bash
for file in ./**/*(.); do if [[ $file =~ .*.pdf ]] pdftk $file input_pw PROMPT output ${file%.pdf}-decrypted.pdf ; done
```

| parameter | description |
| --- | --- |
| ${file} | variable which contains the filename |
| .*.pdf | the input file extension |
| ${file%.pdf}-decrypted.pdf | this cuts off the extension and adds a "-decrypted" plus the extension |

## ... with find command
My use-case for this is copying specific games from my NAS to my handheld's sd card.
```bash
find /path/to/games/roms/gba -type f -iname "*Street Fighter*.gba" -exec cp {} /path/to/sdcard/roms/gba \; 
```

| parameter | description |
| --- | --- |
| -type f | determines the file type `f = file` and `d = directory` |
| -iname "" | anything that matches this string (similar to windows) |
| -exec {}\; | `do` with found files, `{}` = contains the filename, `\;` requiered (marks end of script)  |
