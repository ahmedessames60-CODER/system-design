# Frontend Rules — Vue.js

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
- Feature/component names must be clear and descriptive of their purpose.
- Code must be **clean** and **readable**.
- Code must be **scalable**.
- Code must be **performant** (performance-first).
- Split code into components as much as possible (Single Responsibility).

<!--==========================================================-->
                    Vue.js-Specific Rules
<!--==========================================================-->
- Use the **Composition API** with `<script setup>` instead of the Options API.
- Each component lives in a clear folder based on its feature (feature-based structure), not all dumped into one generic `components` folder in larger projects.
- Shared logic goes into **composables** (`useSomething.js`), never duplicated across components.
- State shared across multiple places goes into a **Pinia store**, not local component state.
- Props and emits must be defined with explicit types (TypeScript, or fully defined `defineProps`/`defineEmits`).
- Don't write API calls directly inside components; put them in a separate `services/` or `api/` layer.
- Use a consistent naming convention:
  - Components: PascalCase (`UserCard.vue`)
  - Composables: camelCase starting with `use` (`useAuth.js`)
  - Other files/folders: kebab-case
- Don't use `v-html` unless truly necessary, and only after confirming proper sanitization.
- Stick to the project's Design System / UI library; don't invent new components that duplicate existing functionality.

<!--==========================================================-->
                    Validation Rules
<!--==========================================================-->
- The **backend is always the source of truth**. The frontend's job is to improve UX, not replace backend validation.
- Any field validated on the backend (regex/rules) must have the **exact same validation** on the frontend.
- Validation rules and their regex patterns are written by hand (no libraries like yup/vee-validate), to keep full control and exact parity with the backend.
- All of this lives in a `helpers/validators` (or `utils/validators`) folder:
  - `patterns.js`: contains only the regex patterns, matching the backend exactly.
  - Validator function files, where each function checks **exactly one thing** (Single Responsibility), e.g.:
    ```
    isValidEmail(value)
    isValidPhone(value)
    isRequired(value)
    ```
    Never write one function that checks multiple conditions at once.
- Error messages live in one central place (`error-messages.js`), never hardcoded inside components, so they're easy to update from a single location.
- Error messages must be shown to the user in a clear, friendly way — as soon as they leave the field (on blur) or while typing (debounced), not only after submit.
- During submit, the button must be disabled/show a loading state until the request finishes, to prevent double submission.

<!--==========================================================-->
                    TypeScript Types Rules
<!--==========================================================-->
- All project types live in a central `types/` folder; types must never be written directly inside components or other files.
- Organization inside `types/`:
  - Small project: a single `index.ts` file with all types.
  - Large project: each feature/module gets its own file (e.g., `user.types.ts`, `product.types.ts`, `auth.types.ts`), all re-exported centrally from `types/index.ts`.
- Always import from the central location, e.g.:
  ```ts
  import { User, Product } from '@/types'
  ```
- Never redefine the same type in more than one place; if a type already exists, extend/edit it instead of creating a similar new one.
- Name types clearly and descriptively (e.g., `User`, `UserPayload`, `ApiResponse<T>`), avoiding generic names like `Data` or `Item` alone.

<!--==========================================================-->
                Best Practices & No Over-Engineering
<!--==========================================================-->
- Follow the language/framework's official best practices and idiomatic patterns — don't invent custom patterns when a standard, well-known approach already exists.
- Keep it simple: solve the actual problem in front of you, not a hypothetical future one (KISS / YAGNI). Don't add abstraction layers, configs, or generalizations "just in case" they're needed later.
- Don't over-engineer: if a plain function/component solves it cleanly, don't wrap it in unnecessary patterns, extra layers, or premature optimization.
- Favor simple, boring, well-tested solutions over clever ones — clarity beats cleverness.
- If a simpler alternative achieves the same result with less complexity, prefer it, even if the more complex version seems "more professional."

<!--==========================================================-->
                    Before Delivery / Review
<!--==========================================================-->
- Make sure there's no leftover `console.log` in the code.
- Make sure each component has a single responsibility, not multiple.
- Make sure there's no duplicated code (DRY).
