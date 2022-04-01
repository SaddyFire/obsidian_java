```java
	/**  
	 ** @param url  访问路径
	 * @param headers  请求头
	 * @param params  请求参数
	 * @return  
	 */
    public String postform(String url, HashMap<String, String> headers, HashMap<String, String> params) {
        try {
            URL url = new URL(url);
            HttpURLConnection connection = (HttpURLConnection) url.openConnection();
            connection.setDoInput(true);
            connection.setDoOutput(true);
            connection.setRequestMethod("POST");
            connection.setRequestProperty("Content-Type", "application/x-www-form-urlencoded; charset=UTF-8");
            if (headers != null)
                for (Map.Entry<String, String> item : headers.entrySet()) {
                    connection.setRequestProperty("strToken", headers.get("strToken"));
                }
            String param = "";
            if (params != null)
                for (Map.Entry<String, String> item : params.entrySet()) {
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
            logger.error("请求失败!详细错误:" + e.getMessage());
            e.printStackTrace();
            return "";
        }
    }
```