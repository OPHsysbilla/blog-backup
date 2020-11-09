---
title: Android Studio 的新东西
date: 2019-06-20 18:45:01
tags:
---

1. `find . | grep hprof-conv` 在~\Library\Android\sdk\下找出`hprof-conv`的位置，再使用命令`./hprof-conv -z infile outfile` 来转换成Eclipse的Mat工具能够识别的hprof文件，注意使用此命令时，指定的outPutFile路径需要已经被创建，否则会一直返回Usage...展示`Usage: hprof-conf [-z] infile outfile`
2. [强行刷新gradle依赖缓存，拉取远程依赖版本](https://blog.csdn.net/T_yoo_csdn/article/details/84950088)：`./gradlew build --refresh-dependencie`。
    > 只想刷新某个指定的依赖: 直接去~/.gradle/caches/modules-2目录下，rm -fr `find . -name xxx`，然后直接重编
    > 删除含有'aaa'的文件夹依赖：`find ~/.gradle/caches -type d | grep 'aaa' | xargs rm -r`