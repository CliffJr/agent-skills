---
name: integrate-digital-dubai-sso
description: Integrate or debug Digital Dubai SSO (DDA OAuth/OIDC) as an optional sign-in/sign-out provider in web, iOS, Android, Expo/React Native, or similar apps. Use this skill whenever the user mentions Digital Dubai SSO, DDA SSO, demoidp.dubai.gov.ae, ssoredirect, an app backend DDA token exchange, "Sign in with Digital Dubai", DDA callback issues, mobile WebView/browser SSO behavior, or asks to add/debug this provider. The skill guides a TDD-first, evidence-based implementation that reads existing auth flows first, asks only for missing facts, preserves existing sign-in providers, and avoids guessing provider/backend contracts.
---

# Integrate Digital Dubai SSO

Use this skill to add or repair Digital Dubai SSO as a feature enhancement, not as a rewrite of the auth system. Challenge unclear assumptions against the code and docs, but ask only for facts that cannot be discovered locally.

## Operating Rules

- Start with code and docs, not guesses. Read the existing sign-in providers, callback handling, environment config, API client, token storage, logout cleanup, localization, icons, and platform deep-link settings.
- Treat Digital Dubai SSO as separate from UAE PASS and other identity providers. Reuse generic patterns only when they are already shared cleanly; do not couple DDA code to another provider's request names or assumptions.
- Use TDD vertically: one behavior test, see it fail, write the smallest implementation, see it pass, then continue.
- Preserve existing sign-in and sign-out behavior unless the user explicitly asks to change it.
- Do not commit secrets, full tokens, or provider credentials. If sensitive token logging is needed, make it development-only and behind an explicit env flag.
- When the provider/backend contract is missing or contradicted by logs, stop and ask for that contract. Do not try random payload shapes until one works.

## First Pass: Discover Before Asking

Scan the repository for these items before asking the user anything:

- Current auth providers and sign-in UI: button components, route names, icons, i18n keys, ordering, loading/error states.
- OAuth/OIDC helpers: state generation, callback parsing, token exchange, userinfo calls, session storage.
- Native/web routing: Expo Router routes, React Navigation routes, framework callback routes, universal links/app links, Android intent filters, iOS URL schemes, associated domains.
- Provider config: `.env.example`, app config, build profiles, CI secrets, runtime config, staging/production separation.
- Backend API contract: endpoint path, HTTP method, headers, body shape, response shape, auth token storage.
- Sign-out cleanup: pending callback state, OAuth state, secure storage, app session token, provider logout endpoint if one exists.
- Tests and commands: unit/integration/e2e frameworks, typecheck, lint, app build commands.
- Docs: environment setup docs, context docs, ADRs, provider notes, issue comments, logs supplied by the user.

Only ask after this scan. Ask one compact set of questions and include recommended answers inferred from the code.

## Questions To Ask Only If Missing

- DDA provider values per environment: authorize URL, token URL, client ID, client secret handling, redirect URI, scope, and whether a userinfo endpoint is required.
- Backend exchange contract: endpoint, method, whether to send DDA `access_token`, `id_token`, or full token response, headers/body, and expected app JWT/session response.
- Native return strategy: whether the registered HTTPS redirect is bridged back to an app link/deep link, and the exact iOS/Android callback URLs.
- UX requirements: button text, icon asset, ordering, whether it should match another provider, and cancel/error copy.
- Sign-out requirements: app-only logout, DDA session logout endpoint, or both.
- Verification target: web only, iOS device/simulator, Android device/emulator, staging build, production build, or Expo dev client.

If code already answers a question, state the evidence instead of asking.

## TDD Workflow

### 1. Inventory The Public Interfaces

Name the public surfaces future tests can exercise:

- Config resolver and authorization URL builder.
- OAuth state generation and validation.
- Callback URL parser/router for HTTPS redirects and native deep links.
- DDA token exchange client.
- Backend JWT/session exchange client.
- Sign-in UI action and callback screen/route.
- Logout cleanup function.

Write a short plan with the first tracer bullet behavior. Do not write all tests up front.

### 2. RED: One Behavior Test At A Time

Good first tests usually cover:

- Authorization URL includes `client_id`, `redirect_uri`, `scope=openid`, `response_type=code`, and a persisted `state`.
- Callback parsing accepts both the registered HTTPS redirect, commonly `/ssoredirect`, and the native return URL/app link if the app uses one.
- State validation rejects missing, expired, or mismatched state and clears invalid state.
- DDA token request sends `grant_type=authorization_code`, `client_id`, `client_secret`, `redirect_uri`, and `code` as `application/x-www-form-urlencoded` unless the provider docs for this integration say otherwise.
- Backend exchange uses the documented app contract. Some app backends accept `POST /user/exchange-dda-token` with `Authorization: Bearer <DDA access_token>`, JSON headers, and an empty body, but confirm this in local code/docs or with the backend owner before applying it.
- UI renders "Sign in with Digital Dubai" with the Digital Dubai icon without removing existing sign-in options.
- Logout clears pending DDA state/callback data along with the app session.

Run the specific test and record the failing result before editing production code.

### 3. GREEN: Implement The Smallest Working Slice

Follow existing app style and module boundaries:

- Add a DDA-specific config helper. Keep staging and production values separate. Reject production builds using staging-looking DDA values when the app has an environment concept.
- Generate a high-entropy OAuth `state`, persist it securely when possible, and validate it before exchanging the code.
- Build the authorize URL with `URLSearchParams`; never hand-concatenate unescaped OAuth parameters.
- On native apps, match the app's established provider UX. If the app uses an in-app WebView and DDA callbacks are not reliably returned through a system browser, load the authorize URL in a WebView and intercept both the HTTPS redirect and native bridge deep links. Do not switch to a system browser unless callback delivery is configured and tested.
- On web apps, use the framework's normal redirect/callback route. Prefer server-side code-to-token exchange when the architecture supports it.
- Exchange the authorization code with the DDA token endpoint, then exchange the DDA token with the app backend exactly as the contract specifies.
- Normalize the backend response only enough to feed the existing app auth/session model. Avoid renaming DDA concepts to another provider's concepts except where a legacy backend response already returns such a field.
- Add development diagnostics that prove where the failure is: authorize URL built, callback received, state valid, DDA token returned, backend exchange status. Redact secrets by default.
- Add sign-out cleanup for DDA pending state/callbacks. Call a DDA logout endpoint only when the provider/backend docs supply one.

Run the focused test after each slice. Keep iterating until the current behavior is green.

### 4. Refactor After Green

Refactor only after tests pass:

- Extract shared OAuth helpers only if they genuinely reduce duplication and do not blur provider differences.
- Tighten error messages so users see clear auth failure/cancel states without raw provider internals.
- Move magic URLs and env names into config/docs.
- Remove temporary logs, or gate sensitive diagnostics behind an explicit dev env flag.

Run the full relevant verification set after refactoring.

## Debugging Rules

Use logs to split the flow into stages:

1. Authorization URL built.
2. DDA login page loaded.
3. Redirect/callback returned with `code` and `state` or provider `error`.
4. State validated.
5. DDA token endpoint returned `access_token` and optionally `id_token`.
6. App backend exchange returned the app JWT/session payload.
7. App stored token and completed login.

Common evidence-based responses:

- `invalid_request` and "client_id was not found": inspect the actual authorize URL emitted by the app. The browser base URL alone is not enough; confirm query parameters are present and encoded.
- Browser/session closes with no callback: inspect redirect bridge, app links/universal links, custom schemes, and whether the user left the auth sheet. Do not change token exchange code for a callback-delivery failure.
- DDA token succeeds but backend exchange fails: log HTTP status, backend message, and non-sensitive `id_token` claims such as issuer, audience, subject, and expiry. Then compare with backend validation rules.
- iOS and Android differ: inspect native URL schemes, Android intent filters, associated domains, WebView navigation interception, and whether a native rebuild is required.
- Password reset or secondary IdP pages fail inside WebView: inspect WebView navigation policy, allowed URL schemes, pop-up behavior, third-party cookies, and external-link handling before changing OAuth parameters.

## Platform Notes

### Web

- Prefer a backend callback route for exchanging code if the app has a server layer.
- Store client secrets only on the server when possible.
- Use secure, HTTP-only app session cookies when that matches the app architecture.

### iOS

- Confirm `CFBundleURLTypes`, associated domains if universal links are used, and any required rebuild after app config changes.
- WebView interception should handle both the HTTPS redirect URI and any bridge deep link back to the app.

### Android

- Confirm intent filters for the app scheme/host and app links if HTTPS callbacks open the app directly.
- Rebuild the native app after changing schemes, package names, intent filters, or plugins.

## Security And Secrets

- A client secret bundled in a mobile or browser app is not truly secret. If the backend can exchange the code, recommend that architecture. If the current backend contract requires client-side DDA token exchange, document the trade-off and keep the value in env/CI secrets, not source.
- Keep OAuth `state` mandatory.
- Never print full tokens by default. If backend debugging requires a full DDA token, add a development-only flag such as `EXPO_PUBLIC_DDA_LOG_TOKENS=true`, print clear `[Sensitive][Full]` logs, and tell the user to disable it after capture.

## Documentation Updates

Update docs as part of the feature when the repo has a place for them:

- `.env.example` with non-secret DDA variable names and safe defaults.
- Environment setup docs with staging/production redirect URI expectations and rebuild notes.
- Domain/context docs only for product language such as "Digital Dubai SSO" as an optional identity provider.
- ADR only if there is a hard-to-reverse decision, such as WebView vs system browser or backend vs client-side token exchange.

## Final Response Shape

When done, report:

- What the code already answered and what, if anything, still needs user/backend confirmation.
- The red-green-refactor cycles completed.
- Files changed and why.
- Verification commands run and their results.
- Platform rebuild requirements, especially for iOS/Android config changes.
- Any residual risk, especially backend contract ambiguity or client-secret exposure.
