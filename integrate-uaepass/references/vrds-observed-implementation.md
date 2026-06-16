# Observed UAE PASS Implementation In VRDS

This file records evidence from the current VRDS codebases so future agents know what already works and where it differs from the official UAE PASS docs.

## 1. VRDS Web Observations

Observed in `vrds-web`:

- Flow type: authorization-code flow
- Redirect starts in the browser
- `state` is generated in the browser and stored in `sessionStorage`
- PKCE is optional and controlled by env
- Callback can land on both `/login` and `/redirect`
- Browser sends `code`, `redirect_uri`, and optional `code_verifier` to a server route
- Server route exchanges the code with UAE PASS using HTTP Basic auth and `application/x-www-form-urlencoded`
- Server route forwards either UAE PASS `id_token` or `access_token` to the VRDS backend endpoint `/user/exchange-token`
- Browser stores only the final VRDS session token in the app auth store
- Logout clears the app session and then optionally redirects through the UAE PASS logout URL

Important web-specific behavior:

- The app refuses to start auth if the current browser origin does not match the redirect URI origin.
- The callback handler maps several cancellation-shaped errors, including `cancelledOnApp`, `cancelled`, `user_cancel`, and some `invalid_request` or `access_denied` cases that actually mean the user cancelled.

## 2. VRDS Mobile Observations

Observed in sibling `vrds-mobile`:

- Flow type: authorization-code flow
- Mobile integration uses an embedded WebView, matching the official mobile guide
- PKCE is enabled by default unless explicitly disabled
- OAuth `state` and PKCE verifier are stored in secure storage
- OAuth `state` expires after 10 minutes and is single-use
- The app stores callback URLs in async storage for up to 2 minutes so callback delivery survives remounts
- The app detects whether UAE PASS is installed and switches `acr_values`
- If UAE PASS is installed, the app rewrites `successURL` and `failureURL` inside the UAE PASS deep link to a local app route such as `vrds://resume_authn?...`
- The app then re-injects the original success URL back into the WebView after returning from UAE PASS
- Mobile token exchange currently happens on-device, not on a backend bridge
- After receiving the UAE PASS access token, the app calls the VRDS backend `/user/exchange-token`
- The app stores both the VRDS token and, when present, the UAE PASS token

Important mobile-specific behavior:

- Android has extra cache-clearing recovery logic before retrying some failed starts.
- WebView cookies are disabled with `sharedCookiesEnabled={false}` and `thirdPartyCookiesEnabled={false}`.
- Mobile sign-in handles both direct redirect-URI callbacks and app deep links that carry `code`, `state`, or error fields.

## 3. Discrepancies Between Official Docs And VRDS

These must be called out rather than silently normalized:

### Discrepancy A: Web uses a backend bridge, docs show direct token and userinfo calls

- Official docs explain the token call and userinfo call directly.
- VRDS web uses a server route so the browser never sees the UAE PASS client secret.

Recommendation:

- Treat the VRDS web pattern as safer for web and prefer it for new web integrations.

### Discrepancy B: Mobile bundles the UAE PASS client secret

- Official docs describe mobile API steps and WebView/app-link behavior but do not deeply discuss secret management.
- VRDS mobile currently stores the client secret in app config and performs the token exchange on-device.

Recommendation:

- Keep this only when the backend cannot own the code exchange yet.
- Prefer migrating mobile to a backend bridge if possible.

### Discrepancy C: PKCE is used in VRDS but not explicitly documented in the fetched UAE PASS pages

- VRDS web supports optional PKCE.
- VRDS mobile uses PKCE by default.
- The fetched UAE PASS authentication pages do not explicitly document `code_challenge` or `code_verifier`.

Recommendation:

- Preserve PKCE where it already works.
- Mark PKCE as tenant-verified only after confirming against real UAE PASS behavior in that tenant/environment.

### Discrepancy D: VRDS web does not call userinfo

- Official docs include a userinfo step after token exchange.
- VRDS exchanges the UAE PASS token with the backend immediately and relies on the backend session response.

Recommendation:

- This is acceptable if the backend owns identity mapping and the app does not need raw UAE PASS claims on the client.

### Discrepancy E: VRDS web callback supports two routes

- Official docs talk about a single registered redirect URI.
- VRDS web handles callbacks on both `/login` and `/redirect`.

Recommendation:

- Use exactly one registered redirect URI per environment when possible.
- Supporting multiple local callback handlers is fine if they all validate the same `state` and only one actual registered URI is used.

## 4. What VRDS Does Not Prove

Do not overclaim based on VRDS:

- It does not prove that every UAE PASS tenant supports PKCE.
- It does not prove that every backend should use `/user/exchange-token`.
- It does not prove that a refresh-token flow is accepted by every tenant.
- It does not prove that WebView is the best UX for every mobile app.

Use VRDS as working evidence, not as universal law.
