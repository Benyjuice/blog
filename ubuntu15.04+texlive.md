---
title: Ubuntu install texlive
date: 2015-04-23
---
##1.安装texlive
```
sudo apt-get install texlive-xetex latex-cjk-all
```
##2.安装texstudio
```sudo apt-get install texstudio```

##3.配置texstudio
打开texstudio
选择Options->Configure TexStudio可以配置语言
在Options->Configure TexStudio->构建 中将默认编译器改为XeLatex即可使用ctex/cjk编辑中文了
