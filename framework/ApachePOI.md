## Apache POI example to read Excel data as map of map

Each column (DataSet1, DataSet2, DataSet3, DataSet4) represents one test data set, while the first column contains the field name

```text
+--------+----------+----------+----------+----------+
| Field  | DataSet1 | DataSet2 | DataSet3 | DataSet4 |
+--------+----------+----------+----------+----------+
| Field1 |    1     |    a     |    w     |    p     |
| Field2 |    2     |    b     |    x     |    q     |
| Field3 |    3     |    c     |    y     |    r     |
| Field4 |    4     |    d     |    z     |    s     |
+--------+----------+----------+----------+----------+
```


Output expected:
```text
[
 {Field1=1, Field2=2, Field3=3, Field4=4},
 {Field1=a, Field2=b, Field3=c, Field4=d},
 {Field1=w, Field2=x, Field3=y, Field4=z},
 {Field1=p, Field2=q, Field3=r, Field4=s}
]
```

# Apache POI Example:
---

```java
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;

import java.io.FileInputStream;
import java.util.*;

public class ExcelReader {

    public static List<Map<String, String>> readTestData(String filePath,
                                                         String sheetName)
            throws Exception {

        List<Map<String, String>> allData = new ArrayList<>();

        FileInputStream fis = new FileInputStream(filePath);
        Workbook workbook = new XSSFWorkbook(fis);

        Sheet sheet = workbook.getSheet(sheetName);

        int totalRows = sheet.getPhysicalNumberOfRows();
        int totalCols = sheet.getRow(0).getLastCellNum();

        // Iterate through DataSet columns
        for (int col = 1; col < totalCols; col++) {

            Map<String, String> dataSet = new LinkedHashMap<>();

            for (int row = 1; row < totalRows; row++) {

                String fieldName =
                        sheet.getRow(row)
                             .getCell(0)
                             .getStringCellValue();

                Cell valueCell =
                        sheet.getRow(row)
                             .getCell(col);

                String value = getCellValue(valueCell);

                dataSet.put(fieldName, value);
            }

            allData.add(dataSet);
        }

        workbook.close();
        fis.close();

        return allData;
    }

    private static String getCellValue(Cell cell) {

        if (cell == null)
            return "";

        return switch (cell.getCellType()) {

            case STRING ->
                    cell.getStringCellValue();

            case NUMERIC ->
                    String.valueOf((long) cell.getNumericCellValue());

            case BOOLEAN ->
                    String.valueOf(cell.getBooleanCellValue());

            default ->
                    "";
        };
    }

    public static void main(String[] args)
            throws Exception {

        List<Map<String, String>> data =
                readTestData(
                        "TestData.xlsx",
                        "Sheet1");

        data.forEach(System.out::println);
    }
}
```

**Output**

```text
{Field1=1, Field2=2, Field3=3, Field4=4}

{Field1=a, Field2=b, Field3=c, Field4=d}

{Field1=w, Field2=x, Field3=y, Field4=z}

{Field1=p, Field2=q, Field3=r, Field4=s}
```


#  Usage with TestNG DataProvider:
---

```java
@DataProvider(name = "excelData")
public Object[][] getData() throws Exception {

    List<Map<String, String>> testData =
            ExcelReader.readTestData(
                    "TestData.xlsx",
                    "Sheet1");

    Object[][] data =
            new Object[testData.size()][1];

    for (int i = 0; i < testData.size(); i++) {

        data[i][0] = testData.get(i);
    }

    return data;
}

```


# Test usage:
---

```java
@Test(dataProvider = "excelData")
public void loginTest(Map<String, String> data) {

    System.out.println("Field1 = "
            + data.get("Field1"));

    System.out.println("Field2 = "
            + data.get("Field2"));
}

```


# Utility to read data using above code:
---

```java
Map<String, String> data = ExcelUtil.getData("LoginData", "DataSet2");

String username = data.get("Username");

String password = data.get("Password");
```


output:
```text
{
 Username=user1,
 Password=pass123,
 Environment=UAT,
 Browser=Edge
}
```

