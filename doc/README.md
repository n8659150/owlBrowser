## 源起

[用 PyQt 快速打造一个浏览器](https://zhuanlan.zhihu.com/p/40634325)

市面上大多数浏览器都是现成下载安装后直接使用的，所谓定制化也仅仅是在给定的范围内实现功能上的个性化配置。在拜读了《用PyQt快速打造一个浏览器》一文后，跟着文章一步步实现了一个简易浏览器。麻雀虽小，五脏俱全，虽说未能像主流浏览器一样拥有如下载，书签等功能，但经使用后发现，即便缺少了某些看似必备的高级功能，依然不影响浏览器正常使用。由此也引发了我的思考：对于浏览器，我们真的需要那么多功能吗？如果连功能也可以定制，那会是怎样一番情景？

## 诞生

本着极简主义（断，舍，离）的精神，这款开源的"猫头鹰浏览器"诞生了。这款浏览器主打"极简","开源","高效"。
- 所谓"极简"，浏览器仅提供基本功能，附加功能由用户自行研究，实现;

- 所谓"开源"，浏览器采用MIT开源协议，用户可根据需求任意替换图标，修改源码，只要你够强，你可以用PyQt实现一个Chrome（我没说笑..；

- 所谓"高效"，浏览器主要采用PyQt5.9提供的QWebEngineView实现网页浏览功能，QWebEngineView基于大名鼎鼎的Webkit内核（Google Chrome浏览器采用此内核）,资源处理速度很快。

## 技术栈
- 源代码：Python 3.5.2 + PyQt5.9.1

- 打包工具：Pyinstaller 3.3.1

## 下载
[下载地址1](https://pan.baidu.com/s/1KaAgqsfU9Ri70vVGZ3a7EA)
[下载地址2](http://www.jackyangli.com/owl_browser/owl.zip)

## 安装 & 运行

下载zip文件解压后双击exe运行即可。目前版本只支持64位 windows 系统

## 版本更新

### 1.10 | 2018.08.09
计划推出msi安装包及其他操作系统平台支持

### 1.09 | 2018.08.06
更新代码，浏览器支持flash和javascript了，终于可以上b站看视频了

### 1.08 | 2018.08.04
图标引用地址变更为相对路径，打包后将会从icons文件夹中查找

### 1.07 | 2018.08.03
书签变更

### 1.06 | 2018.08.03
地址栏字体优化，目前为微软雅黑

### 1.05 | 2018.08.02
发布pyinstaller打包后的exe文件，并针对打包配置对代码进行更改

### 1.04 | 2018.08.02

修复点击书签打开页面后，页面标题不同步更新的bug

### 1.03 | 2018.08.02

开启新的branch,增加基于py3.5和PyQt5.6的实现，对API进行相应变更

### 1.02 | 2018.08.01

加入书签功能，目前包括 百度, github 和 知乎 三个站点

### 1.01 | 2018.08.01

基于py3.4和PyQt5.5实现基础功能


## 开发指南

接下去的篇幅将会简单介绍代码骨架及如何实现浏览器的基本功能

### 代码骨架

本浏览器基于python的图形库PyQt实现：定义一个窗体类，传入窗体控件，并在初始化函数中定义图标及相关控件，在类体中定义控件的交互方法，并在main函数中调用初始化函数，实现浏览器窗体及控件的加载和生成。

```python
from PyQt5 import QtCore
from PyQt5.QtCore import *
from PyQt5.QtWidgets import *
from PyQt5.QtGui import *
from PyQt5.QtWebEngineWidgets import QWebEnginePage, QWebEngineView,QWebEngineSettings
import os
import sys
class MainWindow(QMainWindow):
	def __init__(self, *args, **kwargs):
		super().__init__(*args, **kwargs)
		# 设置窗口标题
		self.setWindowTitle('Owl Browser')
		# 设置窗口图标
		self.setWindowIcon(QIcon('./icons/owl.png'))
        ...
    # 添加新的标签页
    def add_new_tab(self, qurl=QUrl(''), label='新标签页'):
        # 为标签创建新网页
        browser = QWebEngineView()
        self.webSettings = browser.settings()
        self.webSettings.setAttribute(QWebEngineSettings.PluginsEnabled,True)
        self.webSettings.setAttribute(QWebEngineSettings.JavascriptEnabled,True)
        browser.setUrl(qurl)
        i = self.tabs.addTab(browser, label)

        self.tabs.setCurrentIndex(i)

        browser.urlChanged.connect(lambda qurl, browser=browser: self.renew_urlbar(qurl, browser))

        browser.loadFinished.connect(lambda _, i=i, browser=browser: 
            self.tabs.setTabText(i, browser.page().title()))
    ...
    if __name__ == '__main__':
    # 创建应用
    app = QApplication(sys.argv)
    # 创建主窗口
    window = MainWindow()
    # 显示窗口
    window.show()
    # 运行应用，并监听事件
    app.exec_()
```

### 定制浏览器图标
浏览器窗口左上角小图标 和 exe文件的图标可根据需求自行更改

- 左上角小图标

```python 
# 设置窗口图标，图片路径改成对应路径即可
self.setWindowIcon(QIcon('./icons/owl.png'))
```
- exe文件图标

将在"打包"一节讲解

### 定制工具栏图标
工具栏图标如"前进","后退","刷新"等也可定制，代码片段如下：
```python
# 添加导航栏
navigation_bar = QToolBar('Navigation')
# 设定图标
back_button = QAction(QIcon('./icons/back.png'), 'Back', self)
# 为按钮绑定点击事件
back_button.triggered.connect(self.tabs.currentWidget().back)
# 将按钮添加到导航栏上
navigation_bar.addAction(back_button)
```

### 定制书签
和工具栏图标相同，书签的添加也需要经历3个步骤：1. 指定图标;2. 绑定点击事件(书签的跳转函数需要自行定义) 和 3.添加至导航栏：

```python 
def go_to_baidu(self):
    q = QUrl('https://www.baidu.com')
    self.tabs.currentWidget().setUrl(q)
baidu_button = QAction(QIcon('./icons/baidu.png'), 'baidu', self)
baidu_button.triggered.connect(self.go_to_baidu)
navigation_bar.addAction(baidu_button)
```

### 定制地址栏字体
浏览器地址栏的字体也可自行定制：

```python
# 添加 URL 地址栏
self.urlbar = QLineEdit()
font = QFont()
# 设置字体
font.setFamily('微软雅黑')
self.urlbar.setFont(font)
```

### 执行
```shell 
python yourfilename.python
```

### 打包
本浏览器通过pyinstaller打包。
先安装pyinstaller：

```shell
pip install pyinstaller
```

再进入代码文件所在的目录，执行pyinstaller的打包命令，其中--icon后所指定的为打包后exe的图标文件，需为ico格式，且路径需为绝对路径:

```shell
cd owl
pyinstaller -F -w --icon=C:\owl.ico --clean owl.py
```


## 关于此文档

此文档由 [docsify](https://github.com/QingWei-Li/docsify/) 生成 docsify 是一个动
态生成文档网站的工具。不同于 GitBook、Hexo 的地方是它不会生成将 .md 转成 .html
文件，所有转换工作都是在运行时进行。

## License

[The MIT License (MIT)](https://github.com/n8659150/owlBrowser/blob/master/LICENSE)