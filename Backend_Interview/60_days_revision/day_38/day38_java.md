# Day 38 — Java KeyStore, Cipher & Security APIs

**Topics:** JCA · KeyStore · Cipher · TLS · Interview Questions

---

# 1. Java Cryptography Architecture (JCA)

```text
Application
    ↓
Cipher / MessageDigest / Signature  (API)
    ↓
Provider (SunJCE, BouncyCastle, etc.)
    ↓
Native / software implementation
```

Pluggable providers — swap algorithms without changing application code.

---

# 2. KeyStore

Stores cryptographic keys and certificates.

| Type | Format |
|------|--------|
| `JKS` | Legacy Java keystore |
| `PKCS12` | `.p12` / `.pfx` — industry standard |

```java
KeyStore ks = KeyStore.getInstance("PKCS12");
try (InputStream in = Files.newInputStream(Path.of("keystore.p12"))) {
    ks.load(in, "changeit".toCharArray());
}

PrivateKey key = (PrivateKey) ks.getKey("myalias", "keypass".toCharArray());
Certificate cert = ks.getCertificate("myalias");
```

---

# 3. Symmetric Encryption — Cipher

```java
public static byte[] encryptAesGcm(byte[] plaintext, SecretKey key) throws Exception {
    byte[] iv = new byte[12];
    SecureRandom.getInstanceStrong().nextBytes(iv);

    Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding");
    GCMParameterSpec spec = new GCMParameterSpec(128, iv);
    cipher.init(Cipher.ENCRYPT_MODE, key, spec);

    byte[] ciphertext = cipher.doFinal(plaintext);
    // prepend IV to ciphertext for storage
    byte[] result = new byte[iv.length + ciphertext.length];
    System.arraycopy(iv, 0, result, 0, iv.length);
    System.arraycopy(ciphertext, 0, result, iv.length, ciphertext.length);
    return result;
}
```

**Always use AES-GCM** (authenticated encryption) — not AES-ECB.

---

# 4. Key Generation

```java
KeyGenerator keyGen = KeyGenerator.getInstance("AES");
keyGen.init(256);
SecretKey secretKey = keyGen.generateKey();
```

```java
KeyPairGenerator kpg = KeyPairGenerator.getInstance("RSA");
kpg.initialize(2048);
KeyPair pair = kpg.generateKeyPair();
```

---

# 5. Asymmetric — RSA Encrypt/Decrypt

```java
Cipher cipher = Cipher.getInstance("RSA/ECB/OAEPWithSHA-256AndMGF1Padding");
cipher.init(Cipher.ENCRYPT_MODE, publicKey);
byte[] encrypted = cipher.doFinal(plaintext);

cipher.init(Cipher.DECRYPT_MODE, privateKey);
byte[] decrypted = cipher.doFinal(encrypted);
```

RSA for small payloads (keys); hybrid encryption for bulk data.

---

# 6. Digital Signatures

```java
Signature sig = Signature.getInstance("SHA256withRSA");
sig.initSign(privateKey);
sig.update(data);
byte[] signature = sig.sign();

sig.initVerify(publicKey);
sig.update(data);
boolean valid = sig.verify(signature);
```

---

# 7. Message Digest (Hashing)

```java
MessageDigest md = MessageDigest.getInstance("SHA-256");
byte[] hash = md.digest(password.getBytes(StandardCharsets.UTF_8));
```

**Never** store plain passwords. Use **bcrypt/Argon2** (Spring Security `BCryptPasswordEncoder`), not raw SHA-256 for passwords.

---

# 8. TLS / HTTPS

```properties
server.ssl.key-store=classpath:keystore.p12
server.ssl.key-store-password=${KEYSTORE_PASSWORD}
server.ssl.key-store-type=PKCS12
server.ssl.key-alias=tomcat
```

Programmatic:

```java
SSLContext ctx = SSLContext.getInstance("TLS");
ctx.init(keyManagerFactory.getKeyManagers(), trustManagerFactory.getTrustManagers(), null);
```

---

# 9. Secure Random

```java
SecureRandom random = SecureRandom.getInstanceStrong();
byte[] token = new byte[32];
random.nextBytes(token);
```

Avoid `java.util.Random` for security tokens.

---

# 10. Common Interview Pitfalls

| Mistake | Fix |
|---------|-----|
| ECB mode | Use GCM or CBC + HMAC |
| Hardcoded keys in source | KeyStore / env / HSM |
| Weak RSA 1024 | Use 2048+ |
| MD5/SHA1 for security | SHA-256+ |
| Logging secrets | Mask PII and tokens |

---

# Interview Questions

## Q1. KeyStore vs TrustStore?

KeyStore = **your** keys/certs (server identity). TrustStore = **trusted CA** certs to verify peers.

## Q2. Symmetric vs asymmetric?

Symmetric (AES) fast for bulk. Asymmetric (RSA/EC) for key exchange and signatures.

## Q3. What is salt in password hashing?

Random per-user value combined before hash — defeats rainbow tables. BCrypt embeds salt.

## Q4. AES key size?

128/192/256-bit. Prefer 256 for new systems.

## Q5. How Spring Boot loads SSL?

`server.ssl.*` properties → embedded Tomcat configures `SSLContext` from PKCS12.

---

# One-Line Revision

```text
PKCS12 KeyStore for keys; AES-GCM for encryption; SHA256withRSA for signatures; BCrypt for passwords.
```

---

*End of Day 38 Java*
