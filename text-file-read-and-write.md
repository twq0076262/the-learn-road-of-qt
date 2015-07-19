# Qt 学习之路(57): 文本文件读写

二进制文件比较小巧，但是不是人可读的格式。文本文件是一种人可读的格式的文件，为了操作这种文件，我们需要使用 QTextStream 类。QTextStream 和 QDataStream的使用类似，只不过它是操作纯文本文件的。还有一些文本格式，比如 XML、HTML，虽然可以由 QTextStream 生成，但 Qt 提供了更方便的 XML 操作类，这里就不包括这部分内容了。

QTextStream 会自动将 Unicode 编码同操作系统的编码进行转换，这一操作对程序员是透明的。它也会将换行符进行转换，同样不需要你自己去处理。QTextStream 使用16位的 QChar 作为基础的数据存储单位，同样，它也支持 C++标准类型，如 int 等。实际上，这是将这种标准类型与字符串进行了相互转换。

QTextStream 同 QDataStream 使用基本一致，例如下面的代码将把“Thomas M. Disch: 334/n”写入到 tmp.txt 文件中：

```

QFile file("sf-book.txt");  
if (!file.open(QIODevice::WriteOnly)) {  
    std::cerr << "Cannot open file for writing: " 
              << qPrintable(file.errorString()) << std::endl;  
    return;  
}  

QTextStream out(&file);  
out << "Thomas M. Disch: " << 334 << endl; 
```

可以看到，这段代码同前面的 QDataStream 相关代码基本雷同。文本文件的写入比较容易，但是读出就不那么简单了。例如，

```

out << "Denmark" << "Norway"; 
```

是我们写入的代码。我们分别写入两个单词，然后试图以与二进制文件读出的格式相同的形式读出：

```

in >> str1 >> str2; 
```

上面两段代码的 out 和 in 都是 QTextStream 类型的。虽然我们可以正常写入，但读出的时候，str1里面将是 DenmarkNorway，str2 是空的。以文本形式写入数据，是不能区分数据的截断位置的。因为使用 QDataStream 写入的时候，实际上是要在字符串前面写如长度信息的。因此，对于文本文件，更多的是一种全局性质的操作，比如使用 QTextStream::readLine() 读取一行，使用 QTextStream::readAll() 读取所有文本，之后再对获得的 QString 对象进行处理。
默认情况下，QTextStream 使用操作系统的本地编码进行读写。不过你可以使用 setCodec() 函数进行设置，比如

```

stream.setCodec("UTF-8"); 
```

同 <iostream> 类似，QTextStream 也提供了一些用于格式化输出的描述符，称为 stream manipulators。这些描述符放置在输出内容之前，或者是使用相应的函数，用于对后面的输出内容做格式化。具体的描述符如下


|setIntegerBase(int)|
|:----|:---------------|
|0	|读出时自动检测数字前缀|
|2	|二进制|
|8	|八进制|
|10	|十进制|
|16	|十六进制|

|setNumberFlags(NumberFlags)|
|:----|:---------------|
|ShowBase|	显示前缀，二进制显示0b，八进制显示0，十六进制显示0x|
|ForceSign	|在实数前面显示符号|
|ForcePoint	|在数字中显示点分隔符|
|UppercaseBase	|使用大写的前缀，如0B, 0X|
|UppercaseDigits	|使用大写字母做十六进制数字|

|setRealNumberNotation(RealNumberNotation)|
|:----|:---------------|
|FixedNotation	|定点计数表示，如0.000123|
|ScientificNotation	|科学计数法表示，如1.23e-4|
|SmartNotation	|定点或科学计数法表示，自动选择简洁的一种表示法|

|setRealNumberPrecision(int)|
|:--------------------------|
|设置生成的最大的小数位数，默认是6|

|setFieldWidth(int)|
|:--------------------------|
|设置一个字段的最小值，默认是0|

|setFieldAlignment(FieldAlignment)|
|:----|:---------------|
|AlignLeft	|左对齐|
|AlignRight	|右对齐|
|AlignCenter	|中间对齐|
|AlignAccountingStyle	|符号和数字之间对齐|


|setPadChar(QChar)|
|:--------------------|
|设置对齐时填充的字符，默认是空格|

比如，下面的代码

```

out << showbase << uppercasedigits << hex << 12345678;
```

将输出0xBC614E。或者我们可以这样去写：

```

out.setNumberFlags(QTextStream::ShowBase | QTextStream::UppercaseDigits);  
out.setIntegerBase(16);  
out << 12345678; 
```

QTextStream 不仅仅可以输出到 QIODevice 上，也可以输出到 QString 上面，例如

```

QString str;  
QTextStream(&str) << oct << 31 << " " << dec << 25 << endl; 
```
本文出自 “豆子空间” 博客，请务必保留此出处 [http://devbean.blog.51cto.com/448512/193918](http://devbean.blog.51cto.com/448512/193918)