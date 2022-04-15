##### 1. 接口概述
```java
public interface DBhelper {
	Map<String, HashMap> getDataSources() throws Exception;
	/*
	 * 开启事务
	 */
	void StartTrans(Runnable var1);
	//获取数据库类型
	DBType GetDbType(String var1) throws Exception;
	/*
	 * 查询单条数据
	 * param(String datasourceguid, String sqlKey, HashMap<String, Object> params(占位符参数), HashMap<String, Object> vars(修改占位符相关参数), boolean fromSlave(未知))
	 * callback采用内置回调参数, pager为null, 
	 * 并且将查询结果直接 提取 以map形式返回
	 */
	Map QueryMap(String var1, String var2, HashMap<String, Object> var3, HashMap<String, Object> var4, boolean var5) throws Exception;
	/*
	 * 万能sql接口
	 */
	DataTable QueryDataTable(String var1, String var2, HashMap<String, Object> var3, int var4, int var5, HashMap<String, Object> var6, boolean var7) throws Exception;
	
	DataTable QueryCursor(String var1, String var2, HashMap<String, Object> var3, HashMap<String, Object> var4) throws Exception;
	/*
	 * 执行sql, 如果当前语句为 DELETE | UPDATE | INSERT，返回执行后所影响的记录数。否则返回 -1
	 */
	int QueryInt(String var1, String var2, HashMap<String, Object> var3, HashMap<String, Object> var4) throws Exception;
	/*
	 * 执行sql不返回结果
	 */
	void QueryVoid(String var1, String var2, HashMap<String, Object> var3, HashMap<String, Object> var4) throws Exception;
	/**
	 * 批量数据查询, 将结果返回
	 */
	List<DataColumn> GetColumns(String var1, String var2, HashMap<String, Object> var3) throws Exception;
	/**
	 * 查询表列表(String datasourceguid)
	 */
	List<String> GetTables(String var1) throws Exception;
	/**
	 * 获取表参数相关(String datasourceguid, String procName)
	 */
	DataTable GetProcParams(String var1, String var2) throws Exception;
	/**
	 * 是否为select语句
	 */
	boolean IsSelectCommandText(String var1);
	/**
	 * 是否为call语句
	 */
	boolean IsProcCommandText(String var1);
	/*
	 * 返回一个可用的ID
	 */
	Integer GetPkValue(String var1, String var2, String var3) throws Exception;
}
```




## 2. QueryDataTable详解
QueryDataTable() -> QueryCallBack() -> Execute()
##### 01 QueryDataTable
```java
//七个参数: datasourceguid(数据库guid), sqlKey(sqlKey), params(执行参数), startIndex(起始索引), pageSize(页大小), vars(修改占位符相关参数), fromSlave(未知)
public DataTable QueryDataTable(String datasourceguid, String sqlKey, HashMap<String, Object> params, int startIndex, int pageSize, HashMap<String, Object> vars, boolean fromSlave) throws Exception {
	//此处调用 QueryCallBack 方法, 将参数全部放入 , 执行sql语句
	//new SqlCallback() 说明: 此处自定义sql回调类, 将sql语句执行后的结果进行
	return (DataTable)this.QueryCallBack(datasourceguid, sqlKey, params, startIndex, pageSize, vars, new SqlCallback() {
		public Object invoke(Connection conn, ResultSet rs, Sql sql) throws SQLException {
			if (rs == null) {
				return null;
			} else {
				ResultSetMetaData md = rs.getMetaData();
				int columnCount = md.getColumnCount();
				DataTable dt = DBhelperImpl.this.ResultSet2DataTable(rs);
				HashMap<String, DataColumn> dcs = new HashMap();

				for(int i = 1; i <= columnCount; ++i) {
					DataColumn dc = new DataColumn();
					String colName = md.getColumnLabel(i).toUpperCase();
					dc.setColumnName(colName);
					int jdbcType = md.getColumnType(i);
					dc.setJdbcType(jdbcType);
					Class<?> javaType = DBhelperImpl.this.jdbcType2JavaType(jdbcType);
					dc.setDataType(javaType);
					dcs.put(colName, dc);
				}

				dt.setColumns(dcs);
				return dt;
			}
		}
	}, fromSlave);
}

```

##### 02 QueryCallBack

```java
//在此处执行了sql, 并将结果返回
public Object QueryCallBack(String datasourceguid, String sqlKey, HashMap<String, Object> params, int startIndex, int pageSize, HashMap<String, Object> vars, SqlCallback callback, boolean fromSlave) throws Exception {
	Pager pager = null;
	if (pageSize != 0) {
		//计算当前页
		int pageIndex = startIndex / pageSize + 1;
		pager = this.getDao(datasourceguid, fromSlave).createPager(pageIndex, pageSize);
	}
	//调用Execute(), 将datasourceguid, sqlKey, params, 
	Sql sql = this.Execute(datasourceguid, sqlKey, params, callback, pager, vars);
	//将sql结果返回
	return sql.getResult();
}
```

##### 03 Execute
通过`sqlObject.setCallback(callback);` 设置回调
通过 `dao.execute(sqlObject);` 执行sql语句


```java
//datasourceguid, sqlKey, params, callback(回调函数), pager(封装有关page的对象), vars(修改占位符相关参数), fromSlave(未知)
private Sql Execute(String datasourceguid, String sqlKey, HashMap<String, Object> params, SqlCallback callback, Pager pager, HashMap<String, Object> vars, boolean fromSlave) throws Exception {
	if (!this.ecaSqlsConfig.getMap().containsKey(sqlKey)) {
		throw new Exception("未在ecasqls中配置对应的sql键！键值：" + sqlKey);
	} else {
		String strSql = this.ecaSqlsConfig.getMap().get(sqlKey).toString();
		Dao dao = this.getDao(datasourceguid, fromSlave);
		String place;
		if (vars != null) {
			//编译
			Pattern pattern = Pattern.compile("\\$(\\w+)");
			//匹配sql语句
			Matcher matcher = pattern.matcher(strSql);

			while(matcher.find()) {
				String place = matcher.group(0);
				place = matcher.group(1);
				if (vars.keySet().contains(place)) {
					//替换占位符
					strSql = strSql.replace(place, vars.get(place).toString());
				}
			}
		}

		char placeHolder = this.getPlaceHolder(datasourceguid);
		if (params != null) {
			Pattern pattern = Pattern.compile("[@?:](\\w+)");
			Matcher matcher = pattern.matcher(strSql);

			while(matcher.find()) {
				place = matcher.group(0);
				String key = matcher.group(1);
				if (place.charAt(0) != placeHolder) {
					strSql = strSql.replace(place, placeHolder + key);
				}
			}
		}

		Sql sqlObject = Sqls.create(strSql);
		sqlObject.changePlaceholder(placeHolder, '$');
		if (callback != null) {
			sqlObject.setCallback(callback);
		}

		if (params != null) {
			params.forEach((k, v) -> {
				SqlType sqlType = sqlObject.getSqlType();
				if (k.startsWith("OUT") && (sqlType == SqlType.CALL || sqlType == SqlType.EXEC)) {
					if (k.equalsIgnoreCase("OUTC1")) {
						sqlObject.params().set(k, 2012);
					} else {
						sqlObject.params().set(k, 12);
					}
				} else if (v instanceof byte[]) {
					InputStream input = new ByteArrayInputStream((byte[])((byte[])v));
					sqlObject.params().set(k, input);
				} else {
					sqlObject.params().set(k, v);
				}

			});
		}

		if (pager != null) {
			sqlObject.setPager(pager);
		}
		//执行sql语句
		dao.execute(sqlObject);
		this.setOutParams(params, sqlObject);
		return sqlObject;
	}
}
```


## 3. GetPkValue
##### 01 实现类
```java
// 三个参数 (String datasourceguid, String tableName, String fieldName) (数据库guid, 表明, 字段名)
public Integer GetPkValue(String datasourceguid, String tableName, String fieldName) throws Exception {
	HashMap<String, Object> param = new HashMap();
	param.put("tablename", tableName);
	param.put("fieldname", fieldName);
	param.put("OUTpkvalue", "0");
	param.put("ds_id", "1");
	//执行 QueryVoid 
	this.QueryVoid(datasourceguid, "getpkvalue", param);
	// sql: 
	return Convert.parseInt(param.get("OUTpkvalue"));
}
```
##### 02 getpkvalue SQL语句
```yml
#获取最大主键值的存储过程
getpkvalue: 'call GetPKValue (?tablename,?fieldname,?OUTpkvalue,?ds_id)',
```
