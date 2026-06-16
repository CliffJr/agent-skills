# Integrate UAE PASS

This skill helps an agent integrate or debug UAE PASS authentication across web and mobile apps.

It is designed for:

- Web apps with server-side callback routes or backend token-exchange bridges.
- iOS and Android apps with deep links, universal links, app links, embedded WebView auth, or app-to-app UAE PASS handoff.
- Expo and React Native apps that need UAE PASS added without breaking existing sign-in providers.
- Teams that must satisfy UAE PASS onboarding, use-case selection, and staging assessment expectations in addition to the raw OAuth flow.

## What It Enforces

- Inspect existing auth code before asking questions.
- Separate web and mobile flows instead of assuming they are identical.
- Identify the likely official UAE PASS use case before implementation.
- Prefer authorization-code flow as the primary path.
- Preserve PKCE where it already works, but mark tenant verification honestly.
- Verify callback routing, state validation, token exchange, backend exchange, refresh-token handling, and logout cleanup.
- Call out discrepancies between official docs and project-specific implementations.
- Avoid committing secrets or logging full tokens unless explicitly gated in development.

## Example Prompt

```md
[$integrate-uaepass](/path/to/integrate-uaepass/SKILL.md)

Add UAE PASS to this app. Inspect the existing auth flow first, identify the correct UAE PASS use case, and implement the web and/or mobile authorization-code flow without guessing the backend contract.
```

## Install

```bash
npx skills add CliffJr/agent-skills --skill integrate-uaepass
```

For a global install:

```bash
npx skills add CliffJr/agent-skills --skill integrate-uaepass --global
```

## Files

- `SKILL.md` - the reusable agent instructions.
- `references/implementation-playbook.md` - the main web/mobile implementation guide.
- `references/use-cases-and-assessment.md` - official UAE PASS use-case and assessment guidance.
- `references/vrds-observed-implementation.md` - evidence from the VRDS codebases and documented discrepancies.
- `references/design-guidelines.md` - sign-in-button and related design notes.

## License

MIT. See the repository `LICENSE` file.
