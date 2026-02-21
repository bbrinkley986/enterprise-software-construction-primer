# TL;DR — The 10 Things That Matter

This primer exists because AI coding assistants are the most powerful — and most misused — tools in modern software engineering. Everything below is based on building a multi-region financial services platform with AI as part of the engineering team.


1. Survey the lot before breaking ground. AI given “build me a fintech platform” will produce something that compiles and nothing that complies. Define your constraints, regulations, and business rules before writing code.
2. Architecture is a human job. AI cannot make system boundary decisions, choose communication patterns, or design data models that account for business rules it doesn’t know about. Build the blueprints first.
3. Document everything as if your builder has amnesia. AI starts fresh every session. Your architecture decisions, coding conventions, testing patterns, and security standards must be written down explicitly — not absorbed through osmosis.
4. Treat AI as your most productive junior developer. Fast, tireless, eager — and it will confidently do the wrong thing if instructions are ambiguous. Same review process, same standards, same scrutiny.
5. Context files are your most important AI document. A well-structured context file (your “job site binder”) is the difference between AI that follows your patterns and AI that invents its own.
6. Security is never implicit. AI will write insecure code by default — concatenating SQL, skipping authorization checks, logging PII — unless explicitly told not to. Every security requirement must be documented and enforced.
7. Test against reality, not fantasy. For financial services, mock databases prove nothing. Our platform runs ~90% integration tests against real PostgreSQL because regulators demand provable correctness, not theoretical coverage.
8. Build in phases with real dependencies. Foundation first (auth, event bus, shared libraries), then core services, then domain services. The sequence matters — you don’t install fixtures before the plumbing is roughed in.
9. Automate your inspections. Code review alone won’t catch every architectural violation. Build automated compliance checks that validate code against your architecture decisions.
10. AI multiplies whatever you give it. Good specs produce good code at speed. Bad specs produce confidently wrong code at scale. The quality of AI output is a direct measure of the quality of your engineering process.

* **The bottom line:** Build the blueprints. Write the specs. Define the standards. Set up the inspections. **Then** *hand the AI the nail gun.*

Read on for the full playbook, with real examples, templates, and checklists from the build.
