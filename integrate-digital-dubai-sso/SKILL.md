---
name: integrate-digital-dubai-sso
description: Integrate or debug Digital Dubai SSO (DDA OAuth/OIDC) as an optional sign-in/sign-out provider in web, iOS, Android, Expo/React Native, or similar apps. Use this skill whenever the user mentions Digital Dubai SSO, DDA SSO, demoidp.dubai.gov.ae, ssoredirect, "Sign in with Digital Dubai", DDA token exchange, backend token exchange, mobile callback issues, WebView vs browser SSO behavior, or asks to add/debug this provider. The skill guides a TDD-first, evidence-based implementation that reads existing auth flows first, asks only for missing facts, preserves existing sign-in providers, and avoids guessing provider/backend contracts.
---

# Integrate Digital Dubai SSO

Use this skill to add or repair Digital Dubai SSO as a feature enhancement, not as a rewrite of the auth system. The agent should move like `grill-with-docs`: challenge unclear assumptions against the code and docs, but ask only for facts that cannot be discovered locally.

## Operating Rules

- Start with code and docs, not guesses. Read the existing sign-in providers, callback handling, environment config, API client, token storage, logout cleanup, localization, icons, and platform deep-link settings.
- Treat Digital Dubai SSO as separate from any existing auth provider. Reuse generic patterns only when they are already shared cleanly; do not couple DDA code to another provider's request names or assumptions.
- Use TDD vertically: one behavior test, see it fail, write the smallest implementation, see it pass, then continue.
- Preserve existing sign-in and sign-out behavior unless the user explicitly asks to change it.
- Do not commit secrets, full tokens, or provider credentials. If sensitive token logging is needed, make it development-only and behind an explicit env flag.
- When the provider/backend contract is missing or contradicted by logs, stop and ask for that contract. Do not try random payload shapes until one works.

## Reusable DDA Mobile Evidence

Use this as evidence from a working Expo/React Native Digital Dubai SSO integration, not as a universal contract. Always confirm tenant-specific values, redirect URIs, callback bridge URLs, and backend exchange requirements from the app's docs, environment, DDA portal configuration, or backend owner.

Observed DDA STG provider endpoints often look like:

- Authorize URL: `https://demoidp.dubai.gov.ae/mga/sps/oauth/oauth20/authorize`
- Token URL: `https://demoidp.dubai.gov.ae/mga/sps/oauth/oauth20/token`
- Scope: commonly `openid`
- Registered redirect URI: app/tenant-specific HTTPS URL, often ending in `/ssoredirect`
- Native bridge route: app-specific custom scheme or app/universal link that carries `code` and `state` back into the app

Observed token-exchange behavior:

- A working React Native tenant used `POST` form-encoded body with `grant_type=authorization_code`, `client_id`, `client_secret`, `redirect_uri`, and `code`.
- In React Native, setting `credentials: "omit"` on the DDA token `fetch` prevented Android from attaching the DDA WebView/user session cookie to `/token`.
- Do not switch client authentication to HTTP Basic just because another OAuth provider uses it. In one DDA STG tenant, switching to HTTP Basic caused a `200 text/html` response containing the Smart Dubai SSO login page instead of JSON tokens. Use the provider/backend contract for the tenant being debugged.
- App-backend exchange is application-specific. Some apps exchange the DDA `access_token` with their backend using `Authorization: Bearer <DDA access_token>` and no body; others may require `id_token`, the full token response, or a JSON body. Confirm before coding.

Successful iOS and Android evidence for a native app should have this shape:

1. Authorization URL is built with `client_id`, `redirect_uri`, `scope`, `response_type=code`, and `state`.
2. WebView/browser reaches the registered redirect URI with fresh `code` and matching `state`.
3. State validation succeeds.
4. DDA token request returns JSON with an access token and usually `token_type`/`expires_in`; an `id_token` may also be present.
5. Non-sensitive `id_token` claims, when available, show expected `iss`, `aud=<DDA client_id>`, `sub=<DDA user id/email>`, and non-expired `exp`.
6. App-backend exchange returns the app session/JWT/user payload expected by the existing auth model.
7. The app stores the app session and completes login.

## First Pass: Discover Before Asking

Scan the repository for these items before asking the user anything:

- Current auth providers and sign-in UI: button components, route names, icons, i18n keys, ordering, loading/error states.
- OAuth/OIDC helpers: state generation, callback parsing, token exchange, userinfo calls, session storage.
- Native/web routing: Expo Router routes, React Navigation routes, Next/Remix routes, universal links/app links, Android intent filters, iOS URL schemes, associated domains.
- Provider config: `.env.example`, app config, build profiles, CI secrets, runtime config, staging/production separation.
- Backend API contract: endpoint path, HTTP method, headers, body shape, response shape, auth token storage.
- Sign-out cleanup: pending callback state, OAuth state, secure storage, app session token, provider logout endpoint if one exists.
- Tests and commands: unit/integration/e2e frameworks, typecheck, lint, app build commands.
- Docs: environment setup docs, context docs, ADRs, provider notes, issue comments, logs supplied by the user.

Only ask after this scan. Ask one compact set of questions and include recommended answers inferred from the code.

## Questions To Ask Only If Missing

- DDA provider values per environment: authorize URL, token URL, client ID, client secret handling, redirect URI, scope, and whether a userinfo endpoint is required.
- Backend exchange contract: endpoint, method, whether to send DDA `access_token`, `id_token`, or full token response, headers/body, and expected app JWT response.
- Native return strategy: whether the registered HTTPS redirect is bridged back to an app link/deep link, and the exact iOS/Android callback URLs.
- UX requirements: button text, icon asset, ordering relative to existing providers, and cancel/error copy.
- Sign-out requirements: app-only logout, DDA session logout endpoint, or both.
- Verification target: web only, iOS device/simulator, Android device/emulator, QA build, production build, or Expo dev client.

If code already answers a question, state the evidence instead of asking.

## TDD Workflow

### 1. Inventory The Public Interfaces

Name the public surfaces future tests can exercise:

- Config resolver and authorization URL builder.
- OAuth state generation and validation.
- Callback URL parser/router for HTTPS redirects and native deep links.
- DDA token exchange client.
- Backend JWT exchange client.
- Sign-in UI action and callback screen/route.
- Logout cleanup function.

Write a short plan with the first tracer bullet behavior. Do not write all tests up front.

### 2. RED: One Behavior Test At A Time

Good first tests usually cover:

- Authorization URL includes `client_id`, `redirect_uri`, `scope=openid`, `response_type=code`, and a persisted `state`.
- Callback parsing accepts both the registered HTTPS redirect, commonly `/ssoredirect`, and the native return URL/app link if the app uses one.
- State validation rejects missing, expired, or mismatched state and clears invalid state.
- DDA token request sends `grant_type=authorization_code`, `redirect_uri`, and `code`; include `client_id` and `client_secret` according to the tenant's documented client-auth method. If the tenant uses form-body client auth in React Native, include `credentials: "omit"` to avoid Android cookie/session collisions.
- Backend exchange uses the documented app contract. Confirm endpoint path, whether to send DDA `access_token`, `id_token`, or full token response, and whether the backend expects a bearer header, JSON body, or empty body.
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

- Extract shared OAuth helpers only if they genuinely reduce duplication and do not blur DDA-specific behavior with another provider's behavior.
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
- `FBTOAU220E The authenticated client id ... does not match the client id in the request body`: split the problem at the token request. If callback `code` and `state` are fresh and state validates, do not rewrite callback handling. In React Native, suspect Android or WebView session cookies being sent to `/token`; try `credentials: "omit"` while preserving the tenant's documented client-auth method.
- `200` with `content-type: text/html` from the DDA token endpoint and an HTML Smart Dubai SSO page: the token endpoint did not accept the request as a token exchange. Re-check the tenant's client-auth method before changing redirect/callback code; in one working STG tenant, HTTP Basic auth caused this and form-body `client_secret` was required.
- `Digital Dubai IdP did not return an access token`: log status, content type, parsed keys, OAuth error fields, and a redacted text preview. If the response is HTML, treat it as a token-request shape/session issue, not a backend issue.
- Browser/session closes with no callback: inspect redirect bridge, app links/universal links, custom schemes, and whether the user left the auth sheet. Do not change token exchange code for a callback-delivery failure.
- DDA token succeeds but backend exchange fails: log HTTP status, backend message, and non-sensitive `id_token` claims such as issuer, audience, subject, and expiry. Then compare with backend validation rules.
- iOS and Android differ: first compare the stage reached by logs. If both platforms reach callback and state validation, focus on token fetch behavior, cookies/credentials, and request body/header shape. If Android never reaches callback, inspect Android intent filters, WebView navigation interception, app links, and whether a native rebuild is required.
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
- For token exchange failures after a valid callback, remember that Android WebView/session cookies can affect React Native network requests. Prefer proving the fetch request shape and using `credentials: "omit"` before changing OAuth parameters.

## Security And Secrets

- A client secret bundled in a mobile or browser app is not truly secret. If the backend can exchange the code, recommend that architecture. If the current backend contract requires client-side DDA token exchange, document the trade-off and keep the value in env/CI secrets, not source.
- Keep OAuth `state` mandatory.
- Never print full tokens by default. If backend debugging requires a full DDA token, add a development-only flag such as `EXPO_PUBLIC_DDA_LOG_TOKENS=true`, print clear `[Sensitive][Full]` logs, and tell the user to disable it after capture.
- If a user pastes client secrets or full DDA access/id tokens into chat or logs, tell them to rotate the secret/token where possible, disable `EXPO_PUBLIC_DDA_LOG_TOKENS`, and remove full-token logging before committing.

## Documentation Updates

Update docs as part of the feature when the repo has a place for them:

- `.env.example` with non-secret DDA variable names and safe defaults.
- Environment setup docs with QA/prod redirect URI expectations and rebuild notes.
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
