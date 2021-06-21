# vscode使用python提示无法加载

检查本地使用的vscode插件，一多半是因为某几个插件冲突导致的插件失效。可以点击扩展->右上角三个点->禁用所有已安装部分->只开启python所需要的的插件即可。

# 如何将markdown格式代码补全

在主界面中，文件 >>首选项>>用户片段，选择markdown语言，然后json文件即可。

示例如下：

```json
{
	// Place your snippets for markdown here. Each snippet is defined under a snippet name and has a prefix, body and 
	// description. The prefix is what is used to trigger the snippet and the body will be expanded and inserted. Possible variables are:
	// $1, $2 for tab stops, $0 for the final cursor position, and ${1:label}, ${2:another} for placeholders. Placeholders with the 
	// same ids are connected.
	// Example:
	"font_red": {
		"prefix": "font-red",
		"body": [
			"<font color=red>$1</font>$0"
		],
		"description": "红色字体"
	},

	"code_block": {  
        "prefix": "```", 
        "body":   "```\n$1\n```$2"
           // 另外$1代表光标默认位置,$2代表按下tab后会切换至的下一个光标落点,此外shift+tab能够切换至上一个光标落点
    },
	"java_block": {  
        "prefix": "```java", 
        "body":   "```java\n$1\n```$2"
           // 另外$1代表光标默认位置,$2代表按下tab后会切换至的下一个光标落点,此外shift+tab能够切换至上一个光标落点
    },
	"xml_block": {  
        "prefix": "```xml", 
        "body":   "```xml\n$1\n```$2"
           // 另外$1代表光标默认位置,$2代表按下tab后会切换至的下一个光标落点,此外shift+tab能够切换至上一个光标落点
    },
	"json_block": {  
        "prefix": "```json", 
        "body":   "```json\n$1\n```$2"
           // 另外$1代表光标默认位置,$2代表按下tab后会切换至的下一个光标落点,此外shift+tab能够切换至上一个光标落点
    },
}
```