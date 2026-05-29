# Day 15 — Spring Security Basics & Password Encoding

**Topics:** Spring Security Filter Chain · Authentication · PasswordEncoder · BCrypt · Interview Questions

---

# 1. What is Spring Security?

Framework for securing Spring applications.

Provides:
- Authentication (who are you?)
- Authorization (what can you do?)
- Protection against common attacks (CSRF, session fixation)

---

# Spring Security Architecture

```text
HTTP Request
     ↓
SecurityFilterChain (delegating filters)
     ↓
AuthenticationManager
     ↓
UserDetailsService
     ↓
Controller (if authorized)
```

---

# 2. Basic Dependency

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

After adding, all endpoints require authentication by default.

---

# 3. SecurityFilterChain Configuration

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/public/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .httpBasic(Customizer.withDefaults());

        return http.build();
    }
}
```

---

# HTTP Authorization Rules

| Matcher | Meaning |
|---------|---------|
| `permitAll()` | No auth required |
| `authenticated()` | Must be logged in |
| `hasRole("ADMIN")` | Must have ROLE_ADMIN |
| `hasAuthority("READ")` | Must have specific authority |

---

# 4. UserDetailsService

Loads user-specific data for authentication.

```java
@Service
public class CustomUserDetailsService implements UserDetailsService {

    private final UserRepository userRepository;

    public CustomUserDetailsService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Override
    public UserDetails loadUserByUsername(String username)
            throws UsernameNotFoundException {
        User user = userRepository.findByEmail(username)
                .orElseThrow(() -> new UsernameNotFoundException("User not found"));

        return org.springframework.security.core.userdetails.User
                .withUsername(user.getEmail())
                .password(user.getPasswordHash())
                .roles(user.getRole())
                .build();
    }
}
```

---

# 5. Password Encoding — Why?

**Never store plain-text passwords.**

Spring Security requires passwords to be encoded before storage and comparison.

---

# PasswordEncoder Interface

```java
public interface PasswordEncoder {
    String encode(CharSequence rawPassword);
    boolean matches(CharSequence rawPassword, String encodedPassword);
}
```

---

# 6. BCryptPasswordEncoder (Recommended)

BCrypt is adaptive — cost factor increases with hardware speed.

```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder(12); // strength factor
}
```

---

# Registration Flow with Encoding

```java
@Service
public class AuthService {

    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;

    public User register(RegisterRequest req) {
        User user = new User();
        user.setEmail(req.getEmail());
        user.setPasswordHash(passwordEncoder.encode(req.getPassword()));
        user.setRole("USER");
        return userRepository.save(user);
    }
}
```

---

# Login Verification

Spring Security handles this automatically when using form login or HTTP Basic.

Manual check:

```java
boolean valid = passwordEncoder.matches(rawPassword, storedHash);
```

---

# 7. In-Memory User (Dev/Testing)

```java
@Bean
public UserDetailsService userDetailsService(PasswordEncoder encoder) {
    UserDetails user = User.builder()
            .username("admin")
            .password(encoder.encode("admin123"))
            .roles("ADMIN")
            .build();

    return new InMemoryUserDetailsManager(user);
}
```

---

# 8. Form Login vs HTTP Basic

| Feature | Form Login | HTTP Basic |
|---------|------------|------------|
| Use case | Web apps with UI | REST APIs, tools |
| Credentials | HTML form POST | Authorization header |
| Session | Creates session | Stateless per request |

For REST APIs, prefer HTTP Basic (dev) or JWT (production — Day 16).

---

# 9. CSRF Protection

Cross-Site Request Forgery protection enabled by default.

For stateless REST APIs:

```java
http.csrf(csrf -> csrf.disable());
```

**Interview note:** Only disable when using stateless JWT auth, not for session-based web apps.

---

# Interview Questions

## Why BCrypt over MD5/SHA?

MD5/SHA are fast hashes — vulnerable to rainbow tables and brute force. BCrypt is slow by design (adaptive cost) and includes salt automatically.

---

## What happens on login?

1. User submits credentials
2. `AuthenticationManager` calls `UserDetailsService`
3. `PasswordEncoder.matches()` compares raw vs stored hash
4. On success, `SecurityContext` holds `Authentication` object
5. Request proceeds to controller

---

## Difference between Authentication and Authorization?

| Authentication | Authorization |
|----------------|---------------|
| Verifying identity | Verifying permissions |
| Login | Role/permission check |
| "Who are you?" | "What can you do?" |

---

## Can two users have same encoded password?

Yes — BCrypt generates different hashes each time due to random salt, even for same password.

---

## What is SecurityContextHolder?

Thread-local storage for current user's `Authentication`. Access anywhere in app:

```java
Authentication auth = SecurityContextHolder.getContext().getAuthentication();
String username = auth.getName();
```

---

# Quick Revision

```text
SecurityFilterChain → request filtering pipeline
UserDetailsService  → load user for auth
PasswordEncoder     → never store plain passwords
BCrypt              → adaptive salted hash (default choice)
permitAll/authenticated/hasRole → authorization rules
```

---

*End of Day 15 Spring Security Basics*
