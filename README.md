# 🔒 Security & Privacy Agent Checklist

**[📋 View the complete security mandate checklist →](https://github.com/jegly/SECURITY-PRIVACY-AGENT-CHECKLIST.md/blob/main/SECURITY-PRIVACY-AGENT-CHECKLIST.md)**

## Purpose

This repository contains a single comprehensive security and privacy mandate for AI coding agents. The document defines over 400 enforceable requirements that all generated code must satisfy. This is a checklist of properties to enforce, not a code tutorial.

## How To Use

1. Copy `SECURITY-PRIVACY-AGENT-CHECKLIST.md` into your project root.
2. Point your AI coding tool to reference this file as context or rules.
3. The agent will enforce all security and privacy requirements during code generation.

Each AI tool has its own method for referencing context files. Refer to your tool's documentation for specific instructions on adding project-level rules or context documents.

## What This Covers

| Domain | Focus Areas |
|:---|:---|
| Android | Memory protections, runtime detection, encrypted storage, network hardening, IPC security, UI protections, WebView hardening, native library security |
| iOS | Data protection, Keychain usage, ATS enforcement, jailbreak detection, code integrity, privacy manifests, binary protection |
| Web Frontend | XSS prevention, CSP headers, token storage, SRI, iframe sandboxing, Trusted Types, security headers (COOP/COEP) |
| Backend APIs | Authentication, injection defense, SSRF prevention, cryptographic controls, rate limiting, audit logging |
| GraphQL & gRPC | Query depth limiting, complexity analysis, introspection controls, persisted queries, metadata validation |
| Cloud & Serverless | IAM least privilege, secrets management, storage hardening, VPC security, backup encryption |
| CI/CD Pipeline | Secret scanning, dependency auditing, artifact signing, SAST/DAST integration, build provenance |
| AI & LLM Integration | Prompt injection defense, output sanitization, PII scrubbing, agency isolation, abuse monitoring |
| Smart Contracts | Reentrancy protection, access control, randomness, upgradeability, MEV protection, flash loan security |
| IoT & Embedded | Secure boot, OTA updates, hardware root of trust, side-channel mitigations |
| Privacy & Compliance | GDPR, HIPAA, CCPA, data minimization, right to deletion, breach notification |
| Container & Orchestration | Docker security, Kubernetes pod security, network policies, admission control |

## Language Support

Language-specific constraints are included for Java, Kotlin, Swift, Objective-C, JavaScript, TypeScript, Python, Go, Rust, C, C++, C#, PHP, Ruby, Dart/Flutter, Bash, PowerShell, Terraform, CloudFormation, Pulumi, Docker, and Kubernetes.

## Important Notes

- This is not a tutorial. It defines rules that govern code output.
- Agents verify requirements silently and only surface warnings when a conflict exists.
- Security requirements are non-negotiable. Insecure temporary code is never provided.

## License

MIT License. See the LICENSE file for details.
