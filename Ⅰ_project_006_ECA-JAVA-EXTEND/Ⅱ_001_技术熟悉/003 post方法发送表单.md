##### 01 工具包


```java
/**
     * Get方法
     */
    public String get(String uri, HashMap<String, String> headers) {
        try {
            URL url = new URL(uri);
            HttpURLConnection connection = (HttpURLConnection) url.openConnection();
            connection.setDoOutput(true);
            connection.setRequestMethod("GET");
            if (headers != null)
                for (Map.Entry<String, String> item : headers.entrySet()) {
                    connection.setRequestProperty(item.getKey(), item.getValue());
                }
            BufferedReader br = new BufferedReader(
                    new InputStreamReader(connection.getInputStream(), StandardCharsets.UTF_8));
            String line;
            StringBuilder result = new StringBuilder();
            while ((line = br.readLine()) != null) {
                result.append(line).append("\n");
            }
            connection.disconnect();
            return result.toString();
        } catch (Exception e) {
            e.printStackTrace();
            return "";
        }
    }

    /**
     ** @param uri  访问路径
     * @param headers  请求头
     * @param params  请求参数
     * @return
     */
    public String postform(String uri, HashMap<String, String> headers, HashMap<String, String> params) {
        try {
            URL url = new URL(uri);
            HttpURLConnection connection = (HttpURLConnection) url.openConnection();
            connection.setDoInput(true);
            connection.setDoOutput(true);
            connection.setRequestMethod("POST");
            connection.setRequestProperty("Content-Type", "application/x-www-form-urlencoded; charset=UTF-8");
            if (headers != null)
                for (Map.Entry<String, String> item : headers.entrySet()) {
                    connection.setRequestProperty(item.getKey(), item.getValue());
                }
            String param = "";
            if (params != null)
                for (Map.Entry<String, String> item : headers.entrySet()) {
                    param += "&" + item.getKey() + "=" + item.getValue();
                }
            param = param.substring(1);
            PrintWriter pw = new PrintWriter(new BufferedOutputStream(connection.getOutputStream()));
            pw.write(param);
            pw.flush();
            pw.close();
            BufferedReader br = new BufferedReader(
                    new InputStreamReader(connection.getInputStream(), StandardCharsets.UTF_8));
            String line;
            StringBuilder result = new StringBuilder();
            while ((line = br.readLine()) != null) {
                result.append(line).append("\n");
            }
            connection.disconnect();
            return result.toString();
        } catch (Exception e) {
            e.printStackTrace();
            return "";
        }
    }

    /**
     * Post方法发送json数据
     */
    public String postjson(String uri, HashMap<String, String> headers, JSONObject data) {
        try {
            URL url = new URL(uri);
            HttpURLConnection connection = (HttpURLConnection) url.openConnection();
            connection.setDoInput(true);
            connection.setDoOutput(true);
            connection.setRequestMethod("POST");
            connection.setRequestProperty("Content-Type", "application/json;charset=UTF-8");
            if (headers != null)
                for (Map.Entry<String, String> item : headers.entrySet()) {
                    connection.setRequestProperty(item.getKey(), item.getValue());
                }
            PrintWriter pw = new PrintWriter(new BufferedOutputStream(connection.getOutputStream()));
            pw.write(JSON.toJSONString(data));
            pw.flush();
            pw.close();
            BufferedReader br = new BufferedReader(
                    new InputStreamReader(connection.getInputStream(), StandardCharsets.UTF_8));
            String line;
            StringBuilder result = new StringBuilder();
            while ((line = br.readLine()) != null) {
                result.append(line).append("\n");
            }
            connection.disconnect();
            return result.toString();
        } catch (Exception e) {
            e.printStackTrace();
            return "";
        }
    }
```



##### 02 调用该接口访问学信网
核心参数: 
姓名, 身份证
```java
String url = "http://223.4.76.84:8080/ECA_JAVA_WebApi/api/ProvinceInterface/GetEducationInfo";
//请求参数
HashMap param = new HashMap();
param.put("strPersonName", PersonName);
param.put("strIDCard", IdCard);
//请求头参数
String strUserName = "D29F16A673C689B3D5EB1FC004FC8D07";//用户名..
String strPwd = "CCIR.com.cn";//密码
int time = (int) (System.currentTimeMillis() / 1000);
String strPersonName = PersonName;//用户姓名
String strIDCard = IdCard;//信用代码
String strToken = "strUserName=" + strUserName + "&strTimeStamp=" + time + "&strUserPwd=" + strPwd + "&strIDCard=" + strIDCard + "&strPersonName=" + strPersonName;
//请求头字符串AES加密
String AESToken = OutProDockingAesUtil.encrypt(strToken);
AESToken = AESToken.replaceAll("\r|\n", "");//替换掉字符串中所有的换行和空格
HashMap headParam = new HashMap();
headParam.put("strToken", AESToken);
String s1 = postform(url, headParam, param);    //调用接口: Post方法发送form表单
```