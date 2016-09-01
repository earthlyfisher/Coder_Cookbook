`Redis`是一个开源的，基于`key-value`存储，可用来构建高性能，可扩展的应用程序.内部存储其实存储的是二进制，存取时通过序列化反序列化操作.
#简要概述
主要特点：
*.	`Redis`是完全在内存中保存数据的数据库，使用磁盘只是为了持久性目的 
*.	`Redis`相比许多键值数据存储系统有相对丰富的数据类型,有`String`,`Set`,`Map`,`List`
*.	`Redis`可以将数据复制到任意数量的从服务器中
优点:
*.	速度很快
*.	可支持的数据类型丰富
*.	操作是原子操作
*.	多功能实用工具
#`Spring-data-Redis`
由于使用Spring比较多,现展示Spring与Redis集成的操作.
##jar包下载
jedis-**.jar,spring-data-redis-**.jar
##参数配置
```java
redis.pool.maxActive=1024  
redis.pool.maxIdle=200  
redis.pool.maxWait=1000  
redis.pool.testOnBorrow=true  
  
#IP  
redis.host=localhost
#Port  
redis.port=6379  
```
##Spring集成配置
```java
<bean id="prop"
		class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
		<property name="locations">
			<list>
				<value>classpath:redis.properties</value>
			</list>
		</property>
	</bean>

	<bean id="jedisPoolConfig" class="redis.clients.jedis.JedisPoolConfig">
		<property name="maxTotal" value="${redis.pool.maxActive}" />
		<property name="maxIdle" value="${redis.pool.maxIdle}" />
		<property name="maxWaitMillis" value="${redis.pool.maxWait}" />
		<property name="testOnBorrow" value="${redis.pool.testOnBorrow}" />
	</bean>

	<bean id="redisConnectFactory"
		class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory"
		p:host-name="${redis.host}" p:port="${redis.port}" p:pool-config-ref="jedisPoolConfig" />

	<bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate">
		<property name="connectionFactory" ref="redisConnectFactory" />
		<property name="keySerializer">
			<bean
				class="org.springframework.data.redis.serializer.StringRedisSerializer" />
		</property>
		<property name="hashKeySerializer">
			<bean
				class="org.springframework.data.redis.serializer.StringRedisSerializer" />
		</property>
		<property name="valueSerializer">
			<bean
				class="org.springframework.data.redis.serializer.JdkSerializationRedisSerializer" />
		</property>
		<property name="hashValueSerializer">
			<bean
				class="org.springframework.data.redis.serializer.JdkSerializationRedisSerializer" />
		</property>
	</bean>
	```
##数据源使用
通过`redisTemplate`操作具体的数据增删改查等.
###基类用来抽离共有的属性
```java
public class RedisBaseDao<K,V> {

	@Autowired
	protected RedisTemplate<K,V> redisTemplate;
	
	@Resource(name="redisTemplate")
	protected HashOperations<K, Object, Object> hashOperations;

	
}
```
###增加
操作`HashOperations`
```java
// 通过hashmap存储
if (hashOperations.hasKey(KEY, entity.getName())) {
	delete(entity);
	}
hashOperations.put(KEY, entity.getName(), entity);
```
操作`ListOperations`
```java
// 通过list队列存储
// Queue LILO
long count = redisTemplate.opsForList().rightPush(KEY, entity);
```
可以通过自己序列化来处理
```java
long count = redisTemplate.execute(new RedisCallback<Long>() {
			public Long doInRedis(RedisConnection connection) throws DataAccessException {
				JdkSerializationRedisSerializer valueSerializer = (JdkSerializationRedisSerializer) redisTemplate
						.getValueSerializer();
				return connection.lPush(redisTemplate.getStringSerializer().serialize(KEY),
						valueSerializer.serialize(entity));
			}
		});
```
###删除
```java
long count = hashOperations.delete(KEY, entity.getName());
```
###查找
操作`HashOperations`
```java
hashOperations.multiGet(KEY, hashOperations.keys(KEY))
```
操作`ListOperations`
```java
public Customer get(Customer entity) {
		ListOperations<String, Customer> list = redisTemplate.opsForList();
		List<Customer> lists = list.range(KEY, 0, -1);
		// 此处由于配有默认的KEYSerializer,所以不用做序列化处理
		return list.leftPop(KEY);
	}
```
###排序
```java
//sort,不能使用user作为key值
		List<Object> lst=hashOperations.multiGet(KEY, hashOperations.keys(KEY));
		Customer[] arrays=new Customer[lst.size()];
		for(int i=0;i<lst.size();i++){
			arrays[i]=(Customer) lst.get(i);
		}
        SetOperations<String, Customer>  setOper= redisTemplate.opsForSet();
        String setKey = "not use user for key"; 
        setOper.add(setKey,arrays);
        SortQueryBuilder<String> builder = SortQueryBuilder.sort(setKey);  
        builder.alphabetical(true);//对字符串使用“字典顺序”  
        builder.order(Order.DESC);//倒序  
        builder.limit(0, 2);  
        //builder.limit(new Range(0, 2));  
        List<Customer> results = redisTemplate.sort(builder.build());  
        for(Customer item : results){  
            System.out.println(item);  
        }
```
------------------
*Redis存储时不能使用user作为key值*
------------------
