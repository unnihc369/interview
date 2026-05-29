# Day 38 — Spring OAuth2 Resource Server

**Topics:** OAuth2 Roles · JWT Resource Server · SecurityFilterChain · Scopes · Interview Questions

---

# 1. OAuth2 Roles

| Role | Responsibility |
|------|----------------|
| Resource Owner | User granting access |
| Client | App requesting access |
| Authorization Server | Issues tokens (Keycloak, Okta, Auth0) |
| Resource Server | API protecting resources (your Spring app) |

---

# 2. Resource Server Dependency

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
</dependency>
```

```properties
spring.security.oauth2.resourceserver.jwt.issuer-uri=https://auth.example.com/realms/myrealm
# OR
spring.security.oauth2.resourceserver.jwt.jwk-set-uri=https://auth.example.com/realms/myrealm/protocol/openid-connect/certs
```

---

# 3. Security Configuration

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/actuator/health").permitAll()
                .requestMatchers(HttpMethod.GET, "/api/public/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .requestMatchers("/api/orders/**").hasAuthority("SCOPE_orders.read")
                .anyRequest().authenticated()
            )
            .oauth2ResourceServer(oauth2 -> oauth2.jwt(Customizer.withDefaults()));
        return http.build();
    }
}
```

---

# 4. Access JWT Claims in Controller

```java
@GetMapping("/me")
public Map<String, Object> me(@AuthenticationPrincipal Jwt jwt) {
    return Map.of(
        "sub", jwt.getSubject(),
        "email", jwt.getClaimAsString("email"),
        "roles", jwt.getClaimAsStringList("roles")
    );
}
```

---

# 5. Custom JWT → Authorities

```java
@Bean
public JwtAuthenticationConverter jwtAuthenticationConverter() {
    JwtGrantedAuthoritiesConverter scopes = new JwtGrantedAuthoritiesConverter();
    scopes.setAuthorityPrefix("SCOPE_");

    JwtAuthenticationConverter converter = new JwtAuthenticationConverter();
    converter.setJwtGrantedAuthoritiesConverter(jwt -> {
        Collection<GrantedAuthority> authorities = new ArrayList<>();
        authorities.addAll(scopes.convert(jwt));
        List<String> roles = jwt.getClaimAsStringList("roles");
        if (roles != null) {
            roles.forEach(r -> authorities.add(new SimpleGrantedAuthority("ROLE_" + r)));
        }
        return authorities;
    });
    return converter;
}
```

Wire in:

```java
.oauth2ResourceServer(oauth2 -> oauth2
    .jwt(jwt -> jwt.jwtAuthenticationConverter(jwtAuthenticationConverter())))
```

---

# 6. Method Security

```java
@EnableMethodSecurity
@Configuration
public class MethodSecurityConfig {}

@Service
public class OrderService {

    @PreAuthorize("hasAuthority('SCOPE_orders.write')")
    public Order create(OrderRequest req) { ... }

    @PreAuthorize("#userId == authentication.principal.subject")
    public Profile getProfile(String userId) { ... }
}
```

---

# 7. Token Flow (Interview)

```text
1. User logs in at Authorization Server
2. Client receives access_token (JWT)
3. Client calls Resource Server: Authorization: Bearer <token>
4. Resource Server validates signature (JWK), expiry, issuer, audience
5. Maps scopes/roles → GrantedAuthority
6. Allows or denies request
```

---

# 8. Resource Server vs OAuth2 Client

| Resource Server | OAuth2 Client |
|-----------------|---------------|
| Validates incoming JWT | Obtains token (client_credentials, authorization_code) |
| Protects your APIs | Calls other APIs |

```xml
spring-boot-starter-oauth2-client
```

---

# 9. Common Errors

| Error | Cause |
|-------|-------|
| 401 invalid_token | Expired, wrong signature, wrong issuer |
| 403 insufficient_scope | Missing scope claim |
| 401 no bearer | Missing Authorization header |

---

# Interview Questions

## Q1. JWT vs opaque token?

JWT self-contained (claims + signature). Opaque token requires introspection endpoint at auth server.

## Q2. Where is JWT validated?

`BearerTokenAuthenticationFilter` → `JwtDecoder` (Nimbus) using JWK Set.

## Q3. How revoke JWT before expiry?

Short TTL + refresh token; token blocklist in Redis; or session at auth server.

## Q4. `hasRole` vs `hasAuthority`?

`hasRole('ADMIN')` expects `ROLE_ADMIN`. `hasAuthority` uses exact string (e.g. `SCOPE_read`).

## Q5. mTLS vs OAuth2?

mTLS = client certificate at transport layer. OAuth2 = application-layer delegation. Often combined in zero-trust.

---

# One-Line Revision

```text
oauth2-resource-server + issuer-uri; JWT validated automatically; map scopes/roles to authorities.
```

---

*End of Day 38 Spring Boot*
