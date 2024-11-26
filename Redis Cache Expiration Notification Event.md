# Redis Cache Expiration Notification Event

#### 1. Overview

In one of our projects, we encountered a need for Redis cache expiration notification events. This prompted an exploration into Redis expiration notifications and how to integrate them with Spring.

For more information, refer to these resources:

- [Redis EXPIRE command reference](https://redis.io/commands/expire)
- [Spring Data Redis GitHub repository](https://github.com/spring-projects/spring-data-redis)
- [Spring Data Redis new feature documentation](https://docs.spring.io/spring-data/redis/docs/2.1.9.RELEASE/reference/html/#redis:pubsub:subscribe:containers)

------

#### 2. Redis and Spring Integration

Through research, we discovered that Redis cache expiration notifications can be implemented by extending the `KeyExpirationEventMessageListener` class from Spring Data Redis and overriding the `onMessage(Message message, byte[] pattern)` method. By parsing the expired key with `message.toString()`, we can verify if the expired key matches specific business logic and execute the necessary operations. Once this method is customized, the `RedisMessageListenerContainer` must be configured as a Spring Bean.

##### Extending `KeyExpirationEventMessageListener` and Overriding the `onMessage` Method

Example:

```java
@Service
@Slf4j
public class RedisKeyExpirationListener extends KeyExpirationEventMessageListener {

    /**
     * Creates a new {@link RedisKeyExpirationListener} for {@code __keyevent@*__:expired} messages.
     *
     * @param listenerContainer must not be {@literal null}.
     */
    public RedisKeyExpirationListener(RedisMessageListenerContainer listenerContainer) {
        super(listenerContainer);
    }

    @Override
    public void onMessage(Message message, byte[] pattern) {
        // Handle the expired key
    }
}
```

##### Configuring `RedisMessageListenerContainer`

Example:

```java
@Bean
public RedisMessageListenerContainer container(RedisConnectionFactory connectionFactory) {
    final RedisMessageListenerContainer container = new RedisMessageListenerContainer();
    container.setConnectionFactory(connectionFactory);
    return container;
}
```

After configuration, during project startup, Spring will register the `RedisKeyExpirationListener` Bean using its constructor, which takes the `RedisMessageListenerContainer` Bean as an argument. This triggers the `afterPropertiesSet` method of the `KeyspaceEventMessageListener` class for listener registration.

------

#### 3. Key Methods and Workflow

##### Listener Registration

In `RedisMessageListenerContainer`, the `addMessageListener` method adds listeners dynamically:

```java
public void addMessageListener(MessageListener listener, Topic topic) {
    addMessageListener(listener, Collections.singleton(topic));
}
```

The `lazyListen` method ensures that the listener is ready when the Redis connection starts. Eventually, the `SubscriptionTask#run` method handles the Redis subscription, linking the `DispatchMessageListener` back to your custom listener.

------

#### 4. Summary

- By extending `KeyExpirationEventMessageListener` and customizing the `onMessage` method, you can handle Redis expiration notifications.
- Ensure `RedisMessageListenerContainer` is declared as a Spring Bean, using `RedisConnectionFactory` for Redis connection configurations.

This integration allows efficient handling of cache expiration events for enhanced application responsiveness.