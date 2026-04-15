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
| Native Library Hardening | Compile all native libraries with `-fstack-protector-strong`, `-fPIC`, `-Wl,-z,relro,-z,now`, and strip symbols. Enable `FORTIFY_SOURCE=2`. | None |
| WebView JavaScript Interface | Do not expose sensitive Java methods via `addJavascriptInterface` unless absolutely necessary and with strict input validation. | Low |
| Intent Filter Protection | Ensure that `intent-filter` with `BROWSABLE` category validates the `host` and `path` to prevent unauthorized deep link hijacking. | Low |
| Clipboard Monitoring | Periodically clear the clipboard if it contains sensitive data and warn users when pasting into untrusted contexts. | None |
| Screen Pinning | Encourage or enforce screen pinning for high-security flows (e.g., financial transactions). | None |
| Runtime Permission Validation | Re-check permissions before performing sensitive operations even if granted at install, as they can be revoked while app is in background. | None |

### Data Storage & Persistence
| Security Feature | Requirement | Risk of Breakage |
| :--- | :--- | :--- |
| Backup Exclusion | Set `android:allowBackup="false"` and configure `@xml/backup_rules` to exclude databases/preferences containing tokens. | None |
| Encrypted Storage | Use `EncryptedSharedPreferences` (MasterKeys.AES256_GCM) and SQLCipher/Room Encryption for all PII, tokens, or sync metadata. | Low (Increased I/O latency) |
| Keystore Hardware Backing | Use `setIsStrongBoxBacked(true)` for financial/identity keys. Validate Android Key Attestation certificates server-side. | Low (Older device fallback) |
| Cache Directory Enforcement | Never store sensitive files on External Storage. All private data must reside in `Context.getCacheDir()` or `getFilesDir()`. | None |
| Scoped Storage Compliance | Use MediaStore API or Storage Access Framework (SAF) for Android 10+. Do not request legacy `READ_EXTERNAL_STORAGE`. | Low (Breaks old file pickers) |
| Exif Metadata Stripping | Strip EXIF metadata (location, device info) from user-selected images before upload or storage. | None |
| Secure File Deletion | Overwrite files with random data before deletion, or use `File.delete()` followed by `StorageManager` secure delete if available. | Low |
| Database Query Hardening | Use parameterized queries with Room or SQLite. Avoid rawQuery with concatenation; validate all input lengths and types. | None |
| Content Provider Security | Set `android:exported="false"` for internal providers. For exported providers, enforce strict permissions and validate `_id` and selection args. | Low |

### Network & IPC (Inter-Process Communication)
| Security Feature | Requirement | Risk of Breakage |
| :--- | :--- | :--- |
| Network Security Config | Define `network_security_config.xml` to explicitly block cleartext HTTP traffic for all domains except local dev IPs. | Low (If API uses HTTPS) |
| Certificate Pinning | Implement SHA-256 certificate pinning for backend APIs (with backup pins). Enforce Certificate Transparency. | Medium (Cert rotation required) |
| WebView Hardening | Disable JavaScript unless essential, disable File Access, use `WebViewAssetLoader`. Enable SafeBrowsing. Never load `file://`. | Low |
| PendingIntent Immutability | Use `FLAG_IMMUTABLE` for all PendingIntents unless explicit `FLAG_MUTABLE` is strictly required by the OS. | None |
| Exported Components | Set `android:exported="false"` on all Activities, Services, and Receivers unless explicitly meant for cross-app IPC. | Low (Breaks external intents) |
| Intent Redirection / Deep Links | Validate target component package before launching received Intents. Validate all deep link schemes/hosts against allowlist. | Low |
| Network Timeouts & Retries | Set connect/read/write timeouts (e.g., 10s/30s/30s). Implement exponential backoff with jitter. Never retry non-idempotent requests. | Low |
| WebSocket Security | Use `wss://` exclusively. Validate Origin header server-side. Implement ping/pong to detect dead connections. | None |
| mTLS for Backend Services | Where possible, implement mutual TLS using client certificates stored in Keystore. | Medium |
| DNS over HTTPS | Use DoH to prevent DNS spoofing and monitoring. | Low |
| GraphQL Client Security | Validate response shape, limit query depth client-side, and never send sensitive operations in GET requests. | Low |

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
| UserDefaults Protection | Never store sensitive data in UserDefaults. Use Keychain instead. | None |
| File Protection for Downloads | Mark downloaded files with `NSFileProtectionComplete` unless they must be accessed in background. | None |
| SQLite Encryption | Use SQLCipher with a Keychain-stored key for all local SQLite databases. | Low |
| App Group Security | Apply file protection attributes manually to all files written to shared containers. | None |

### Network & Code Integrity
| Security Feature | Requirement | Risk of Breakage |
| :--- | :--- | :--- |
| ATS (App Transport Security) | Do not disable ATS (`NSAllowsArbitraryLoads`) in production. Require TLS 1.3. | High (Breaks old HTTP APIs) |
| SSL Pinning | Implement TrustKit or native URLSession pinning. | Medium (Cert rotation required) |
| Jailbreak & Debugger Detection | Check for Cydia/Sileo, `/bin/bash`, and `P_TRACED` flag. Use `ptrace(PT_DENY_ATTACH)`. | Low (False positives) |
| Anti-Swizzling & Hooking | Verify `IMP` address for critical Objective-C methods. Check `DYLD_INSERT_LIBRARIES`. | Low |
| Symbol Stripping & Obfuscation | Enable "Strip Swift Symbols", set "Symbols Hidden by Default" to YES. Use `-O` optimization. | None |
| Binary Protection | Compile with `-fobjc-arc`, `-fstack-protector-strong`, and `-Wl,-dead_strip` to remove unused code. | None |
| Certificate Transparency | Enforce CT validation for all TLS connections. | Low |
| Network Extension Security | If using VPN/Network Extensions, validate all incoming data and enforce strict memory protections. | Medium |

### Privacy & Permissions
| Security Feature | Requirement | Risk of Breakage |
| :--- | :--- | :--- |
| ATT (App Tracking Transparency) | Never access IDFA without displaying the ATT prompt and receiving explicit authorization. Never fallback to IDFV for tracking. | High (Ad attribution) |
| Privacy Manifests | Include `NSPrivacyAccessedAPIs` in `Info.plist` with accurate reasons (Mandatory for iOS 17+). | None |
| Location Privacy | Request `kCLLocationAccuracyReduced` by default. Only request Precise Location if the app core function requires it. | None |
| Photo Library Access | Use `PHPickerViewController` to avoid full photo library access. | None |
| Microphone & Camera | Request permission only at point of use; provide clear purpose strings. | None |
| Calendar/Contacts Access | Never request access unless core functionality; scrub any stored data when no longer needed. | None |
| Background Tasks | Use `BGTaskScheduler` only for essential background work; do not use for tracking. | None |

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
| JSON Injection | Never embed untrusted JSON inside `<script>` tags. Use `JSON.stringify` with escaping. | None |
| CSS Injection | Do not concatenate user input into `<style>` tags. Use `CSS.escape()` for dynamic class names. | None |
| Iframe Sandboxing | Use `sandbox` attribute with minimal permissions. Never allow `allow-same-origin` without `allow-scripts`. | Low |
| Web Worker Origin | Ensure workers are loaded from same origin and do not have access to DOM. | None |

### State Management, Storage & APIs
| Security Feature | Requirement | Risk of Breakage |
| :--- | :--- | :--- |
| Token Storage | Store Auth tokens ONLY in `httpOnly`, `Secure`, `SameSite=Strict` cookies. Never use `localStorage` or `sessionStorage` for JWTs/refresh tokens. | Medium (Requires backend align) |
| Web Worker Isolation | Offload cryptography and sensitive data processing to Web Workers to isolate from main thread XSS. | Low (Message passing overhead) |
| PostMessage Validation | Validate the `origin` of all `postMessage` events using exact matching (no `indexOf`). | None |
| CSRF Tokens | Use Anti-CSRF tokens for forms/mutations if `SameSite` cookies are not fully sufficient for the architecture. | Low |
| Subresource Integrity (SRI) | Generate `integrity="..."` attribute hashes for all external CDNs/scripts. | Low (CDN updates break app) |
| Cache API Security | Do not store authenticated responses or PII in Cache API. | None |
| IndexedDB Encryption | Encrypt sensitive data before storing in IndexedDB using WebCrypto with non-extractable keys. | Low |
| Push Notification Privacy | Do not include sensitive data in push payloads. Use silent pushes to trigger secure fetches. | None |
| Service Worker Security | Ensure service workers do not cache sensitive pages; validate fetch events and prevent request smuggling. | Low |

### Browser Features & Headers
| Security Feature | Requirement | Risk of Breakage |
| :--- | :--- | :--- |
| HTTPS Enforcement | Redirect all HTTP to HTTPS. Use `Strict-Transport-Security` with `includeSubDomains` and `max-age=31536000`. | None |
| Permissions Policy | Set restrictive `Permissions-Policy` header to disable camera, microphone, geolocation unless needed. | Low |
| Referrer Policy | Use `Referrer-Policy: strict-origin-when-cross-origin`. | None |
| X-Content-Type-Options | Set `X-Content-Type-Options: nosniff`. | None |
| X-Frame-Options | Set `X-Frame-Options: DENY` or `SAMEORIGIN` as appropriate. | Low (Breaks embedding) |
| Cross-Origin Opener Policy (COOP) | Set `Cross-Origin-Opener-Policy: same-origin` for process isolation. | Low |
| Cross-Origin Embedder Policy (COEP) | Set `Cross-Origin-Embedder-Policy: require-corp` if using SharedArrayBuffer. | Low |

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
| Multi-Factor Authentication (MFA) | Require MFA for sensitive actions (password change, email update, high-value transactions). | Low |
| OAuth 2.1 Compliance | Use PKCE for all OAuth flows, even confidential clients. Validate state parameter. | None |
| API Key Rotation | Support automated rotation; accept both old and new keys during grace period. | Low |
| Session Fixation Protection | Regenerate session ID after login and privilege change. | None |
| Account Recovery Security | Use time-limited, single-use recovery tokens sent to verified email/phone. | None |

### Input Validation & Injection Defense
| Security Feature | Requirement | Risk of Breakage |
| :--- | :--- | :--- |
| Parameterized Queries | Use parameterized SQL/NoSQL queries exclusively. String concatenation for database queries is absolutely forbidden. | None |
| SSRF Prevention | Validate/allowlist outbound HTTP requests. Explicitly block requests to internal IP ranges (10.0.0.0/8, 169.254.169.254). | Low |
| XXE Protection | Disable DOCTYPE declarations and external entities in all XML parsers. | None |
| Command/Path Injection | Never pass input to shell execution. Use argument arrays. Normalize and strictly bounds-check all file paths. | None |
| File Upload Security | Validate via magic bytes, limit file size, strip EXIF, randomize filenames, and store outside the web root (e.g., in S3). | Low |
| Webhooks Verification | Require and validate HMAC signatures on all incoming webhooks using a securely stored shared secret. | None |
| GraphQL Injection Prevention | Use static query whitelisting or persisted queries. Validate all variables against schema. | Low |
| NoSQL Injection | Use typed operators (e.g., `$eq`) and validate that user input is not an object with operator keys. | None |
| LDAP Injection | Use parameterized LDAP filters; escape special characters. | None |
| Email Header Injection | Strip `\r` and `\n` from all user-supplied email addresses. | None |
| Deserialization Safety | Never deserialize untrusted data with `ObjectInputStream`, `pickle`, `BinaryFormatter`, etc. Use JSON with strict schemas. | None |

### Infrastructure & Configuration
| Security Feature | Requirement | Risk of Breakage |
| :--- | :--- | :--- |
| Health Endpoint Obfuscation | Health endpoints must not expose internal system details, DB statuses, or stack traces to unauthenticated requests. | None |
| Secrets Management | Never hardcode secrets. Load dynamically from Vault or cloud Secret Managers. | None |
| HTTP Security Headers | Enforce HSTS, `X-Content-Type-Options: nosniff`, and `X-Frame-Options: DENY`. | None |
| Zero Trust & mTLS | Assume network breach. Implement mutual TLS (mTLS) between all internal microservices. | Low (Setup overhead) |
| Container Security | Run as non-root user, read-only root filesystem, drop all capabilities, use distroless base images. | Low |
| Dependency Scanning | Integrate SCA (Snyk, Trivy) in CI/CD; block builds on Critical/High CVEs. | Low |
| Immutable Infrastructure | Deploy new versions as fresh instances; never patch running containers. | None |
| Resource Limits | Set CPU/memory limits, implement circuit breakers, and graceful shutdown (30s). | None |
| Audit Logging | Log all security events in structured JSON with `user_id_hash`, `ip_hash`, `outcome`. Ensure logs are append-only. | None |
| Error Handling | Never expose stack traces or internal details to clients. Return generic error messages. | None |
| Database Encryption | Enable TDE or disk-level encryption. Encrypt sensitive columns at application layer. | Low |

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
| Persisted Queries | Use persisted queries (allow-list) to prevent arbitrary query execution. | Medium |
| Error Message Redaction | Mask internal error details in GraphQL error responses. Return only a generic error message to clients. | None |
| Alias Limiting | Limit the number of aliases per query to prevent resource exhaustion. | Low |

### gRPC & Protobuf
| Security Feature | Requirement | Risk of Breakage |
| :--- | :--- | :--- |
| Max Message Size | Configure explicit maximum send/receive message sizes (e.g., 4MB) to prevent memory exhaustion (OOM) attacks. | Low |
| Metadata Validation | Treat gRPC metadata exactly like HTTP headers. Validate and sanitize all incoming metadata (do not log auth tokens). | None |
| Channel Security | Require `TransportCredentials` (TLS/mTLS) for all channels. Never use `InsecureChannelCredentials` in production. | High (Breaks local non-TLS setups) |
| Payload Reflection | Disable gRPC Server Reflection in production environments. | None |
| Rate Limiting | Implement rate limiting per method or client certificate. | Low |
| Timeout Enforcement | Set deadlines on all RPCs to prevent hanging connections. | None |
| Input Validation | Validate all Protobuf messages against custom constraints beyond schema (e.g., string lengths, ranges). | None |

---

## 6. CLOUD & SERVERLESS SECURITY CHECKLIST

| Security Feature | Requirement | Risk of Breakage |
| :--- | :--- | :--- |
| IAM Least Privilege | Assign single-purpose IAM roles to every Lambda/Function. Never use wildcards (`*`) in `Action` or `Resource` policies. | Medium (Strict access setup) |
| S3 / Storage Buckets | Block Public Access at the account/bucket level. Enforce AES-256 server-side encryption (SSE-S3 or SSE-KMS). | None |
| Serverless Environment Vars | Do not store plain text secrets in Lambda environment variables; fetch at runtime via KMS/Secrets Manager. | None |
| API Gateway Hardening | Enable WAF, require API keys for usage plans, set strict payload size limits, and enable detailed execution logging. | Low |
| Cloud Metadata Protection | Ensure IMDSv2 is enforced on all EC2 instances to prevent SSRF from extracting instance credentials. | Low |
| CloudTrail / Audit Logs | Enable CloudTrail on all regions; store logs in a separate account with immutable storage. | None |
| VPC & Network Security | Place sensitive resources in private subnets; use VPC endpoints for AWS services; restrict security group egress. | Low |
| Container Registry | Scan images for vulnerabilities; block pushes with Critical CVEs; use signed images only. | Low |
| Secrets Rotation | Automate rotation of database credentials and API keys using Secrets Manager. | Medium |
| Backup Encryption | Ensure all automated backups are encrypted with KMS and access is restricted. | None |
| DDoS Protection | Use AWS Shield / Cloudflare; configure rate-based rules in WAF. | Low |

---

## 7. CI/CD & DEVSECOPS PIPELINE CHECKLIST

| Security Feature | Requirement | Risk of Breakage |
| :--- | :--- | :--- |
| Secret Scanning | Implement pre-commit and CI-level secret scanning (e.g., Gitleaks, TruffleHog). Fail builds if secrets are detected. | None |
| Dependency Scanning | Run SCA (Snyk, Dependabot, Trivy) on all builds. Block deployment on Critical/High CVEs. | Low |
| Runner Isolation | Execute CI/CD jobs in ephemeral, isolated environments. Do not share runner states between distinct pipeline runs. | None |
| Artifact Signing | Sign container images and binaries (e.g., via Sigstore/Cosign) before pushing to registries. Verify signatures before deploy. | Low (Process change) |
| Immutable Infrastructure | Deploy new immutable instances/containers. Never SSH into production or patch running containers. | None |
| Pipeline Security | Protect pipeline definitions with branch protection rules; require reviews for changes to CI config. | Low |
| Static Analysis (SAST) | Integrate SAST tools (Semgrep, CodeQL) into PR checks; block merges on high-severity findings. | Low (Noise) |
| Dynamic Analysis (DAST) | Run automated DAST scans against staging environment before production deployment. | Low |
| Infrastructure as Code Scanning | Scan Terraform/CloudFormation for misconfigurations (Checkov, tfsec). | None |
| Build Provenance | Generate SLSA provenance for all artifacts to verify build integrity. | Low |

---

## 8. AI & LLM INTEGRATION SECURITY

| Security Feature | Requirement | Risk of Breakage |
| :--- | :--- | :--- |
| Prompt Injection Defense | Treat all user input to LLMs as hostile. Use system prompts to restrict behavior. Delimit user input strictly (e.g., `<user_input>`). | Low |
| Output Sanitization | Treat LLM output as untrusted user input. Sanitize, type-check, and parse strictly before executing or rendering LLM output. | None |
| PII Scrubbing | Strip or mask PII, credentials, and sensitive app state before sending context to external LLM APIs. | Medium (Context loss) |
| Agency Isolation | If the LLM has tools/functions, restrict tool permissions. An LLM must not have direct DB write access without human-in-the-loop or strict scoped limits. | High (Limits AI capabilities) |
| Model Theft Prevention | Implement rate limiting on LLM endpoints; do not expose raw model outputs without output validation. | Low |
| Training Data Poisoning Awareness | For fine-tuned models, ensure training data is sanitized and free of malicious instructions. | N/A |
| Logging & Monitoring | Log all LLM prompts and completions (redacted) for abuse detection; set up alerts for suspicious patterns. | Medium (Privacy concerns) |

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
| Gas Limit & DoS | Avoid loops over unbounded arrays; use pull payments for distributing funds. | Low |
| Signature Replay Protection | Use EIP-712 structured data hashing and include nonce/chainID in signed messages. | Low |
| Arithmetic Precision | Use fixed-point math libraries (e.g., PRBMath) for high-precision calculations. | Low |
| Delegatecall Safety | Ensure that `delegatecall` is only used with trusted, verified contracts and that storage layout is compatible. | High |

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
| Secure Storage | Store sensitive data (keys, certificates) in secure element or encrypted flash with hardware-backed key. | Low |
| Side-Channel Mitigations | Use constant-time operations for cryptographic comparisons; avoid branching on secret data. | Medium |
| Supply Chain Security | Verify authenticity of all third-party libraries and toolchains; sign all firmware images. | Low |

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
| Data Portability | Provide users with ability to export their data in a machine-readable format (JSON/CSV). | Low |
| Breach Notification | Have a process to notify affected users and authorities within required timeframe (e.g., 72 hours under GDPR). | None |
| Privacy by Default | Set all privacy settings to the most restrictive option by default. | Low |

---

## 12. LANGUAGE-SPECIFIC SECURITY CONSTRAINTS

### Java / Kotlin
*   **Deserialization**: Block `ObjectInputStream` on untrusted data. Use Jackson/Gson with typing disabled.
*   **Cryptography**: Use `javax.crypto` with explicit algorithms (e.g., `AES/GCM/NoPadding`).
*   **Randomness**: Use `SecureRandom`; never `java.util.Random`. Use `MessageDigest.isEqual()` for constant-time comparisons.
*   **XXE**: Use `DocumentBuilderFactory` with `FEATURE_SECURE_PROCESSING`.
*   **Log Injection**: Sanitize user input before logging; strip newline characters.
*   **Expression Language**: Avoid evaluating user input with JEXL, MVEL, or SpEL without strict sandboxing.

### C / C++
*   **Memory Safety**: Banned APIs: `strcpy`, `strcat`, `sprintf`, `gets`. Use bounded equivalents. Explicitly `NULL` pointers after free.
*   **Compilation**: Compile with `-fstack-protector-strong`, `-fPIE`, `-fcf-protection=full`, and `-D_FORTIFY_SOURCE=2`.
*   **Format Strings**: Never pass user strings directly to `printf`.
*   **Integer Overflows**: Use compiler built-ins for checked arithmetic.
*   **Race Conditions**: Use mutexes; avoid TOCTOU.

### Python
*   **Execution**: Ban `eval()`, `exec()`, `shell=True` in `subprocess`.
*   **Deserialization**: Ban `pickle` and `yaml.load` (use `yaml.safe_load`).
*   **Web Frameworks**: Enable Django `SecurityMiddleware`. Never run Flask with `debug=True` in production. Use Pydantic for input validation.
*   **Templates**: Do not use user input as template string; enable autoescape.
*   **XML Parsing**: Use `defusedxml` for untrusted XML.
*   **Secrets**: Use `os.environ` or Secret Manager; no hardcoded keys.

### JavaScript / TypeScript (Node.js)
*   **Prototype Pollution**: Use `Object.create(null)` for maps. Avoid deep-merge utilities without pollution checks.
*   **Execution**: Ban `eval()` and `new Function()`.
*   **Regex DoS**: Avoid nested quantifiers. Use safe regex libraries or timeouts.
*   **Dependencies**: Run `npm audit --audit-level=high` in CI. Require lockfiles.
*   **Child Processes**: Use `execFile` or `spawn` with argument array; avoid `exec`.
*   **Crypto**: Use `crypto.randomBytes`; `timingSafeEqual`.

### Go
*   **SQL Injection**: Rely on `database/sql` placeholders. No `fmt.Sprintf` for SQL.
*   **Path Traversal**: Use `filepath.Clean()` and `filepath.Rel()`.
*   **Goroutine Leaks**: Ensure all goroutines have exit conditions and context cancellation.
*   **Unsafe**: Ban `unsafe` package usage unless strictly required for FFI.
*   **Templates**: Use `html/template` for HTML; `text/template` only for plain text.
*   **Crypto**: Use `crypto/rand`; never `math/rand`.

### Rust
*   **Unsafe**: Minimize `unsafe` blocks. If used, thoroughly document safety invariants.
*   **Memory/Crypto**: Use `zeroize` crate for sensitive data. Use `ring` or `rustls`.
*   **Integer Overflow**: Rely on `wrapping_*`, `checked_*`, or `saturating_*` math functions where overflow is possible.
*   **Panic Safety**: Handle Mutex poisoning; use `parking_lot`.
*   **FFI**: Validate all pointers and lengths; use `std::ffi::CStr`.

### C# / .NET
*   **SQL Injection**: Use Entity Framework Core with LINQ. If raw SQL is needed, use `SqlParameter`.
*   **XSS & CSRF**: Use `[ValidateAntiForgeryToken]` for mutations. Rely on Razor's auto-encoding.
*   **Crypto**: Use `RandomNumberGenerator.Create()`. Ban `MD5CryptoServiceProvider`.
*   **Config**: Keep secrets in `appsettings.json` strictly local; use Azure Key Vault / AWS Secrets for prod.
*   **Deserialization**: Avoid `BinaryFormatter`; use `System.Text.Json` with strict settings.

### PHP
*   **Database**: Use PDO with prepared statements. Ban `mysqli_query` with string concatenation.
*   **XSS & Types**: Use `htmlspecialchars(..., ENT_QUOTES, 'UTF-8')`. Enable strict typing (`declare(strict_types=1)`).
*   **Configuration**: Disable `allow_url_include`, `expose_php`, and `display_errors` in `php.ini`. Use `password_hash()` with `PASSWORD_ARGON2ID`.
*   **File Uploads**: Validate MIME type with `finfo`; store outside web root.
*   **Session Security**: Use `session.cookie_httponly=1`, `session.cookie_secure=1`, `session.cookie_samesite=Strict`.

### Ruby / Rails
*   **Mass Assignment**: Enforce strict `strong_parameters` in controllers.
*   **SQL Injection**: Use ActiveRecord's parameterized `.where("id = ?", params[:id])`. Never interpolate strings into `.where`.
*   **Rendering**: Do not use `html_safe` on user input.
*   **CSRF**: Use `protect_from_forgery` with `with: :exception`.
*   **Secrets**: Use Rails credentials or environment variables; never hardcode.

### Swift (iOS)
*   **Keychain**: Use `kSecAttrAccessibleWhenUnlockedThisDeviceOnly`.
*   **UserDefaults**: Never store sensitive data.
*   **Crypto**: Use `CryptoKit`; avoid custom algorithms.
*   **Networking**: Enforce ATS; implement certificate pinning with `URLSessionDelegate`.

### Dart / Flutter
*   **Storage**: Use `flutter_secure_storage` for tokens; never `shared_preferences` for secrets.
*   **Network**: Enforce HTTPS; implement certificate pinning with `HttpClient` and `SecurityContext`.
*   **Obfuscation**: Use `flutter build --obfuscate --split-debug-info` for release.
*   **Jailbreak/Root Detection**: Use `flutter_jailbreak_detection` package.
*   **WebView**: Disable JavaScript if not needed; use `WebViewAssetLoader` for local content.

### Bash / Shell Scripts
*   **Injection**: Always quote variables; use `"$var"`. Never `eval`.
*   **Temp Files**: Use `mktemp` for temporary files; clean up with `trap`.
*   **Permissions**: Avoid running scripts as root; drop privileges.
*   **Secrets**: Never hardcode passwords; use environment variables or secret managers.
*   **Input Validation**: Validate all inputs against allowlists.

### PowerShell
*   **Execution Policy**: Use `Set-ExecutionPolicy RemoteSigned`; avoid `Bypass`.
*   **Script Block Logging**: Enable for auditing.
*   **Secrets**: Use `SecretManagement` module; never plaintext in scripts.
*   **Input**: Avoid `Invoke-Expression` with user input.

### Infrastructure as Code (Terraform, CloudFormation, Pulumi)
*   **Secrets**: Use `sensitive = true` and remote state encryption. Never hardcode.
*   **S3 Buckets**: Enforce `block_public_acls = true`, `encryption`, and `versioning`.
*   **Security Groups**: Never open `0.0.0.0/0` for SSH/RDP; restrict to specific IPs.
*   **IAM**: Follow least privilege; avoid `*` in policies.
*   **State Files**: Store remotely with locking and encryption.

### Docker & Container Security
*   **Base Images**: Use minimal/distroless images; avoid `latest` tag.
*   **USER**: Never run as root; specify non-root `USER`.
*   **COPY**: Use `COPY --chown` to set ownership; avoid `ADD` with URLs.
*   **Secrets**: Use Docker secrets or build-time arguments with caution; never embed in image.
*   **Capabilities**: Drop all (`--cap-drop=ALL`) and add only required ones.
*   **Read-only Rootfs**: Mount root filesystem as read-only.
*   **Healthchecks**: Implement but do not expose sensitive info.

### Kubernetes Security
*   **Pod Security**: Use `securityContext` with `runAsNonRoot: true`, `readOnlyRootFilesystem: true`.
*   **Network Policies**: Deny all ingress/egress by default; allow only necessary traffic.
*   **Secrets**: Use `SealedSecrets` or External Secrets Operator; never plain `Secret` in Git.
*   **RBAC**: Follow least privilege; avoid cluster-admin for applications.
*   **Admission Controllers**: Enable PodSecurity or OPA/Gatekeeper to enforce policies.
*   **Image Scanning**: Use admission webhook to block images with critical CVEs.

---

## 13. AGENT RESPONSE & EXECUTION GUIDELINES

When parsing user requests and generating code, the AI Agent **must silently verify and enforce the checklist above**. 

1. **Mandatory Enforcement**: If a user request implicitly or explicitly conflicts with a requirement (e.g., "Disable ATS for easier debugging", "Store JWTs in localStorage", "Create an unauthenticated backend route for testing"), you must output a **warning comment** in the generated code explaining the security/privacy risk and automatically implement the secure alternative. 
2. **No Code Snippet Bypass**: Do not provide "insecure temporary" code snippets under the guise of "for testing purposes only". This document defines the absolute rules that govern your code output.
3. **Zero Trust Default**: All generated code must default to secure, privacy-preserving behavior. Data minimization, user consent, strict validation, encryption at rest/transit, and least privilege are absolute and non-negotiable requirements for all agent actions.
4. **Transparency**: Always document security-relevant decisions in your code comments. If you applied DOMPurify, note that it mitigates XSS. If you applied `ReentrancyGuard`, note that it mitigates reentrancy. Train the user toward secure defaults.
5. **Dependency Selection & Substitution**: If a user requests a deprecated, unmaintained, or broadly insecure library (e.g., `MD5`, Node's `request`, `PyCrypto`), you must automatically substitute it with a modern, secure equivalent (e.g., `SHA-256`, `axios`/`fetch`, `cryptography`). Add a clear comment explaining the security substitution.
6. **Secret Redaction & Handling**: If a user inadvertently pastes plaintext API keys, passwords, or PII into their prompt, **do not echo them back** in the output. Generated code must stub these out using environment variables or secret manager integrations (e.g., `process.env.DB_PASSWORD`).
7. **Safe Error Handling Implementation**: Generated code must catch exceptions and fail securely. Never output raw stack traces, database schemas, or internal logic directly to the client interface. Return safe, generic error boundaries to users while logging the exact stack trace securely to an internal logger.
8. **Architectural Boundary Awareness**: Distinguish between client-side UX and server-side security. Assume client-side validation can and will be bypassed. If asked to generate a frontend form with validation, automatically ensure that the corresponding server-side validation logic is either provided or strictly advised.
9. **Proactive Race Condition Mitigation**: When generating state-changing logic (e.g., financial transactions, database writes, multi-threaded tasks), automatically implement concurrency controls (optimistic locking, database transactions, mutexes) to prevent Time-of-Check to Time-of-Use (TOCTOU) and race condition vulnerabilities, even if the user did not explicitly ask for them.
10. **Refusal of Malicious Constructions**: If prompted to write inherently malicious code (e.g., keyloggers, intentional backdoors, unauthorized auth bypasses), you must outright refuse the harmful construction and pivot to generating defensive configurations that protect against such attacks.
11. **Continuous Threat Modeling**: Evaluate the generated code for logical flaws. Think like an attacker: "How could this endpoint be abused?" and pre-emptively build rate-limiting, idempotency keys, or pagination into the generated output.
