poi创建excel表打不开——文件格式与扩展名不匹配

使用的XSSFWorkbook创建的xls，打开的时候会有这样的提示：
 
```bash
HSSF － 提供读写Microsoft Excel XLS格式档案的功能。

XSSF － 提供读写Microsoft Excel OOXML XLSX格式档案的功能。

HWPF － 提供读写Microsoft Word DOC97格式档案的功能。

XWPF － 提供读写Microsoft Word DOC2003格式档案的功能。

HSLF － 提供读写Microsoft PowerPoint格式档案的功能。

HDGF － 提供读Microsoft Visio格式档案的功能。

HPBF － 提供读Microsoft Publisher格式档案的功能。

HSMF － 提供读Microsoft Outlook格式档案的功能。
```

所以：创建xls文件需要使用HSSF，同理创建xlsx文件就要使用XSSF，这样就可以正常打开了。