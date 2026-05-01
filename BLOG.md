# Why I Built a Local-First Data Anonymizer for AI Workflows

*On the quiet privacy risks of uploading research data to AI tools — and a practical solution.*

---

There is a moment that most researchers working with AI tools have probably experienced: you have a dataset open, you want a quick analysis or a code snippet that handles your specific column structure, and the fastest path forward is to paste the data directly into the chat.

It works. It's useful. And it carries a risk that is easy to overlook.

---

## The Problem With Uploading Real Data to AI Systems

When you upload a file or paste data into an AI tool, that data enters a system you do not control. Depending on the platform, the service's data handling policies, and your account type, your data may be:

- **Logged** for safety review or quality assurance
- **Retained** for a period after the session ends
- **Used to improve models** if you haven't explicitly opted out
- **Accessible to employees** under certain review conditions

For many use cases, this is an acceptable trade-off. For researchers working with unpublished experimental results, proprietary datasets, patient-adjacent data, sample identifiers tied to ongoing studies, or commercially sensitive measurements — it is not.

The risk is not hypothetical or paranoid. It is simply a consequence of how most cloud AI systems are architected: your data goes to a server, and what happens after that is governed by a terms-of-service document that most people do not read carefully.

---

## What Researchers Actually Need

The most common use of AI tools in research workflows is not to share the data — it's to get help *working with* the data. You want to:

- Generate analysis code that handles your column structure
- Debug a script that processes your specific format
- Ask for statistical guidance on your experimental design
- Explore visualization options

None of these tasks actually require the AI to see your real values. A dataset with fake sample IDs, anonymized group labels, and plausible-but-synthetic numeric measurements is almost always sufficient for these purposes. The AI doesn't know or care whether `SAMPLE_001` used to be `ACC_2024_Cohort_B_strain_47`.

The problem is that anonymizing data properly, consistently, and quickly has always required extra tooling — or trusting cloud services with the original data to do it. Neither option felt right.

---

## Building DataMask

DataMask started as a straightforward idea: a single HTML file that anonymizes data entirely in the browser, with no backend, no upload, and no retained state.

The key design constraint was that the tool had to be useful *before* you trust any external service. That meant:

**Processing must be local.** Every transformation — noise injection, column shuffling, cryptographic hashing — happens inside the browser's JavaScript engine. The data never touches a server.

**Text identifiers must be consistently replaced.** Numeric anonymization is well-understood, but research data is full of string identifiers: sample names, strain IDs, experimental group labels, metadata tags. These needed to be replaced with consistent tokens (`SAMPLE_001`, `SAMPLE_002`, …) that preserve relationships without leaking anything real.

**The anonymization must be strong enough to be meaningful.** A tool that adds 1% noise to your measurements isn't protecting much. DataMask offers three tiers: a fast Lite mode for internal testing, a Strong mode that applies sigmoid distortion and cross-column mixing with parameters that are never stored, and a Vault mode that uses SHA-256 to generate cryptographically irreversible synthetic values with zero mathematical link to the originals.

**The workflow must be fast.** Researchers are busy. The tool has four steps: upload, select columns, choose a protection mode, download. That's it.

---

## Why Client-Side Matters

Running entirely in the browser is not just a technical choice — it's an architectural statement about where trust should live.

When a tool processes your data locally, you can verify exactly what it does. The source code is in the page. There is no API call to audit, no server log to request, no data retention policy to read carefully. The data is processed and stays on your machine.

This is especially important for tools designed to protect sensitive data. A cloud anonymization service that handles your data in order to anonymize it is, by definition, a service that has seen your original data. The privacy guarantee depends entirely on how much you trust that service's infrastructure, employees, and policies.

A local tool removes that dependency entirely.

---

## The Practical Upside

In practice, this approach works well for the most common AI-assisted research workflows:

**Getting analysis code written.** Paste an anonymized version of your dataset into the chat. The AI can see your column structure, your data types, your approximate value ranges. It can generate working code. You then run that code on your real data, locally, without it ever going anywhere.

**Debugging data processing scripts.** Share a synthetic version of your problematic dataset. The AI can help you trace the issue without seeing proprietary identifiers or real measurement values.

**Exploring statistical approaches.** Anonymized data with preserved distributional shape (DataMask's Vault mode, with shape-preservation enabled) gives the AI enough information to give meaningful statistical guidance without exposing your actual results.

**Sharing with collaborators.** Anonymized data can be shared with external collaborators, submitted to review processes, or attached to early drafts without concern about premature disclosure.

---

## What DataMask Is Not

DataMask is not a formal data de-identification tool compliant with HIPAA, GDPR, or other regulatory frameworks. It does not perform entity recognition on free text, does not scrub metadata from file formats, and does not provide certified anonymization guarantees.

It is a practical utility for researchers who want a fast, local, trustworthy way to prepare datasets for use in AI-assisted workflows — and who understand that the goal is to share the *structure* of the data, not the data itself.

If you are working with data that requires regulatory-grade anonymization, you should use tools built and certified for that purpose.

---

## Open Source

DataMask is a single HTML file. You can read the entire source code in a few minutes. There are no hidden dependencies, no bundled analytics, no obfuscated logic.

It is open source under the MIT license — use it, fork it, embed it, improve it.

→ [github.com/mbaffour/datamask](https://github.com/mbaffour/datamask)

---

*Built with passion for science and discovery.*
