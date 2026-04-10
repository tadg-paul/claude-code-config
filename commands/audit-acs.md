Review the acceptance criteria for the current issue. Do not write any code or modify any files.

1. **Summarize the functionality.** In plain language, describe what this issue delivers from the user's perspective. Post this summary in chat.

2. **Enumerate edge cases.** For each AC, list the edge cases, boundary conditions, error states, and unusual inputs that could falsify it. Be adversarial — think about what could go wrong, not just what should go right.

3. **Identify missing ACs.** Are there user-facing behaviours, error conditions, or integration points implied by the problem statement that no AC addresses? List them.

4. **Check AC quality.** Run the AC self-audit from ISSUES.md:
   - Does each AC describe a system state, not a test action?
   - Does it contain any forbidden language?
   - Does it pass the litmus test?

5. **Summarize.** List all findings: missing edge cases, missing ACs, quality violations. Recommend specific additions or rewrites.

Do not proceed past this audit. Wait for Taḋg's response.
