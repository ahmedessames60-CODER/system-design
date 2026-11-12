# Backend Rules — Laravel

<!--==========================================================-->
                        General Principles
<!--==========================================================-->
- When writing code, add a comment header for each section, like this:
```
<!--==========================================================-->
                          Section Name
<!--==========================================================-->
```
- Follow **SOLID** principles when writing code.
- Don't add extra code that wasn't asked for — implement only what's required.
- Don't add anything unless explicitly told to.
- Don't create folders outside the project's established structure.
- Before starting work, create an **execution plan** and share it first, before writing any code.

<!--==========================================================-->
                    Naming & Code Organization
<!--==========================================================-->
- Feature/class names must be clear and descriptive of their purpose.
- Code must be **clean** and **readable**.
- Code must be **scalable**.
- Code must be **performant** (performance-first, especially for queries).
- Split code into classes/layers as much as possible (Single Responsibility).

<!--==========================================================-->
                    Laravel-Specific Rules
<!--==========================================================-->
- Use a **Service / Repository pattern** for business logic; don't put heavy logic directly in Controllers.
- Controllers stay **thin**: receive the request, call a Service/Action, return a response — that's it.
- Use **Form Request classes** for validation; don't validate directly inside the controller.
- Use **API Resources** (`JsonResource`) to format responses; never return raw Models.
- Use **Eloquent relationships** instead of raw queries wherever possible, and watch out for the **N+1** problem (use `with()`).
- Migrations must stay organized: never modify an already-run migration directly — create a new migration instead.
- Use **Enums** instead of magic strings/numbers.
- Use **Dependency Injection** instead of `new Class()` inside code, to keep things testable.
- Routes are split into clear, organized files (e.g., `api.php` split into route groups by feature/module).
- Every API response follows a consistent shape (e.g., `{ status, message, data }`) across the whole project.
- Handle errors with centralized **exception handling** (custom exceptions + handler); never return ad-hoc errors.
- Never let sensitive data (passwords, tokens) be logged or returned in a response.

<!--==========================================================-->
                    Validation Rules
<!--==========================================================-->
- The **backend is always the source of truth**: even if the frontend already validated, the backend must re-validate everything (never trust the client).
- Any regex/rule defined on the backend must be mirrored exactly on the frontend (it's the team's responsibility to keep them in sync).
- Use Form Request classes for all validation, and return clear error messages (`messages()` method) in the project's language — not Laravel's default messages.

<!--==========================================================-->
                    UUID Rules
<!--==========================================================-->
- Use **UUID** instead of auto-increment IDs as the Primary Key (or at least as the public identifier) for models exposed via the API, so no one can infer record counts or guess other IDs (e.g., `/users/1`, `/users/2`).
- In migrations, define the field with `$table->uuid('id')->primary();` (or a separate `uuid` column if the primary ID must stay auto-increment for relationship performance reasons).
- In the model, use a shared trait (e.g., `HasUuid`) instead of duplicating the same boot logic in every model.
- Make sure the UUID generates correctly and that no error or exception occurs on insert (e.g., a non-nullable field with no default, or the trait missing from the model).
- Make sure all route model binding and relationships work correctly with the new UUID, and that nothing still treats the ID as a number (type mismatch).
- Actually test the feature (Create/Update/Delete/Fetch) to confirm there are no errors before considering it done.

<!--==========================================================-->
                Best Practices & No Over-Engineering
<!--==========================================================-->
- Follow Laravel's official best practices and idiomatic conventions — don't invent custom patterns when a standard, well-known Laravel approach already exists.
- Keep it simple: solve the actual problem in front of you, not a hypothetical future one (KISS / YAGNI). Don't add abstraction layers, configs, or generalizations "just in case" they're needed later.
- Don't over-engineer: if a straightforward controller/service solves it cleanly, don't wrap it in unnecessary patterns, extra layers, or premature optimization.
- Favor simple, boring, well-tested solutions over clever ones — clarity beats cleverness.
- If a simpler alternative achieves the same result with less complexity, prefer it, even if the more complex version seems "more professional."

<!--==========================================================-->
                    Checks Before Delivery
<!--==========================================================-->
- After any change, run the appropriate checks (e.g., `composer` checks or whatever commands the project uses) and make sure there are no syntax errors or breakage before considering the feature done.

<!--==========================================================-->
                    Before Delivery / Review
<!--==========================================================-->
- Make sure there's no leftover `dd()` or `dump()` in the code.
- Make sure queries are optimized and free of N+1 issues.
- Make sure every endpoint has proper validation.
