# 利用python读取图片实现ocr

1、下载Tesseract-OCR，地址：https://digi.bib.uni-mannheim.de/tesseract/，选择对应版本下载即可

2、下载支持中文的版本，地址：https://tesseract-ocr.github.io/tessdoc/Data-Files，选择繁体和简体中文如下：

![](https://cdn.jsdelivr.net/gh/talkzhang/imgs-bed@master/image/20210712134307.png)

选择之后将下载的文件放至Tesseract-OCR安装目录下的tessdata即可。

3、安装所需执行的依赖：

```python
pip install pytesseract
pip install pillow
```

# 编码实现

```python
# encoding=utf8
import pytesseract
from PIL import Image

# 设置该变量路径 如果不设置该依赖会找不到配置项
pytesseract.pytesseract.tesseract_cmd = r'E:\program\Tesseract-OCR\tesseract.exe'

# 读取图片
im = Image.open('C:\\Users\\Zhang Peike\\Desktop\\1.jpg')
# 识别文字 指定简体中文 默认是英文
string = pytesseract.image_to_string(im, lang='chi_sim')
print(string)
```

以上是python实现图片OCR简单总结和了解。