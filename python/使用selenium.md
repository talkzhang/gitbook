# 使用selenium实现自动化页面操作

selenium 是一个web应用测试工具，能够真正的模拟人去操作浏览器，可以实现web页面自动化点击，自动化测试很方便，同时也可以用于某些网站批量提交的操作。

# 准备

在使用前，需要准备：

1. [chrome]/浏览器              https://www.google.cn/chrome/
2. [chromedriver]/浏览器驱动   http://chromedriver.storage.googleapis.com/index.html

注意chromedriver和chrome浏览器的版本需要对应，例如：

![](https://gitee.com/hongqigg/imgs-bed/raw/master/image/chromevier-version.png)

如果chrome版本是92的版本，可以直接通过网盘下载驱动：

链接：https://pan.baidu.com/s/102gIrR6IvjGwrw4fqvp69Q 
提取码：1ucb

# code

在写代码之前，需要将chromedriver所写脚本放至一个目录下最方便；当然还有别的方式，没去研究，因为该使用只是一时之需。

打开浏览器，并访问google首页：

```python
from selenium import webdriver

browser = webdriver.Chrome()
browser.get("https://www.google.com")
```

接下来实现一次google搜索，通过F12查看搜索框，可以看到该搜索框没有id标识，那就可以通过Xpath来获取该元素，并输入想要输入的东西，然后因为我们搜索时是通过回车键来完成的，所以实现回车操作即可

获取搜索框Xpath：

![](https://gitee.com/hongqigg/imgs-bed/raw/master/image/find_element.gif)

获取到该Xpath，编码：

```python
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.common.keys import Keys

def demo():
    # 前台开启浏览器模式
    driver = webdriver.Chrome()
    driver.get('https://www.google.com')
    text_elem = driver.find_element_by_xpath('/html/body/div[1]/div[3]/form/div[1]/div[1]/div[1]/div/div[2]/input')
    text_elem.send_keys('spring framework')
    text_elem.send_keys(Keys.ENTER)
    # 在通过vscode启动时，如果该线程运行结束会自动关闭浏览器，所以休眠100s看效果 可以通过IDLE来运行该脚本就不会自动关闭浏览器了
    time.sleep(100)

if __name__ == '__main__':
    demo()
```

可以看到，实现了一次搜索spring框架的流程。

**注意**：在通过vscode启动时，如果该线程运行结束会自动关闭浏览器，所以休眠100s看效果 可以通过IDLE来运行该脚本就不会自动关闭浏览器了

看下效果：

![](https://gitee.com/hongqigg/imgs-bed/raw/master/image/running.gif)

一个简单的流程大致如此，除了通过Xpath，还可以通过id、class来获取页面元素，很简答，不做赘述，自行搜索即可。

如果是自动化测试的场景，那就可以定义一些变量完成登录、页面点击的操作。

如果是批量提交的操作，那就可以准备数据源，比如excel，通过遍历excel来自动完成页面的批量化操作，提高效率，解放人力。