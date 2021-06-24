> 介绍在java中使用redis的框架jedis。

- 添加maven依赖

```xml
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>2.9.0</version>
</dependency>
```

# 普通配置

```java
public class RedisConfig {
	@Bean
	public JedisPoolConfig jedisPoolConfig(){
		JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
		jedisPoolConfig.setMaxIdle(10);
		jedisPoolConfig.setMinIdle(5);
		jedisPoolConfig.setMaxTotal(20);
		return jedisPoolConfig;
	}
	@Bean
	public JedisPool jedisPool(JedisPoolConfig jedisPoolConfig){
		return new JedisPool(jedisPoolConfig, url, port, timeout, password);
	}
}
```

- 获取连接

```java
Jedis resource = jedisPool.getResource();
```

- 执行redis指令

```java

```

