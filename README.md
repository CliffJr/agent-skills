# agent-skills

A personal collection of reusable skills, built and refined over time. Some are adapted from existing skills to better fit specific needs; others are original.

## Skills

### integrate-digital-dubai-sso

Guides agents through integrating or debugging Digital Dubai SSO as an optional OAuth/OIDC sign-in provider across web, iOS, Android, Expo, and React Native apps.

Use it when the task mentions Digital Dubai SSO, DDA SSO, `demoidp.dubai.gov.ae`, `ssoredirect`, DDA token exchange, mobile callback handling, or "Sign in with Digital Dubai".

```md
[$integrate-digital-dubai-sso](./integrate-digital-dubai-sso/SKILL.md)

Integrate Digital Dubai SSO into this app using TDD. First inspect the existing auth flow, then ask only for missing details.
```

## Install Locally

Copy a skill folder into your local agent skills directory:

```bash
mkdir -p ~/.agents/skills
cp -R integrate-digital-dubai-sso ~/.agents/skills/integrate-digital-dubai-sso
```

Then call it from any project:

```md
[$integrate-digital-dubai-sso](/Users/<you>/.agents/skills/integrate-digital-dubai-sso/SKILL.md)
```

## Safety

Do not commit OAuth client secrets, access tokens, ID tokens, captured auth logs, or real `.env` files. Skills in this repo should describe workflows and guardrails, not private project credentials.
