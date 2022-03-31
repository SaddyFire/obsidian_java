## 1. 增操作
##### 01 QueryDataTable
```java
//七个参数: datasourceguid(数据库guid), sqlKey(sqlKey), params(执行参数), startIndex(查询起点索引), pageSize(查询数据), vars(), fromSlave()
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
	if (pageSize != 0) {
		int pageIndex = startIndex / pageSize + 1;
		pager = this.getDao(datasourceguid, fromSlave).createPager(pageIndex, pageSize);
	}

	Sql sql = this.Execute(datasourceguid, sqlKey, params, callback, pager, vars);
	return sql.getResult();
}
```