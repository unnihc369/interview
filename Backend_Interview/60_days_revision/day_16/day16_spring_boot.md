# Day 16 — JWT Authentication & UserDetailsService

**Topics:** JWT Structure · Stateless Auth · UserDetailsService · Spring Security Integration

---

# 1. Why JWT?

Traditional session auth stores state on server. JWT is **stateless** — token carries claims, server validates signature.

Benefits:
- Scales horizontally (no session stickiness)
- Works across microservices
- Mobile/SPA friendly

---

# 2. JWT Structure

```text
header.payload.signature
```

Each part is Base64URL encoded, joined by `.`

---

# JWT Parts

| Part | Contains |
|------|----------|
| Header | Algorithm (HS256, RS256), type (JWT) |
| Payload | Claims (sub, exp, roles, custom) |
| Signature | HMAC or RSA sign(header + payload) |

Example payload:

```json
{
  "sub": "user@example.com",
  "roles": ["USER"],
  "iat": 1717000000,
  "exp": 1717003600
}
```

---

# 3. JWT Flow

```text
Login (username/password)
        ↓
UserDetailsService validates
        ↓
Generate JWT with claims
        ↓
Return token to client
        ↓
Client sends: Authorization: Bearer <token>
        ↓
JwtFilter validates signature + expiry
        ↓
Set SecurityContext → controller
```

---

# 4. Dependencies

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.12.5</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.12.5</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>0.12.5</version>
    <scope>runtime</scope>
</dependency>
```

---

# 5. UserDetailsService Implementation

```java
@Service
@RequiredArgsConstructor
public class CustomUserDetailsService implements UserDetailsService {

    private final UserRepository userRepository;

    @Override
    public UserDetails loadUserByUsername(String username)
            throws UsernameNotFoundException {
        User user = userRepository.findByEmail(username)
                .orElseThrow(() -> new UsernameNotFoundException(
                        "User not found: " + username));

        return org.springframework.security.core.userdetails.User.builder()
                .username(user.getEmail())
                .password(user.getPasswordHash())
                .authorities(user.getRoles().stream()
                        .map(r -> new SimpleGrantedAuthority("ROLE_" + r))
                        .toList())
                .accountLocked(user.isLocked())
                .disabled(!user.isActive())
                .build();
    }
}
```

---

# 6. JWT Utility Service

```java
@Service
public class JwtService {

    @Value("${jwt.secret}")
    private String secretKey;

    @Value("${jwt.expiration-ms:3600000}")
    private long expirationMs;

    public String generateToken(UserDetails userDetails) {
        Map<String, Object> claims = new HashMap<>();
        claims.put("roles", userDetails.getAuthorities().stream()
                .map(GrantedAuthority::getAuthority).toList());

        return Jwts.builder()
                .claims(claims)
                .subject(userDetails.getUsername())
                .issuedAt(new Date())
                .expiration(new Date(System.currentTimeMillis() + expirationMs))
                .signWith(getSigningKey())
                .compact();
    }

    public String extractUsername(String token) {
        return parseClaims(token).getSubject();
    }

    public boolean isTokenValid(String token, UserDetails userDetails) {
        String username = extractUsername(token);
        return username.equals(userDetails.getUsername()) && !isExpired(token);
    }

    private Claims parseClaims(String token) {
        return Jwts.parser()
                .verifyWith(getSigningKey())
                .build()
                .parseSignedClaims(token)
                .getPayload();
    }

    private SecretKey getSigningKey() {
        byte[] keyBytes = Decoders.BASE64.decode(secretKey);
        return Keys.hmacShaKeyFor(keyBytes);
    }

    private boolean isExpired(String token) {
        return parseClaims(token).getExpiration().before(new Date());
    }
}
```

---

# 7. JWT Authentication Filter

```java
@Component
@RequiredArgsConstructor
public class JwtAuthFilter extends OncePerRequestFilter {

    private final JwtService jwtService;
    private final UserDetailsService userDetailsService;

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain)
            throws ServletException, IOException {

        String authHeader = request.getHeader("Authorization");
        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            filterChain.doFilter(request, response);
            return;
        }

        String token = authHeader.substring(7);
        String username = jwtService.extractUsername(token);

        if (username != null &&
                SecurityContextHolder.getContext().getAuthentication() == null) {
            UserDetails userDetails = userDetailsService.loadUserByUsername(username);
            if (jwtService.isTokenValid(token, userDetails)) {
                UsernamePasswordAuthenticationToken authToken =
                        new UsernamePasswordAuthenticationToken(
                                userDetails, null, userDetails.getAuthorities());
                SecurityContextHolder.getContext().setAuthentication(authToken);
            }
        }
        filterChain.doFilter(request, response);
    }
}
```

---

# 8. Security Configuration

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    private final JwtAuthFilter jwtAuthFilter;

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())
            .sessionManagement(s -> s.sessionCreationPolicy(STATELESS))
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**").permitAll()
                .anyRequest().authenticated()
            )
            .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }

    @Bean
    public AuthenticationManager authManager(AuthenticationConfiguration config)
            throws Exception {
        return config.getAuthenticationManager();
    }
}
```

---

# 9. Login Endpoint

```java
@RestController
@RequestMapping("/api/auth")
@RequiredArgsConstructor
public class AuthController {

    private final AuthenticationManager authManager;
    private final JwtService jwtService;

    @PostMapping("/login")
    public ResponseEntity<TokenResponse> login(@RequestBody LoginRequest req) {
        Authentication auth = authManager.authenticate(
                new UsernamePasswordAuthenticationToken(req.email(), req.password()));
        UserDetails user = (UserDetails) auth.getPrincipal();
        String token = jwtService.generateToken(user);
        return ResponseEntity.ok(new TokenResponse(token));
    }
}
```

---

# Interview Questions

## JWT vs Session — when to use which?

| JWT | Session |
|-----|---------|
| Stateless, microservices | Stateful, monolith |
| Token revocation harder | Easy logout (invalidate session) |
| Self-contained claims | Server stores session data |

Use JWT for SPAs/mobile/APIs. Use sessions for traditional server-rendered apps.

---

## How to revoke JWT before expiry?

Maintain token blacklist in Redis. Check blacklist in filter. Or use short expiry + refresh tokens.

---

## Where to store JWT on client?

- **HttpOnly cookie** — XSS resistant (preferred for web)
- **localStorage** — vulnerable to XSS, common in SPAs
- Never in URL query params

---

## What is UserDetailsService role in JWT flow?

Called: Validates credentials at login. At subsequent requests, JWT filter may reload UserDetails to verify user still active and get fresh authorities.

---

## HS256 vs RS256?

| HS256 | RS256 |
|-------|-------|
| Symmetric (shared secret) | Asymmetric (private sign, public verify) |
| Single service | Microservices verify with public key only |

---

# Quick Revision

```text
JWT = header.payload.signature (stateless)
UserDetailsService → load user for auth
JwtAuthFilter → validate token per request
STATELESS session → no server-side session
Refresh tokens → extend session without long-lived access token
```

---

*End of Day 16 JWT & UserDetailsService*
