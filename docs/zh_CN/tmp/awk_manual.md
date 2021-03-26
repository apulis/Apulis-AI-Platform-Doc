awk '{ gsub(/AAAA/,"BBBB"); print $0 }' t.txt


awk 替换后写入文件awk的sub/gsub函数用来替换字符串
1
	
sub(/regexp/, replacement, target)

注意第三个参数target，如果忽略则使用$0作为参数，即整行文本。

例子1：替换单个串

只把每行的第一个AAAA替换为BBBB
1
	
awk '{ sub(/AAAA/,"BBBB"); print $0 }' t.txt

例子2：替换所有的串

把每一行的所有AAAA替换为BBBB
1
	
awk '{ gsub(/AAAA/,"BBBB"); print $0 }' t.txt

例子3：替换满足条件的行的串

只在出现字符串CCCC的前提下，将行中所有AAAA替换为BBBB
1
2
3
	
awk '/CCCC/ { gsub(/AAAA/,"BBBB"); print $0; next }
            { print $0 }
    ' t.txt


**参考**
-----------------------------------------------------------------------------

1. [wk 替换文本字符串内容](http://www.pinhuba.com/linux/101488.htm)