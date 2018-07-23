---
title: Item2使用sz rz命令从服务下载上传文件
date: 2018-07-12 21:27:37
tags:
---

## 背景
如何从linux服务下载/上传文件

 - 使用scp命令
 - 使用sz命令


这里主要介绍使用sz命令下载文件

## 安装
 - 安装lrzsz `hobrew install llssz`
 - 在本地编写shell用于接受sz命令下载下来的文件

上传shell

`iterm2-send-zmodem.sh`
```
    #!/bin/bash
    
    
    #Author: Matt Mastracci (matthew@mastracci.com)
    
    
    # AppleScript from http://stackoverflow.com/questions/4309087/cancel-button-on-osascript-in-a-bash-script
    
    
    # licensed under cc-wiki with attribution required
    
    
    # Remainder of script public domain
    
    
    osascript -e 'tell application "iTerm2" to version' > /dev/null 2>&1 && NAME=iTerm2 || NAME=iTerm
    if [[ $NAME = "iTerm" ]]; then
        FILE=`osascript -e 'tell application "iTerm" to activate' -e 'tell application "iTerm" to set thefile to choose file with prompt "Choose a file to send"' -e "do shell script (\"echo \"&(quoted form of POSIX path of thefile as Unicode text)&\"\")"`
    else
        FILE=`osascript -e 'tell application "iTerm2" to activate' -e 'tell application "iTerm2" to set thefile to choose file with prompt "Choose a file to send"' -e "do shell script (\"echo \"&(quoted form of POSIX path of thefile as Unicode text)&\"\")"`
    fi
    if [[ $FILE = "" ]]; then
        echo Cancelled.
        # Send ZModem cancel
        echo -e \\x18\\x18\\x18\\x18\\x18
        sleep 1
        echo
        echo \# Cancelled transfer
    else
        /usr/local/bin/sz "$FILE" -e -b
        sleep 1
        echo
        echo \# Received $FILE
    fi  
```

下载shell

`iterm2-recv-zmodem.sh`

```
    #!/bin/bash
    # Author: Matt Mastracci (matthew@mastracci.com)
    # AppleScript from http://stackoverflow.com/questions/4309087/cancel-button-on-osascript-in-a-bash-script
    # licensed under cc-wiki with attribution required 
    # Remainder of script public domain

    osascript -e 'tell application "iTerm2" to version' > /dev/null 2>&1 && NAME=iTerm2 || NAME=iTerm
    if [[ $NAME = "iTerm" ]]; then
        FILE=`osascript -e 'tell application "iTerm" to activate' -e 'tell application "iTerm" to set thefile to choose folder with prompt "Choose a folder to place received files in"' -e "do shell script (\"echo \"&(quoted form of POSIX path of thefile as Unicode text)&\"\")"`
    else
        FILE=`osascript -e 'tell application "iTerm2" to activate' -e 'tell application "iTerm2" to set thefile to choose folder with prompt "Choose a folder to place received files in"' -e "do shell script (\"echo \"&(quoted form of POSIX path of thefile as Unicode text)&\"\")"`
    fi

    if [[ $FILE = "" ]]; then
        echo Cancelled.
        # Send ZModem cancel
        echo -e \\x18\\x18\\x18\\x18\\x18
        sleep 1
        echo
        echo \# Cancelled transfer
    else
        cd "$FILE"
        /usr/local/bin/rz -E -e -b
        sleep 1
        echo
        echo
        echo \# Sent \-\> $FILE
    fi
```
## 配置iterm2

- 设置Iterm2的Tirgger特性，profiles-default-editProfiles-Advanced中的Tirgger

```
    添加两条trigger，分别设置Regular expression，Action，Parameters，Instant如下：
        1.第一条
            Regular expression: rz waiting to receive.\*\*B0100
            Action: Run Silent Coprocess
            Parameters: /usr/local/bin/iterm2-send-zmodem.sh
            Instant: checked
        2.第二条
            Regular expression: \*\*B00000000000000
            Action: Run Silent Coprocess
            Parameters: /usr/local/bin/iterm2-recv-zmodem.sh
            Instant: checked
```

**备注**:注意两个文件的权限