​
使用的POI版本为3.14  

在sax模式下读取一个20万行的xlsx报错：org.apache.xmlbeans.XmlException: java.io.CharConversionException: Characters larger than 4 bytes are not supported: byte 0xb1 implies a length of more than 4 bytes  

到处找答案说是xlsx文件有乱码，sax转换为xml其解析时xml文件中有超过了4byte的字符，故抛出此异常。  

各种研究均无法解决，最后发现如下代码，将读取xlsx的代码替换为如下，成功解决，原因不明。  

此处给出大神的[原地址](https://download.csdn.net/download/zwyjg/9606945)  

我自己的代码对以上链接的源码做了些修改。  

ExcelTool.java：  
```java
package Tools;

import java.io.File;
import java.util.ArrayList;
import java.util.LinkedHashMap;
import java.util.List;
import java.util.Map;

import org.apache.poi.openxml4j.opc.OPCPackage;
import org.apache.poi.openxml4j.opc.PackageAccess;
import org.apache.poi.ss.usermodel.Cell;
import org.apache.poi.ss.usermodel.DataFormatter;
import org.apache.poi.ss.usermodel.Row;
import org.apache.poi.ss.usermodel.Sheet;
import org.apache.poi.ss.usermodel.Workbook;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;

import Tools.RowDataProcesser;
import Tools.Xlsx2ListData;

public class ExcelTool {
	/**
	 * 读取xlsx文件，不适用于读取大文件
	 */
	public static List<List<String>> read(String filePath) throws Exception {
		List<List<String>> result = new ArrayList<>();
		Workbook wb = new XSSFWorkbook(new File(filePath));
		for (Sheet sheet : wb) {
			for (Row row : sheet) {
				List<String> rowData = getRowData(row);
				result.add(rowData);
			}
		}
		wb.close();
		return result;
	}

	/**
	 * 读取xlsx文件，不适用于读取大文件
	 * 
	 * @return map: key->sheet name, value->rowData
	 */
	public static Map<String, List<List<String>>> readMultiSheet(String filePath)
			throws Exception {
		Map<String, List<List<String>>> result = new LinkedHashMap<>();
		Workbook wb = new XSSFWorkbook(new File(filePath));
		for (Sheet sheet : wb) {
			List<List<String>> sheetResult = new ArrayList<>();
			for (Row row : sheet) {
				List<String> rowData = getRowData(row);
				sheetResult.add(rowData);
			}
			result.put(sheet.getSheetName(), sheetResult);
		}
		wb.close();
		return result;
	}

	/**
	 * 以SAX模式读取xlsx文件，读取后将结果全部加载到内存，适用于读取较大文件
	 * 
	 * @param filePath
	 *            文件路径
	 * @param minColumns
	 *            补齐到多少列，-1表示不补齐
	 */
	public static List<List<Object>> readBigFile(String filePath, int minColumns)
			throws Exception {
		File xlsxFile = new File(filePath);
		OPCPackage p = OPCPackage.open(xlsxFile.getPath(), PackageAccess.READ);
		Xlsx2ListData xlsx2ListData = new Xlsx2ListData(p, minColumns, null);
		List<List<Object>> result = xlsx2ListData.process();
		p.close();
		return result;
	}

	/**
	 * 以SAX方式读取xlsx文件，读取每一行调用RowDataProcesser，适用于读超大文件
	 * 
	 * @param filePath
	 *            文件路径
	 * @param minColumns
	 *            补齐到多少列，-1表示不补齐
	 * @param rowDataProcesser
	 *            处理每一行的数据
	 */
	public static void readBigFile(String filePath, int minColumns,
			RowDataProcesser rowDataProcesser) throws Exception {
		File xlsxFile = new File(filePath);
		OPCPackage p = OPCPackage.open(xlsxFile.getPath(), PackageAccess.READ);
		Xlsx2ListData xlsx2ListData = new Xlsx2ListData(p, minColumns,
				rowDataProcesser);
		xlsx2ListData.process();
		p.close();
	}

	private static List<String> getRowData(Row row) {
		List<String> rowData = new ArrayList<>();
		int cellNum = row.getLastCellNum();
		for (int i = 0; i < cellNum; i++) {
			Cell cell = row.getCell(i);
			if (cell == null) {
				rowData.add("");
				continue;
			}
			switch (cell.getCellType()) {
			case Cell.CELL_TYPE_STRING:
				rowData.add(cell.getRichStringCellValue().getString());
				break;
			case Cell.CELL_TYPE_NUMERIC:
				rowData.add(new DataFormatter().formatCellValue(cell));
				break;
			case Cell.CELL_TYPE_BOOLEAN:
				rowData.add(cell.getBooleanCellValue() + "");
				break;
			case Cell.CELL_TYPE_FORMULA:
				rowData.add(cell.getCellFormula());
				break;
			case Cell.CELL_TYPE_BLANK:
			case Cell.CELL_TYPE_ERROR:
			default:
				rowData.add("");
				break;
			}
		}
		return rowData;
	}
}
```
RowDataProcesser.java：  
```java
package Tools;

import java.util.List;

import model.Tweet;

public interface RowDataProcesser {

	/**
	 * 处理一行的数据
	 * @param rowData: 该行数据
	 */
	public void processRowData(List<Object> rowData);
}
```
Xlsx2ListData.java:  
```java
package Tools;

import java.io.IOException;
import java.io.InputStream;
import java.util.ArrayList;
import java.util.List;

import javax.xml.parsers.ParserConfigurationException;

import org.apache.poi.openxml4j.exceptions.OpenXML4JException;
import org.apache.poi.openxml4j.opc.OPCPackage;
import org.apache.poi.ss.usermodel.DataFormatter;
import org.apache.poi.ss.util.CellAddress;
import org.apache.poi.ss.util.CellReference;
import org.apache.poi.util.SAXHelper;
import org.apache.poi.xssf.eventusermodel.ReadOnlySharedStringsTable;
import org.apache.poi.xssf.eventusermodel.XSSFReader;
import org.apache.poi.xssf.eventusermodel.XSSFSheetXMLHandler;
import org.apache.poi.xssf.eventusermodel.XSSFSheetXMLHandler.SheetContentsHandler;
import org.apache.poi.xssf.model.StylesTable;
import org.apache.poi.xssf.usermodel.XSSFComment;
import org.xml.sax.ContentHandler;
import org.xml.sax.InputSource;
import org.xml.sax.SAXException;
import org.xml.sax.XMLReader;

public class Xlsx2ListData {
	/**
	 * Uses the XSSF Event SAX helpers to do most of the work of parsing the
	 * Sheet XML, and outputs the contents as a List.
	 */
	private class SheetToList implements SheetContentsHandler {
		private boolean firstCellOfRow = false;
		private int currentRow = -1;
		private int currentCol = -1;
		private Object defaultValue = "";

		List<Object> currRowData = new ArrayList<>();
		private List<List<Object>> data = new ArrayList<>();

		private void outputMissingRows(int number) {
			for (int i = 0; i < number; i++) {
				List<Object> rowData = new ArrayList<>();
				for (int j = 0; j < minColumns; j++) {
					rowData.add(defaultValue);
				}
				processRowData(rowData);
			}
		}

		private void processRowData(List<Object> rowData) {
			if (rowDataProcesser != null) {
				rowDataProcesser.processRowData(rowData);
			} else {
				data.add(rowData);
			}
			currRowData = new ArrayList<>();
		}

		public List<List<Object>> getData() {
			return data;
		}

		public void startRow(int rowNum) {
			// If there were gaps, output the missing rows
			outputMissingRows(rowNum - currentRow - 1);
			// Prepare for this row
			firstCellOfRow = true;
			currentRow = rowNum;
			currentCol = -1;
		}

		public void endRow(int rowNum) {
			// Ensure the minimum number of columns
			for (int i = currentCol; i < minColumns - 1; i++) {
				currRowData.add(defaultValue);
			}
			processRowData(currRowData);
		}

		@Override
		public void cell(String cellReference, String formattedValue,
				XSSFComment comment) {
			if (firstCellOfRow) {
				firstCellOfRow = false;
			}

			// gracefully handle missing CellRef here in a similar way as
			// XSSFCell does
			if (cellReference == null) {
				cellReference = new CellAddress(currentRow, currentCol)
						.formatAsString();
			}

			// Did we miss any cells?
			int thisCol = (new CellReference(cellReference)).getCol();
			int missedCols = thisCol - currentCol - 1;
			for (int i = 0; i < missedCols; i++) {
				currRowData.add(defaultValue);
			}
			currentCol = thisCol;

			currRowData.add(formattedValue);
		}

		public void headerFooter(String text, boolean isHeader, String tagName) {
			// Skip, ignore headers or footers
		}
	}

	// /////////////////////////////////////

	private final OPCPackage xlsxPackage;

	/**
	 * Number of columns to read starting with leftmost
	 */
	private final int minColumns;

	private final RowDataProcesser rowDataProcesser;

	/**
	 * Creates a new XLSX -> List converter
	 *
	 * @param pkg
	 *            The XLSX package to process
	 * @param minColumns
	 *            The minimum number of columns to output, or -1 for no minimum
	 * @param rowDataProcesser
	 *            process row data
	 */
	public Xlsx2ListData(OPCPackage pkg, int minColumns,
			RowDataProcesser rowDataProcesser) {
		this.xlsxPackage = pkg;
		this.minColumns = minColumns;
		this.rowDataProcesser = rowDataProcesser;
	}

	/**
	 * Parses and shows the content of one sheet using the specified styles and
	 * shared-strings tables.
	 *
	 * @param styles
	 * @param strings
	 * @param sheetInputStream
	 */
	public void processSheet(StylesTable styles,
			ReadOnlySharedStringsTable strings,
			SheetContentsHandler sheetHandler, InputStream sheetInputStream)
			throws IOException, ParserConfigurationException, SAXException {
		DataFormatter formatter = new DataFormatter();
		InputSource sheetSource = new InputSource(sheetInputStream);
		try {
			XMLReader sheetParser = SAXHelper.newXMLReader();
			ContentHandler handler = new XSSFSheetXMLHandler(styles, null,
					strings, sheetHandler, formatter, false);
			sheetParser.setContentHandler(handler);
			sheetParser.parse(sheetSource);
		} catch (ParserConfigurationException e) {
			throw new RuntimeException("SAX parser appears to be broken - "
					+ e.getMessage());
		}
	}

	/**
	 * Initiates the processing of the XLS workbook file to List.
	 *
	 * @throws IOException
	 * @throws OpenXML4JException
	 * @throws ParserConfigurationException
	 * @throws SAXException
	 */
	public List<List<Object>> process() throws IOException, OpenXML4JException,
			ParserConfigurationException, SAXException {
		ReadOnlySharedStringsTable strings = new ReadOnlySharedStringsTable(
				this.xlsxPackage);
		XSSFReader xssfReader = new XSSFReader(this.xlsxPackage);
		StylesTable styles = xssfReader.getStylesTable();
		XSSFReader.SheetIterator iter = (XSSFReader.SheetIterator) xssfReader
				.getSheetsData();

		List<List<Object>> result = new ArrayList<>();
		while (iter.hasNext()) {
			InputStream stream = iter.next();
			SheetToList sheetToList = new SheetToList();
			processSheet(styles, strings, sheetToList, stream);
			result.addAll(sheetToList.getData());
			stream.close();
		}
		return result;
	}

}
```

​
