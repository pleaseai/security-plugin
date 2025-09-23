# Gemini CLI Security Extension

The Security extension is an open-source Gemini CLI extension, built to enhance your repository's security posture. The extension adds a new command to Gemini CLI that analyzes code changes to identify a variety of security risks and vulnerabilities.

## Features

- **AI-powered security analysis**: Leverages Gemini's advanced capabilities to provide intelligent and context-aware security analysis.
- **Focused analysis**: Specifically designed to analyze code changes within pull requests, helping to identify and address vulnerabilities early in the development process.
- **Open source**: The extension is open source and distributed under the Apache 2.0 license.
- **Integrated with Gemini CLI**: As a Google-developed extension, it integrates seamlessly into the Gemini CLI environment, making security an accessible part of your workflow.
- **Expandable scope**: The extension is designed with an extensible architecture, allowing for future expansion of detected security risks and more advanced analysis techniques.

## Installation

Install the Security extension by running the following command from your terminal *(requires Gemini CLI v0.4.0 or newer)*:

```bash
gemini extensions install https://github.com/google-gemini/gemini-cli-security
```

## Use the extension

The Security extension adds the `/security:analyze` command to Gemini CLI which analyzes code changes on your current branch for common security vulnerabilities and provides an intelligent, Gemini-powered security report to improve the repository's security posture.

Important: This report is a first-pass analysis, not a complete security audit. Use in combination with other tools and manual review.

Note: The /security:analyze command is currently designed for interactive use. Support for non-interactive sessions is planned for a future release (tracked in [issue #20](https://github.com/google-gemini/gemini-cli-security/issues/20)).

## GitHub Integration

Coming soon!

## Benchmark

To evaluate the quality and effectiveness of our security analysis, we benchmarked the extension against a real-world dataset of known vulnerabilities.

### Methodology

Our evaluation process is designed to test the extension's ability to identify vulnerabilities in code changes.

1. **Dataset**: We used the [OpenSSF CVE Benchmark](https://github.com/ossf-cve-benchmark/ossf-cve-benchmark), a dataset containing GitHub repositories of real applications in TypeScript / JavaScript. For each vulnerability, the dataset provides the commit containing the vulnerable code (`prePatch`) and the commit where the vulnerability was fixed (`postPatch`).
2. **Analysis Target**: For each CVE, we set up the repository, found the introducing patch with the help of [archeogit](https://github.com/samaritan/archeogit), and added that patch to our local environment.
3. **Report Generation**: We ran the `/security:analyze` command on this diff to generate a security report.
4. **Validation**: Since the dataset has a small number of repositories, we manually reviewed all the generated security reports and compared with the ground truth to calculate the final precision and recall numbers.

We are now actively working to automate the evaluation framework and enrich our datasets by adding new classes of vulnerabilities.

### Results

Our evaluation on this dataset yielded a precision of **90%** and a recall of **93%**.

* **Precision (90%)** measures the accuracy of our detections. Of all the potential vulnerabilities the extension identified, 90% were actual security risks.
* **Recall (93%)** measures the completeness of our coverage. The extension successfully identified 93% of all the known vulnerabilities present in the dataset.

## Types of vulnerabilities

The Security extension scans files for the following vulnerabilities:

### Secrets management

- **Hardcoded secrets**: Credentials such as API keys, private keys, passwords and connection strings, and symmetric encryption keys embedded directly in the source code

### Insecure data handling

- **Weak cryptographic algorithms**: Weak or outdated cryptographic algorithms, including any instances of DES, Triple DES, RC4, or ECB mode in block ciphers
- **Logging of sensitive information**: Logging statements that might write passwords, PII, API keys, or session tokens to application or system logs
- **Personally identifiable information (PII) handling violations**: Improper storage, insecure transmission, or any use of PII that may violate data privacy regulations
- **Insecure deserialization**: Code that deserializes data from untrusted sources without proper validation, which could allow an attacker to execute arbitrary code

### Injection vulnerabilities

- **Cross-site scripting (XSS)**: Instances where unsanitized or improperly escaped user input is rendered directly into HTML, which could allow for script execution in a user's browser
- **SQL injection (SQLi)**: Database queries that are constructed by concatenating strings with raw, un-parameterized user input
- **Command injection**: Code that executes system commands or cloud functions using user-provided input without proper sanitization
- **Server-side request forgery (SSRF)**: Code that makes network requests to URLs provided by users without validation, which could allow an attacker to probe internal networks or services
- **Server-side template injection (SSTI)**:  Instances where user input is directly embedded into a server-side template before it is rendered

### Authentication

- **Authentication bypass**: Improper session validation, insecure "remember me" functionality, or custom authentication endpoints that lack brute-force protection
- **Weak or predictable session tokens**: Tokens that are predictable, lack sufficient entropy, or are generated from user-controllable data
- **Insecure password reset**: Predictable reset tokens, leakage of tokens in logs or URLs, and insecure confirmation of a user's identity

### LLM Safety
- **Insecure Handling of LLM Inputs (Prompts)**: Analyzes how prompts are constructed, identifying risks from concatenating untrusted data sources or embedding sensitive information (e.g., API keys, PII) directly in prompts.
- **Unsafe Execution of LLM-generated content**: Detects when code or commands returned by the LLM are executed directly (e.g., via `eval()`, shell execution, or scripting libraries) without proper sandboxing or validation.
- **Injection Vulnerabilities from LLM Output**: Identifies cases where LLM output is concatenated into backend-sensitive functions, such as SQL queries, OS commands, or web page rendering (XSS).
- **Unsanitized Rendering of LLM Outputs**: Finds vulnerabilities where LLM-generated content (like HTML or Markdown) is rendered on a client without adequate sanitization, potentially leading to Cross-Site Scripting (XSS).
- **Insecure Plugin/Tool Usage**: Detects when LLM output is used to select or provide input to external plugins or tools, which could result in code injection or privilege escalation.
- **Insecure Logging or Storage of Sensitive LLM Outputs**: Finds instances where sensitive data from LLM interactions (which may include PII, credentials, or proprietary information) is logged or stored insecurely.
- **Improper Trust in LLM Outputs for Security Decisions**: Detects cases where critical security logic (such as authorization, access control, or validation) is based directly on manipulable LLM output.

## Resources

- [Gemini CLI extensions](https://github.com/google-gemini/gemini-cli/blob/main/docs/extension.md): Documentation about using extensions in Gemini CLI
- Blog post (coming soon!): More information about the Security extension
- [GitHub issues](https://github.com/google-gemini/gemini-cli-security/issues): Report bugs or request features

## Legal

- License: [Apache License 2.0](https://github.com/google-gemini/gemini-cli-security/blob/main/LICENSE)
- Security: [Security Policy](https://github.com/google-gemini/gemini-cli-security/blob/main/SECURITY.md)
