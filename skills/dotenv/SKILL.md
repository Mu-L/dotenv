---
name: dotenv
description: Load environment variables from a .env file into process.env for Node.js applications. Use when configuring apps with environment-specific secrets, setting up local development environments, managing API keys and database URLs, parsing .env file contents, or populating environment variables programmatically. Triggers on requests involving .env files, process.env, environment variable loading, twelve-factor app config, or Node.js secrets management.
---

# dotenv

**Install:** `npm install dotenv`

## Choose Your Workflow

| Goal | Start Here |
|------|------------|
| Load `.env` into `process.env` | [Basic Usage](#basic-usage) |
| Use with ES6 `import` | [ES6 Import](#es6-import) |
| Load multiple `.env` files | [Multiple Files](#multiple-files) |
| Parse `.env` content manually | [Parse](#parse) |
| Populate a custom object | [Populate](#populate) |
| Debug why vars aren't loading | [Debug Mode](#debug-mode) |
| Encryption, multiple envs, CI/CD | [dotenvx skill](../dotenvx/SKILL.md) |

---

## Basic Usage

Create a `.env` file in the root of your project:

```ini
# .env
S3_BUCKET="YOURS3BUCKET"
SECRET_KEY="YOURSECRETKEYGOESHERE"
DATABASE_URL="postgres://localhost/mydb"
```

Load it as early as possible in your application:

```js
// CommonJS
require('dotenv').config()
console.log(process.env.S3_BUCKET) // "YOURS3BUCKET"
```

**Never commit `.env` to source control.** Add it to `.gitignore`:

```
.env
```

---

## ES6 Import

Use the side-effect import to ensure dotenv loads before other modules:

```js
// index.mjs — CORRECT
import 'dotenv/config'
import express from 'express'
```

**Common pitfall:** Do NOT call `dotenv.config()` before imports — ES6 modules are hoisted, so this won't work:

```js
// index.mjs — INCORRECT (express loads before dotenv runs)
import * as dotenv from 'dotenv'
dotenv.config()
import express from 'express' // process.env is still empty here!
```

Always use `import 'dotenv/config'` as the first import statement.

---

## Config Options

```js
require('dotenv').config({
  path: '/custom/path/to/.env', // default: path.resolve(process.cwd(), '.env') — also accepts an array
  encoding: 'utf8',             // default: 'utf8'
  debug: true,                  // default: false — logs what's happening
  override: true,               // default: false — overwrite existing process.env values
  quiet: true,                  // default: false — suppress the injection log message
  processEnv: myObject,         // default: process.env — write to a custom object instead
})
```

**Config returns** `{ parsed: { KEY: 'value', ... } }` on success, or `{ error: Error, parsed: {} }` on failure.

```js
const result = require('dotenv').config()
if (result.error) throw result.error
console.log(result.parsed) // { S3_BUCKET: 'YOURS3BUCKET', ... }
```

---

## Multiple Files

Pass an array to `path`. Files are processed in order; the **first value wins** (unless `override: true`, in which case the **last value wins**).

```js
// Load .env.local first, fall back to .env
require('dotenv').config({ path: ['.env.local', '.env'] })
```

Existing `process.env` values are never overwritten unless `override: true` is set.

---

## Parse

Parse a string or Buffer containing `.env` content — returns a plain object without touching `process.env`.

```js
const dotenv = require('dotenv')

const config = dotenv.parse('S3_BUCKET=mybucket\nSECRET=abc123')
console.log(config) // { S3_BUCKET: 'mybucket', SECRET: 'abc123' }

// From a Buffer
const buf = Buffer.from('BASIC=basic')
const parsed = dotenv.parse(buf) // { BASIC: 'basic' }
```

---

## Populate

Write parsed values into any target object (not just `process.env`):

```js
const dotenv = require('dotenv')
const parsed = { HELLO: 'world', DEBUG: 'true' }

// Write to process.env
dotenv.populate(process.env, parsed)

// Write to a custom object (leaves process.env untouched)
const target = {}
dotenv.populate(target, parsed, { override: true, debug: true })
console.log(target) // { HELLO: 'world', DEBUG: 'true' }
```

---

## Debug Mode

Turn on debug logging to diagnose missing or unexpected values:

```js
require('dotenv').config({ debug: true })
```

This prints which file is loaded, which keys are set, and why any keys were skipped (e.g. already set in the environment).

---

## .env File Format

```ini
# Comments start with #
BASIC=basic                          # → { BASIC: 'basic' }
EMPTY=                               # → { EMPTY: '' }
QUOTED="hello world"                 # → { QUOTED: 'hello world' }
SINGLE='quoted'                      # → { SINGLE: 'quoted' }
BACKTICK=`has 'single' and "double"` # → preserves inner single and double quotes
HASH_VALUE="value with # hash"       # # inside quotes is NOT a comment
WHITESPACE=  trimmed value           # → { WHITESPACE: 'trimmed value' }
MULTILINE="first\nsecond"            # → newline expanded in double quotes
JSON={"foo": "bar"}                  # → { JSON: '{"foo": "bar"}' }
export MY_VAR=value                  # export keyword is stripped → { MY_VAR: 'value' }
```

---

## Preload (no code changes required)

Load dotenv before your app starts using Node's `--require` flag:

```sh
node -r dotenv/config your_script.js
node -r dotenv/config your_script.js dotenv_config_path=/custom/.env
DOTENV_CONFIG_DEBUG=true node -r dotenv/config your_script.js
```
