---
layout: theme:post
title: "Python 检测生僻字"
date: 2016-01-09T21:25:49+08:00
---

最近碰到一个需求，要求检测字段是否包含生僻字以及一些非法字符如 `~!@#$%^&* `。

首先想到的就是利用 python 的正则表达式来匹配非法字符，然后找出非法记录。然而理想总是丰满的，现实却是残酷的。在实现的过程中，才发现自己对于字符编码、以及 python 内部字符串表示的相关知识的缺乏。在这期间，踩过了不少坑，到最后虽然还有些模糊的地方，但总算有一个总体清晰的了解。在此记录下心得，避免以后在同一个地方跌倒。

以下的测试环境是 ArcGIS 10.3 自带的 python 2.7.8 环境，不保证其他 python 环境也适用。

### python 正则表达式

python 中的正则功能由内嵌的 re 函数库提供，主要用到 3 个函数。`re.compile()` 提供可重用的正则表达式，`match()` 和 `search()` 函数返回匹配结果，两者之间的区别在于： `match()` 从指定位置开始匹配，`search()` 会从指定位置向后搜索直到找到匹配字符串。例如下面的代码中，`match_result` 从第一个字符 f 开始匹配，匹配失败返回空值；`search_result` 从 f 开始向后搜索，直到找到第一个匹配的字符 a, 然后通过 `group()` 函数输出匹配结果为字符 a。

```python
import re

pattern = re.compile('[abc]')
match_result = pattern.match('fabc')
if match_result:
	print match_result.group()

search_result = pattern.search('fabc')
if search_result:
	print search_result.group()
```

以上的实现方式需要先编译一个 pattern，然后再进行匹配。实际上，我们可以直接利用 `re.match(pattern, string)` 函数来实现相同的功能。但是直接匹配的方式没有先编译再匹配的方式灵活，首先是正则表达式没办法重用，如果大量数据进行同一模式匹配，意味着每次都需要内部编译，造成性能损失；另外，`re.match()` 函数没有 `pattern.match()` 功能强大，后者可以指定从哪个位置开始匹配。


### 编码问题

了解 python 正则的基本功能后，剩下的事情就是找到一个合适的正则表达式来匹配生僻字和非法字符。非法字符很简单，采用以下 pattern 就可以实现匹配：
```python
pattern = re.compile(r'[~!@#$%^&* ]')
```

然而对于生僻字的匹配，着实难倒了我。首先是对于生僻字的定义，什么样的字算生僻字？经过咨询项目经理，规定非 GB2312 的字符属于生僻字。接下来的问题是，如何匹配 GB2312 字符？

经过查询，GB2312 的范围是 `[\xA1-\xF7][\xA1-\xFE]`，其中汉字区的范围是 `[\xB0-\xF7][\xA1-\xFE]`。因此，添加生僻字匹配后的表达式为：
```python
pattern = re.compile(r'[~!@#$%^&* ]|[^\xA1-\xF7][^\xA1-\xFE]')
```

问题似乎是顺理得当地解决了，然而我还是 too simple too naive 。由于要判断的字符串都是从图层文件读取的，arcpy 贴心地将读取的字符编码为 unicode 格式。因此，我需要找出 GB2312 字符集在 unicode 中的编码范围。但现实是，GB2312 字符集在 unicode 中的分布并不是连续的，使用正则表示这个范围必定是非常复杂的。使用正则表达式匹配生僻字的构想，似乎陷入了死胡同。


### 解决方案

既然提供的字符串是 unicode 格式，那么我可不可以将其转换为 GB2312 再进行匹配呢？实际上是不行，因为 unicode 字符集要远大于 GB2312 字符集，因此 `GB2312 => unicode` 总是可以实现的，而反过来 `unicode => GB2312` 不一定能成功。

这突然为我提供了另外一种思路，假设一个字符串的 `unicode => GB2312` 转换会失败，那么是不是恰恰说明了它不属于 GB2312 字符集？所以，我使用 `unicode_string.encode('GB2312')` 函数尝试转换字符串，捕获 `UnicodeEncodeError` 异常来识别生僻字。

最终的代码如下：
```python
import re

def is_rare_name(string):
	pattern = re.compile(u"[~!@#$%^&* ]")
	match = pattern.search(string)
	if match:
		return True

	try:
        string.encode("gb2312")
    except UnicodeEncodeError:
    	return True

    return False
```

