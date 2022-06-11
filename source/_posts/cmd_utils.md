---
title: Some useful command scripts
date: 2022-06-11 11:19
tags:  
    - Tools
    - Command
    - Shell
---
Record some common shortcut command tools.
<!--more-->
## 1.cmd_utils.sh
``` shell
#!/bin/bash

alias chrome="open -a 'Google Chrome'"

# show top fragment of an android app
alias topf="adb shell dumpsys activity top"

# show top activity of an android app
alias topa="adb shell \"dumpsys activity activities | grep mResumedActivity\""

# input text to a input box of android devices
alias input="adb shell input text $(echo "$*" | sed 's/ /\%s/g')"

# print connected android device 
# full infomation: adb shell getprop | grep "model\|version.sdk\|manufacturer\|hardware\|platform\|revision\|serialno\|product.name\|brand"
alias dinfo="adb shell getprop | grep \"ro.product.model\|ro.product.brand\|ro.build.version.sdk\|ro.build.version.release\""

# clear text from a input box of android devices
function cls(){
	adb shell input keyevent KEYCODE_MOVE_EDN
	adb shell input keyevent --longpress $(printf 'KEYCODE_DEL %.0s' {1..30})
}

# take a screen shot of connected device
function cap(){
	adb shell screencap -p /sdcard/screenshot.png
	adb pull /sdcard/screenshot.png "/Users/luoyanlin/Desktop/$1-$(date +"%Y%m%d%H%M%S").png"
}


# Convert video to gif file.
# Usage: video2gif video_file (scale) (fps)
function video2gif() {
  ffmpeg -y -i "${1}" -vf fps=${3:-10},scale=${2:-320}:-1:flags=lanczos,palettegen "${1}.png"
  ffmpeg -i "${1}" -i "${1}.png" -filter_complex "fps=${3:-10},scale=${2:-320}:-1:flags=lanczos[x];[x][1:v]paletteuse" "${1}".gif
  rm "${1}.png"
}

# 启动 shizuku
function sku(){
	adb shell sh /storage/emulated/0/Android/data/moe.shizuku.privileged.api/start.sh
}


# 打印一些记不住的命令
fun cmd(){
	echo "# 上传文件到主机\nscp -r Desktop/mvvm/G1002/G1002.mp4 ubuntu@ten:/opt/photos"
	echo "# 上传目录到主机\nscp -r Desktop/mvvm/G1002 ubuntu@ten:/opt/photos"
	echo "# 从主机下载文件\nscp username@servername:/path/filename /tmp/local_destination"
	echo "# 从主机下载目录\nscp -r username@servername:remote_dir/ /tmp/local_dir"

	echo "# .tar 包压缩\ntar -cvf ntfh.tar ntfh/"
}
```
## 2. add cmd_utils.sh path to sys env path 👉 ~/.zprofile or ～/.bash_profile (legacy system)
``` shell
vim ~/.zprofile
# Add the following content
source /Users/username/sh/cmd_utils.sh
```

## 3. source file and restart terminal
``` shell
# execute 👇 in terminal to make it effective immediately 
$ source ~/.zprofile 
```


