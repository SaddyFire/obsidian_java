##### 1. 概述
```java
public interface DBhelper {
	Map<String, HashMap> getDataSources() throws Exception;
	/*
	 * 开启事务
	 */
	void StartTrans(Runnable var1);

	DBType GetDbType(String var1) throws Exception;
	/*
	 * 查询单条数据
	 */
	Map QueryMap(String var1, String var2, HashMap<String, Object> var3, HashMap<String, Object> var4, boolean var5) throws Exception;
	/*
	 * 查询多条数据
	 */
	DataTable QueryDataTable(String var1, String var2, HashMap<String, Object> var3, int var4, int var5, HashMap<String, Object> var6, boolean var7) throws Exception;

	DataTable QueryCursor(String var1, String var2, HashMap<String, Object> var3, HashMap<String, Object> var4) throws Exception;
	/*
	 * 返回影响函数, 增删改
	 */
	int QueryInt(String var1, String var2, HashMap<String, Object> var3, HashMap<String, Object> var4) throws Exception;

	void QueryVoid(String var1, String var2, HashMap<String, Object> var3, HashMap<String, Object> var4) throws Exception;

	List<DataColumn> GetColumns(String var1, String var2, HashMap<String, Object> var3) throws Exception;

	List<String> GetTables(String var1) throws Exception;

	DataTable GetProcParams(String var1, String var2) throws Exception;

	boolean IsSelectCommandText(String var1);

	boolean IsProcCommandText(String var1);
	/*
	 * 插入表
	 */
	Integer GetPkValue(String var1, String var2, String var3) throws Exception;
}
```




## 2. QueryDataTable
##### 01 QueryDataTable
```java
//七个参数: datasourceguid(数据库guid), sqlKey(sqlKey), params(执行参数), startIndex(起始索引), pageSize(页大小), vars(未知), fromSlave(未知)
public DataTable QueryDataTable(String datasourceguid, String sqlKey, HashMap<String, Object> params, int startIndex, int pageSize, HashMap<String, Object> vars, boolean fromSlave) throws Exception {
	//此处调用 QueryCallBack 回调方法, 将参数全部放入 , 同时调用 SqlCallback() 匿名内部类 操作
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
//此处
public Object QueryCallBack(String datasourceguid, String sqlKey, HashMap<String, Object> params, int startIndex, int pageSize, HashMap<String, Object> vars, SqlCallback callback, boolean fromSlave) throws Exception {
	Pager pager = null;
	//此处将
	if (pageSize != 0) {
		//计算当前页
		int pageIndex = startIndex / pageSize + 1;
		//
		pager = this.getDao(datasourceguid, fromSlave).createPager(pageIndex, pageSize);
	}
	//调用Execute(), 将datasourceguid, sqlKey, params, 
	Sql sql = this.Execute(datasourceguid, sqlKey, params, callback, pager, vars);
	return sql.getResult();
}
```

