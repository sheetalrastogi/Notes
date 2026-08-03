## Create an Apache POI Java utility function to read an excel as List of Map of data in following format:

```text
| Field    | DataSet1 | DataSet2 | DataSet3 | DataSet4 | DataSet5 |
|----------|----------|----------|----------|----------|----------|
| Field1   | 1        | a        | v        | p        | a1       |
| Field2   | 2        | b        | w        | q        | a2       |
| Field3   | 3        | c        | x        | r        | a3       |
| Field4   | 4        | d        | y        | s        | a4       |
```


# Apache POI utility that reads the Excel structure where:

- Column 1 contains field names.
- Each subsequent column (DataSet1, DataSet2, etc.) represents one test data set.
- Returns a List<Map<String, String>>.
- Each Map represents one dataset.

**Excel Structure**:
```text
+--------+----------+----------+----------+----------+----------+
| Field  | DataSet1 | DataSet2 | DataSet3 | DataSet4 | DataSet5 |
+--------+----------+----------+----------+----------+----------+
| Field1 |    1     |    a     |    v     |    p     |    a1    |
| Field2 |    2     |    b     |    w     |    q     |    a2    |
| Field3 |    3     |    c     |    x     |    r     |    a3    |
| Field4 |    4     |    d     |    y     |    s     |    a4    |
+--------+----------+----------+----------+----------+----------+
```

**Expected Output**:
```text
[
    {Field1=1, Field2=2, Field3=3, Field4=4},
    {Field1=a, Field2=b, Field3=c, Field4=d},
    {Field1=v, Field2=w, Field3=x, Field4=y},
    {Field1=p, Field2=q, Field3=r, Field4=s},
    {Field1=a1, Field2=a2, Field3=a3, Field4=a4}
]
```

# Apache POI Utility method:
---

```java
package utils;

import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;

import java.io.FileInputStream;
import java.io.IOException;
import java.util.*;

public class ExcelUtils {

    /**
     * Reads Excel data where:
     * Row 0     -> Dataset names
     * Column 0  -> Field names
     * Remaining cells -> Data
     */
    public static List<Map<String, String>> readDataSets(
            String filePath,
            String sheetName) throws IOException {

        List<Map<String, String>> dataSets = new ArrayList<>();

        try (FileInputStream fis = new FileInputStream(filePath);
             Workbook workbook = new XSSFWorkbook(fis)) {

            Sheet sheet = workbook.getSheet(sheetName);

            if (sheet == null) {
                throw new RuntimeException(
                        "Sheet not found : " + sheetName);
            }

            int totalRows = sheet.getPhysicalNumberOfRows();
            int totalColumns = sheet.getRow(0).getLastCellNum();

            // Iterate through each DataSet column
            for (int col = 1; col < totalColumns; col++) {

                Map<String, String> testData =
                        new LinkedHashMap<>();

                for (int row = 1; row < totalRows; row++) {

                    String fieldName =
                            getCellValue(
                                    sheet.getRow(row)
                                            .getCell(0));

                    String value =
                            getCellValue(
                                    sheet.getRow(row)
                                            .getCell(col));

                    testData.put(fieldName, value);
                }

                dataSets.add(testData);
            }
        }

        return dataSets;
    }

    private static String getCellValue(Cell cell) {

        if (cell == null)
            return "";

        switch (cell.getCellType()) {

            case STRING:
                return cell.getStringCellValue().trim();

            case NUMERIC:

                if (DateUtil.isCellDateFormatted(cell)) {
                    return cell.getDateCellValue().toString();
                }

                double numericValue =
                        cell.getNumericCellValue();

                if (numericValue == (long) numericValue) {
                    return String.valueOf(
                            (long) numericValue);
                }

                return String.valueOf(numericValue);

            case BOOLEAN:
                return String.valueOf(
                        cell.getBooleanCellValue());

            case FORMULA:
                return cell.getCellFormula();

            default:
                return "";
        }
    }
}
```

**Example usage**
---

```java
public class ExcelReaderDemo {

    public static void main(String[] args)
            throws Exception {

        List<Map<String, String>> testData =
                ExcelUtils.readDataSets(
                        "TestData.xlsx",
                        "Sheet1");

        testData.forEach(System.out::println);
    }
}
```


# TestNG DataProvider example:
---

```java
@DataProvider(name = "excelData")
public Object[][] getExcelData()
        throws Exception {

    List<Map<String, String>> data =
            ExcelUtils.readDataSets(
                    "TestData.xlsx",
                    "Sheet1");

    Object[][] testData =
            new Object[data.size()][1];

    for (int i = 0; i < data.size(); i++) {
        testData[i][0] = data.get(i);
    }

    return testData;
}
```


# Test method usage:
---

```java
@Test(dataProvider = "excelData")
public void sampleTest(
        Map<String, String> data) {

    System.out.println(
            "Field1 = " + data.get("Field1"));

    System.out.println(
            "Field2 = " + data.get("Field2"));

    System.out.println(
            "Field3 = " + data.get("Field3"));

    System.out.println(
            "Field4 = " + data.get("Field4"));
}
```

