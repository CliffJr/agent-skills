# UAE PASS Design Guidelines Notes

Use this file when the task includes sign-in UI.

## Official Source

The official UAE PASS design-guidelines pages point to downloadable button-guideline PDFs and official PNG/SVG assets. Use those official assets whenever possible instead of redrawing the mark.

Official pages used for this skill:

- `https://docs.uaepass.ae/guidelines/design-guidelines`
- `https://docs.uaepass.ae/guidelines/design-guidelines/uaepass-button-guideline`
- `https://docs.uaepass.ae/guidelines/design-guidelines/error-messages`

## Required Practical Rules

- Use official UAE PASS button artwork from the official SVG/PNG asset packs linked by UAE PASS docs.
- Do not redraw the UAE PASS logo, alter its proportions, or replace it with a generic lock/user icon.
- Keep the button clearly identifiable as UAE PASS.
- Maintain adequate padding and spacing around the mark and label.
- Keep contrast high enough for accessibility.
- Do not bury the UAE PASS action in plain inline text when a primary sign-in button is expected.

## Error Message Guidance

The official guidelines include standardized service-provider-facing error copy for some scenarios. When product requirements allow, align user-facing restriction/error messages with the official wording, especially for:

- unverified or visitor account restrictions
- signature qualification restrictions
- account-linking problems

## Safe Fallback If Exact Design Specs Are Missing In Code

If the target repo does not yet contain official UAE PASS assets:

1. Download and use the official SVG or PNG assets from the UAE PASS docs.
2. Keep the button on a neutral background.
3. Do not recolor the mark.
4. Use a clear label such as `Sign in with UAE PASS`.
5. Leave a code comment or docs note saying the final visual should be checked against the official button-guideline PDF.
