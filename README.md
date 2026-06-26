# AI-Generated Code Review Checklist

AI can produce a large amount of plausible-looking code quickly. Human review should therefore focus less on formatting and individual lines, and more on whether the change solves the right problem, fits the system, and is supported by meaningful evidence.

Builds, formatting, linting, type checks, standard security scans, secret scanning, dependency vulnerability checks, and license checks should run automatically before human review. Also, run AI-reviews, aka reviews performed by one or more AI agents, also before asking for a peer-review.

## What the PR should explain

Before reviewing the code, the pull request should briefly state:

* **What is changing:** the user or system outcome and relevant acceptance criteria.
* **How it was solved:** the chosen approach and any important existing code, pattern, or dependency that was reused or deliberately not reused.
* **What changed around it:** new dependencies, configuration, APIs, schema changes, permissions, feature flags, or operational implications.
* **How it was checked:** tests run, manual verification, limitations, and anything intentionally deferred.

---

## 1. Does this solve the real problem?

**Why this matters:** AI often finds the quickest way to make an error, test, or visible symptom disappear. That may look successful while leaving the underlying problem untouched.

**Review questions**

* Does the change meet the actual requirement, including important non-happy-path behaviour?
* Does it fix the cause rather than conceal the symptom?
* Is a temporary workaround clearly labelled and consciously accepted as such?
* Are assumptions about business rules, APIs, or system behaviour supported by evidence?

**Warning signs**

* An exception is caught and ignored, converted into a generic success response, or replaced with a silent fallback.
* Validation, permissions, alerts, or logging become weaker.
* A test was removed, skipped, loosened, or changed only so the new implementation passes.
* The PR says “we can improve/fix this later” without explaining the current limitation and its impact.
* The change is much larger than the stated problem requires.

---

## 2. Does it fit the way this system already works?

**Why this matters:** AI can produce locally sensible code while gradually creating a second architecture: new folders, helper layers, patterns, naming conventions, or duplicate implementations that make the codebase harder to understand and maintain.

**Review questions**

* Is there already an established way to solve this problem in the codebase?
* Does the change reuse existing components, services, utilities, or patterns where that would be clearer?
* Is the code located in the expected place, with responsibilities kept reasonably separate?
* Does it introduce a new concept, layer, or pattern only where there is a clear need?

**Warning signs**

* A new generic `Helper`, `Manager`, `Adapter`, `Utils`, or service layer appears for a small local problem.
* The agent builds its own version of functionality that already exists elsewhere.
* A feature introduces a different state-management, error-handling, logging, API, or data-access pattern from the rest of the system.
* Domain logic, database access, API calls, and UI behaviour are mixed in one large component or function.
* The PR touches several unrelated modules merely to establish a new pattern.

---

## 3. Is the dependency decision sound?

**Why this matters:** AI may add a library because it is easy to suggest, or reimplement something complex because it is easy to generate. Neither choice is automatically right.

**Review questions**

* Is a new dependency necessary, or can an approved framework feature, internal library, or existing package solve this?
* Conversely, is the agent implementing security-sensitive or complex functionality that should use a mature, approved library?
* Does the package fit team standards for maintenance, licensing, security, compatibility, and support?
* Is the selected package and version real, appropriate, and correctly added?

**Warning signs**

* A small feature adds several new direct or transitive dependencies.
* Lock files change, but the PR does not explain why.
* An unfamiliar package is added without evidence that it is maintained and approved.
* The code hand-rolls authentication, encryption, date parsing, validation, retries, file handling, or other well-solved infrastructure concerns.
* The agent introduces a package, API, version, or configuration option that nobody can verify in a clean environment.

---

## 4. What happens outside the happy path?

**Why this matters:** AI-generated code often looks complete because it handles the example it was given. Production systems fail at boundaries: missing data, retries, slow services, duplicate requests, and old clients.

**Review questions**

* What happens with invalid, missing, duplicated, delayed, partial, or unexpected input?
* What happens when an external system is slow, unavailable, or returns something unexpected?
* Could retries create duplicate payments, emails, jobs, writes, or events?
* Does the user receive an accurate failure state rather than a false impression of success?
* Could this break an existing integration, API consumer, migration path, or stored data?

**Warning signs**

* Broad `catch` blocks, empty exception handlers, or default values that hide a failed operation.
* External calls without timeouts, clear error handling, or deliberate retry behaviour.
* Database, API, or schema changes without compatibility or migration consideration.
* A fallback path that returns stale, incomplete, or misleading data.
* Logic that assumes data is always present, ordered, current, or valid.

---

## 5. Do the tests provide real evidence?

**Why this matters:** When AI writes both the implementation and the tests, both may reflect the same misunderstanding. Passing tests are useful evidence, but not proof that the requirement was understood correctly.

**Review questions**

* Would the tests fail if the main behaviour were subtly wrong?
* Do they test meaningful outcomes, including important errors and edge cases?
* Is the right level of testing used: unit, integration, contract, end-to-end, regression, or manual verification?
* Were existing tests changed because the product behaviour legitimately changed, or just because the implementation changed?

**Warning signs**

* Tests only check that a method returns exactly what the new implementation produces.
* Tests mock away every meaningful external dependency or integration boundary.
* A test was deleted, skipped, or weakened without a clear explanation.
* There are many generated tests but none covering failure, compatibility, or real user behaviour.
* Coverage increased, but the essential acceptance criterion is still not tested.

---

# Additional checks when relevant

## 6. Does the change cross a security or data boundary?

Use this section whenever the change touches users, permissions, data, external input, storage, infrastructure, or AI tools.

**Why this matters:** AI does not reliably understand your organisation’s security model, data classification, tenancy rules, or permission structure from local code alone.

**Review questions**

* Is authentication and authorisation enforced in the right place?
* Can one user, tenant, role, or service access data or actions belonging to another?
* Is user-controlled input safe when used in queries, HTML, paths, commands, URLs, or logs?
* Could sensitive data, secrets, tokens, or personal information be exposed or retained unnecessarily?
* Does the change introduce an AI-specific risk, such as untrusted content influencing tool calls or retrieving data across boundaries?

**Warning signs**

* New endpoints, permissions, database queries, cloud roles, file uploads, redirects, shell calls, or external integrations.
* Raw queries, dynamic commands, direct object references, or values copied directly from user input.
* Secrets or personal data appearing in logs, tests, error messages, prompts, or configuration.
* A change to access-control logic without review from the relevant owner.

---

## 7. Is this change reviewable and owned by a human?

Use this section for large PRs, public APIs, shared systems, migrations, significant user-facing changes, and changes that affect other teams.

**Why this matters:** AI can generate large PRs faster than people can understand them. A green pipeline does not compensate for a change that nobody can confidently explain, maintain, or reverse.

**Review questions**

* Is the PR small enough to review with confidence, or should it be split by behaviour or risk boundary?
* Are unrelated refactorings, formatting changes, generated files, and dependency updates separated from the actual feature?
* Does the right domain, architecture, security, UX, accessibility, or platform owner need to review it?
* Is there a human who understands the design and accepts responsibility for maintaining it?

**Warning signs**

* A large PR combines a feature, refactoring, dependency upgrade, config change, migration, and generated-file update.
* Unrelated changes make it difficult to see what behaviour actually changed.
* Nobody can explain why a particular approach was chosen without referring back to the agent conversation.
* The change affects shared infrastructure, public APIs, data models, or security-sensitive code without an appropriate owner reviewing it.

---

## Where human attention should go first

For every AI-generated PR, review these in order:

1. Did it fix the real problem without hiding failure?
2. Does it fit the existing architecture and reuse the right things?
3. Are dependency choices sound?
4. What can fail outside the happy path?
5. Do the tests prove the behaviour?

Use sections 6 and 7 when the PR changes security, data, permissions, external integrations, infrastructure, public contracts, shared systems, user-facing behaviour, or spans a large and difficult-to-review scope.

Do not spend scarce human review time on issues that reliable automation should catch first, such as formatting, import ordering, simple naming, lint rules, type errors, known vulnerable package versions, leaked secrets, or routine style violations.
