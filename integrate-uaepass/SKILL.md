---
name: integrate-uaepass
description: Integrate or debug UAE PASS authentication in web, iOS, Android, Expo/React Native, or similar apps. Use this skill whenever the user mentions UAE PASS, uaepass.ae, id.uaepass.ae, stg-id.uaepass.ae, Sign in with UAE PASS, OAuth callback problems, deep links, universal links, WebView vs browser auth, token exchange, PKCE, refresh tokens, or logout. This skill teaches the full authorization-code flow end to end, covers both mobile and web separately, points out official-docs vs project-implementation discrepancies, and includes copy-pasteable request/response examples.
---

# Integrate UAE PASS

Use this skill to add or repair UAE PASS as a sign-in provider without guessing OAuth details. Treat it as a full implementation playbook for a less-capable agent: explain terms, read the repo first, and do not skip steps that are only "obvious" to humans.

## What To Read First

Read these files in order before editing code:

1. `references/implementation-playbook.md`
2. `references/use-cases-and-assessment.md`
3. `references/vrds-observed-implementation.md`

If the task includes UI work, also read:

4. `references/design-guidelines.md`

## Operating Rules

- Start with evidence. Read the existing auth UI, callback routes, token storage, environment config, backend exchange code, and logout code before asking the user anything.
- Keep web and mobile mentally separate. UAE PASS documents them as different integration shapes, and the current VRDS codebases also implement them differently.
- Prefer the authorization-code flow. It is the documented primary path in the official UAE PASS authentication guide.
- Choose the correct UAE PASS use-case category before changing code. UAE PASS onboarding and assessment expect the service provider to submit a use-case flow diagram with a reference like `UC 1.1.1` or `UC 1.2.3`.
- Treat PKCE as supported-but-not-fully-documented by UAE PASS. If the target codebase already uses PKCE, preserve it. If you add PKCE to a fresh integration, mark it as "best effort, verify with tenant" unless you can prove it against tenant behavior.
- Never commit client secrets, full access tokens, or full id tokens.
- If the app has a backend, prefer server-side code exchange on web and seriously consider it on mobile too. A client secret inside a mobile app is not a real secret.
- If the backend exchange contract is missing, stop and ask for that contract instead of inventing one.

## First Pass: Discover Before Asking

Scan the repository for these items:

- Sign-in screen, provider buttons, icons, translations, and loading/error states.
- OAuth helpers for `state`, PKCE verifier/challenge, callback parsing, and secure storage.
- Web callback routes, mobile deep links, Android intent filters, iOS URL schemes, associated domains, and universal/app links.
- Environment variables for client ID, client secret, authorize URL, token URL, userinfo URL, logout URL, scopes, and redirect URIs.
- Backend exchange code that turns a UAE PASS token into the app's own session token.
- Logout cleanup for both the app session and the UAE PASS single-sign-out hop.
- Tests, lint/typecheck commands, and any auth docs already in the repo.

Only ask after this scan. Ask one compact set of questions and include the best inferred answers from code.

## Questions To Ask Only If Code And Docs Do Not Answer Them

- Which environment is being integrated right now: staging or production?
- What are the exact registered redirect URIs for web, iOS, and Android?
- Does the backend want the UAE PASS `access_token`, the `id_token`, or the full token response?
- Should mobile exchange the authorization code directly with UAE PASS, or should the backend own the code exchange?
- Is PKCE required by this tenant, merely tolerated, or unsupported?
- Should logout clear only the app session, or also redirect through the UAE PASS logout endpoint?

## Workflow

1. Read the playbook and determine whether the task is web, mobile, or both.
2. Read `references/use-cases-and-assessment.md` and identify the likely UAE PASS use case and linking model.
3. Map the target repo onto the required components:
   - authorize URL builder
   - callback handler
   - code-to-token exchange
   - backend session exchange
   - token storage
   - refresh path
   - logout path
4. Follow the relevant section in `references/implementation-playbook.md`.
5. Check `references/vrds-observed-implementation.md` for working patterns and known discrepancies.
6. If there is sign-in button work, apply `references/design-guidelines.md`.
7. Verify the full flow with concrete logs at each stage.

## Required Final Output

When you finish an implementation or debugging task, report:

- Which flow you implemented: web, mobile, or both.
- Which UAE PASS environment you targeted: staging or production.
- Which UAE PASS use case you implemented or inferred, for example `UC 1.1.1` or `UC 1.2.2`.
- Where the code now builds the authorize URL, handles callbacks, exchanges code for tokens, refreshes tokens, and logs out.
- Any discrepancy between the official UAE PASS docs and the target repo's implementation.
- Any remaining unverified item, especially PKCE support, backend token contract, or native callback registration.
