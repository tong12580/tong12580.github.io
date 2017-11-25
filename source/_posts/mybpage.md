title: MyBatis分页的拓展--合并高级查询
date: 2016/03/08 21:29:00
tags:
    - page
    - java
    - myBatis
categories:
    - myBatis
---

***MyBatis分页的拓展--合并查询***
在网上有很多关于MyBatis拦截器分页的办法，可缺少关于合并查询的方法。本文将讲述这一过程的具体实现。
话不多说，直接贴代码：

applicationContext.xml里这样配置：
```
	<!-- MyBatis配置 -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource" />
		<!-- mybatis配置文件路径 -->
        <property name="configLocation" value="WEB-INF/conf/myBatisConfig.xml"/>    
    </bean> 
    <bean id="sqlSessionTemplate" class="org.mybatis.spring.SqlSessionTemplate">
       <constructor-arg index="0" ref="sqlSessionFactory" />
       <!-- 这个执行器会批量执行更新语句, 还有SIMPLE 和 REUSE -->
       <constructor-arg index="1" value="BATCH" />
    </bean>
    
 	 <!-- 扫描basePackage接口 -->
   <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    	<property name="annotationClass" value="org.springframework.stereotype.Repository" />
    	<!-- 映射器接口文件的包路径， -->
       <property name="basePackage" value="com.stock.dao" />
   </bean>
```
<br>
接下来是Page分页部分：

```
public class Page<T> {
	
	private int pageNo = 1;//页码，默认是第一页
    private int pageSize = 15;//每页显示的记录数，默认是15
    private int totalRecord;//总记录数
    private int totalPage;//总页数
    private List<?> results;//对应的当前页记录
    private Map<String, Object> params = new HashMap<String, Object>();//其他的参数我们把它分装成一个Map对象
    
    /** 页码*/
    public int getPageNo() {
       return pageNo;
    }
    
    public void setPageNo(int pageNo) {
       this.pageNo = pageNo;
    }
    
    /** 每页记录数*/
    public int getPageSize() {
       return pageSize;
    }
 
    public void setPageSize(int pageSize) {
       this.pageSize = pageSize;
    }
    
    /** 总页数*/
    public int getTotalRecord() {
       return totalRecord;
    }
 
    public void setTotalRecord(int totalRecord) {
    	
       this.totalRecord = totalRecord;
       //在设置总页数的时候计算出对应的总页数，在下面的三目运算中加法拥有更高的优先级，所以最后可以不加括号。
       int totalPage = totalRecord%pageSize==0 ? totalRecord/pageSize : totalRecord/pageSize + 1;
       this.setTotalPage(totalPage);
       
       if(0!=totalRecord)
    	   params.clear();
    }
 
    public int getTotalPage() {
       return totalPage;
    }
 
    public void setTotalPage(int totalPage) {
       this.totalPage = totalPage;
    }
 
    public List<?> getResults() {
       return results;
    }
 
    public void setResults(List<?> results) {
       this.results = results;
    }
   
    public Map<String, Object> getParams() {
       return params;
    }
   
    public void setParams(Map<String, Object> params) {
       this.params = params;
    }
 
    @Override
    public String toString() {
       StringBuilder builder = new StringBuilder();
       builder.append("Page [pageNo=").append(pageNo).append(", pageSize=")
              .append(pageSize).append(", results=").append(results).append(
                     ", totalPage=").append(totalPage).append(
                     ", totalRecord=").append(totalRecord).append("]");
       return builder.toString();
    }
}
```
接着，是拦截器

```
@Intercepts( {
    @Signature(method = "prepare", type = StatementHandler.class, args = {Connection.class}) })
public class PageInterceptor implements Interceptor {
	
	private static Logger log = Logger.getLogger(PageInterceptor.class);
    private String databaseType;//数据库类型，不同的数据库有不同的分页方法
    
    /**
     * 拦截后要执行的方法
     */
    public Object intercept(Invocation invocation) throws Throwable {
      
       RoutingStatementHandler handler = (RoutingStatementHandler) invocation.getTarget();
       //通过反射获取到当前RoutingStatementHandler对象的delegate属性
       StatementHandler delegate = (StatementHandler)ReflectUtil.getFieldValue(handler, "delegate");
  
      
       BoundSql boundSql = delegate.getBoundSql();

       Object obj = boundSql.getParameterObject();

       if (obj instanceof Page<?>) {
           Page<?> page = (Page<?>) obj;
        
           MappedStatement mappedStatement = (MappedStatement)ReflectUtil.getFieldValue(delegate, "mappedStatement");
           
           Connection connection = (Connection)invocation.getArgs()[0];
       
           String sql = boundSql.getSql();
          
           this.setTotalRecord(page,mappedStatement, connection);
          
           String pageSql = this.getPageSql(page, sql);

           ReflectUtil.setFieldValue(boundSql, "sql", pageSql);
       }
       return invocation.proceed();
    }
 
 
    /**
     * 拦截器对应的封装原始对象的方法
     */
    public Object plugin(Object target) {
       return Plugin.wrap(target, this);
    }
 
    /**
     * 设置注册拦截器时设定的属性
     */
    public void setProperties(Properties properties) {
       this.databaseType = properties.getProperty("databaseType");
    }
   
    /**
     * 根据page对象获取对应的分页查询Sql语句，这里只做了两种数据库类型，Mysql和Oracle
     * 其它的数据库都 没有进行分页
     *
     * @param page 分页对象
     * @param sql 原sql语句
     * @return
     */
    private String getPageSql(Page<?> page, String sql) {
       StringBuffer sqlBuffer = new StringBuffer(sql);
       if ("mysql".equalsIgnoreCase(databaseType)) {
           return getMysqlPageSql(page, sqlBuffer);
       } else if ("oracle".equalsIgnoreCase(databaseType)) {
           return getOraclePageSql(page, sqlBuffer);
       }
       return sqlBuffer.toString();
    }
   
    /**
     * 获取Mysql数据库的分页查询语句
     * @param page 分页对象
     * @param sqlBuffer 包含原sql语句的StringBuffer对象
     * @return Mysql数据库分页语句
     */
    private String getMysqlPageSql(Page<?> page, StringBuffer sqlBuffer) {
    	
       sqlBuffer =new StringBuffer(getParamSql(page, sqlBuffer));
    	//计算第一条记录的位置，Mysql中记录的位置是从0开始的。
       int offset = (page.getPageNo() - 1) * page.getPageSize();
       sqlBuffer.append(" limit ").append(offset).append(",").append(page.getPageSize());
       return sqlBuffer.toString();
    }
   
    /**
     * 获取Oracle数据库的分页查询语句
     * @param page 分页对象
     * @param sqlBuffer 包含原sql语句的StringBuffer对象
     * @return Oracle数据库的分页查询语句
     */
    private String getOraclePageSql(Page<?> page, StringBuffer sqlBuffer) {
       //计算第一条记录的位置，Oracle分页是通过rownum进行的，而rownum是从1开始的
       int offset = (page.getPageNo() - 1) * page.getPageSize() + 1;
       sqlBuffer.insert(0, "select u.*, rownum r from (").append(") u where rownum < ").append(offset + page.getPageSize());
       sqlBuffer.insert(0, "select * from (").append(") where r >= ").append(offset);
       //上面的Sql语句拼接之后大概是这个样子：
       //select * from (select u.*, rownum r from (select * from t_user) u where rownum < 31) where r >= 16
       return sqlBuffer.toString();
    }
   
    /**
     * 给当前的参数对象page设置总记录数
     *
     * @param page Mapper映射语句对应的参数对象
     * @param mappedStatement Mapper映射语句
     * @param connection 当前的数据库连接
     */
    private void setTotalRecord(Page<?> page,
           MappedStatement mappedStatement, Connection connection) {

       BoundSql boundSql = mappedStatement.getBoundSql(page);
      
       String sql = boundSql.getSql();
     
       String countSql = this.getCountSql(page,sql);
      
       List<ParameterMapping> parameterMappings = boundSql.getParameterMappings();
    
       BoundSql countBoundSql = new BoundSql(mappedStatement.getConfiguration(), countSql, parameterMappings, page);
      
       ParameterHandler parameterHandler = new DefaultParameterHandler(mappedStatement, page, countBoundSql);
       //通过connection建立一个countSql对应的PreparedStatement对象。
       PreparedStatement pstmt = null;
       ResultSet rs = null;
       try {
           pstmt = connection.prepareStatement(countSql);
           //通过parameterHandler给PreparedStatement对象设置参数
           parameterHandler.setParameters(pstmt);
          
           rs = pstmt.executeQuery();
           if (rs.next()) {
              int totalRecord = rs.getInt(1);
              //给当前的参数page对象设置总记录数
              page.setTotalRecord(totalRecord);
           }
       } catch (SQLException e) {
           e.printStackTrace();
       } finally {
           try {
              if (rs != null)
                  rs.close();
               if (pstmt != null)
                  pstmt.close();
           } catch (SQLException e) {
              e.printStackTrace();
           }
       }
    }
    
    /**
     * 拼接参数 
     * @param page
     * @param sql
     * @return
     */
   private String getParamSql(Page<?> page, StringBuffer sqlBuffer){
	   
	   Map<String, Object> params =page.getParams();
	   
	   	if(null!=params){
	   		boolean first= true; 
	   		for(Map.Entry<String, Object> entry :params.entrySet() ){
	   			if(first){
	   				sqlBuffer.append(" where ").append(entry.getKey()).append(" = ").append(entry.getValue());
	   				first=!first;
	   			}else{
	   				sqlBuffer.append(" and ").append(entry.getKey()).append(" = ").append(entry.getValue());
	   			}
	   		}
	   	}
	   
	   return sqlBuffer.toString();  
   }
    
    /**
     * 根据原Sql语句获取对应的查询总记录数的Sql语句
     * @param sql
     * @return
     */
    private String getCountSql(Page<?> page,String sql) {
    	
    	String countSql	=	null;
    	int index 		=	sql.indexOf("from");
    	countSql		=	"select count(*) " + sql.substring(index);
    	countSql		=	getParamSql(page, new StringBuffer(countSql));
    	
       return countSql;
    }
   
    /**
     * 利用反射进行操作的一个工具类
     *
     */
    private static class ReflectUtil {
       /**
        * 利用反射获取指定对象的指定属性
        * @param obj 目标对象
        * @param fieldName 目标属性
        * @return 目标属性的值
        */
       public static Object getFieldValue(Object obj, String fieldName) {
           Object result = null;
           Field field = ReflectUtil.getField(obj, fieldName);
           if (field != null) {
              field.setAccessible(true);
              try {
                  result = field.get(obj);
              } catch (IllegalArgumentException e) {
            	  log.error(e.getMessage());
                  e.printStackTrace();
              } catch (IllegalAccessException e) {
                  log.error(e.getMessage());
                  e.printStackTrace();
              }
           }
           return result;
       }
      
       /**
        * 利用反射获取指定对象里面的指定属性
        * @param obj 目标对象
        * @param fieldName 目标属性
        * @return 目标字段
        */
       private static Field getField(Object obj, String fieldName) {
           Field field = null;
          for (Class<?> clazz=obj.getClass(); clazz != Object.class; clazz=clazz.getSuperclass()) {
              try {
                  field = clazz.getDeclaredField(fieldName);
                  break;
              } catch (NoSuchFieldException e) {
                  //这里不用做处理，子类没有该字段可能对应的父类有，都没有就返回null。
              }
           }
           return field;
       }
 
       /**
        * 利用反射设置指定对象的指定属性为指定的值
        * @param obj 目标对象
        * @param fieldName 目标属性
         * @param fieldValue 目标值
        */
       public static void setFieldValue(Object obj, String fieldName,
              String fieldValue) {
           Field field = ReflectUtil.getField(obj, fieldName);
           if (field != null) {
              try {
                  field.setAccessible(true);
                  field.set(obj, fieldValue);
              } catch (IllegalArgumentException e) {
            	  log.error(e.getMessage());
                  e.printStackTrace();
              } catch (IllegalAccessException e) {
            	  log.error(e.getMessage());
                  e.printStackTrace();
              }
           }
        }
    }
}

```
然后是mybatis配置文件在这里我命名为myBatisConfig.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

	<!-- 开启懒加载 -->
	<settings>
		<setting name="lazyLoadingEnabled" value="true"/>
		<setting name="aggressiveLazyLoading" value="false"/>
		<setting name="cacheEnabled" value="true" />
	</settings>
	  <typeAliases>
       <typeAlias type="com.stock.common.mybatis.Page" alias="page"/>
    </typeAliases>
	<plugins>
       <plugin interceptor="com.stock.common.mybatis.PageInterceptor">
           <property name="databaseType" value="Mysql"/>
       </plugin>
    </plugins>
	<mappers>
	     <mapper resource="com/stock/mybatis-conf/jikeUser.xml" />
	</mappers>
</configuration>
```
最后是Mappers文件 xx.xml

```
<mapper namespace="com.xx.dao.xxDao">
<select id="findBy" parameterType="page" resultType="java.util.LinkedHashMap" useCache="true">
		<![CDATA[
			select * from jikeuser 
		]]>
	</select>
```
对应DAO接口
```
@Repository
public interface xxDao {
	
	List<Map<String, Object>> findBy(Page<Map<String, Object>> params);
}
```
如此以来，就成功啦！
