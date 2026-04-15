# SECURITY-PRIVACY-AGENT-CHECKLIST.md

[![Security Mandate](https://img.shields.io/badge/Security%20Mandate-View%20Checklist-blue)](https://github.com/jegly/SECURITY-PRIVACY-AGENT-CHECKLIST.md/blob/main/SECURITY-PRIVACY-AGENT-CHECKLIST.md)



## Purpose

This document contains mandatory security and privacy requirements for AI coding agents. All generated code must satisfy these constraints. This is a comprehensive checklist of properties to enforce, not a code tutorial.

Agents must follow privacy-by-design principles, enforce least privilege, use secure defaults, and implement data minimization everywhere. When a user request conflicts with any requirement, the agent must silently enforce the secure path and insert a clear warning comment explaining the risk and the recommended long-term fix. Security cannot be bypassed for convenience.

## What This Is

A single-source-of-truth security mandate containing over three hundred enforceable requirements spanning mobile applications, web frontends, backend APIs, cloud infrastructure, AI and LLM integrations, smart contracts, and embedded systems. Use this checklist to ensure every line of generated code follows secure-by-default and privacy-first principles.

## Coverage

| Domain | Requirements |
|:---|:---|
| Android Application Security | Memory protections, runtime detection, encrypted storage, network hardening, IPC security, UI protections, logging controls, permissions |
| iOS Application Security | Data protection, Keychain usage, ATS enforcement, jailbreak detection, code integrity, privacy manifests |
| Web Frontend Security | XSS prevention, CSP headers, token storage, state management, SRI, iframe sandboxing, postMessage validation |
| Backend and API Security | Authentication, authorization, injection defense, SSRF prevention, file upload validation, cryptographic controls |
| GraphQL and gRPC Security | Query depth limiting, complexity analysis, introspection controls, metadata validation, reflection disabling |
| Cloud and Serverless Security | IAM least privilege, S3 bucket hardening, secrets management, API gateway configuration, IMDSv2 enforcement |
| CI and CD Pipeline Security | Secret scanning, dependency auditing, runner isolation, artifact signing, immutable infrastructure |
| AI and LLM Integration Security | Prompt injection defense, output sanitization, PII scrubbing, agency isolation, tool permission restriction |
| Smart Contract and Web3 Security | Reentrancy protection, integer safety, access control, randomness generation, upgradeability patterns, MEV protection |
| IoT and Embedded Systems Security | Hardware root of trust, secure boot, OTA update signing, interface locking, transport encryption |
| Privacy and Compliance | GDPR right to deletion, HIPAA audit logging, data minimization, consent management, cross-border transfer controls |

## Language Support

Language-specific security constraints are defined for twelve plus programming languages including Java, Kotlin, Swift, Objective-C, JavaScript, TypeScript, Python, Go, Rust, C, C++, CSharp, PHP, and Ruby.

## How To Use

### For AI Coding Agents

Add the checklist file to your project root and configure your AI coding tool to reference it as context or rules.

**Cursor IDE**
Create a `.cursorrules` file in your project root and add a directive to consult `security-privacy-agent-checklist.md` and enforce all applicable requirements during code generation.

**GitHub Copilot**
Place the checklist in your workspace and reference it in `.github/copilot-instructions.md`. Copilot will incorporate the security constraints into its suggestions.

**Continue.dev**
Add the checklist file path to the `contextFiles` array in your Continue configuration file. The agent will load the mandate as persistent context.

**Aider**
Launch aider with the read flag pointing to the checklist file using `aider --read security-privacy-agent-checklist.md`.

**Claude Code / Claude Agent**
Add the checklist as project context or upload it at the start of your session. Reference it explicitly when requesting code generation to ensure Claude enforces the security requirements.

**Windsurf (Codeium)**
Place the checklist in your workspace root. Windsurf automatically indexes project files and will incorporate the security mandate into its context when generating code.

**Antigravity**
Add the checklist to your project context directory. Antigravity will reference the mandate during code generation sessions and enforce the defined security constraints.

**General System Prompt**
If your AI tool supports custom system prompts or project instructions, prepend or append the entire mandate document to ensure it governs all generated output.

### For Manual Code Review

Use this document as a verification checklist during security reviews. Each requirement includes a risk assessment indicating potential breakage when enforced.

### For Team Onboarding

Share this mandate with development teams to establish baseline security expectations for all generated and reviewed code.

## Document Structure

Each security requirement follows a three-column format.

| Column | Purpose |
|:---|:---|
| Security Feature | Name of the security control being enforced |
| Requirement | Mandatory implementation requirement that must be satisfied |
| Risk of Breakage | Assessment of potential impact when the requirement is strictly enforced |

## Customization

Fork this repository and adapt the mandate to your organization specific needs.

- Remove sections irrelevant to your technology stack to reduce context size
- Adjust risk tolerance levels based on your threat model
- Add organization specific compliance requirements such as SOC2 or FedRAMP
- Include internal security contacts or escalation paths

## Important Notes

- This is not a tutorial and contains no code examples. It defines rules that govern code output
- Agents should verify requirements silently and only surface warnings when user requests explicitly conflict
- Security requirements are non-negotiable. Do not provide insecure temporary code snippets
- Always document security-relevant decisions in generated code comments

## Related Standards

This mandate incorporates guidance from OWASP Mobile Security Testing Guide, OWASP Web Security Testing Guide, OWASP API Security Top Ten, CWE Top Twenty Five Most Dangerous Software Weaknesses, NIST Cybersecurity Framework, and GDPR and HIPAA compliance requirements.

## Contributing

Contributions are welcome. Please ensure additions follow the established table format, include accurate risk assessments, and reference authoritative security standards where applicable.

## License

MIT License. See the LICENSE file for details.

## Disclaimer

This mandate provides security guidance but cannot guarantee complete protection against all threats. Always conduct proper security reviews and penetration testing for production applications.

