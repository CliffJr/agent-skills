# UAE PASS Implementation Playbook

This file is the main working guide. It teaches a less-capable agent how to produce a working UAE PASS integration from scratch.

## 1. Terms You Must Understand

- `Authorization code flow`: the user signs in at UAE PASS, UAE PASS redirects back with a short-lived `code`, and your app exchanges that code for tokens.
- `Redirect URI`: the callback URL already registered with UAE PASS. The URI in code must exactly match the URI registered in the UAE PASS portal.
- `State`: a random one-time string your app generates before redirecting the user. It protects against CSRF and callback mix-ups.
- `PKCE`: an extra OAuth protection layer. The app creates a `code_verifier`, hashes it into a `code_challenge`, sends the challenge in the authorize request, then later sends the verifier in the token request.
- `Access token`: the token used to call UAE PASS APIs such as `userinfo`.
- `Refresh token`: a longer-lived token used to request a new access token without sending the user through sign-in again.
- `Userinfo endpoint`: the UAE PASS endpoint that returns the authenticated user's profile claims.

## 2. Official UAE PASS Baseline

The official UAE PASS authentication docs describe OAuth 2.0 authorization-code flow for web and an embedded-WebView-based flow for mobile.

Official environment endpoints:

- Staging authorize: `https://stg-id.uaepass.ae/idshub/authorize`
- Staging token: `https://stg-id.uaepass.ae/idshub/token`
- Staging userinfo: `https://stg-id.uaepass.ae/idshub/userinfo`
- Staging logout: `https://stg-id.uaepass.ae/idshub/logout`
- Production authorize: `https://id.uaepass.ae/idshub/authorize`
- Production token: `https://id.uaepass.ae/idshub/token`
- Production userinfo: `https://id.uaepass.ae/idshub/userinfo`
- Production logout: `https://id.uaepass.ae/idshub/logout`

Default documented scopes:

- Citizen/resident baseline scope: `urn:uae:digitalid:profile:general`
- Common richer scope seen in docs and working code: `urn:uae:digitalid:profile:general urn:uae:digitalid:profile:general:profileType`
- Visitor flow often adds: `urn:uae:digitalid:profile:general:unifiedId`

Documented `acr_values`:

- Standard web auth: `urn:safelayer:tws:policies:authentication:level:low`
- Mobile app-installed flow: `urn:digitalid:authentication:flow:mobileondevice`

## 3. Prerequisites

Before you touch code, collect these facts:

- UAE PASS environment: staging or production
- Client ID
- Client secret
- Registered redirect URI for web
- Registered redirect URI for mobile, or the HTTPS redirect URI that the mobile app will bridge back into the app
- Whether the backend exchanges the code itself or expects the app to exchange code and pass a UAE PASS token downstream
- The backend endpoint that converts UAE PASS identity into your app session
- Required scopes
- Whether logout must hit UAE PASS logout

If any of these are missing, say exactly what is missing and why you cannot safely guess it.

## 4. Canonical End-To-End Flow

Use this order unless the target app already has a proven variation:

1. Generate high-entropy `state`.
2. If PKCE is enabled, generate `code_verifier` and `code_challenge` using SHA-256 and Base64URL encoding.
3. Build the authorize URL.
4. Send the user to UAE PASS.
5. Receive the callback at the registered redirect URI.
6. Validate `state`.
7. Exchange the authorization `code` for tokens.
8. Call UAE PASS `userinfo` if the app or backend needs profile claims directly.
9. Exchange the UAE PASS token with the app backend if the app has its own JWT/session model.
10. Store the app session token securely.
11. Store UAE PASS tokens only if the product truly needs them later, for example for signing or token refresh.
12. Implement logout and optionally the UAE PASS logout redirect.

## 5. Web Implementation

### Recommended Architecture

Use this unless the target project cannot support a server route:

- Browser builds the authorize URL and redirects to UAE PASS.
- Browser callback handler validates `state`.
- Browser sends `code`, `redirect_uri`, and optional `code_verifier` to a server route on the same app origin.
- Server route calls the UAE PASS token endpoint with the client secret.
- Server route optionally calls `userinfo`.
- Server route exchanges the UAE PASS token with the app backend.
- Browser receives only the final app session payload.

This keeps the client secret off the browser.

### Web Session And CORS Rules

- Start the OAuth flow from the same browser origin that owns the registered redirect URI. If the app starts on `http://localhost:3000` but UAE PASS redirects to `https://qa.example.com/callback`, the callback page cannot safely read the original `state` stored in `sessionStorage`.
- Prefer a same-origin server route such as `/auth-bridge/uaepass` or `/api/uaepass/callback-exchange` for the code exchange.
- If the browser must call a different origin directly, you must configure CORS deliberately. Do not assume UAE PASS endpoints or your backend already allow browser cross-origin credentials.
- If your app uses cookie sessions instead of a JWT in local storage, prefer HTTP-only secure cookies for the final app session.

### Environment Variables

Use names like these:

```bash
NEXT_PUBLIC_UAE_PASS_CLIENT_ID=your_client_id
UAE_PASS_CLIENT_SECRET=your_client_secret
NEXT_PUBLIC_UAE_PASS_REDIRECT_URI=https://app.example.com/uaepass/callback
NEXT_PUBLIC_UAE_PASS_AUTH_URL=https://stg-id.uaepass.ae/idshub/authorize
UAE_PASS_TOKEN_URL=https://stg-id.uaepass.ae/idshub/token
UAE_PASS_USERINFO_URL=https://stg-id.uaepass.ae/idshub/userinfo
NEXT_PUBLIC_UAE_PASS_SCOPE="urn:uae:digitalid:profile:general urn:uae:digitalid:profile:general:profileType"
NEXT_PUBLIC_UAE_PASS_ACR_VALUES=urn:safelayer:tws:policies:authentication:level:low
NEXT_PUBLIC_UAE_PASS_USE_PKCE=true
```

### Build The Authorization Request

Copy-pasteable example:

```ts
const params = new URLSearchParams({
  response_type: "code",
  client_id: clientId,
  redirect_uri: redirectUri,
  scope,
  acr_values: "urn:safelayer:tws:policies:authentication:level:low",
  state,
});

if (codeChallenge) {
  params.set("code_challenge", codeChallenge);
  params.set("code_challenge_method", "S256");
}

const authorizeUrl = `${authUrl}?${params.toString().replace(/\+/g, "%20")}`;
window.location.assign(authorizeUrl);
```

### Handle The Callback

Required checks:

- If `error` is present, show a user-safe error and stop.
- If `code` is missing, fail.
- If stored `state` is missing, treat the session as expired.
- If returned `state` does not match stored `state`, fail and clear local OAuth state.
- If PKCE was enabled but no stored `code_verifier` exists, fail and clear local OAuth state.

### Exchange Code For Tokens On The Server

Official docs show a token request that uses Basic authorization with `client_id:client_secret`.

Copy-pasteable example:

```ts
const basicAuth = Buffer.from(`${clientId}:${clientSecret}`).toString("base64");

const formData = new URLSearchParams({
  grant_type: "authorization_code",
  redirect_uri: redirectUri,
  code,
});

if (codeVerifier) {
  formData.set("code_verifier", codeVerifier);
}

const tokenRes = await fetch(tokenUrl, {
  method: "POST",
  headers: {
    Authorization: `Basic ${basicAuth}`,
    "Content-Type": "application/x-www-form-urlencoded",
    Accept: "application/json",
  },
  body: formData.toString(),
});
```

Expected success fields usually include:

```json
{
  "access_token": "string",
  "token_type": "Bearer",
  "expires_in": 3600,
  "refresh_token": "string",
  "scope": "string",
  "id_token": "string"
}
```

### Get User Information

If your app or backend needs claims directly from UAE PASS:

```ts
const userInfoRes = await fetch(userInfoUrl, {
  headers: {
    Authorization: `Bearer ${accessToken}`,
    Accept: "application/json",
  },
});
```

### Refresh Tokens

The official pages fetched for this skill do not document the refresh-token request explicitly, but UAE PASS token responses include `refresh_token` in examples and observed code types. Treat this as standard OAuth 2.0 behavior and verify it with your tenant before shipping.

Best-effort refresh example:

```ts
const formData = new URLSearchParams({
  grant_type: "refresh_token",
  refresh_token: refreshToken,
});

const refreshRes = await fetch(tokenUrl, {
  method: "POST",
  headers: {
    Authorization: `Basic ${basicAuth}`,
    "Content-Type": "application/x-www-form-urlencoded",
    Accept: "application/json",
  },
  body: formData.toString(),
});
```

Mark this branch in code comments as tenant-verified or unverified.

### Logout

Official logout URL shape:

```txt
https://stg-id.uaepass.ae/idshub/logout?redirect_uri=https://app.example.com/login
```

Do both:

1. Clear the app session locally and on your backend.
2. If the user authenticated through UAE PASS and product requirements expect single sign-out, redirect through the UAE PASS logout URL.

## 6. Mobile Implementation

### Official Mobile Shape

The official mobile guide assumes:

- The app uses an embedded WebView.
- The app checks whether UAE PASS is installed.
- If installed, it starts a mobile-on-device flow.
- The app intercepts the UAE PASS deep link, rewrites success and failure callback URLs to the app's own scheme, opens the UAE PASS app, then resumes the WebView flow when the app returns.
- If the app is not installed, the app keeps the user in the embedded WebView and UAE PASS pushes confirmation to another device.

### Required Native Setup

Register all callback paths before debugging runtime code:

- Custom URL scheme for the app, for example `vrds://`
- Android intent filters for the scheme/host or app link
- iOS `CFBundleURLTypes`
- Universal links or app links only if the product uses them intentionally
- Expo/React Native `Linking.canOpenURL` support for UAE PASS schemes

Official mobile-app detection details from UAE PASS docs:

- Android staging package ID: `ae.uaepass.mainapp.stg`
- iOS production scheme: `uaepass://`
- iOS staging scheme: `uaepassstg://`

In practice, Android apps often also probe `uaepass://` or `uaepassstg://` via `canOpenURL`-style checks in addition to package visibility rules.

### Recommended Mobile Architecture

Safer architecture:

- Use WebView or system browser only for the authorize step.
- Receive the HTTPS redirect or bridge deep link in the app.
- Send the `code` to your backend.
- Let the backend exchange code for tokens using the client secret.

Fallback architecture used in some existing apps:

- Mobile app exchanges the code directly with UAE PASS using a bundled client secret.
- Mobile app then exchanges the UAE PASS access token with the app backend.

If you must use the fallback, document the secret-exposure tradeoff in code comments or docs.

### Build The Authorization URL

Use the same OAuth parameters as web. Only `acr_values` changes when the UAE PASS app is installed.

App installed:

```txt
acr_values=urn:digitalid:authentication:flow:mobileondevice
```

App not installed:

```txt
acr_values=urn:safelayer:tws:policies:authentication:level:low
```

### Intercept And Rewrite UAE PASS App Links

The official guide requires you to:

1. Detect the UAE PASS deep link such as `uaepass://...` or `uaepassstg://...`.
2. Read its `successURL` and `failureURL`.
3. Replace those URLs with your app's own deep link, for example:

```txt
uaepassstg://...?successURL=yourapp:///resume_authn?url=<encoded_original_success>&failureURL=yourapp:///resume_authn?url=<encoded_original_failure>
```

4. Open the rewritten link so the UAE PASS app handles authentication.
5. When the app receives `yourapp:///resume_authn?...`, extract the original URL and load it back inside the WebView.
6. Continue until the WebView reaches the registered redirect URI with `code` and `state`.

### Token Exchange

If mobile exchanges the code directly:

```ts
const basicAuth = base64(`${clientId}:${clientSecret}`);

const body = new URLSearchParams({
  grant_type: "authorization_code",
  redirect_uri: redirectUri,
  code,
});

if (codeVerifier) {
  body.set("code_verifier", codeVerifier);
}

const response = await fetch(tokenUrl, {
  method: "POST",
  headers: {
    Authorization: `Basic ${basicAuth}`,
    "Content-Type": "application/x-www-form-urlencoded",
    Accept: "application/json",
  },
  body: body.toString(),
});
```

Then exchange the UAE PASS token with the app backend:

```ts
const backendRes = await fetch(`${API_BASE_URL}/user/exchange-token`, {
  method: "POST",
  headers: {
    Authorization: `Bearer ${uaePassAccessToken}`,
    "App-Type": Platform.OS === "ios" ? "ios" : "android",
    "Content-Type": "application/json",
    Accept: "application/json",
  },
  body: "{}",
});
```

### Token Storage

Store:

- OAuth `state` and PKCE verifier in secure storage
- Callback URLs that may arrive before the auth screen remounts
- App JWT/session token in secure storage
- UAE PASS access token only if the product really needs it later

Do not store secrets or tokens in plain async storage if secure storage is available. Use secure storage for tokens and async storage only for non-secret callback-resume breadcrumbs.

### Refresh Tokens

If your product needs long-lived UAE PASS API access on mobile:

1. Keep the UAE PASS `refresh_token` in secure storage.
2. Refresh through the same token endpoint.
3. Rotate stored tokens on every successful refresh.
4. If refresh fails with invalid_grant or expired token, clear UAE PASS token state and send the user through full sign-in again.

Because the official docs fetched for this skill do not spell this out, mark the refresh implementation as tenant-verified only after you prove it.

### Logout

Do these in order:

1. Clear pending OAuth state, PKCE verifier, and any stored callback URL.
2. Clear app session token.
3. Clear any stored UAE PASS token if present.
4. If product requirements need UAE PASS SSO logout, open the logout URL with a valid redirect back to your sign-in screen.

## 7. PKCE Guidance

UAE PASS official pages used for this skill document the authorization-code flow but do not explicitly document PKCE parameters. However, both VRDS web and VRDS mobile codebases already support PKCE successfully enough to preserve it.

Therefore:

- If the target repo already uses PKCE, keep it.
- If you add PKCE to a fresh integration, label it as "best effort / verify with tenant".
- Use `code_challenge_method=S256`.
- Store the `code_verifier` securely and clear it immediately after code exchange or hard failure.

Copy-pasteable browser/mobile PKCE helper:

```ts
async function toBase64UrlSha256(value: string): Promise<string> {
  const encoded = new TextEncoder().encode(value);
  const hashBuffer = await crypto.subtle.digest("SHA-256", encoded);
  const bytes = new Uint8Array(hashBuffer);
  const hash = btoa(String.fromCharCode(...bytes));
  return hash.replace(/\+/g, "-").replace(/\//g, "_").replace(/=+$/, "");
}
```

## 8. Common Failure Modes And How To Debug Them

Split the flow into stages and log each stage:

1. Authorize URL built
2. UAE PASS login page opened
3. Callback received
4. `state` validated
5. Token endpoint returned JSON
6. Backend exchange succeeded
7. App session stored

Common issues:

- Redirect URI mismatch
  - Symptom: UAE PASS returns `invalid_request` or never returns to the app.
  - Fix: compare the exact runtime `redirect_uri` string against the value registered in UAE PASS. Exact means scheme, host, path, trailing slash, and encoding.
- Missing or expired `state`
  - Symptom: callback arrives but app rejects it.
  - Fix: persist `state` before redirecting, give it a TTL, clear it after single use.
- Missing PKCE verifier
  - Symptom: token exchange fails after callback.
  - Fix: make sure the verifier survives app backgrounding and callback routing.
- User cancellation
  - Symptom: callback contains `error`, or mobile waiting page reports failure.
  - Fix: map cancellation to a normal user-facing message, not a scary crash message.
- Token endpoint returns HTML instead of JSON
  - Symptom: parsing fails or response looks like a login page.
  - Fix: inspect the request shape, auth method, cookies, and whether you accidentally hit the authorize flow instead of token flow.
- Backend rejects UAE PASS token
  - Symptom: UAE PASS token exchange succeeds, but app login still fails.
  - Fix: confirm whether the backend expects `access_token`, `id_token`, or something else; log redacted issuer, audience, subject, and expiry claims if relevant.
- Callback lands on the wrong browser origin on web
  - Symptom: callback page cannot read stored `state`.
  - Fix: start the auth flow only from the same origin that owns the redirect URI.
- Clock skew during token validation
  - Symptom: backend says token expired even though login just completed.
  - Fix: check server/device clock drift and token validation leeway.

## 9. Minimum Verification Checklist

- Sign-in button uses official UAE PASS styling/assets
- Authorize URL contains expected query parameters
- Callback returns with `code` and matching `state`
- Code exchange succeeds
- Userinfo call succeeds if used
- Backend exchange succeeds
- App session is stored
- Logout clears local session
- UAE PASS logout redirect works if required
- At least one failure path is tested: user cancel or mismatched `state`

## 10. Onboarding And Assessment Checklist

Before calling the work done, also check these business-flow items from the UAE PASS use-case guidelines:

- The likely use case is identified, for example `UC 1.1.1`, `UC 1.2.2`, or `UC 1.3.1`.
- UAE PASS sign-in is a separate action from local username/password sign-in.
- The implemented linking method matches the chosen use case.
- If automatic linking is used, the linking field is unique and already verified.
- If manual linking is used, the user completes a real SP-side account-ownership step.
- If new-user registration is supported, UAE PASS-populated fields are non-editable.
- The SP stores UAE PASS UUID after first successful link or registration.
- The implementation does not prompt the user to create or receive a local password inside the UAE PASS registration journey.
- Web and mobile client credentials are treated as channel-specific unless UAE PASS explicitly approved otherwise.
