---
title: Linux下vim配置clang-format的方法
date: 2018-08-17 21:46:40
tags: linux
---

团队写代码经常会遇到代码风格不一致的问题，让团队代码显得非常混乱，统一的代码风格非常有助于团队成员的阅读。

Google的C++代码规范基本已经是业内通用的代码风格参考了，配合clang-format这个工具可以快速地使得代码符合Google代码风格。

clang-format是由LLVM做出的专门做代码format的工具。下载下来后，使用方法非常简单：
```
clang-format -style=Google -i xxx.h
```

但是每次写完之后都统一format一遍，非常费劲，可以把clang-format跟vim结合起来使用。

#### 安装vim插件`rhysd/vim-clang-format`
使用vim的插件管理工具Bundle可以快速安装。
在`.vim/bundles.vim`中添加`Bundle "rhysd/vim-clang-format"`，执行`BundleInstall`即可。

#### 配置`clang-format`
在linux下，需要先把`clang-format`的二进制放在`$PATH`路径，记住，要设置`clang-format`的父目录，不需要直接把这个二进制名字贴上。

##### 配置1
在`clang-format`二进制所在的目录下，执行：

```
clang-format -dump-config -style=Google > .clang-format
```

会生成一个`.clang-format`文件，这个文件在`.vimrc`中可以配置为使用的格式：

```
let g:clang_format#command = 'clang-format'
nmap <F4> :ClangFormat<cr>
autocmd FileType c ClangFormatAutoEnable
let g:clang_format#detect_style_file = 1
```

##### 配置2
配置2，不检测`.clang-format`配置文件，直接把配置放在`.vimrc`中。
```
let g:clang_format#command = 'clang-format'
nmap <F4> :ClangFormat<cr>
autocmd FileType c ClangFormatAutoEnable
let g:clang_format#detect_style_file = 0

let g:clang_format#style_options = {
        \ "Language" : "Cpp",
        \ "BasedOnStyle" : "Google",
        \ "AccessModifierOffset" : -1,
        \ "AlignAfterOpenBracket" : "true",
        \ "AlignEscapedNewlinesLeft" : "true",
        \ "AlignOperands" : "true",
        \ "AlignTrailingComments" : "true",
        \ "AllowAllParametersOfDeclarationOnNextLine" : "true",
        \ "AllowShortBlocksOnASingleLine" : "false",
        \ "AllowShortCaseLabelsOnASingleLine" : "false",
        \ "AllowShortIfStatementsOnASingleLine" : "true",
        \ "AllowShortLoopsOnASingleLine" : "true",
        \ "AllowShortFunctionsOnASingleLine" : "All",
        \ "AlwaysBreakAfterDefinitionReturnType" : "false",
        \ "AlwaysBreakTemplateDeclarations" : "true",
        \ "AlwaysBreakBeforeMultilineStrings" : "true",
        \ "BreakBeforeBinaryOperators" : "None",
        \ "BreakBeforeTernaryOperators" : "true",
        \ "BreakConstructorInitializersBeforeComma" : "false",
        \ "BinPackParameters" : "true",
        \ "BinPackArguments" : "true",
        \ "ColumnLimit" : 80,
        \ "ConstructorInitializerAllOnOneLineOrOnePerLine" : "true",
        \ "ConstructorInitializerIndentWidth" : 4,
        \ "DerivePointerAlignment" : "true",
        \ "ExperimentalAutoDetectBinPacking" : "false",
        \ "IndentCaseLabels" : "true",
        \ "IndentWrappedFunctionNames" : "false",
        \ "IndentFunctionDeclarationAfterType" : "false",
        \ "MaxEmptyLinesToKeep" : 1,
        \ "KeepEmptyLinesAtTheStartOfBlocks" : "false",
        \ "NamespaceIndentation" : "None",
        \ "ObjCBlockIndentWidth" : 2,
        \ "ObjCSpaceAfterProperty" : "false",
        \ "ObjCSpaceBeforeProtocolList" : "false",
        \ "PenaltyBreakBeforeFirstCallParameter" : 1,
        \ "PenaltyBreakComment" : 300,
        \ "PenaltyBreakString" : 1000,
        \ "PenaltyBreakFirstLessLess" : 120,
        \ "PenaltyExcessCharacter" : 1000000,
        \ "PenaltyReturnTypeOnItsOwnLine" : 200,
        \ "PointerAlignment" : "Left",
        \ "SpacesBeforeTrailingComments" : 2,
        \ "Cpp11BracedListStyle" : "true",
        \ "Standard" : "Auto",
        \ "IndentWidth" : 2,
        \ "TabWidth" : 8,
        \ "UseTab" : "Never",
        \ "BreakBeforeBraces" : "Attach",
        \ "SpacesInParentheses" : "false",
        \ "SpacesInSquareBrackets" : "false",
        \ "SpacesInAngles" : "false",
        \ "SpaceInEmptyParentheses" : "false",
        \ "SpacesInCStyleCastParentheses" : "false",
        \ "SpaceAfterCStyleCast" : "false",
        \ "SpacesInContainerLiterals" : "true",
        \ "SpaceBeforeAssignmentOperators" : "true",
        \ "ContinuationIndentWidth" : 4 }

```

#### 验证配置成功与否
用vim打开任一文件，执行：`echo executable('clang-format') `看看输出是否为1，若是则代表`clang-format`在vim中是可执行的。

可以打开任一`C/C++`文件，直接按`F4`执行`clang-format`了。
