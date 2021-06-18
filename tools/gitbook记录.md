# 版本

本文档使用的 GitBook 版本为 3.2.3。

# 安装过程记录

首先本地环境需要支持node，具体node安装自行google。

## 安装gitbook

```bash
npm install gitbook-cli -g
```

查看gitbook版本

```bash
gitbook -V
```

## 初始化

进入指定目录下执行命令完成初始化：

```bash
gitbook init
```

初始化完成后会在当前文件夹下生成几个文件，特别注意SUMMARY.md，这里定义的是整个记录的目录机构。

## 构建章节结构

运行下面命令来构建章节结构：

```bash
$ gitbook build
```

或者直接运行下面命令可在浏览器查看书本信息

```bash
$ gitbook serve
```

浏览器中输入http://localhost:4000，即可查看




## 插件

gitbook使用插件，需要在当前工作的目录下创建book.json文件，并编辑它，

示例：

```json
{
    "title": "干货笔记",
    "description": "日常、笔记",
    "author": "zhangpk",
    "plugins": [
        "chapter-fold", 
        "lightbox",
        "-lunr", "-search","search-pro",
        "code",
        "hide-element"
    ],
    "pluginsConfig": {
        "hide-element": {
            "elements": [".gitbook-link"]
        }
    },
    "language": "zh-hans"
}
```

说明：

gitbook的插件前缀都是：npm install gitbook-plugin-xxx，这种方式来完成安装，示例：

### 中文搜索
- search-pro 需要去除自带的搜索 "-lunr", "-search"
### 左侧目录可折叠
- chapter-fold
### 图片弹窗
- lightbox
### 隐藏的元素
- hide-element


## 导出

如果是要导出PDF，ePub或者mobi格式的电子书时，需要安装Calibre电子书阅读/管理器和命令行工具，不然可能会报错“EbookError: Error during ebook generation: 'ebook-convert'”。

calibre 官网: https://calibre-ebook.com/

windowns下载完直接安装后，会自动在系统环境变量中添加该路径，所以理论上安装完毕之后就可以进行导出操作。