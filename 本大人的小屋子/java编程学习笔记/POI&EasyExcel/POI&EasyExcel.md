
# poi

![[img/Pasted image 20240112141350.png]]



![[img/Pasted image 20240112143259.png]]



Excel 2007版本最多支持1048576行 
![[img/Pasted image 20240112143436.png]]

Excel 2003版本最多支持行 65536行
![[img/Pasted image 20240112143524.png]]




> [!问题] pol存在的问题
> 内存损耗过大
>


 *poi测试依赖


```
<dependencies>  
<dependency>  
<groupId>org.apache.poi</groupId>  
<artifactId>poi</artifactId>  
<version>4.1.2</version>  
</dependency>  
<dependency>  
<groupId>org.apache.poi</groupId>  
<artifactId>poi-ooxml</artifactId>  
<version>4.1.2</version>  
</dependency>  
</dependencies>
```

## 写入
```java
public class Main {  
public static void main(String[] args) throws IOException {  
Main.writeExCEL03();  
  
}  
public static void writeExCEL03() throws IOException {  
Workbook h1=new HSSFWorkbook();  
Sheet sheet=h1.createSheet("03测试");  
  
// 写入数据时，先指定行/row,从得到Row对象  
Row row=sheet.createRow(0);  
  
// row对象指定列，得到具体的可以执行写入操作的单元格对象cell  
Cell cell=row.createCell(0);  
  
// 设置写入数据  
cell.setCellValue("test");  
  
// 输出文件  
FileOutputStream output=new FileOutputStream("./03test.xls");  
h1.write(output);  
output.close();  
}  
public static void writeExCEL07() throws IOException{  
Workbook h1=new XSSFWorkbook();  
Sheet sheet=h1.createSheet("07测试");  
  
// 写入数据时，先指定行/row,从得到Row对象  
Row row=sheet.createRow(0);  
  
// row对象指定列，得到具体的可以执行写入操作的单元格对象cell  
Cell cell=row.createCell(0);  
  
// 设置写入数据  
cell.setCellValue("test");  
  
// 输出文件  
FileOutputStream output=new FileOutputStream("./07test.xlsx");  
h1.write(output);  
output.close();  
  
}
  
  
}


```




## 批量写入

03版本会将数据先写入缓存中，一次性写入，速度极快
07版本是通过读取全部行的数据，内存消耗大，海量数据时可能造成内存溢出，速度较慢。

```java
class batch{  
  
public static void BatchWrite03() throws IOException {  
long start=System.currentTimeMillis();  
  
Workbook h1=new HSSFWorkbook();  
Sheet sheet=h1.createSheet("03Batch");  
  
  
for (int row=0;row<1000;row++){  
Row row1=sheet.createRow(row);  
for (int i=0;i<255;i++){  
Cell cell=row1.createCell(i);  
cell.setCellValue(i+1);  
}  
}  
  
FileOutputStream output=new FileOutputStream("./03Batch.xls");  
h1.write(output);  
output.close();  
  
long end=System.currentTimeMillis();  
  
System.out.println((end-start)/1000);  
}  
}
```




![[img/Pasted image 20240114093306.png]]


## 读取







