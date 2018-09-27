## 关于format函数与占位符相关问题

在工作中我们经常性的会使用`format`函数来代替使用占位符，一般来说绝大多数占位符能完成的事情`format`函数都可以完成，但是最近我遇到一个问题，反映出了`format`函数的局限性。比如说：我们想做一个输入Excel文件，然后完成`sql`语句字符串自动化替换

```python
import xlrd

file_path = input("请输入文件路径及文件名：")
# 打开Excel
excel = xlrd.open_workbook(file_path)
# 获取sheet页
table = excel.sheet_by_index(0)
# 获取行数
nrows = table.nrows
# 获取列数
ncols = table.ncols
# 第一行为表名，获取表名
table_name = table.row_values(0)[0]
# 第二行为字段名，获取字段名列表
field_name = table.row_values(1)
# 这个时候format函数传参就是个问题，因为你不知道Excel里有个什么玩意儿
sql_r = """insert into "{}"(""" + "{}," * (ncols - 1) + "{}) values(" + "'{}'," * (ncols - 1) + "'{}'"
sql = str1.format(？？？)
```

这个时候大家可以看到`format`函数在这种情况下就无法满足需求,但是使用占位符只要添加几行代码就可以完成我们的需求

```python
import xlrd
import copy

file_path = input("请输入文件路径及文件名：")
# 打开Excel
excel = xlrd.open_workbook(file_path)
# 获取sheet页
table = excel.sheet_by_index(0)
# 获取行数
nrows = table.nrows
# 获取列数
ncols = table.ncols
# 第一行为表名，获取表名
table_name = table.row_values(0)[0]
# 第二行为字段名，获取字段名列表
field_name = table.row_values(1)
# 定义列表，仅存表名
list1 = [table_name]
# for循环将字段名添加到列表中
for i in range(ncols):
    list1.append(field_name[i])
# for循环遍历所有行
for j in range(2, nrows):
    # 确保每次循环时列表里都只有表名和字段名
    list2 = copy.deepcopy(list1)
    # 获取行数据
    rows_values = table.row_values(j)
    # for循环将各字段对应的数据添加到列表中
    for i in range(ncols):
        list2.append(rows_values[i])
    # 占位符的语法是"%s%s%s" %('a','s','d'),可以看出%后面是一个元组，所以把list1转为元组
    tuple1 = tuple(list2)
	sql_r = """insert into "%s"(""" + "%s," * (ncols - 1) + "%s) values(" + "'%s'," * (ncols - 1) + "'%s')"
    # 替换
    sql = sql_r %tuple1
```

