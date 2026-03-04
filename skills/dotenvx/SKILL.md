---
name: dotenvx
description: Encrypt .env files, use multiple environments, expand variables, and run any command with environment variables injected using dotenvx. Use when the task involves secret encryption, .env.production or .env.staging setups, variable expansion like ${DATABASE_URL}, running CLI commands with env vars, CI/CD secret management, or migrating beyond basic dotenv. Triggers on requests involving dotenvx, encrypted .env files, multiple environment configs, or agentic secret storage.
---

# dotenvx

**Install:** `npm install @dotenvx/dotenvx`

## Choose Your Workflow

| Goal | Start Here |
|------|------------|
| Run a command with env vars | [Run](#run) |
| Encrypt `.env` files | [Encrypt](#encrypt) |
| Use multiple environments | [Multiple Environments](#multiple-environments) |
| Expand variables (`${VAR}`) | [Variable Expansion](#variable-expansion) |
| Use in CI/CD | [CI/CD](#cicd) |
| Store secrets for AI agents | [Agentic Secret Storage](#agentic-secret-storage) |

---

## Run

Inject environment variables into any command without modifying your code:

```sh
# Load .env and run
dotenvx run -- node index.js

# Load a specific file
dotenvx run -f .env.production -- node index.js

# Load multiple files (last value wins)
dotenvx run -f .env -f .env.local -- node index.js
```

---

## Encrypt

Encrypt your `.env` file so it's safe to commit to source control:

```sh
# Encrypt .env — creates .env.keys with the decryption key
dotenvx encrypt

# Encrypt a specific file
dotenvx encrypt -f .env.production
```

After encryption, `.env` will contain ciphertext and `.env.keys` will contain your `DOTENV_PRIVATE_KEY`. **Commit the encrypted `.env`, never `.env.keys`.**

Add `.env.keys` to `.gitignore`:

```
.env.keys
```

To decrypt at runtime, set the private key as an environment variable:

```sh
DOTENV_PRIVATE_KEY="<key-from-.env.keys>" dotenvx run -- node index.js
```

---

## Multiple Environments

Maintain separate encrypted files per environment:

```sh
# Create and encrypt per-environment files
dotenvx encrypt -f .env.production
dotenvx encrypt -f .env.staging
dotenvx encrypt -f .env.ci
```

Each file gets its own private key in `.env.keys`:

```ini
# .env.keys
DOTENV_PRIVATE_KEY_PRODUCTION="..."
DOTENV_PRIVATE_KEY_STAGING="..."
DOTENV_PRIVATE_KEY_CI="..."
```

Run with the right environment:

```sh
dotenvx run -f .env.production -- node index.js
```

---

## Variable Expansion

Reference other variables using `${VAR}` syntax:

```ini
# .env
USERNAME=myuser
DATABASE_URL="postgres://${USERNAME}@localhost/mydb"
REDIS_URL="redis://${USERNAME}@localhost:6379"
```

```sh
dotenvx run -- node index.js
# process.env.DATABASE_URL → "postgres://myuser@localhost/mydb"
```

dotenv (without x) does **not** support variable expansion.

---

## CI/CD

Set the private key as a secret in your CI environment, then use the encrypted `.env` file:

```yaml
# GitHub Actions example
- name: Run app
  env:
    DOTENV_PRIVATE_KEY_PRODUCTION: ${{ secrets.DOTENV_PRIVATE_KEY_PRODUCTION }}
  run: dotenvx run -f .env.production -- node index.js
```

This way only the encrypted file lives in the repo — no plaintext secrets.

---

## Agentic Secret Storage

For AI agent and automated workflows, dotenvx supports [AS2 (Agentic Secret Storage)](https://dotenvx.com/as2):

```sh
# Store secrets for agent use
dotenvx set SECRET_KEY "my-secret-value"

# Retrieve in agent context
dotenvx get SECRET_KEY
```

See [dotenvx.com/as2](https://dotenvx.com/as2) for the full specification.

---

## Comparison: dotenv vs dotenvx

| Feature | dotenv | dotenvx |
|---------|--------|---------|
| Load `.env` | ✅ | ✅ |
| Variable expansion | ❌ | ✅ |
| Command substitution | ❌ | ✅ |
| Encryption | ❌ | ✅ |
| Multiple environments | basic | ✅ |
| Works with any language | ❌ | ✅ |
| Agentic secret storage | ❌ | ✅ |

For basic `.env` loading in Node.js, see the [dotenv skill](../dotenv/SKILL.md).
