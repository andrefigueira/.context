# Prompt: Security Audit

## Context Files to Include
```
.context/auth/security.md
.context/auth/overview.md
.context/ai-rules.md
.context/anti-patterns.md
```

## Prompt

```
Perform a security audit on:

**Scope**: [File/component/feature to audit]

Check for:

**Authentication & Authorization**
- [ ] Auth bypass vulnerabilities
- [ ] Missing authorization checks
- [ ] Privilege escalation paths
- [ ] Session management issues

**Input Handling**
- [ ] SQL injection
- [ ] XSS vulnerabilities
- [ ] Command injection
- [ ] Path traversal
- [ ] Unvalidated redirects

**Data Protection**
- [ ] Sensitive data exposure
- [ ] Insecure data storage
- [ ] Missing encryption
- [ ] Logging sensitive data

**Configuration**
- [ ] Hardcoded secrets
- [ ] Debug mode in production
- [ ] Insecure defaults
- [ ] Missing security headers

**Dependencies**
- [ ] Known vulnerable packages
- [ ] Outdated dependencies

For each finding:
1. Severity (Critical/High/Medium/Low)
2. Location in code
3. Attack scenario
4. Remediation steps
```
