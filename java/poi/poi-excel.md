# poi 解决 excel-2003(xlsx)与2007(xls)不兼容的问题<small>3.5</small>

- 代码片段
```java
@Override
public List<MagiCADCollectingListDTO> process(ResolveRequest request) {
    logger.info("1.整理清单");
    try (InputStream is = new ByteArrayInputStream(request.getFileBytes())) {
        Workbook workbook = null;
        if (POIFSFileSystem.hasPOIFSHeader(is)) { // xls
            workbook = new HSSFWorkbook(is);
        } else if (POIXMLDocument.hasOOXMLHeader(is)) { // xlsx
            workbook = new XSSFWorkbook(is);
        } else {
            throw new IllegalArgumentException("文件格式不正确");
        }
        Sheet sheet = workbook.getSheetAt(0);
        return parseSheet(sheet);
    } catch (IOException e) {
        logger.severe("整理清单异常：" + e.getMessage());
    }
    return null;
}
  
private static List<MagiCADCollectingListDTO> parseSheet(Sheet sheet) {
    // 获取第一行数据
    int firstRowNum = sheet.getFirstRowNum();
    // 解析每一行的数据，构造数据对象
    int rowStart = firstRowNum + 1;
    int rowEnd = sheet.getPhysicalNumberOfRows();
    // 返回结果
    List<MagiCADCollectingListDTO> dtos = new ArrayList<>(rowEnd);
    for (int rowNum = rowStart; rowNum < rowEnd; rowNum++) {
        Row row = sheet.getRow(rowNum);
        if (null == row) {
            continue;
        }
        MagiCADCollectingListDTO dto = convertRowToData(row);
        if (null == dto) {
            continue;
        }
        dtos.add(dto);
    }
    return dtos;
}
 
private static MagiCADCollectingListDTO convertRowToData(Row row) {
    // 过滤掉 code 中不是数字的
    String code = row.getCell(2).getStringCellValue();
    if (!code.matches("^[0-9]*$")) {
        logger.warning("code: [" + code + "] 为非数字，过滤掉");
        return null;
    }
    MagiCADCollectingListDTO dto = new MagiCADCollectingListDTO();
    dto.setNo(row.getCell(0).getStringCellValue());
    dto.setCode(code);
    dto.setType(row.getCell(3).getStringCellValue());
    dto.setName(row.getCell(4).getStringCellValue());
    dto.setProjectFeature(row.getCell(5).getStringCellValue());
    dto.setExpression(row.getCell(6).getStringCellValue());
    dto.setUnit(row.getCell(7).getStringCellValue());
    dto.setProjectQuantity(new BigDecimal(row.getCell(8).getNumericCellValue()));
    dto.setRemark(row.getCell(9).getStringCellValue());
    return dto;
}
```

* 说明

> `POIFSFileSystem.hasPOIFSHeader(is)` 校验是否为 xls
> `POIXMLDocument.hasOOXMLHeader(is)` 校验是否为 xlsx