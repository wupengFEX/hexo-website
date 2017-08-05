---
title: iOS Crash 栈解析
date: 2016-11-27 16:54:22
tags: [iOS, Crash, 栈解析]
---

> 作者寄语：该文章讲解iOS Crash栈解析的一些基本用法，用于入门级别以及日常问题处理，分别从准备工作，获取crash，获取符号表，解析前确认，解析等步骤进行讲解。

#### 准备工作
- 栈解析需要三个文件，分别是 `.crash, symbolicatecrash, .dSYM`

####  获取crash
- 苹果系统(OS X)：`~/Library/Logs/CrashReporter/MobileDevice/`
- 其他：`xcode->window->Devices-> <DEVICE_NAME> ->View Devices Logs`

#### 获取symbolicatecrash

- `find /Applications/Xcode.app -name symbolicatecrash -type f`

#### 获取dSYM

- 如果是打包平台或者其他打包工具，可以通过在其上找到.dSYM
- 如果是Xcode开发中的app，可以在`commend+r`之后在Products中找到`Products->xx.app->show in finder`

#### 解析前确认uuid

- 只有当xx.app, xxx.app.dSYM, crash文件这三者的uuid一致才能够解析出正确的日志文件

#### 查看xx.app的uuid

- `dwarfdump --uuid xx.app/xx`

#### 查看xxx.app.dSYM的uuid

- `dwarfdump --uuid xx.app.dSYM/Contents/Resources/DWARF/xx`

#### 查看crash文件的uuid

- 位于crash日志中的Binary Images中的第一行尖括号内

#### 解析crash

- 将`.crash, symbolicatecrash, .dSYM`放在一个文件夹中
- `export DEVELOPER_DIR=/Applications/Xcode.app/Contents/Developer`
- 命令行执行 `./symbolicatecrash yy.crash xx.dSYM > xx.log`