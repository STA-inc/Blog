情景：有一个一条记录有20个字段，共30万条记录的xlsx文件，文件大小大约30兆，需要将其内容读取后写入数据库。  

这意味着需要向数据库写入30万条数据，读取xlsx方面，用的是POI，用了几种方式做，性能对比如下：  

一、MyBatis的xml里逐条写入：一条记录生成一条SQL语句并执行一次，这种做法性能过低，不作讨论。  

二、MyBatis的xml里用foreach方式，一次写入2000条数据  

1.POI:XSSFWorkbook读xlsx：  
```java
List<List<Object>> result = new ArrayList<>();
		Workbook wb = new XSSFWorkbook(new File(filePath));
		for (Sheet sheet : wb) {
			for (Row row : sheet) {
				List<Object> rowData = getRowData(row);
				result.add(rowData);
			}
		}
		wb.close();
		return result;
``` 
 这种方式在xlsx内有30万条数据的情况下报错。  

    2.POI:SAX模式读xlsx:  
```java
File xlsxFile = new File(filePath);
		OPCPackage p = OPCPackage.open(xlsxFile.getPath(), PackageAccess.READ);
		Xlsx2ListData xlsx2ListData = new Xlsx2ListData(p, minColumns, null);
		List<List<Object>> result = xlsx2ListData.process();
		p.close();
		return result;
```
使用该方法一次读取30万条数据，形成一个size=30万的list,然后每次写入2000条数据，耗时1320s。   

3.POI：SAX方式读取xlsx文件，读取每一行调用RowDataProcesser：   
```java
File xlsxFile = new File(filePath);
		OPCPackage p = OPCPackage.open(xlsxFile.getPath(), PackageAccess.READ);
		Xlsx2ListData xlsx2ListData = new Xlsx2ListData(p, minColumns,
				rowDataProcesser);
		xlsx2ListData.process();
		p.close();
```
该方式使用RowDataProcesser接口，读取一行就写入list，list满后向数据库批量写入，同样的机器，同样的数据，耗时1356s  

第二，第三种方式读取xlsx文件分别耗时：13s，12s也就是说，数据库IO占了大部分时间。  

结论：SAX模式读取大文件（30万条数据）向数据库写入的情景下，SAX模式的两种读取方式性能差距不大，真正的性能瓶颈是数据库I/O，针对这个问题，尝试增大list容量，减少数据库IO次数。  

将list容量增大至1万，即每次向数据库写入1万条数据。  

结果报错了：PacketTooBigException  

目前发现最快的方法：  

直接上代码：  
```java
SqlSession session = sqlSessionFactory.openSession(ExecutorType.BATCH);
try {
    SimpleTableMapper mapper = session.getMapper(SimpleTableMapper.class);
    List<SimpleTableRecord> records = getRecordsToInsert(); // not shown

    BatchInsert<SimpleTableRecord> batchInsert = insert(records)
            .into(simpleTable)
            .map(id).toProperty("id")
            .map(firstName).toProperty("firstName")
            .map(lastName).toProperty("lastName")
            .map(birthDate).toProperty("birthDate")
            .map(employed).toProperty("employed")
            .map(occupation).toProperty("occupation")
            .build()
            .render(RenderingStrategy.MYBATIS3);

    batchInsert.insertStatements().stream().forEach(mapper::insert);

    session.commit();
} finally {
    session.close();
}
```
基本思想是将 MyBatis session 的 executor type 设为 Batch ，然后多次执行插入语句。
