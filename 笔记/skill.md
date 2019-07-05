### 常用基础技术点

#### 1.导出excel

用easypoi做导出<http://easypoi.mydoc.io/>

```java
1.导出对应实体类
	/**
     * 资源上架状态：0 没上架 1 上架
     */
    @Excel(name="资源上架状态", width = 20, replace={ "没上_0", "上_1" }, suffix = "架", isImportField = "true_st")
    private Integer isShow;

    /**
     * 方案类型
     */
    @Excel(name="方案类型", replace={ "视频_1", "ppt_2","图片_3", "解决方案_4"}, isImportField = "true_st")
    private Integer resourceType;



2.解决导出文件名乱码问题
String fileName="方案Top"+LocalDate.now();
			//设置返回响应头
			response.setContentType("application/xls;charset=UTF-8");
			response.setHeader("Content-Disposition", "attachment;filename=\""+ new String(fileName.getBytes("gbk"), "iso8859-1") + ".xls"+"\"");
			OutputStream out = response.getOutputStream();
			workbook.write(out);
			out.flush();
			out.close();


```

#### 2.BigDecimal金额计算

```java
1.BigDecimal比较相等，不能用equals，要用compareTo
bigDecimal.compareTo(BigDecimal.ZERO) != 0
2.金额的基本运算
如果使用new BigDecimal()时，用new BigDecimal("100")，或者直接使用BigDecimal.valueof(double)

public class BigDecimalUtil {

    private BigDecimalUtil() {

    }
	//加法
    public static BigDecimal add(double v1, double v2) {// v1 + v2
        BigDecimal b1 = new BigDecimal(Double.toString(v1));
        BigDecimal b2 = new BigDecimal(Double.toString(v2));
        return b1.add(b2);
    }
	//减法
    public static BigDecimal sub(double v1, double v2) {
        BigDecimal b1 = new BigDecimal(Double.toString(v1));
        BigDecimal b2 = new BigDecimal(Double.toString(v2));
        return b1.subtract(b2);
    }
	//乘法
    public static BigDecimal mul(double v1, double v2) {
        BigDecimal b1 = new BigDecimal(Double.toString(v1));
        BigDecimal b2 = new BigDecimal(Double.toString(v2));
        return b1.multiply(b2);
    }
	//除法
    public static BigDecimal div(double v1, double v2) {
        BigDecimal b1 = new BigDecimal(Double.toString(v1));
        BigDecimal b2 = new BigDecimal(Double.toString(v2));
        // 2 = 保留小数点后两位   ROUND_HALF_UP = 四舍五入
        return b1.divide(b2, 2, BigDecimal.ROUND_HALF_UP);// 应对除不尽的情况
    }
}


```

