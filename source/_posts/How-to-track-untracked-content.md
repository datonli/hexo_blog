---
title: git其他lib untrackable怎么办
date: 2018-09-01 01:54:32
tags: git
---
git其他人的lib放到自己的lib路径下，会出现修改了别人lib内容无法`git add`的情形，显示`untracked content`。简单3步解决这个问题：
1. 删除别人lib中的`.git`目录，只在根目录下有；
2. `git rm -rf --cached /path/to/other_lib_directories`
3. `git add .` 正常执行
