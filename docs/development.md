# Development

This document provides instructions for developing the Google Workspace
extension.

## Development setup

### Prerequisites

- **Node.js:** Use `~20.19.0` for development because of an upstream development
  dependency issue. Production use supports Node.js `>=20`.
- **Git**

### Clone and install

```bash
git clone https://github.com/gemini-cli-extensions/workspace.git
cd workspace
npm ci
```

Use your fork's URL when cloning a fork. `npm ci` installs the exact dependency
versions recorded in `package-lock.json` and runs the root `prepare` build.

### Build

```bash
npm run build
```

This builds the workspace packages, including the MCP server and headless login
utility. See the root and `workspace-server` `package.json` files for the exact
scripts.

## Tests, lint, and formatting

Run the test suite and lint checks from the repository root:

```bash
npm run test
npm run lint
```

Check formatting using the script defined in `package.json`:

```bash
npm run format:check
```

To apply formatting instead, run:

```bash
npm run format:fix
```

Type-check the server with:

```bash
npx tsc --noEmit --project workspace-server
```

To run tests for one file, pass its path from the repository root:

```bash
npm run test -- workspace-server/src/__tests__/GmailService.test.ts
```

CI runs lint, Prettier checking, type checking, tests, and a build. Run the
relevant checks before submitting changes. Optionally install a pre-commit hook:

```bash
cat <<'EOF' > .git/hooks/pre-commit
#!/bin/sh
npm run test && npm run lint && npm run format:check
EOF
chmod +x .git/hooks/pre-commit
```

## Testing with Gemini CLI

```bash
gemini extensions uninstall google-workspace
npm ci
npm run build
gemini extensions link .
gemini extensions list
gemini --debug
```

Then prompt Gemini CLI to test the feature or fix.

## Coding conventions

- Follow the styles and patterns already used in the codebase.
- Consult [GEMINI.md](../GEMINI.md) for AI-assisted development guidance.
- Pay attention to import paths; ESLint enforces restrictions on imports across
  packages.
- Tool names use dot notation in source (for example, `docs.create`). For
  Gemini CLI, `--use-dot-names` preserves those names at runtime.

## Project structure

- `workspace-server/`: MCP server package.
  - `src/`: server implementation.
    - `__tests__/`: unit tests.
    - `auth/`: authentication and token storage.
    - `cli/`: command-line tools such as headless OAuth login.
    - `features/`: feature configuration and OAuth scope registry.
    - `services/`: Google Workspace service logic.
    - `utils/`: shared helpers and configuration.
- `scripts/`: build, release, authentication, and development utilities.
- `commands/` and `skills/`: Gemini CLI extension commands and skills.

## Authentication

The extension uses OAuth 2.0. Authentication utilities are available from the
repository root:

```bash
npm run auth-utils -- <command>
```

Commands include `login`, `clear`, `expire`, `status`, and `help`. For remote
or headless environments (SSH, WSL, Cloud Shell, or VMs), run:

```bash
npm run auth-utils -- login
```

Open the printed OAuth URL in a browser, complete Google sign-in, and paste the
credentials JSON into the CLI when prompted. The CLI reads credentials from the
terminal (`/dev/tty` on Unix or `CON` on Windows), rather than process stdin.
Use `--force` to authenticate again when credentials already exist.

### Token storage

Credentials are stored in the OS keychain when available, using service name
`gemini-cli-workspace-oauth` and account name `main-account`. If the keychain is
unavailable, the extension uses AES-256-GCM encrypted file storage.

The encrypted fallback files are `gemini-cli-workspace-token.json` and
`.gemini-cli-workspace-master-key`. They are created with restrictive file and
directory permissions. To force file storage, set:

```bash
export GEMINI_CLI_WORKSPACE_FORCE_FILE_STORAGE=true
```
