# 解决方案一
使用iconv命令对excel的编码进行转换

```
iconv -f UTF8 -t GB18030 ~/Downloads/a.csv > ~/Downloads/b.csv
```

若是抛出如下类似错误
```
iconv: conversion from UTF8 unsupported

iconv: try 'iconv -l' to get the list of supported encodings

```

即表明编码的名字可能不是UTF8.那么按照提示输入`iconv -l`查看输出中类似UTF8的名字是什么
```
ANSI_X3.4-1968 ANSI_X3.4-1986 ASCII CP367 IBM367 ISO-IR-6 ISO646-US ISO_646.IRV:1991 US US-ASCII CSASCII
UTF-8  # 这个可能就是我们需要的
ISO-10646-UCS-2 UCS-2 CSUNICODE
UCS-2BE UNICODE-1-1 UNICODEBIG CSUNICODE11
UCS-2LE UNICODELITTLE
ISO-10646-UCS-4 UCS-4 CSUCS4
UCS-4BE
UCS-4LE
UTF-16
UTF-16BE
UTF-16LE
...
```

重新试一下
```
iconv -f UTF-8 -t GB18030 ~/Downloads/a.csv > ~/Downloads/b.csv
```

没有抛出错误,成功了.用Excel再次打开,发现中文正常显示了

# 解决方案二

下载安装**WPS For Mac**.
打开即显示正常.亲测有效.