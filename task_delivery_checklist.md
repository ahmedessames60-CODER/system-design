# Task Delivery & Code Review Checklist (Defensive Engineering)

<!--==========================================================-->
                    1. Functionality & Logic
<!--==========================================================-->
- [ ] **Edge Cases:** Are functions & helpers tested against zero, negative, null, undefined, or empty data inputs?
- [ ] **UI States:** Are all UI states covered gracefully (Loading, Empty Data, Error, and Success)?
- [ ] **Single Responsibility:** Does each function and component perform exactly one task without over-complication?
- [ ] **Type Safety:** Are props, emits, payloads, and response interfaces strictly typed without generic or `any` fallbacks?

<!--==========================================================-->
                2. User Journey & E2E Validation
<!--==========================================================-->
- [ ] **Critical Journeys:** Is the entire feature flow (Open -> Interact -> Validate -> Submit -> Success Feedback) working seamlessly?
- [ ] **Form & Input Validation:** Is validation triggered at the right time (on blur, or debounced on input) matching backend rules exactly?
- [ ] **Double Submission:** Is the action button disabled with a loading state while requests are pending to prevent duplicate triggers?

<!--==========================================================-->
                3. Performance, Memory & Network
<!--==========================================================-->
- [ ] **List Virtualization:** Are large lists rendered via virtualization/pagination instead of overwhelming the DOM?
- [ ] **Memory Cleanups:** Are all active event listeners, intervals, timeouts, and subscriptions properly cleaned up on unmount?
- [ ] **Race Conditions:** Are stale or pending API calls cancelled (using `AbortController`) during rapid user input changes?
- [ ] **Code Splitting:** Are heavy or non-critical components lazy-loaded?

<!--==========================================================-->
                    4. Security & Sanitization
<!--==========================================================-->
- [ ] **Input Cleaning:** Is input sanitized against XSS before rendering via raw HTML (`v-html` / `dangerouslySetInnerHTML`)?
- [ ] **No Credentials Leakage:** Are API keys, tokens, or sensitive values removed from logs, responses, and codebase?
- [ ] **Auth Interceptors:** Do secure endpoints automatically receive the authorization headers without manual intervention?

<!--==========================================================-->
                5. UX, Accessibility & Styling
<!--==========================================================-->
- [ ] **Design Tokens:** Are colors, spacing, and radii sourced strictly from project tokens without hardcoded values?
- [ ] **Dark & Light Themes:** Has the component been tested in both dark and light modes to ensure high contrast and readability?
- [ ] **Keyboard & Screen Readers:** Is the interactive component fully accessible via `Tab` / `Esc` with correct `aria` attributes?
- [ ] **Responsive Design:** Is the layout fluid and free of unwanted horizontal scrolling across all viewports?

<!--==========================================================-->
                6. Architecture & Rules Alignment
<!--==========================================================-->
- [ ] **Separation of Concerns:** Is business logic kept inside Composables/Services/Hooks rather than directly inside UI templates?
- [ ] **Backend Source of Truth:** Does the frontend enhance UX without replacing backend authority?
- [ ] **Localization & Error Messages:** Are error texts pulled from a central error mapping file rather than hardcoded string literals?

<!--==========================================================-->
                    7. Pre-Delivery Cleanup
<!--==========================================================-->
- [ ] **No Debug Statements:** Are all `console.log()`, `dd()`, `dump()`, or temporary dev flags removed?
- [ ] **Clean Code & DRY:** Is there zero redundant or duplicated code across components or handlers?
- [ ] **Git Commits:** Are commit messages clear, descriptive, and linked to the specific task/feature scope?