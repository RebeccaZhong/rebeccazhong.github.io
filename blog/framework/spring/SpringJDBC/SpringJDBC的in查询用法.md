<!-- MarkdownTOC -->
- [JdbcTemplate的query方法](#JdbcTemplate的query方法)
- [NamedParameterJdbcTemplate的queryForList方法](#NamedParameterJdbcTemplate的queryForList方法)
<!-- /MarkdownTOC -->


### JdbcTemplate的query方法
JdbcTemplate的query中sql是用`?`占位符的方式进行传参的。
方法定义如下：
```java
<T> List<T> query(String sql, Object[] args, int[] argTypes, RowMapper<T> rowMapper)
```
这个方法本身不支持in查询，我们在调用前，可以自己封装一层方法，变相的实现in查询。
```java
public <T> List<T> querySupportIn(String sql, Object[] args, RowMapper<T> rowMapper) throws DataAccessException {
	Object[] obj = this.changeMessage(sql, args);
	return this.getJdbcTemplate().query((String)obj[0], (Object[])obj[1], (int[])obj[2], rowMapper);
}
private Object[] changeMessage(String sql, Object[] args) throws DataAccessException {
	List<Object> params = new ArrayList<Object>();
	List<Integer> types = new ArrayList<Integer>();
	List<Object[]> paramList = new ArrayList<Object[]>();
	for(int i=0;i<args.length;i++){
		Object arg = args[i];
		if(arg.getClass().isArray()){
			Object[] array = (Object[]) arg;
			if(array.length == 0){
				throw new RuntimeException("Array param can not be empty!");
			}
			Integer type = null;
			for(Object obj : array){
				if(type == null){
					type = this.getTypes(obj);
				}
				params.add(obj);
				types.add(type);
			}
			if(array.length > 1){
				paramList.add(new Object[]{i, getParamSymbol(array.length)});
			}
		}else if(Collection.class.isAssignableFrom(arg.getClass()) ){
			Collection<?> list = (Collection<?>) arg;
			if(list.size() == 0){
				throw new RuntimeException("Collection param can not be empty!");
			}
			Integer type = null;
			for(Object obj : list){
				if(type == null){
					type = this.getTypes(obj);
				}
				params.add(obj);
				types.add(type);
			}
			if(list.size() > 1){
				paramList.add(new Object[]{i, getParamSymbol(list.size())});
			}
		}else{
			params.add(arg);
			types.add(this.getTypes(arg));
		}
	}
	//根据序号替换单个?为多个数组?
	if(!paramList.isEmpty()){
		StringBuffer sqlb = new StringBuffer();
		String sqls = sql.toString();
		int count = -1;
		int index = -1;
		for(Object[] obj : paramList){
			int paramIndex = (Integer)obj[0];
			while(true){
				count++;
				index = sqls.indexOf("?", index + 1);
				if(index == -1 ){
					throw new RuntimeException("Paramater size > '?' size!");
				}
				if(paramIndex == count){
					sqlb.append(sqls.substring(0,index))
						.append((String)obj[1]);
					sqls = sqls.substring(index+1);
					index = -1;
					break;
				}
			}
		}
		sqlb.append(sqls);
		sql = sqlb.toString();
	}

	int[] tys = new int[types.size()];
	for(int i=0;i<types.size();i++){
		tys[i] = types.get(i);
	}
	if(logger.isInfoEnabled()){
		logger.info("sql  : "+sql.toString());
		logger.info("param: "+params);
		logger.info("types: "+types);
	}
	return new Object[]{sql , params.toArray(), tys};
}
```
changeMessage方法的主要作用就是获取Object[] args参数中所有的个数（包括数组套数组的个数），根据args的个数，把传入的String sql参数中的所有`?`占位符重新替换一遍。

使用如下方式调用
```java
String sql = "SELECT * FROM t1 WHERE type = ? AND id IN (?)";
String[] ids = new String[]{"a", "b"};
Object[] params = new Object[]{sql, ids};
List<Test> result = this.querySupportIn(sql, params, new BeanPropertyRowMapper<>(Test.class));
```
在querySupportIn内部调用的changeMessage方法中，处理后的sql就变成`SELECT * FROM t1 WHERE type = ? AND id IN (?,?)`了，这样就变相实现了in查询。

### NamedParameterJdbcTemplate的queryForList方法
NamedParameterJdbcTemplate的queryForList方法中sql是用*具名参数*`SQL按名称(以冒号开头)`的方式进行传参的。
方法定义如下：
```java
<T> List<T> queryForList(String sql, SqlParameterSource paramSource, Class<T> elementType) throws DataAccessException
```
使用如下方式调用
```java
String sql = "SELECT * FROM t1 WHERE type = :type AND id IN (:ids)";
Map<String, Object> paramsMap = new HashMap<>();
paramsMap.put("type", 1);
String[] ids = new String[]{"a", "b"};
paramsMap.put("ids", ids);
List<Test> result = namedParameterJdbcTemplate.query(sql, paramsMap, new BeanPropertyRowMapper<>(Test.class));
```
NamedParameterJdbcTemplate的query方法，本身就支持in查询，此处不用再做封装处理。

## 总结
NamedParameterJdbcTemplate的使用好处：如果参数比较多，并且参数位置或顺序可能变化的情况下，使用NamedParameterJdbcTemplate是非常方便的！

**参考资料**
[详解jdbcTemplate和namedParameterJdbcTemplate](https://www.jianshu.com/p/1bdc0e26a7e4)
