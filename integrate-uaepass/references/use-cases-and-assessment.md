# UAE PASS Use Cases And Assessment

Use this file before implementation planning. It explains how UAE PASS expects service providers to choose an approved business flow, not just a technical OAuth flow.

## Why This Matters

UAE PASS says the service provider should submit a use-case flow diagram early in onboarding, include the official use-case number, and expect UAE PASS to assess the staging implementation against that approved flow before go-live. That means a technically working OAuth flow can still be wrong if it implements the wrong linking or registration journey.

## Official Sources

- Use-Cases overview: `https://docs.uaepass.ae/guidelines/use-cases`
- Standard Authentication Scenarios: `https://docs.uaepass.ae/guidelines/use-cases/standard-authentication-scenarios-for-service-provider-use-cases`
- Standard Digital Signature Scenarios: `https://docs.uaepass.ae/guidelines/use-cases/standard-digital-signature-scenarios-for-service-provider-use-cases`
- Standard Implementation Guidelines: `https://docs.uaepass.ae/guidelines/use-cases/standard-implementation-guidelines`

## What To Decide Before Coding

Figure out which of these three broad authentication categories the service provider needs:

- Existing users only
- Existing users plus new user registration
- New user registration only

Then decide how linking works:

- Automatic linking
- Manual linking
- Automatic plus manual linking

If the product also uses UAE PASS digital signing, identify whether it is:

- `2.1.1` single signer digital signature flow
- `2.1.2` multiple signer digital signature flow

## Authentication Use-Case Shortlist

### Category 1: Existing Users Only

- `UC 1.1.1`: existing users with automatic linking only
  - Use when the SP already has a unique and verified attribute in its database.
  - UAE PASS says `SPUUID` and Emirates ID number are the acceptable unique verified linking attributes called out in this flow.
  - Brief acceptable SOP guidance from the official table: SOP2 and SOP3.

- `UC 1.1.2`: existing users with manual linking only
  - Use when the SP does not already have a unique verified linking attribute.
  - The account is linked after the user proves ownership through the SP's own local authentication.
  - Official table allows SOP1, SOP2, and SOP3.

- `UC 1.1.3`: existing users with automatic and manual linking
  - Use when some existing users can be auto-linked and others need manual linking.

### Category 2: Existing Users And New User Registration

- `UC 1.2.1`: automatic linking for existing users and new user registration
- `UC 1.2.2`: manual linking for existing users and new user registration
- `UC 1.2.3`: automatic and manual linking for existing users plus new user registration

For all three `1.2.x` flows, the official guidance says:

- The SP should determine whether the person is an existing user with no online access or a truly new user.
- The user should not be prompted to create a username and password during the UAE PASS registration journey.
- If the SP creates credentials in the backend, they must not be shared over email or SMS.
- Data auto-populated from UAE PASS must be non-editable.

### Category 3: New User Registration Only

- `UC 1.3.1`: new user registration only
  - Use when only new-user onboarding is allowed through the SP channel.
  - The same rules apply: do not prompt for username/password during the UAE PASS registration journey, do not share backend-created credentials, and keep UAE PASS-populated fields non-editable.

## How To Choose Automatic vs Manual Linking

Choose automatic linking when:

- The SP already has a unique and verified attribute for the user.
- The attribute can be matched safely against UAE PASS data.

Official acceptable automatic-linking methods include:

- Verified Emirates ID match
- Verified email match, if email is unique at the SP
- Verified email plus mobile-number match
- For visitor onboarding, email or mobile-based linking

Choose manual linking when:

- The SP does not have a unique and verified attribute already stored.
- The user must prove account ownership through the SP's own IAM.

Official manual-linking methods include:

- Asking for SP local username and password
- Asking the user to answer secret questions

## Implementation-Assessment Rules UAE PASS Explicitly Checks

These points should be treated as assessment requirements, not optional UX advice.

- UAE PASS login and local login must be independent.
  - Do not merge `Sign in with UAE PASS` into the existing username/password flow.
- Linking identifiers must be unique and verified.
- The chosen SOP level should be shown in the submitted use-case diagram.
- UAE PASS UUID must be stored after linking or registration.
- Verified UAE PASS should ideally not be linked to non-verified local accounts, and vice versa.
- During UAE PASS registration, do not ask users to create local usernames or passwords.
- During UAE PASS sign-in, do not offer local password change.
- Do not share newly created local account credentials during or after UAE PASS registration.
- Plan complete release coverage where possible.
  - UAE PASS says best experience comes from covering automatic linking, manual linking, and new-user registration together for go-live.
- Stage onboarding uses dev or stage redirect URLs; production redirect URLs are required later for go-live after sign-off.
- UAE PASS client credentials are channel-specific.
  - Web credentials and mobile credentials should not be reused across channels without UAE PASS approval.
- Avoid using UAE PASS for a narrow, partial journey only.
  - UAE PASS says it should support the full authentication journey, not just isolated tasks like updating email/mobile data.

## Verified Attribute Rules

The official implementation guidelines define what counts as a verified attribute. This matters for deciding whether automatic linking is allowed.

Examples called out by UAE PASS:

- Emirates ID can be verified through online biometric/ICA/MOI methods or face-to-face/agent-assisted methods.
- Email and mobile can be verified through OTP/password or activation-link flows.

A user-entered Emirates ID, email, or mobile number by itself is not considered verified unless one of the accepted verification methods has already happened.

## Additional SP Verification That Is Allowed

The official guidelines allow extra SP verification, but only at the right time:

- Email/mobile OTP verification during account linking is accepted.
- OTP verification during sign-up is accepted after UAE PASS authentication is completed.
- Security-question setup should happen after UAE PASS authentication, not before it.
- PIN/Face ID setup should also happen after the UAE PASS journey.

## How To Apply This In A Coding Task

Before editing code, write one short sentence in your notes or final response:

- `Likely UAE PASS use case: UC 1.1.1 automatic linking for existing users only`
- `Likely UAE PASS use case: UC 1.2.2 manual linking plus new-user registration`

Then confirm the code matches that use case:

- Does the UI separate UAE PASS sign-in from local sign-in?
- Does linking use a verified unique attribute, or a real manual-linking step?
- If this is a registration flow, are UAE PASS-populated fields non-editable?
- Is UUID stored after first successful link or registration?
- Are web and mobile credentials kept channel-specific?

If any answer is no, say the implementation is not just incomplete technically; it may also fail UAE PASS onboarding assessment.
