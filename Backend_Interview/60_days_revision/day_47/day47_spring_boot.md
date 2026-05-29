# Day 47 — Spring Security & JWT Review

**Topics:** Filter chain · JWT flow · UserDetailsService · Method security · Interview Q&A

---

# 1. Spring Security Architecture

```text
HTTP Request
    ↓
SecurityFilterChain (ordered filters)
    ↓
AuthenticationManager
    ↓
SecurityContext (ThreadLocal)
    ↓
Controller (@PreAuthorize checks)
```

---

# 2. Core Components

| Component | Role |
|-----------|------|
| `SecurityFilterChain` | Defines which URLs need auth, which filters apply |
| `AuthenticationManager` | Validates credentials |
| `UserDetailsService` | Loads user by username |
| `PasswordEncoder` | BCrypt hash/verify |
| `JwtTokenProvider` | Generate/validate JWT |
| `SecurityContextHolder` | Stores current Authentication |

---

# 3. JWT Flow (Stateless Auth)

```text
1. POST /auth/login { username, password }
2. Server validates → generates JWT (header.payload.signature)
3. Client stores token (memory / httpOnly cookie)
4. Subsequent requests: Authorization: Bearer <token>
5. JwtAuthFilter validates signature + expiry → sets SecurityContext
6. Controller executes with authenticated principal
```

---

# 4. Security Configuration (Spring Boot 3)

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfig {

    private final JwtAuthFilter jwtAuthFilter;
    private final UserDetailsService userDetailsService;

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
            .csrf(csrf -> csrf.disable())  // stateless JWT — disable CSRF
            .sessionManagement(s -> s.sessionCreationPolicy(STATELESS))
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/auth/**", "/actuator/health").permitAll()
                .requestMatchers("/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .authenticationProvider(daoAuthProvider())
            .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class)
            .exceptionHandling(e -> e
                .authenticationEntryPoint((req, res, ex) ->
                    res.sendError(HttpServletResponse.SC_UNAUTHORIZED))
            )
            .build();
    }

    @Bean
    public AuthenticationProvider daoAuthProvider() {
        DaoAuthenticationProvider provider = new DaoAuthenticationProvider();
        provider.setUserDetailsService(userDetailsService);
        provider.setPasswordEncoder(passwordEncoder());
        return provider;
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(12);
    }
}
```

---

# 5. JWT Token Provider

```java
@Component
public class JwtTokenProvider {

    @Value("${jwt.secret}")
    private String secret;

    @Value("${jwt.expiration-ms:86400000}")
    private long expirationMs;

    public String generateToken(UserDetails user) {
        Date now = new Date();
        return Jwts.builder()
            .subject(user.getUsername())
            .claim("roles", user.getAuthorities().stream()
                .map(GrantedAuthority::getAuthority).toList())
            .issuedAt(now)
            .expiration(new Date(now.getTime() + expirationMs))
            .signWith(getSigningKey())
            .compact();
    }

    public String extractUsername(String token) {
        return parseClaims(token).getSubject();
    }

    public boolean isValid(String token, UserDetails user) {
        String username = extractUsername(token);
        return username.equals(user.getUsername()) && !isExpired(token);
    }

    private Claims parseClaims(String token) {
        return Jwts.parser()
            .verifyWith(getSigningKey())
            .build()
            .parseSignedClaims(token)
            .getPayload();
    }

    private SecretKey getSigningKey() {
        byte[] keyBytes = Decoders.BASE64.decode(secret);
        return Keys.hmacShaKeyFor(keyBytes);
    }
}
```

---

# 6. JWT Authentication Filter

```java
@Component
@RequiredArgsConstructor
public class JwtAuthFilter extends OncePerRequestFilter {

    private final JwtTokenProvider tokenProvider;
    private final UserDetailsService userDetailsService;

    @Override
    protected void doFilterInternal(HttpServletRequest req,
                                    HttpServletResponse res,
                                    FilterChain chain)
            throws ServletException, IOException {

        String header = req.getHeader("Authorization");
        if (header == null || !header.startsWith("Bearer ")) {
            chain.doFilter(req, res);
            return;
        }

        String token = header.substring(7);
        try {
            String username = tokenProvider.extractUsername(token);
            if (username != null &&
                SecurityContextHolder.getContext().getAuthentication() == null) {

                UserDetails user = userDetailsService.loadUserByUsername(username);
                if (tokenProvider.isValid(token, user)) {
                    var auth = new UsernamePasswordAuthenticationToken(
                        user, null, user.getAuthorities());
                    auth.setDetails(new WebAuthenticationDetailsSource()
                        .buildDetails(req));
                    SecurityContextHolder.getContext().setAuthentication(auth);
                }
            }
        } catch (JwtException e) {
            res.sendError(HttpServletResponse.SC_UNAUTHORIZED, "Invalid token");
            return;
        }
        chain.doFilter(req, res);
    }
}
```

---

# 7. Auth Controller

```java
@RestController
@RequestMapping("/auth")
@RequiredArgsConstructor
public class AuthController {

    private final AuthenticationManager authManager;
    private final JwtTokenProvider tokenProvider;

    @PostMapping("/login")
    public AuthResponse login(@Valid @RequestBody LoginRequest req) {
        Authentication auth = authManager.authenticate(
            new UsernamePasswordAuthenticationToken(req.username(), req.password()));
        String token = tokenProvider.generateToken((UserDetails) auth.getPrincipal());
        return new AuthResponse(token, "Bearer");
    }

    @PostMapping("/register")
    public ResponseEntity<Void> register(@Valid @RequestBody RegisterRequest req) {
        userService.register(req);
        return ResponseEntity.status(HttpStatus.CREATED).build();
    }
}
```

---

# 8. UserDetailsService Implementation

```java
@Service
@RequiredArgsConstructor
public class CustomUserDetailsService implements UserDetailsService {

    private final UserRepository userRepo;

    @Override
    public UserDetails loadUserByUsername(String username) {
        User user = userRepo.findByEmail(username)
            .orElseThrow(() -> new UsernameNotFoundException(username));

        return org.springframework.security.core.userdetails.User.builder()
            .username(user.getEmail())
            .password(user.getPasswordHash())
            .roles(user.getRole().name())
            .disabled(!user.isActive())
            .build();
    }
}
```

---

# 9. Method-Level Security

```java
@RestController
@RequestMapping("/api/v1/orders")
public class OrderController {

    @GetMapping("/{id}")
    @PreAuthorize("hasRole('USER') and @orderSecurity.isOwner(#id, authentication)")
    public OrderResponse get(@PathVariable Long id) { ... }

    @DeleteMapping("/{id}")
    @PreAuthorize("hasRole('ADMIN')")
    public void delete(@PathVariable Long id) { ... }
}

@Component("orderSecurity")
public class OrderSecurityService {
    public boolean isOwner(Long orderId, Authentication auth) {
        return orderRepo.findById(orderId)
            .map(o -> o.getUserEmail().equals(auth.getName()))
            .orElse(false);
    }
}
```

---

# 10. JWT vs Session

| | JWT (Stateless) | Session (Stateful) |
|---|-----------------|----------------------|
| Storage | Client-side | Server-side (Redis) |
| Scale | Easy horizontal scale | Requires sticky session or shared store |
| Revocation | Hard (until expiry) | Easy (delete session) |
| Size | Larger headers | Small session cookie |
| CSRF | Less risk (no cookie) | Need CSRF token |

**Refresh tokens:** Short-lived access token (15 min) + long-lived refresh token (7 days) stored httpOnly.

---

# 11. Testing Security

```java
@WebMvcTest(OrderController.class)
@Import(SecurityConfig.class)
class OrderControllerSecurityTest {

    @Autowired MockMvc mockMvc;
    @MockBean OrderService orderService;

    @Test
    @WithMockUser(roles = "USER")
    void authenticatedUser_canAccess() throws Exception {
        mockMvc.perform(get("/api/v1/orders/1"))
               .andExpect(status().isOk());
    }

    @Test
    void unauthenticated_returns401() throws Exception {
        mockMvc.perform(get("/api/v1/orders/1"))
               .andExpect(status().isUnauthorized());
    }
}
```

---

# Interview Questions

## Q1. Where is JWT validated?

In `OncePerRequestFilter` (before `UsernamePasswordAuthenticationFilter`). Sets `SecurityContextHolder`.

## Q2. How to revoke JWT before expiry?

Token blacklist in Redis, short expiry + refresh token rotation, or version claim invalidated on password change.

## Q3. JWT stored where?

Prefer httpOnly Secure cookie (XSS-safe) or memory (SPA). Avoid localStorage if XSS risk.

## Q4. BCrypt vs SHA-256 for passwords?

BCrypt is slow by design (salted, adaptive cost). SHA-256 is fast — vulnerable to rainbow tables.

## Q5. @PreAuthorize vs @Secured?

`@PreAuthorize` uses SpEL (flexible: `@orderSecurity.isOwner(...)`). `@Secured` only role names.

---

# One-Line Revision

```text
JWT flow = login → signed token → Bearer header → JwtAuthFilter → SecurityContext; configure SecurityFilterChain + UserDetailsService + BCrypt + @PreAuthorize.
```

---

*End of Day 47 Spring Boot*
