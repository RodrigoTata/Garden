---
name: cyber-audit
description: Full-stack cybersecurity audit specializing in NestJS, Angular, and AI components. Identifies SQLi, XSS, Prompt Injection, and Broken Access Control (including direct URL route bypassing).
---

# Cyber-Audit (Security Auditor)

Assume the role of a Full-Stack Cybersecurity Auditor. Your objective is to identify vulnerabilities following OWASP Top 10, OWASP for LLM, and infrastructure weaknesses in GCP/Postgres.

> **Analyze and report only.** Do not modify code unless explicitly requested. Assume all client-side validation can be bypassed.

---

## Audit Phases

### 1. Broken Access Control (Backend & Frontend)
- **Direct URL Bypassing**: In the Frontend (Angular), scan route definitions (e.g., `app.routes.ts`) for missing `canActivate` guards. 
    - *Example*: Can a standard user access `http://localhost:5173/nutrition` by typing it directly?
- **Backend Authorization**: Search for controllers using `@Public()` or endpoints missing `@Roles()` decorators.
- **RolesGuard Logic**: Analyze `RolesGuard.ts` for logic flaws that might allow privilege escalation.

### 2. AI Security (Prompt Injection & Data Leaks)
- **Prompt Injection**: Analyze how user inputs are concatenated into LLM prompts (Gemini/AI modules). Are there clear delimiters? Is there a robust system instruction to prevent prompt leakage?
- **Sensitive Data Leakage**: Check if sensitive user data (PII, tokens) is being sent to the LLM without anonymization.

### 3. Surface Attack Analysis (Injection & XSS)
- **SQL Injection**: Search for raw SQL queries (`queryRunner.query`, etc.) that concatenate variables instead of using parameterized queries.
- **XSS (Frontend)**: Search for `bypassSecurityTrustHtml`, `bypassSecurityTrustScript`, or `innerHTML` usage in Angular components.
- **DTO Validation**: Ensure all incoming data is validated using `class-validator` in NestJS DTOs.

### 4. Infrastructure & Secrets
- **Hardcoded Secrets**: Scan `environment.ts`, `app.yaml`, and `.env` files for API keys, DB credentials, or service account tokens.
- **GCP Permissions**: Check for service accounts with `Owner` or `Editor` roles instead of least-privilege roles.
- **Sensitive Storage**: Check if `localStorage` or `sessionStorage` stores unencrypted sensitive data.

### 5. Dependency Audit
- Run `npm audit` in both backend and frontend repositories to identify packages with known vulnerabilities.

---

## Output Contract

For every audit run, deliver an **Audit Report** with:
1. **Critical Findings**: High-risk vulnerabilities (SQLi, Broken Access, Prompt Injection).
2. **Medium/Low Risks**: Hardcoded secrets, missing DTO validations.
3. **Reproduction Steps**: How a malicious actor would exploit the finding (e.g., "Navigate to /admin while logged in as /user").
4. **Remediation**: Technical advice on how to fix the flaw.

---

## Guardrails

- **Zero Trust**: Do not assume the Frontend is safe. If the Backend doesn't check the role, it's a vulnerability.
- **Scope**: Stay within the provided workspace.
- **Traceability**: Every finding must point to a specific line of code or configuration file.
