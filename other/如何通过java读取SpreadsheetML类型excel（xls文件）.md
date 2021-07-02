���ͨ��java��ȡSpreadsheetML����excel��xls�ļ���

## ʲô��SpreadsheetML��ʽ��excel

�����˾ҵ����Ҫ��excel��������ͨ��poi��workbook����ʧ�ܵ����⣬�ļ�����û���κ��𻵣���ֵ��ǣ�ͨ�����±��ı��༭��������ֱ�ӽ����ɾ���xml��ǩ���ı���Ϣ��Ҳ���������ʲô�����½��������������ģ�

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?mso-application progid="Excel.Sheet"?>
<Workbook xmlns="urn:schemas-microsoft-com:office:spreadsheet" xmlns:o="urn:schemas-microsoft-com:office:office" xmlns:x="urn:schemas-microsoft-com:office:excel" xmlns:ss="urn:schemas-microsoft-com:office:spreadsheet" xmlns:html="http://www.w3.org/TR/REC-html40">
  <Worksheet ss:Name="����xxx">
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

�е��±ƣ�û�����������������ļ��������Ŵ��ڼ�����������ʼ���google�����ͨ����ƪ��[ԭ�ĵ���](https://www.cnblogs.com/sungcong/archive/2013/02/19/2916611.html)�������˽�����excel������ʲô������ԭ����������ҳ��Ƕ�׿��Դ�ֱ���Ķ����͵�excel�����Խ���`SpreadsheetML`��ʽ��excel��

## ��SpreadsheetML��ʽ��excel���ж�д

���˰��죬����֪���������͵��ļ��Ǹ�ʲô�ã�����Ҫ����֪������ʲô�����ˣ���������Ծͼ��ˣ�˳�������ң�����java���Բ���SpreadsheetML���͵��ļ�����ר�Ų��������࣬��ΪXelem�Ŀ���Դ����������ǣ�������java��Ŀ��ʹ��maven�ֿ���������������û��ָ����������mavenά���ġ�

![](https://gitee.com/hongqigg/imgs-bed/raw/master/image/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_16100861032214.png)

�����ⶼ����ʲô���⣬�����ҵ��������������������صģ�������������û����ô�졣

�������ӣ�(http://xelem.sourceforge.net/)[http://xelem.sourceforge.net/]�������ӹ��������أ�����������������zipѹ��������ѹ������ļ��п��Կ�������jar����xelem.jar��xelem_src.3.1.jar������Ŀǰ��˵��ֻ��Ҫ��װxlem.jar�Ϳ��������ҵ���Ҫ������ͨ��java���м򵥵Ķ�д��

�ҽ������غõ������ϴ����˰ٶ����̣������պ�������ء�

����: https://pan.baidu.com/s/1tmai5qXoJXA2gsz_PwN5jA ��ȡ��: zjag 

���ض�xelem.jarִ��mvn install�������Ǽ�ʾ����
```
mvn install:install-file -Dfile=xelem.jar -DgroupId=nl.fountain.xelem -DartifactId=sdk-202008122004 -Dversion=3.1.0 -Dpackaging=jar
```

���������ʾ����

```java
public class XelemExcelUtil {

    public XelemExcelUtil() {}


    /**
     * ��ȡSpreadsheetML����excel
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
            throw new Exception("excel�޷����������ļ�������");
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
                    throw new Exception("excel�޷����������ļ�������");
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

END������ѧϰ֮�󣬾����²۴󲿷ֶ����ǲ�ס��������������ͨ��ѧϰ�������ܽ������Ч�ķ�����������⣬˵�����κ�ѧϰ������ѧϰ�������ķ�����