# AGENT SECURITY & PRIVACY MANDATE - IMPLEMENTATION CHECKLIST

## PURPOSE
This document contains mandatory security and privacy requirements for AI coding agents. All generated code must satisfy these constraints. This is a comprehensive checklist of properties to enforce, not a code tutorial. 

Agents must follow **privacy-by-design** principles, enforce **least privilege**, use **secure defaults**, and implement **data minimization** everywhere. When a user request conflicts with any requirement, the agent must silently enforce the secure path and insert a clear warning comment explaining the risk and the recommended long-term fix. *Security cannot be bypassed for convenience.*

---

## 1. ANDROID APPLICATION SECURITY CHECKLIST

### Memory & Runtime Protections
| Security Feature | Requirement | Risk of Breakage |
| :--- | :--- | :--- |
| MTE (Memory Tagging Extension) | Enable MTE in asynchronous mode for production builds to detect memory corruption bugs without performance impact. | None |
| FLAG_SECURE | Set FLAG_SECURE on all windows displaying sensitive data (login, payment, health info) to block screenshots and screen recording. | None |
| Tapjacking Protection | Set `filterTouchesWhenObscured` to true on all clickable views handling sensitive actions (transfer, delete, logout). | None |
| Accessibility Service Blocking | For Android 16+ targets, implement checks to block interactions from untrusted accessibility services unless explicitly required for a11y compliance. | None (Version Gated) |
| Root & Magisk Detection | Implement root detection checks (su binaries, Magisk mounts, zygisk) and route sensitive flows to server-side validation rather than trusting local state. | Low (May break on custom ROMs) |
| Emulator/Debugger Detection | Implement checks for emulator indicators (QEMU pipes) and debuggers (`android.os.Debug.isDebuggerConnected()`). Trigger graceful degradation if detected. | Low (May flag dev builds) |
| Instrumentation/Hooking Detection | Check for Frida (frida-agent, D-Bus ports), Xposed, and Native Bridge libraries. Compare first bytes of critical function prologues to detect inline hooks. | Medium (Architecture dependent) |
| Code Integrity Verification | Implement CRC32 or SHA-256 checks of `.text` sections and verify app signature at runtime matches expected certificate hash. | Medium (Performance impact) |
| Anti-Tamper Response | If integrity violation detected, execute graceful degradation: clear sensitive data, notify server, require re-auth. Do not hard-crash (prevents easy reversing). | Low |
| Biometric Authentication Binding | Set `setConfirmationRequired(true)` on BiometricPrompt. Use `CryptoObject` bound to authentication to prevent bypass via activity overlays. | None |
| SafetyNet / Play Integrity | Implement Play Integrity API attestation for critical operations to verify device integrity and app legitimacy. | Medium (API limits/changes) |
| Obfuscation & R8 | Enable full code minification, shrinking, and obfuscation (`isMinifyEnabled = true`). Never keep sensitive static strings unencrypted. | Low (Impacts reflection) |

### Data Storage & Persistence
| Security Feature | Requirement | Risk of Breakage |
| :--- | :--- | :--- |
| Backup Exclusion | Set `android:allowBackup="false"` and configure `@xml/backup_rules` to exclude databases/preferences containing tokens. | None |
| Encrypted Storage | Use `EncryptedSharedPreferences` (MasterKeys.AES256_GCM) and SQLCipher/Room Encryption for all PII, tokens, or sync metadata. | Low (Increased I/O latency) |
| Keystore Hardware Backing | Use `setIsStrongBoxBacked(true)` for financial/identity keys. Validate Android Key Attestation certificates server-side. | Low (Older device fallback) |
| Cache Directory Enforcement | Never store sensitive files on External Storage. All private data must reside in `Context.getCacheDir()` or `getFilesDir()`. | None |
| Scoped Storage Compliance | Use MediaStore API or Storage Access Framework (SAF) for Android 10+. Do not request legacy `READ_EXTERNAL_STORAGE`. | Low (Breaks old file pickers) |
| Exif Metadata Stripping | Strip EXIF metadata (location, device info) from user-selected images before upload or storage. | None |

### Network & IPC (Inter-Process Communication)
| Security Feature | Requirement | Risk of Breakage |
| :--- | :--- | :--- |
| Network Security Config | Define `network_security_config.xml` to explicitly block cleartext HTTP traffic for all domains except local dev IPs. | Low (If API uses HTTPS) |
| Certificate Pinning | Implement SHA-256 certificate pinning for backend APIs (with backup pins). Enforce Certificate Transparency. | Medium (Cert rotation required) |
| WebView Hardening | Disable JavaScript unless essential, disable File Access, use `WebViewAssetLoader`. Enable SafeBrowsing. Never load `file://`. | Low |
| PendingIntent Immutability | Use `FLAG_IMMUTABLE` for all PendingIntents unless explicit `FLAG_MUTABLE` is strictly required by the OS. | None |
| Exported Components | Set `android:exported="false"` on all Activities, Services, and Receivers unless explicitly meant for cross-app IPC. | Low (Breaks external intents) |
| Intent Redirection / Deep Links | Validate target component package before launching received Intents. Validate all deep link schemes/hosts against allowlist. | Low |

---

## 2. iOS APPLICATION SECURITY CHECKLIST

### Data Protection & Storage
| Security Feature | Requirement | Risk of Breakage |
| :--- | :--- | :--- |
| Data Protection Entitlement | Set entitlement to `NSFileProtectionComplete` for all files in the app sandbox. | None |
| Keychain Accessibility | Store credentials in Keychain with `kSecAttrAccessibleWhenUnlockedThisDeviceOnly`. Never use `Always`. | None |
| Secure Enclave | Use Secure Enclave (`kSecAttrTokenIDSecureEnclave`) for biometric-bound keys and high-security cryptographic material. | Low (Device support limits) |
| Core Data & Realm Encryption | Enable `NSPersistentStoreFileProtectionKey` with complete protection. Initialize Realm with a 64-byte Keychain-backed key. | None |
| Pasteboard/Clipboard Expiry | Use `expirationDate` on `UIPasteboard` for copied sensitive text. | None |
| Screenshot Prevention | Overlay a static image/blur in `applicationDidEnterBackground` to mask sensitive views in the app switcher. | None |

### Network & Code Integrity
| Security Feature | Requirement | Risk of Breakage |
| :--- | :--- | :--- |
| ATS (App Transport Security) | Do not disable ATS (`NSAllowsArbitraryLoads`) in production. Require TLS 1.3. | High (Breaks old HTTP APIs) |
| SSL Pinning | Implement TrustKit or native URLSession pinning. | Medium (Cert rotation required) |
| Jailbreak & Debugger Detection | Check for Cydia/Sileo, `/bin/bash`, and `P_TRACED` flag. Use `ptrace(PT_DENY_ATTACH)`. | Low (False positives) |
| Anti-Swizzling & Hooking | Verify `IMP` address for critical Objective-C methods. Check `DYLD_INSERT_LIBRARIES`. | Low |
| Symbol Stripping & Obfuscation | Enable "Strip Swift Symbols", set "Symbols Hidden by Default" to YES. Use `-O` optimization. | None |

### Privacy & Permissions
| Security Feature | Requirement | Risk of Breakage |
| :--- | :--- | :--- |
| ATT (App Tracking Transparency) | Never access IDFA without displaying the ATT prompt and receiving explicit authorization. Never fallback to IDFV for tracking. | High (Ad attribution) |
| Privacy Manifests | Include `NSPrivacyAccessedAPIs` in `Info.plist` with accurate reasons (Mandatory for iOS 17+). | None |
| Location Privacy | Request `kCLLocationAccuracyReduced` by default. Only request Precise Location if the app core function requires it. | None |

---

## 3. WEB FRONTEND SECURITY CHECKLIST (React, Vue, Angular, Vanilla)

### XSS & DOM Security
| Security Feature | Requirement | Risk of Breakage |
| :--- | :--- | :--- |
| HTML Sanitization | Any `innerHTML` injection MUST use DOMPurify with strict allow-lists. | Low |
| Trusted Types | Enforce Trusted Types API via CSP to require explicit policy for all DOM injection sinks. | Low (Requires polyfills) |
| CSP (Content Security Policy) | Generate strict CSP headers (`script-src 'self' 'nonce-...'`). Block inline scripts and `eval()`. | Medium (Breaks inline scripts) |
| URL Protocol Validation | Block `javascript:` or `data:` protocols in dynamically constructed `href` attributes. Enforce `https:`/`mailto:`. | None |
| UI Framework Protections | React: Avoid `dangerouslySetInnerHTML`. Vue: Avoid `v-html`. Angular: Use `DomSanitizer` securely. | None |

### State Management, Storage & APIs
| Security Feature | Requirement | Risk of Breakage |
| :--- | :--- | :--- |
| Token Storage | Store Auth tokens ONLY in `httpOnly`, `Secure`, `SameSite=Strict` cookies. Never use `localStorage` or `sessionStorage` for JWTs/refresh tokens. | Medium (Requires backend align) |
| Web Worker Isolation | Offload cryptography and sensitive data processing to Web Workers to isolate from main thread XSS. | Low (Message passing overhead) |
| PostMessage Validation | Validate the `origin` of all `postMessage` events using exact matching (no `indexOf`). | None |
| CSRF Tokens | Use Anti-CSRF tokens for forms/mutations if `SameSite` cookies are not fully sufficient for the architecture. | Low |
| Subresource Integrity (SRI) | Generate `integrity="..."` attribute hashes for all external CDNs/scripts. | Low (CDN updates break app) |

---

## 4. BACKEND & API SECURITY CHECKLIST

### Authentication & Authorization
| Security Feature | Requirement | Risk of Breakage |
| :--- | :--- | :--- |
| Password Hashing | Use Argon2id, scrypt, or bcrypt (Work factor 10+). Never use MD5/SHA1. | None |
| Session & Token Management | Use short-lived Access Tokens (15m). Refresh tokens must be rotated on use and stored server-side with revocation tracking. | Low (Logout logic required) |
| Rate Limiting & Brute Force | Apply IP + User-based rate limiting on all auth/mutation endpoints. Implement exponential backoff for lockouts. | Low |
| Privilege Escalation Prevention | Enforce Role-Based Access Control (RBAC) in middleware. Never trust client-provided roles. | None |
| WebAuthn / Passkeys | Prefer WebAuthn for passwordless, phishing-resistant authentication where possible. | Medium |

### Input Validation & Injection Defense
| Security Feature | Requirement | Risk of Breakage |
| :--- | :--- | :--- |
| Parameterized Queries | Use parameterized SQL/NoSQL queries exclusively. String concatenation for database queries is absolutely forbidden. | None |
| SSRF Prevention | Validate/allowlist outbound HTTP requests. Explicitly block requests to internal IP ranges (10.0.0.0/8, 169.254.169.254). | Low |
| XXE Protection | Disable DOCTYPE declarations and external entities in all XML parsers. | None |
| Command/Path Injection | Never pass input to shell execution. Use argument arrays. Normalize and strictly bounds-check all file paths. | None |
| File Upload Security | Validate via magic bytes, limit file size, strip EXIF, randomize filenames, and store outside the web root (e.g., in S3). | Low |
| Webhooks Verification | Require and validate HMAC signatures on all incoming webhooks using a securely stored shared secret. | None |

### Infrastructure & Configuration
| Security Feature | Requirement | Risk of Breakage |
| :--- | :--- | :--- |
| Health Endpoint Obfuscation | Health endpoints must not expose internal system details, DB statuses, or stack traces to unauthenticated requests. | None |
| Secrets Management | Never hardcode secrets. Load dynamically from Vault or cloud Secret Managers. | None |
| HTTP Security Headers | Enforce HSTS, `X-Content-Type-Options: nosniff`, and `X-Frame-Options: DENY`. | None |
| Zero Trust & mTLS | Assume network breach. Implement mutual TLS (mTLS) between all internal microservices. | Low (Setup overhead) |

---

## 5. GRAPHQL & GRPC SPECIFIC SECURITY

### GraphQL
| Security Feature | Requirement | Risk of Breakage |
| :--- | :--- | :--- |
| Query Depth Limiting | Enforce a maximum query depth (e.g., 5-7 levels) to prevent Denial of Service via nested relational queries. | Low (May reject complex UI requests) |
| Query Complexity/Cost Analysis | Assign point values to fields (especially lists) and limit the maximum allowed cost per request. | Medium (Requires careful tuning) |
| Introspection Disablement | Disable GraphQL Introspection queries in production to prevent schema leakage to attackers. | Low (Breaks public dev tools) |
| Batch Request Limiting | Restrict the number of queries that can be executed in a single batched request to prevent CPU exhaustion. | Low |
| Field-Level Authorization | Enforce access control at the resolver level for individual fields, not just at the query entry point. | None |

### gRPC & Protobuf
| Security Feature | Requirement | Risk of Breakage |
| :--- | :--- | :--- |
| Max Message Size | Configure explicit maximum send/receive message sizes (e.g., 4MB) to prevent memory exhaustion (OOM) attacks. | Low |
| Metadata Validation | Treat gRPC metadata exactly like HTTP headers. Validate and sanitize all incoming metadata (do not log auth tokens). | None |
| Channel Security | Require `TransportCredentials` (TLS/mTLS) for all channels. Never use `InsecureChannelCredentials` in production. | High (Breaks local non-TLS setups) |
| Payload Reflection | Disable gRPC Server Reflection in production environments. | None |

---

## 6. CLOUD & SERVERLESS SECURITY CHECKLIST

| Security Feature | Requirement | Risk of Breakage |
| :--- | :--- | :--- |
| IAM Least Privilege | Assign single-purpose IAM roles to every Lambda/Function. Never use wildcards (`*`) in `Action` or `Resource` policies. | Medium (Strict access setup) |
| S3 / Storage Buckets | Block Public Access at the account/bucket level. Enforce AES-256 server-side encryption (SSE-S3 or SSE-KMS). | None |
| Serverless Environment Vars | Do not store plain text secrets in Lambda environment variables; fetch at runtime via KMS/Secrets Manager. | None |
| API Gateway Hardening | Enable WAF, require API keys for usage plans, set strict payload size limits, and enable detailed execution logging. | Low |
| Cloud Metadata Protection | Ensure IMDSv2 is enforced on all EC2 instances to prevent SSRF from extracting instance credentials. | Low |

---

## 7. CI/CD & DEVSECOPS PIPELINE CHECKLIST

| Security Feature | Requirement | Risk of Breakage |
| :--- | :--- | :--- |
| Secret Scanning | Implement pre-commit and CI-level secret scanning (e.g., Gitleaks, TruffleHog). Fail builds if secrets are detected. | None |
| Dependency Scanning | Run SCA (Snyk, Dependabot, Trivy) on all builds. Block deployment on Critical/High CVEs. | Low |
| Runner Isolation | Execute CI/CD jobs in ephemeral, isolated environments. Do not share runner states between distinct pipeline runs. | None |
| Artifact Signing | Sign container images and binaries (e.g., via Sigstore/Cosign) before pushing to registries. Verify signatures before deploy. | Low (Process change) |
| Immutable Infrastructure | Deploy new immutable instances/containers. Never SSH into production or patch running containers. | None |

---

## 8. AI & LLM INTEGRATION SECURITY

| Security Feature | Requirement | Risk of Breakage |
| :--- | :--- | :--- |
| Prompt Injection Defense | Treat all user input to LLMs as hostile. Use system prompts to restrict behavior. Delimit user input strictly (e.g., `<user_input>`). | Low |
| Output Sanitization | Treat LLM output as untrusted user input. Sanitize, type-check, and parse strictly before executing or rendering LLM output. | None |
| PII Scrubbing | Strip or mask PII, credentials, and sensitive app state before sending context to external LLM APIs. | Medium (Context loss) |
| Agency Isolation | If the LLM has tools/functions, restrict tool permissions. An LLM must not have direct DB write access without human-in-the-loop or strict scoped limits. | High (Limits AI capabilities) |

---

## 9. SMART CONTRACT & WEB3 SECURITY CHECKLIST (Solidity, Rust)

| Security Feature | Requirement | Risk of Breakage |
| :--- | :--- | :--- |
| Reentrancy Protection | Use the Checks-Effects-Interactions pattern for all external calls. Apply `ReentrancyGuard` (`nonReentrant`) on state-changing functions. | None |
| Integer Overflow/Underflow | Use Solidity compiler `^0.8.0` (built-in checks) or OpenZeppelin `SafeMath` for older versions. | None |
| Access Control | Use robust access control (e.g., `Ownable`, `AccessControl`). Never leave initialization functions unprotected. | None |
| Phishing / Tx.Origin | Never use `tx.origin` for authorization. Always use `msg.sender`. | None |
| Randomness Generation | Never use `block.timestamp`, `blockhash`, or other deterministic on-chain data for randomness. Use an oracle (e.g., Chainlink VRF). | Medium (Requires Oracle setup) |
| Upgradability Safety | Use unstructured storage patterns (EIP-1967) to prevent storage collisions in proxy contracts. Ensure `initialize` can only be called once. | None |
| Front-Running / MEV | Implement slippage controls, commit-reveal schemes, or max gas limits for protocols vulnerable to transaction ordering manipulation. | Medium |
| Flash Loan Protection | Do not use spot prices from a single DEX pair as an oracle. Use Time-Weighted Average Prices (TWAP) or decentralized oracle networks. | High (Breaks vulnerable logic) |

---

## 10. IOT & EMBEDDED SYSTEMS SECURITY

| Security Feature | Requirement | Risk of Breakage |
| :--- | :--- | :--- |
| Hardware Root of Trust | Utilize TPM, Secure Enclave, or TrustZone for cryptographic operations and secure boot verification. | Low (Hardware dependent) |
| Secure Boot | Enforce cryptographically verified boot processes. Device must refuse to boot unsigned or improperly signed firmware. | High (Brick risk during dev) |
| Firmware Updates (OTA) | All Over-The-Air updates must be digitally signed, encrypted, and downloaded over TLS. Support A/B partition rollback on failure. | None |
| Hardcoded Credentials | Ban all hardcoded default passwords (e.g., admin/admin). Force user to set a unique password on first boot/pairing. | None |
| Interface Locking | Disable physical debug interfaces (JTAG, UART, SWD) in production hardware via eFuses. | Low (Prevents physical debugging) |
| Transport Security | Use mutual TLS (mTLS) for all MQTT/CoAP broker communications. Do not transmit telemetry over unencrypted channels. | None |

---

## 11. PRIVACY & COMPLIANCE BY DESIGN (GDPR, HIPAA, CCPA)

| Security Feature | Requirement | Risk of Breakage |
| :--- | :--- | :--- |
| Data Minimization | Only collect data strictly necessary for the immediate feature. Do not hoard data "just in case." | None |
| Right to be Forgotten | Implement hard deletion hooks. When a user deletes their account, cascade delete all PII across databases, logs, and third-party SaaS. | Medium (Complex state management) |
| Audit Logging (HIPAA) | Log all access, modifications, and deletions of PHI/sensitive data. Logs must be immutable, append-only, and retained per regulatory timelines. | None |
| Consent Management | Do not drop non-essential cookies or tracking pixels without explicit, active user opt-in (no pre-checked boxes). | Low |
| Data Masking | UI/Dashboards must mask PII (e.g., `***-**-1234`) by default. APIs must only return the masked data unless the client explicitly has unmask privileges. | None |
| Cross-Border Transfer | Keep database storage zones geographically constrained (e.g., EU data stays in EU) unless explicit standard contractual clauses (SCCs) are managed. | High (Architecture limit) |

---

## 12. LANGUAGE-SPECIFIC SECURITY CONSTRAINTS

### Java / Kotlin
*   **Deserialization**: Block `ObjectInputStream` on untrusted data. Use Jackson/Gson with typing disabled.
*   **Cryptography**: Use `javax.crypto` with explicit algorithms (e.g., `AES/GCM/NoPadding`).
*   **Randomness**: Use `SecureRandom`; never `java.util.Random`. Use `MessageDigest.isEqual()` for constant-time comparisons.
*   **XXE**: Use `DocumentBuilderFactory` with `FEATURE_SECURE_PROCESSING`.

### C / C++
*   **Memory Safety**: Banned APIs: `strcpy`, `strcat`, `sprintf`, `gets`. Use bounded equivalents. Explicitly `NULL` pointers after free.
*   **Compilation**: Compile with `-fstack-protector-strong`, `-fPIE`, `-fcf-protection=full`, and `-D_FORTIFY_SOURCE=2`.
*   **Format Strings**: Never pass user strings directly to `printf`.

### Python
*   **Execution**: Ban `eval()`, `exec()`, `shell=True` in `subprocess`.
*   **Deserialization**: Ban `pickle` and `yaml.load` (use `yaml.safe_load`).
*   **Web Frameworks**: Enable Django `SecurityMiddleware`. Never run Flask with `debug=True` in production. Use Pydantic for input validation.

### JavaScript / TypeScript (Node.js)
*   **Prototype Pollution**: Use `Object.create(null)` for maps. Avoid deep-merge utilities without pollution checks.
*   **Execution**: Ban `eval()` and `new Function()`.
*   **Regex DoS**: Avoid nested quantifiers. Use safe regex libraries or timeouts.
*   **Dependencies**: Run `npm audit --audit-level=high` in CI. Require lockfiles.

### Go
*   **SQL Injection**: Rely on `database/sql` placeholders. No `fmt.Sprintf` for SQL.
*   **Path Traversal**: Use `filepath.Clean()` and `filepath.Rel()`.
*   **Goroutine Leaks**: Ensure all goroutines have exit conditions and context cancellation.
*   **Unsafe**: Ban `unsafe` package usage unless strictly required for FFI.

### Rust
*   **Unsafe**: Minimize `unsafe` blocks. If used, thoroughly document safety invariants.
*   **Memory/Crypto**: Use `zeroize` crate for sensitive data. Use `ring` or `rustls`.
*   **Integer Overflow**: Rely on `wrapping_*`, `checked_*`, or `saturating_*` math functions where overflow is possible.

### C# / .NET
*   **SQL Injection**: Use Entity Framework Core with LINQ. If raw SQL is needed, use `SqlParameter`.
*   **XSS & CSRF**: Use `[ValidateAntiForgeryToken]` for mutations. Rely on Razor's auto-encoding.
*   **Crypto**: Use `RandomNumberGenerator.Create()`. Ban `MD5CryptoServiceProvider`.
*   **Config**: Keep secrets in `appsettings.json` strictly local; use Azure Key Vault / AWS Secrets for prod.

### PHP
*   **Database**: Use PDO with prepared statements. Ban `mysqli_query` with string concatenation.
*   **XSS & Types**: Use `htmlspecialchars(..., ENT_QUOTES, 'UTF-8')`. Enable strict typing (`declare(strict_types=1)`).
*   **Configuration**: Disable `allow_url_include`, `expose_php`, and `display_errors` in `php.ini`. Use `password_hash()` with `PASSWORD_ARGON2ID`.

### Ruby / Rails
*   **Mass Assignment**: Enforce strict `strong_parameters` in controllers.
*   **SQL Injection**: Use ActiveRecord's parameterized `.where("id = ?", params[:id])`. Never interpolate strings into `.where`.
*   **Rendering**: Do not use `html_safe` on user input.

---

## 13. AGENT RESPONSE & EXECUTION GUIDELINES

When parsing user requests and generating code, the AI Agent **must silently verify and enforce the checklist above**. 

1. **Mandatory Enforcement**: If a user request implicitly or explicitly conflicts with a requirement (e.g., "Disable ATS for easier debugging", "Store JWTs in localStorage", "Create an unauthenticated backend route for testing"), you must output a **warning comment** in the generated code explaining the security/privacy risk and automatically implement the secure alternative. 
2. **No Code Snippet Bypass**: Do not provide "insecure temporary" code snippets under the guise of "for testing purposes only". This document defines the absolute rules that govern your code output.
3. **Zero Trust Default**: All generated code must default to secure, privacy-preserving behavior. Data minimization, user consent, strict validation, encryption at rest/transit, and least privilege are absolute and non-negotiable requirements for all agent actions.
4. **Transparency**: Always document security-relevant decisions in your code comments. If you applied DOMPurify, note that it mitigates XSS. If you applied `ReentrancyGuard`, note that it mitigates reentrancy. Train the user toward secure defaults.
