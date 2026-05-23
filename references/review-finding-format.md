# Review Finding Format

Use this standard for review agents and review skills.

## Finding Shape

Return one block per finding:

```text
<severity>: Source: <agent>; <file>:L<line>
Problem: <concrete issue in one sentence>
Why: <simple explanation of why this matters>
Fix: <practical change>
Example: <example of how this problem can happen>
```

Severity values:

- `bug`: broken behavior, security issue, data loss, or a high-impact regression
- `risk`: likely future failure, fragile behavior, unclear contract, or maintainability problem
- `nit`: small cleanup that is worth mentioning because it removes confusion
- `q`: blocking question only

Required fields:

- `Problem`: concrete issue in one sentence.
- `Why`: simple explanation of why this matters.
- `Fix`: practical change.
- `Example`: example of how this problem can happen.

Output rules:

- Start with `Found issues:` when there are findings.
- Return exactly `No findings` when there are none.
- Separate findings with a blank line.
- Keep each issue concise, but include `Problem`, `Why`, `Fix`, and `Example`.
- Do not add files-reviewed lists, summaries, praise, or long evidence sections.
