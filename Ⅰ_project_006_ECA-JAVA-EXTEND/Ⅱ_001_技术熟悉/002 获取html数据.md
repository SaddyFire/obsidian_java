```java
public static JSONObject ConvertXMLtoJSON(String xml) {

 /*
 String xml = "<?xml version= 1.0  encoding= UTF-8  ?><xjxl reqid= d5dc5f45b7934cdea5d6099ab02f1cad ><xl checkrs= 1 ><xm>丁民坚</xm><zjhm>360424197405200577</zjhm><cc>硕士研究生</cc><yxmc>浙江工商大学</yxmc><zymc>工商管理硕士</zymc><zsbh>*</zsbh><byrq>20120613</byrq><rxrq>20100901</rxrq><xxxs>全日制</xxxs></xl></xjxl>";
 * */
	Document doc;
	try {
		doc = DocumentHelper.parseText(xml);
		JSONObject json = new JSONObject();
		dom4j2Json(doc.getRootElement(), json);
		return json;
	} catch (DocumentException e) {
	}
	return null;
}
```