`Redis`��һ����Դ�ģ�����`key-value`�洢�����������������ܣ�����չ��Ӧ�ó���.�ڲ��洢��ʵ�洢���Ƕ����ƣ���ȡʱͨ�����л������л�����.
#��Ҫ����
��Ҫ�ص㣺
*.	`Redis`����ȫ���ڴ��б������ݵ����ݿ⣬ʹ�ô���ֻ��Ϊ�˳־���Ŀ�� 
*.	`Redis`�������ֵ���ݴ洢ϵͳ����Էḻ����������,��`String`,`Set`,`Map`,`List`
*.	`Redis`���Խ����ݸ��Ƶ����������Ĵӷ�������
�ŵ�:
*.	�ٶȺܿ�
*.	��֧�ֵ��������ͷḻ
*.	������ԭ�Ӳ���
*.	�๦��ʵ�ù���
#`Spring-data-Redis`
����ʹ��Spring�Ƚ϶�,��չʾSpring��Redis���ɵĲ���.
##jar������
jedis-**.jar,spring-data-redis-**.jar
##��������
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
##Spring��������
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
##����Դʹ��
ͨ��`redisTemplate`���������������ɾ�Ĳ��.
###�����������빲�е�����
```java
public class RedisBaseDao<K,V> {

	@Autowired
	protected RedisTemplate<K,V> redisTemplate;
	
	@Resource(name="redisTemplate")
	protected HashOperations<K, Object, Object> hashOperations;

	
}
```
###����
����`HashOperations`
```java
// ͨ��hashmap�洢
if (hashOperations.hasKey(KEY, entity.getName())) {
	delete(entity);
	}
hashOperations.put(KEY, entity.getName(), entity);
```
����`ListOperations`
```java
// ͨ��list���д洢
// Queue LILO
long count = redisTemplate.opsForList().rightPush(KEY, entity);
```
����ͨ���Լ����л�������
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
###ɾ��
```java
long count = hashOperations.delete(KEY, entity.getName());
```
###����
����`HashOperations`
```java
hashOperations.multiGet(KEY, hashOperations.keys(KEY))
```
����`ListOperations`
```java
public Customer get(Customer entity) {
		ListOperations<String, Customer> list = redisTemplate.opsForList();
		List<Customer> lists = list.range(KEY, 0, -1);
		// �˴���������Ĭ�ϵ�KEYSerializer,���Բ��������л�����
		return list.leftPop(KEY);
	}
```
###����
```java
//sort,����ʹ��user��Ϊkeyֵ
		List<Object> lst=hashOperations.multiGet(KEY, hashOperations.keys(KEY));
		Customer[] arrays=new Customer[lst.size()];
		for(int i=0;i<lst.size();i++){
			arrays[i]=(Customer) lst.get(i);
		}
        SetOperations<String, Customer>  setOper= redisTemplate.opsForSet();
        String setKey = "not use user for key"; 
        setOper.add(setKey,arrays);
        SortQueryBuilder<String> builder = SortQueryBuilder.sort(setKey);  
        builder.alphabetical(true);//���ַ���ʹ�á��ֵ�˳��  
        builder.order(Order.DESC);//����  
        builder.limit(0, 2);  
        //builder.limit(new Range(0, 2));  
        List<Customer> results = redisTemplate.sort(builder.build());  
        for(Customer item : results){  
            System.out.println(item);  
        }
```
------------------
*Redis�洢ʱ����ʹ��user��Ϊkeyֵ*
------------------
