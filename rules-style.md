# Style Rules — Design System

<!--==========================================================-->
                    General Visual Identity
<!--==========================================================-->
- Overall style: dark-first theme with a **Neon Glow** effect on key elements (buttons, card borders, focus states), combined with glassmorphism (transparent/blurred) layered surfaces (modals/overlays).
- Every important interactive element (primary button, card, input on focus) must have a **subtle glow** around it instead of being flat.
- Depth through transparency: layered elements (overlay/modal) are separated from what's behind them using transparency + `backdrop-filter: blur()`, not just a different background color.
- Everything is rounded, not sharp: consistent border-radius across cards, buttons, and inputs — not a random different value per element.

<!--==========================================================-->
                    Colors (Design Tokens)
<!--==========================================================-->
- Colors are never hardcoded (`#00f2fe`) throughout the codebase; they're defined once as **CSS variables / Tailwind theme tokens** and used from that single source. Example:
  ```css
  :root {
    --color-primary: #0072ff;      /* main action color */
    --color-primary-glow: #00f2fe; /* accompanying glow color */
    --color-danger: #ff4d4d;       /* delete / danger */
    --color-danger-glow: #cc0000;
  }
  ```
- Every core color gets a transparent variant (`rgba(...)` or `/10`, `/20` in Tailwind) used for:
  - Subtle borders around cards (`border-color: rgba(primary, 0.25)`).
  - Badge/status backgrounds (`background: rgba(primary, 0.1)`).
  - The glow's box-shadow (`box-shadow: 0 0 25px rgba(primary, 0.1)`).
- Success/error/warning colors each follow the same logic (a core color + a transparent variant for glow and backgrounds).

<!--==========================================================-->
                    Buttons
<!--==========================================================-->
- Primary button: gradient background (e.g., `linear-gradient(135deg, colorA, colorB)`), with a `box-shadow` glow in the same color family.
- Hover: lifts the button slightly (`translateY(-1px)`) and increases the glow strength (a more visible `box-shadow`).
- Danger button (delete): same logic, using the danger tokens instead.
- Secondary button (cancel): no gradient, a mostly transparent or neutral background, subtle border; hover just shifts color without a strong glow.
- All buttons have a smooth `transition` (`0.2s ease`) on transform and shadow — no abrupt changes.

<!--==========================================================-->
                    Cards & Modals
<!--==========================================================-->
- Card: near-solid background, a subtle transparent border in the core color, a very light ambient glow around it, plus a regular drop shadow for depth.
- Modal overlay: a semi-transparent layer over the whole page + `backdrop-filter: blur()`, appearing with a transition (opacity + scale), never popping in abruptly.
- The modal card itself follows the same logic as the main card, but with a slightly stronger glow since it's the current focal point.
- Confirmation modals (e.g., delete) use the danger color instead of primary for the border, glow, and icon.

<!--==========================================================-->
                    Inputs
<!--==========================================================-->
- Normal input: a background darker/calmer than the card, with a subtle neutral border.
- On focus: the border takes on the core color plus a subtle glow (`box-shadow`), and any icon inside the input (if present) also takes on that same core color.
- Icons inside inputs (if any) are positioned consistently the same way across every form.

<!--==========================================================-->
                    Spacing
<!--==========================================================-->
- All spacing follows the **Tailwind spacing scale** based on 8px increments (i.e., use `p-2` (8px), `p-3` (12px), `p-4` (16px), `p-6` (24px), `p-8` (32px), etc.).
- No arbitrary spacing values (e.g., `p-[13px]` or `gap-[10px]`) unless absolutely necessary and no scale value comes close.
- The `gap` between elements within the same group (form, buttons) stays consistent across the whole project, not a different value per component.

<!--==========================================================-->
                    Light Mode / Dark Mode
<!--==========================================================-->
- This style is designed dark-first, but **must work correctly in both modes**:
  - Colors (from the tokens above) get a light-mode variant (light backgrounds, dark text) following the same core-color-and-glow logic, not identical values.
  - The glow in light mode must be much subtler than in dark mode — a strong glow on a light background looks messy, not elegant.
  - Transparency (rgba) and blur values used in modals must be checked in both modes, since the same value can look great in dark mode and insufficient in light mode.
- Use Tailwind's official dark mode mechanism (`dark:` variant) with the tokens — never toggle styles manually via JS conditionals.
- Before considering any component done, verify visually (or at least in code) that it works in both modes and that no text or element disappears/becomes unreadable in either (e.g., dark text on a dark background by mistake).

<!--==========================================================-->
                Best Practices & No Over-Engineering
<!--==========================================================-->
- Follow standard, well-known Tailwind/CSS best practices — don't invent custom patterns when a standard approach already exists.
- Keep it simple: solve the actual visual requirement in front of you, not a hypothetical future one (KISS / YAGNI). Don't add extra utility layers, configs, or generalized theming systems "just in case."
- Don't over-engineer: if a straightforward set of utility classes achieves the look cleanly, don't wrap it in unnecessary custom CSS, extra abstraction, or premature optimization.
- Favor simple, well-tested styling approaches over clever ones — clarity beats cleverness.
- If a simpler alternative achieves the same visual result with less complexity, prefer it, even if the more complex version seems "more professional."

<!--==========================================================-->
                    Before Delivery
<!--==========================================================-->
- Make sure no hardcoded color is repeated manually anywhere instead of coming from the tokens.
- Make sure all spacing comes from the Tailwind scale with no arbitrary values.
- Make sure the component has been checked in both light mode and dark mode and both are visually consistent with the rest of the project.
