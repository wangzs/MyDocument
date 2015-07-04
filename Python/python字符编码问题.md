# Python字符编码问题
> 字符串在Python内部的表示是unicode编码，因此，在做编码转换时，通常需要以unicode作为中间编码，即先将其他编码的字符串解码（decode）成unicode，再从unicode编码（encode）成另一种编码。

python的decode和encode是相对于python自己的unicode编码来说的。
**比如如果想对gb2312编码的中文字符转为unicode编码，需要解码decode，即将其解密为python的unicode编码；如果一个unicode编码的字符串需要转为其他格式编码，即属于一个加密的过程，需要编码encode**

```python
str = "中文字符"
print str		// 中文字符
print str.decode('gb2312')	// 中文字符

// decode的作用是：将参数所对应的编码格式的字符串转为unicode编码。
"哈".decode('gb2312')	// u'\u54c8'
'abc'.decode('gb2312')	 // u'abc'

// encode的作用是：将unicode编码的字符串转为参数对应的编码字符串
u'\u54c8'.encode('gb2312')	// '\xb9\xfe'
'abc'.encode('gb2312')		// 'abc'
```
==如果需要将某种非unicode编码的转为另一种非unicode编码的字符串，需要先decode在encode==
例：str.decode('gbk').encode('utf-8')



###### 对形如`'\u54c8'`如何输出为中文字符
```python
'\u54c8'.decode('unicode-escape')	// u'\u54c8'，这样才可以当成中文字符转为unicode编码

unicode('\u54c8')	// u'\\u54c8' 即未把\u看成转义字符，直接当成ascii
```


###### 用来判断是否为unicode
> isinstance(s, unicode)

###### 如何获得系统的默认编码？
```python
#!/usr/bin/env python
#coding=utf-8
import sys
printsys.getdefaultencoding()
```












