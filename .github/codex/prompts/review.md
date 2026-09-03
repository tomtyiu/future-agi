Review this pull request. Focus on:

Correctness: Logic errors, off-by-one bugs, unhandled edge cases

Security: Input validation, auth checks, data exposure, SQL injection, XSS, command injection, and path traversal. Provie secure code if applicable. Flag any dependencies with known CVEs (check npm audit) ,dependency security (new packages with known vulnerabilities).

Audit: Search known CVEs and security advisories that may be related to the codebase, its dependencies, frameworks, libraries, APIs, and implementation patterns. For each potentially relevant CVE, verify that the affected component and vulnerable version or code pattern actually exist in the repository, then assess whether the vulnerable path is reachable. Report only CVEs that are directly relevant or plausibly applicable, with evidence showing why they match the code.

Performance: N+1 queries, unnecessary computation, missing indexes

Maintainability: Overly complex functions, missing error handling, unclear naming

Quality control: high quality secure code

Rules:

Flag only P0 (must fix before merge) and P1 (should fix before merge) issues
Include the file path and line number for each finding
Suggest a specific fix, not just "this could be better" and show code related to the fix.
If the PR is clean, say so explicitly
Do not flag: Style preferences (formatting, naming conventions) unless they violate project standards Test file structure choices Comment content or documentation style
