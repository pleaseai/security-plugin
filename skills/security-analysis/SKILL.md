---
name: Security Vulnerability Analysis
description: Identify security vulnerabilities using SAST methodology with MCP tools for code scanning. Use when reviewing code security, analyzing vulnerabilities, auditing changes, or when user mentions security, vulnerabilities, SQL injection, XSS, authentication, secrets, or security audit.
allowed-tools: Read, Grep, Glob, mcp__securityServer__scan_file, mcp__securityServer__analyze_code
---

# Security Vulnerability Analysis

Expert security analysis following senior security engineer methodology.

## Core Principles

- **Assume All External Input is Malicious**
- **Principle of Least Privilege**
- **Fail Securely**: Never expose sensitive info in errors

## Vulnerability Categories

### 1. Hardcoded Secrets
- API keys, passwords, private keys
- Database connection strings
- Decode base64 strings for hidden credentials

### 2. Broken Access Control
- **IDOR**: Verify ownership checks on user-supplied IDs
- **Missing Authorization**: Check admin/permission guards
- **Privilege Escalation**: User shouldn't modify own role
- **Path Traversal**: Sanitize file paths from user input

### 3. Insecure Data Handling
- Weak crypto (DES, MD5, SHA1, RSA < 2048)
- PII in logs or transmitted over HTTP
- Insecure deserialization from untrusted sources

### 4. Injection Vulnerabilities
- **SQL Injection**: String concatenation in queries
- **XSS**: `dangerouslySetInnerHTML` with unsanitized input
- **Command Injection**: User input in shell commands
- **SSRF**: User-provided URLs without allowlist
- **SSTI**: User input in server templates

### 5. Authentication Flaws
- Weak session tokens (predictable, insufficient randomness)
- Insecure password reset flows
- Missing brute-force protection

### 6. LLM Safety
- **Prompt Injection**: User input in LLM prompts
- **Unsafe Output**: LLM to eval(), exec(), SQL, HTML
- **Flawed Logic**: Security decisions based on LLM output
- **Excessive Tool Permissions**: File writes, network, shell

## Severity Assessment

| Level | Impact | Examples |
|-------|--------|----------|
| **Critical** | RCE, full compromise, data exfiltration | SQL→RCE, hardcoded root creds |
| **High** | Read/modify sensitive data, DoS | Stored XSS, IDOR on critical data |
| **Medium** | Limited data access, requires interaction | Reflected XSS, PII in logs |
| **Low** | Minimal impact, complex exploit | Verbose errors, limited path traversal |

## Reporting Requirements

Each finding must include:
- **Vulnerability**: Brief name (e.g., "SQL Injection")
- **Severity**: Critical/High/Medium/Low
- **Location**: File path and line numbers
- **Line Content**: Exact vulnerable code
- **Description**: Impact and explanation
- **Recommendation**: Clear remediation steps

## High-Fidelity Principles

Before reporting, verify:
1. ✅ Executable, non-test code?
2. ✅ Can point to specific lines?
3. ✅ Direct evidence, not speculation?
4. ✅ Developer can fix in this code?
5. ✅ Plausible negative impact in production?

**ALL must be YES to report.**

## Tool Usage

Offer users:
1. `/security:analyze` for automated comprehensive scan
2. Manual review based on conversation

Use MCP tools (`scan_file`, `analyze_code`) or READ-ONLY tools for analysis.

For complete methodology, see [GEMINI.md](./GEMINI.md)