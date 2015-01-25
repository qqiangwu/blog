Markdown这种格式真是小巧便利，所以有什么理由不使用它呢。

我在电脑中使用的是Notepad++，所以我也懒得加专门的Markdown编辑器了，直接为Notepad++添加Markdown支持。

### 添加语言支持
Github上有适用于Notepad++的Markdown语言包，导入后Notepad++就有Markdown的高亮效果了。

### 预览
当然，预览功能肯定比高亮功能要重要的多了。将正在编辑的md文件转换成html文件很容易，比如在`运行`中输入：

> `python -m markdown $(FULL_CURRENT_PATH)`

而问题在于，如何显示。实际上可以写一些脚本，然后在`运行`中执行，但这样问题似乎麻烦了。我发现Notepad++中存在一个PreviewHTML的插件，并且，其提供Filter，可以用来将其它格式转换成html。Nice！

于是，我写出了这样的Filter文件：

```
; Content of Filter.ini file
[Markdown]
Extension=.md
Language=Markdown
Command=python -m markdown "%1"
```

它已经完全work了，可是烦人的是，它给出的结果和我在网页上用showdown给出的结果不一样！（其实showdown给出的结果和github给出的结果也不一样，真是烦人！），所以我只能用showdown了。

用nodejs下载了showdown，写了一样简单的脚本，然后模仿bower这些工具为其添加了快捷方式。但它在Filter里面根本不work，最终，纠结了N久之后，总算搞出个正确的东西：

```
; Content of Filter.ini file
[Markdown]
Extension=.md
Language=Markdown
Command=node "C:\\Users\\Float\\AppData\\Roaming\\npm\\node_modules\\showdown\\run.js" "%1"
```

That's all!