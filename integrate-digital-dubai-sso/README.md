# Integrate Digital Dubai SSO

This skill helps an agent integrate or debug Digital Dubai SSO as an optional OAuth/OIDC sign-in provider.

It is designed for:

- Web apps with server-side callback routes.
- iOS and Android apps with deep links, universal links, app links, or in-app WebView auth screens.
- Expo and React Native apps that already have another sign-in provider and need Digital Dubai added without breaking existing flows.

## What It Enforces

- Inspect existing auth code before asking questions.
- Ask only for provider/backend facts that are not available in the repo.
- Use a TDD red-green-refactor loop.
- Keep Digital Dubai SSO separate from UAE PASS or other identity-provider assumptions.
- Verify callback routing, OAuth state, token exchange, backend exchange, and sign-out cleanup.
- Avoid committing secrets or logging full tokens unless explicitly gated in development.

## Example Prompt

```md
[$integrate-digital-dubai-sso](/path/to/integrate-digital-dubai-sso/SKILL.md)

Add Digital Dubai SSO to this app. Use TDD, preserve existing sign-in providers, and ask only for missing provider or backend contract details.
```

## Install

```bash
npx skills add CliffJr/agent-skills --skill integrate-digital-dubai-sso
```

For a global install:

```bash
npx skills add CliffJr/agent-skills --skill integrate-digital-dubai-sso --global
```

## Files

- `SKILL.md` - the reusable agent instructions.
- `evals/evals.json` - sample prompts for testing whether the skill guides agents correctly.

## License

MIT. See the repository `LICENSE` file.
