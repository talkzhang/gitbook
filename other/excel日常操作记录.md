> 虽然不经常用excel做数据筛选操作，但是偶尔还是会有这种需求，每次来都是去科普，没往心里记，这次索性做个记录，以后有什么excel计算的场景直接看自己的案例更清晰。

## 两个sheet页筛选

例如有excel表内原始数据（在sheet1内）如下：

![](https://cdn.jsdelivr.net/gh/talkzhang/imgs-bed@master/image/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_16105907874158.png)

由于某些情况，你将某些发生异常的订单给拉了出来，但是你只知道订单号，这个金额由于特殊原因无法获取，于是把这些异常的订单，新建了sheet2来进行记录，如下：

![](https://cdn.jsdelivr.net/gh/talkzhang/imgs-bed@master/image/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_16105910278239.png)

现在需要知道每条订单对应的金额应该是多少，可以使用vlookup函数来进行解决。

在sheet2页内B1列新建一列[金额]，将鼠标放在B2，选择`公式-插入函数-选择vlookup`：

![](https://cdn.jsdelivr.net/gh/talkzhang/imgs-bed@master/image/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_1610591251907.png)

解释一下要填写的几个值的描述：

- Lookup_value 选择要基于哪个列去查询，这里选择A2。
- Table_array 选择查询的区域，可以选多个。
- Col_index_num 选择查询区域的第几列。
- Range_lookup 这个主要是说明精确匹配（0）还是模糊匹配（1）

想要通过sheet2的异常订单号列作为条件，所以`Lookup_value设置为A2`，因为要从sheet1内查询，所以`Table_array设置为Sheet1!A:C`，因为金额在我们选中区域内的第三列，所以`Col_index_num设置为3`，因为要精确匹配，所以`Range_lookup设置为0`。

具体设置如下：

![](https://cdn.jsdelivr.net/gh/talkzhang/imgs-bed@master/image/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_16105918782352.png)

填充后效果如下，姓名列也是按相同的方式进行了设置，

![](https://cdn.jsdelivr.net/gh/talkzhang/imgs-bed@master/image/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_16105930221551.png)

## 如何实现快速下拉

选中公式列，`ctrl+shift+end`，选中后再公式文本框执行`ctrl+enter`，即可完成。

# 如何对数据分组

还是上面的例子，筛选出来异常订单的金额、用户姓名之后，如何按用户进行分组，然后得出每个人下的金额呢？

可以使用数据透视表，在excel内`插入-数据透视表`，对话框中选中数据区域，

![](https://cdn.jsdelivr.net/gh/talkzhang/imgs-bed@master/image/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_1610593390592.png)

求和值设置`金额`列，行标签设置为`用户姓名`即可搞定。

![](https://cdn.jsdelivr.net/gh/talkzhang/imgs-bed@master/image/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_16105934885814.png)

如上，搞定，后续有别的需要再来记录。

# N/A如何替换

例如要替换为空字符串

```
iferror((表达式), '')
```