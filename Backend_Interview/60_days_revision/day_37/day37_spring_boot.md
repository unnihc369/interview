# Day 37 — Spring WebSockets & @MessageMapping

**Topics:** STOMP · WebSocket Config · @MessageMapping · SockJS · Security · Interview Questions

---

# 1. WebSocket vs HTTP

| HTTP | WebSocket |
|------|-----------|
| Request-response | Full-duplex persistent connection |
| Client polls for updates | Server push in real time |
| Higher latency for chat/live | Low latency bidirectional |

Spring supports raw WebSocket + **STOMP** messaging protocol on top.

---

# 2. Dependencies

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
```

---

# 3. WebSocket Configuration

```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        registry.enableSimpleBroker("/topic", "/queue");
        registry.setApplicationDestinationPrefixes("/app");
    }

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/ws")
                .setAllowedOriginPatterns("*")
                .withSockJS();
    }
}
```

```text
Client → /app/*  → @MessageMapping handlers
Broker → /topic/*, /queue/*  → subscriptions
```

---

# 4. @MessageMapping Controller

```java
@Controller
public class ChatController {

    @MessageMapping("/chat.send")
    @SendTo("/topic/room.{roomId}")
    public ChatMessage send(ChatMessage message, @DestinationVariable String roomId) {
        return messageService.save(message);
    }

    @MessageMapping("/chat.private")
    public void sendPrivate(@Payload ChatMessage msg, SimpMessageHeaderAccessor header) {
        String userId = header.getUser().getName();
        messagingTemplate.convertAndSendToUser(
            msg.getRecipientId(), "/queue/messages", msg);
    }
}
```

Client sends to: `/app/chat.send`  
Subscribers receive from: `/topic/room.123`

---

# 5. SimpMessagingTemplate (Server Push)

```java
@Service
@RequiredArgsConstructor
public class NotificationService {

    private final SimpMessagingTemplate messagingTemplate;

    public void notifyUser(String userId, Notification n) {
        messagingTemplate.convertAndSendToUser(userId, "/queue/notifications", n);
    }

    public void broadcast(String topic, Object payload) {
        messagingTemplate.convertAndSend("/topic/" + topic, payload);
    }
}
```

---

# 6. Client Flow (STOMP.js sketch)

```javascript
const socket = new SockJS('/ws');
const stomp = Stomp.over(socket);
stomp.connect({}, () => {
  stomp.subscribe('/topic/room.123', (msg) => console.log(JSON.parse(msg.body)));
  stomp.send('/app/chat.send', {}, JSON.stringify({ text: 'Hello', roomId: '123' }));
});
```

---

# 7. Security

```java
@Configuration
public class WebSocketSecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.authorizeHttpRequests(auth -> auth
            .requestMatchers("/ws/**").authenticated());
        return http.build();
    }
}
```

Pass JWT in STOMP connect headers; validate in `ChannelInterceptor`.

```java
@Override
public void configureClientInboundChannel(ChannelRegistration registration) {
    registration.interceptors(new AuthChannelInterceptor());
}
```

---

# 8. Scaling WebSockets

| Challenge | Solution |
|-----------|----------|
| Sticky sessions | Session affinity at load balancer |
| Cross-server broadcast | Redis Pub/Sub or RabbitMQ relay |
| Connection limits | Horizontal scale + connection registry |

Spring supports **external broker relay** (RabbitMQ) instead of in-memory simple broker.

---

# Interview Questions

## Q1. STOMP vs raw WebSocket?

STOMP adds frame types (SUBSCRIBE, SEND, MESSAGE), routing, heartbeats — easier than parsing raw frames.

## Q2. SockJS purpose?

Fallback transports (xhr-streaming, long-polling) when WebSocket blocked by proxies/firewalls.

## Q3. @SendTo vs SimpMessagingTemplate?

`@SendTo` for simple broadcast from handler; template for programmatic push from any service.

## Q4. How authenticate WebSocket?

JWT in CONNECT frame; `ChannelInterceptor` validates before message reaches `@MessageMapping`.

## Q5. Difference from SSE?

SSE = server-to-client only over HTTP. WebSocket = bidirectional. Use SSE for live feeds; WebSocket for chat/games.

---

# One-Line Revision

```text
@EnableWebSocketMessageBroker; @MessageMapping on /app; subscribe /topic; SockJS + JWT interceptor for auth.
```

---

*End of Day 37 Spring Boot*
