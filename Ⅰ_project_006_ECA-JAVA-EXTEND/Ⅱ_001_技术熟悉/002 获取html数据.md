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


```java
    /**
     * xml转json
     *
     * @param element
     * @param json
     */
    public static void dom4j2Json(Element element, JSONObject json) {
        // 如果是属性
        for (Object o : element.attributes()) {
            Attribute attr = (Attribute) o;
            if (!isEmpty(attr.getValue())) {
                json.put("@" + attr.getName(), attr.getValue());
            }
        }
        List<Element> chdEl = element.elements();
        if (chdEl.isEmpty() && !isEmpty(element.getText())) {// 如果没有子元素,只有一个值
            json.put(element.getName(), element.getText());
        }

        for (Element e : chdEl) {// 有子元素
            if (!e.elements().isEmpty()) {// 子元素也有子元素
                JSONObject chdjson = new JSONObject();
                dom4j2Json(e, chdjson);
                Object o = json.get(e.getName());
                if (o != null) {
                    JSONArray jsona = null;
                    if (o instanceof JSONObject) {// 如果此元素已存在,则转为jsonArray
                        JSONObject jsono = (JSONObject) o;
                        json.remove(e.getName());
                        jsona = new JSONArray();
                        jsona.add(jsono);
                        jsona.add(chdjson);
                    }
                    if (o instanceof JSONArray) {
                        jsona = (JSONArray) o;
                        jsona.add(chdjson);
                    }
                    json.put(e.getName(), jsona);
                } else {
                    if (!chdjson.isEmpty()) {
                        json.put(e.getName(), chdjson);
                    }
                }

            } else {// 子元素没有子元素
                for (Object o : element.attributes()) {
                    Attribute attr = (Attribute) o;
                    if (!isEmpty(attr.getValue())) {
                        json.put("@" + attr.getName(), attr.getValue());
                    }
                }
                if (!e.getText().isEmpty()) {
                    json.put(e.getName(), e.getText());
                }
            }
        }
    }


```