---
title: Python 使用 pkgutil 访问包内资源
date: 2022-10-31 17:42:21.424
updated: 2022-10-31 17:42:21.424
categories: 
- Dev
tags: 
- Python
---

## 结论

`pkgutil` 是 `python` 的内置模块，可以用来打开包内文件。

使用的时候如下：

```py
import pkgutil

def foo():
    data: bytes = pkgutil.get_data(__name__, 'file.txt')
    content: str = data.decode('utf-8')
    print(content)
```

如果你的函数需要一个文件，比如 `PIL` 打开图片，你可以用 `io.BytesIO` 来模拟，如下：

```py
import io
import pkgutil

from PIL import Image


def bar():
    fp = io.BytesIO(pkgutil.get_data(__name__, 'test.png'))
    img = Image.open(fp)
```

## 配合 setup

如果说你想让这个包安装到 `pip` 里，那需要我们配置一下 `setup.cfg` 或 `setup.py`，可以参考这个链接：https://setuptools.pypa.io/en/latest/userguide/datafiles.html

就这样，拜拜。
