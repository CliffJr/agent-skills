# agent-skills

[![skills.sh](https://skills.sh/b/CliffJr/agent-skills)](https://skills.sh/CliffJr/agent-skills)

A personal collection of reusable skills, built and refined over time. Some are adapted from existing skills to better fit specific needs; others are original.

## Skills

### integrate-digital-dubai-sso

Guides agents through integrating or debugging Digital Dubai SSO as an optional OAuth/OIDC sign-in provider across web, iOS, Android, Expo, and React Native apps.

Use it when the task mentions Digital Dubai SSO, DDA SSO, `demoidp.dubai.gov.ae`, `ssoredirect`, DDA token exchange, mobile callback handling, or "Sign in with Digital Dubai".

Install only this skill:

```bash
npx skills add CliffJr/agent-skills --skill integrate-digital-dubai-sso
```

Install it globally for supported agents:

```bash
npx skills add CliffJr/agent-skills --skill integrate-digital-dubai-sso --global
```

```md
[$integrate-digital-dubai-sso](./integrate-digital-dubai-sso/SKILL.md)

Integrate Digital Dubai SSO into this app using TDD. First inspect the existing auth flow, then ask only for missing details.
```

### integrate-uaepass

Guides agents through integrating or debugging UAE PASS authentication across web, iOS, Android, Expo, and React Native apps, with separate mobile and web flows, official use-case guidance, and onboarding-assessment checks.

Use it when the task mentions UAE PASS, `uaepass.ae`, `id.uaepass.ae`, `stg-id.uaepass.ae`, Sign in with UAE PASS, OAuth callback issues, deep links, universal links, PKCE, token exchange, refresh tokens, logout, or UAE PASS onboarding/use-case assessment.

Install only this skill:

```bash
npx skills add CliffJr/agent-skills --skill integrate-uaepass
```

Install it globally for supported agents:

```bash
npx skills add CliffJr/agent-skills --skill integrate-uaepass --global
```

```md
[$integrate-uaepass](./integrate-uaepass/SKILL.md)

Integrate UAE PASS into this app. Inspect the existing auth flow first, identify the correct UAE PASS use case, then implement the web and/or mobile authorization-code flow without guessing missing OAuth or backend contract details.
```

## Install From GitHub

The skills.sh CLI installs skills directly from public GitHub repositories:

```bash
npx skills add CliffJr/agent-skills --list
npx skills add CliffJr/agent-skills --skill integrate-digital-dubai-sso
npx skills add CliffJr/agent-skills --skill integrate-uaepass
```

You can also install from the full GitHub URL:

```bash
npx skills add https://github.com/CliffJr/agent-skills --skill integrate-digital-dubai-sso
npx skills add https://github.com/CliffJr/agent-skills --skill integrate-uaepass
```

## Manual Local Install

If you do not want to use `npx`, copy a skill folder into your local agent skills directory:

```bash
mkdir -p ~/.agents/skills
cp -R integrate-digital-dubai-sso ~/.agents/skills/integrate-digital-dubai-sso
cp -R integrate-uaepass ~/.agents/skills/integrate-uaepass
```

Then call it from any project:

```md
[$integrate-digital-dubai-sso](/Users/<you>/.agents/skills/integrate-digital-dubai-sso/SKILL.md)
[$integrate-uaepass](/Users/<you>/.agents/skills/integrate-uaepass/SKILL.md)
```

## Publishing

`skills.sh` lists public GitHub repos. After adding or updating a skill here:

1. Commit the changes in this repo.
2. Push the branch to `https://github.com/CliffJr/agent-skills`.
3. Wait for `skills.sh` to re-index the public repo badge/listing.

If the repo is private, or the branch is not pushed, `skills.sh` will not list the new skill.

## Safety

Do not commit OAuth client secrets, access tokens, ID tokens, captured auth logs, or real `.env` files. Skills in this repo should describe workflows and guardrails, not private project credentials.

## License

MIT. See [LICENSE](./LICENSE).
