# Gemini PR Security Scan Instructions

## Objective

Perform a targeted security audit focusing on the changes introduced in the current pull request. Your primary goal is to identify and report on vulnerabilities, security misconfigurations, and weaknesses introduced by these specific modifications.

While the analysis must focus on the code within the diff, you may need to examine the surrounding, unchanged code to understand the full context and impact of the changes.

## Phase 0: Change Analysis

### 0.1. Identify Code Modifications
*   **Action:** Determine the precise set of changes to be audited.
*   **Procedure:**
    *   Use `git diff --merge-base origin/main` (or the repository's main branch) to generate a diff.
    *   This diff defines the primary scope of the audit. All subsequent analysis should be centered on the vulnerabilities *introduced* by these changes.

## Phase 1: Dependency and Environment Analysis

### 1.1. Vulnerable Dependency Scan
*   **Action:** Check for vulnerabilities in any newly added or updated project dependencies.
*   **Procedure:**
    *   Conduct this analysis only if there were new dependencies added, i.e. if any lock files were present in the diff.
    *   Examine the diff of the relevant lock file (`package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`).
    *   If dependencies were added or their versions were changed, run the appropriate audit command:
        *   `npm audit`
        *   `yarn audit`
        *   `pnpm audit`
    *   Analyze the output for any moderate, high, or critical severity vulnerabilities introduced by the dependency changes.

### 1.2. Check for Outdated Dependencies
*   **Action:** Identify if any newly added dependencies are significantly out of date.
*   **Procedure:**
    *   Use `npm outdated` (or a similar command) and cross-reference its output with the newly added dependencies from the diff.
    *   Report major version differences for these new dependencies, as they could introduce unsupported or vulnerable code.

## Phase 2: Static Application Security Testing (SAST)

### 2.1. Hardcoded Secrets
*   **Action:** Scan the modified lines of code for any newly hardcoded secrets.
*   **Procedure:**
    *   Within the diff, scan files for credentials embedded directly in the source code and search for common patterns of API keys, passwords, private keys, connection strings, and symmetric encryption keys.
    *   Decode any newly introduced base64 strings and search for these patterns within the output.

### 2.2. Broken Access Control

*   **Action:** Scan the modified lines of code for flaws in how user permissions are enforced.
*   **Procedure:**
    *   **Insecure Direct Object Reference (IDOR):** Check API endpoints and data access functions that use user-supplied identifiers (like user_id, order_id, or file_id) without also verifying that the authenticated user has permission to access that specific object.
    *   **Missing Function-Level Access Control:** Review API endpoints, serverless functions, and other entry points to ensure that they perform robust authorization checks (e.g., verifying user roles or permissions) before executing sensitive logic.
    *   **Privilege Escalation Flaws:** Look for code paths where a user can modify their own role or permissions, or where administrative functions can be called by non-administrative users.
    *   **Improper Service Account Implementation:** Check for service accounts with overly broad permissions and identify where their keys are stored and used. Ensure they follow the principle of least privilege.
    *   **Path Traversal / Local File Inclusion (LFI):** Examine any code that handles file paths based on user input and check for insufficient sanitization that would allow an attacker to access files outside of the intended directory (e.g., using ../).

### 2.3. Insecure Data Handling

*   **Action:** Scan the modified lines of code for weaknesses in data encryption, storage, and processing.
*   **Procedure:**
    *   **Weak Cryptographic Algorithms:** Flag any instances of weak or outdated cryptographic algorithms, such as DES, Triple DES, RC4, or ECB mode in block ciphers, or DSA, blowfish, or twofish in asymmetric ciphers. Flag instances of public key algorithms with insufficent key length, such as RSA with fewer than 2048 bits or ECC with fewer than 256 bits..
    *   **Logging of Sensitive Information:** Scan the code for logging statements that might write passwords, PII, API keys, or session tokens to application or system logs.
    *   **PII Handling Violations:** Look for improper storage (e.g., unencrypted or improperly permissioned), insecure transmission, or any use of Personal Identifiable Information (PII) that may violate data privacy regulations.
    *   **Insecure Deserialization:** Look for any code that deserializes data from untrusted sources without proper validation, which could allow an attacker to execute arbitrary code.

### 2.4. Injection Vulnerabilities
*   **Action:** Look for vulnerabilities related to improper handling of user-supplied input.
*   **Procedure:**
    *   **SQL Injection:** Review all code that constructs database queries. Ensure that ORMs or parameterized queries are used exclusively. Flag any instance of string concatenation or interpolation used to build queries with user input.
    *   **Cross-Site Scripting (XSS):** Check for any instances where unsanitized or improperly escaped user input is rendered directly into HTML, which could allow for script execution in a user's browser. Examine all points where user input is rendered in the UI. If it is a React project, focus on the usage of `dangerouslySetInnerHTML`. Check `src/app/page.tsx` and other components for direct rendering of state or props that could originate from user input without proper sanitization.
    *   **Command Injection:** If the application uses `child_process` or any other method to execute shell commands, verify that user input is not part of the command string. Look for any code that executes system commands or cloud functions using user-provided input without proper sanitization.
    *   **Server-Side Request Forgery (SSRF):** Examine code that makes network requests to URLs provided by users and check for a lack of validation, which could allow an attacker to probe internal networks or services.
    *   **Server-Side Template Injection (SSTI):** Check for any instances where user input is directly embedded into a server-side template before it is rendered.

### 2.5. Authentication
*   **Action:** Analyze any modifications to authentication logic, understanding their interaction with the existing codebase.
*   **Procedure:**
    *   **Authentication Bypass:** In the diff, review authentication logic for weaknesses, such as improper session validation, insecure "remember me" functionality, or custom authentication endpoints that lack brute-force protection.
    *   **Weak or Predictable Session Tokens:** In the diff, analyze how session tokens are generated and managed. Look for tokens that are predictable, lack sufficient entropy, or are generated from user-controllable data (e.g., a simple Base64 encoding of a user ID).
    *   **Insecure Password Reset:** Analyze the password reset implementation for security flaws. Check for predictable reset tokens, leakage of tokens in logs or URLs, and whether the flow confirms a user's identity securely.

## Phase 3: Reporting

*   **Action:** Create a clear, actionable report separating vulnerabilities introduced by this change from pre-existing ones.

---

### **A. Newly Introduced Vulnerabilities**
For each vulnerability introduced by the current pull request, provide the following:

*   **Vulnerability:** A brief name for the issue (e.g., "Cross-Site Scripting," "Hardcoded API Key").
*   **Severity:** Critical, High, Medium, or Low.
*   **Location:** The file path and line number(s) where the vulnerability was introduced.
*   **Description:** A short explanation of the vulnerability and the potential impact stemming from this change.
*   **Recommendation:** A clear suggestion on how to remediate the issue within the new code.

---

### **B. Pre-existing Vulnerabilities**
If, during your analysis of the new code, you identify a significant pre-existing vulnerability in an interacting part of the codebase, report it with the following distinctions:

*   **Vulnerability:** A brief name for the issue, prefixed with "[Pre-existing]".
*   **Severity:** `Informational` (unless it is critically severe and directly exploitable by the new code).
*   **Location:** The file path and line number(s) of the pre-existing issue.
*   **Description:** A short explanation of the vulnerability and a note that it was discovered during the audit of the current PR but was not introduced by it.
*   **Recommendation:** Suggest a potential solution to address the issue.

Begin the scan.