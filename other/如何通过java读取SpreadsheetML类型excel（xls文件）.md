如何通过java读取SpreadsheetML类型excel（xls文件）

## 什么是SpreadsheetML格式的excel

最近公司业务需要，excel解析出现通过poi的workbook解析失败的问题，文件本身没有任何损坏，奇怪的是，通过记事本文本编辑器都可以直接解析成具有xml标签的文本信息，也不清楚这是什么，大致解析出来是这样的：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?mso-application progid="Excel.Sheet"?>
<Workbook xmlns="urn:schemas-microsoft-com:office:spreadsheet" xmlns:o="urn:schemas-microsoft-com:office:office" xmlns:x="urn:schemas-microsoft-com:office:excel" xmlns:ss="urn:schemas-microsoft-com:office:spreadsheet" xmlns:html="http://www.w3.org/TR/REC-html40">
  <Worksheet ss:Name="报表xxx">
    <Table ss:ExpandedColumnCount="100" ss:ExpandedRowCount="1000000" ss:DefaultColumnWidth="54.0" ss:DefaultRowHeight="13.5">
 <Row>
  <Cell>
    <Data ss:Type="String">...</Data>
  </Cell>
</Row>
    </Table>
  </Worksheet>
</Workbook>
```

有点懵逼，没有遇到过这种奇葩文件啊，秉着存在即合理的信念，开始疯狂google，最后通过这篇：[原文点我](https://www.cnblogs.com/sungcong/archive/2013/02/19/2916611.html)，终于了解这种excel适用于什么环境，原来是用于网页内嵌套可以打开直接阅读类型的excel，可以叫它`SpreadsheetML`格式的excel。

## 对SpreadsheetML格式的excel进行读写

搞了半天，终于知道这种类型的文件是干什么用，最重要的是知道它叫什么名字了，接下来相对就简单了，顺着往下找，发现java语言操作SpreadsheetML类型的文件，有专门操作工具类，名为Xelem的库可以处理它，但是，如果你的java项目是使用maven仓库来管理依赖，是没有指定依赖库在maven维护的。

![](https://cdn.jsdelivr.net/gh/talkzhang/imgs-bed@master/image/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_16100861032214.png)

不过这都不是什么问题，可以找到官网，官网有依赖下载的，不过可能下载没有那么快。

官网链接：(http://xelem.sourceforge.net/)[http://xelem.sourceforge.net/]，如果你从官网上下载，首先下载下来的是zip压缩包，解压后进入文件夹可以看到两个jar包，xelem.jar，xelem_src.3.1.jar，对我目前来说，只需要安装xlem.jar就可以满足我的需要，就是通过java进行简单的读写。

我将我下载好的依赖上传到了百度网盘，方便日后进行下载。

链接: https://pan.baidu.com/s/1tmai5qXoJXA2gsz_PwN5jA 提取码: zjag 

本地对xelem.jar执行mvn install，以下是简单示例：
```
mvn install:install-file -Dfile=xelem.jar -DgroupId=nl.fountain.xelem -DartifactId=sdk-202008122004 -Dversion=3.1.0 -Dpackaging=jar
```

工具类代码示例：

```java
public class XelemExcelUtil {

    public XelemExcelUtil() {}


    /**
     * 读取SpreadsheetML类型excel
     *
     * @param in
     * @return
     * @throws Exception
     */
    public static List<Map<String, Object>> readExcel(InputStream in,int size) throws Exception {
        ExcelReader excelReader = new ExcelReader();
        InputSource inputSource = new InputSource(in);
        Workbook workbook = excelReader.getWorkbook(inputSource);
        List<String> sheetNameList = workbook.getSheetNames();

        Worksheet worksheet = workbook.getWorksheet(sheetNameList.get(0));
        List<Map<String, Object>> list = new ArrayList<>();
        Collection<Row> rows = worksheet.getRows();
        for (int i = 1; i <= rows.size(); i++) {
            Row row = worksheet.getRowAt(i);
            Collection<Cell> cells = row.getCells();
            int cellSize = size;
            Map<String, Object> map = new HashMap<>();
            for (int j = 1; j <= cellSize; j++) {
                String cellKey = toExcelColumnName(j);
                Cell cell = row.getCellAt(j);
                Object data = cell.getData();
                map.put(cellKey, data);
            }
            list.add(map);
        }

        return list;
    }

    public static String toExcelColumnName(int data){
        String s = new String();
        while (data > 0){
            int m = data % 26;
            if (m == 0) {
                m = 26;
            }
            s = (char)(m + 64) + s;
            data = (data - m) / 26;
        }
        return s;
    }

    public static byte[] writeExcel(InputStream is,List<Map<String,String>> list) throws Exception {
        Workbook workbook;
        ExcelReader excelReader = new ExcelReader();
        try {
            InputSource inputSource = new InputSource(is);
            workbook = excelReader.getWorkbook(inputSource);
        } catch (Exception e) {
            throw new Exception("excel无法解析或者文件流错误");
        }
        List<String> sheetNameList = workbook.getSheetNames();
        Worksheet worksheet = workbook.getWorksheet(sheetNameList.get(0));
        for (Map<String, String> data : list) {
            for (Map.Entry<String, String> entry : data.entrySet()) {
                String key = entry.getKey();
                String value = entry.getValue();
                char[] buf = key.toUpperCase().toCharArray();
                StringBuffer rowIdBuffer = new StringBuffer();
                StringBuffer cellIdBuffer = new StringBuffer();

                for (int i = 0; i < buf.length; i++) {
                    char c = buf[i];
                    if (c < 'A' || c > 'Z') {
                        rowIdBuffer.append(c);
                    } else {
                        cellIdBuffer.append(c);
                    }
                }

                int cellId = formExcelColumnName(cellIdBuffer.toString());
                try {
                    int rowId = Integer.parseInt(rowIdBuffer.toString());
                    Row row = worksheet.getRowAt(rowId );
                    if (row == null) {
                        row = worksheet.addRowAt(rowId );
                    }
                    Cell cell = row.getCellAt(cellId );
                    if (cell == null) {
                        cell = row.addCellAt(cellId );
                    }
                    cell.setData(value);
                } catch (Exception e) {
                    throw new Exception("excel无法解析或者文件流错误");
                }
            }
        }
        ByteArrayOutputStream out = new ByteArrayOutputStream();
        new XSerializer().serialize(workbook,out);
        return out.toByteArray();
    }

    public static int formExcelColumnName(String data){
        if (StringUtils.isEmpty(data)) {
            return 0;
        }
        int n = 0;
        char[] buf = data.toUpperCase().toCharArray();
        for (int i = buf.length - 1, j = 1; i >= 0; i--, j *= 26) {
            char c = buf[i];
            if (c < 'A' || c > 'Z') {
                return 0;
            }
            n += ((int) c - 64) * j;
        }
        return n;
    }

}
```

END，我们学习之后，经常吐槽大部分东西记不住容易遗忘，但是通过学习我们能总结出更高效的方法来解决问题，说白了任何学习都是在学习解决问题的方法。