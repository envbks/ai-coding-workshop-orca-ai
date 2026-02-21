---
name: aura-fetch-sources
description: "You MUST invoke this skill when you need deeper context about any dependency or internal service - npm packages, pip packages, or internal git repositories. Use when: (1) you need to understand how a library/package works beyond types and docs, (2) user says 'fetch source for X', 'get context for X', 'how does X work internally', 'show me the source of X', (3) you need to integrate with or debug an internal service, (4) user references a specific version of a package or internal repo, (5) you encounter an unfamiliar dependency and need implementation details, (6) user says 'I need to use library X' and you lack context about its API/internals. Supports npm (node_modules), pip (site-packages), and internal git repos (via reposWorkingDir in ~/.agents/aura.json)."
---

# Fetch Sources

Fetch source code and context from dependencies to give you deeper understanding beyond types and documentation.

## Source Types

### 1. npm Package

**Locate the version in use:**
```bash
# Search package-lock.json for exact version
cat package-lock.json | grep -A 5 '"<package-name>"'
```

**Fetch from node_modules:**
1. Check if `node_modules/<package-name>` exists
2. If YES - read the source files you need from there
3. If NO - run `npm ci` and retry:
   ```bash
   npm ci
   ```
4. After install, read from `node_modules/<package-name>`

**What to read:**
- `package.json` (entry points, exports map)
- Main entry file (from `main` or `exports` field)
- Relevant source files for the API/feature you need
- `README.md` if you need usage patterns
- Type definitions if separate (`@types/<package-name>`)

### 2. pip Package

**Locate the package:**
```bash
pip show <package-name>
```

This gives you the `Location` field (e.g., `/usr/lib/python3.11/site-packages`).

**If not installed:**
```bash
pip install <package-name>
pip show <package-name>
```

**If a specific version is needed:**
```bash
pip install <package-name>==<version>
pip show <package-name>
```

**What to read:**
- Navigate to `<Location>/<package_name>/` (note: underscores, not dashes)
- `__init__.py` (public API surface)
- Relevant module files for the feature you need

### 3. Internal Git Repository

**Resolve the repos working directory:**

1. Read `~/.agents/aura.json` and check for `reposWorkingDir` field
2. If `reposWorkingDir` exists - use that path
3. If NOT - ask the user: "Where do you keep your internal (aura) git repositories?"
   - If user provides a path - use it and save to `~/.agents/aura.json` as `reposWorkingDir`
   - If user doesn't know - default to parent directory of current project (`..`)

**Locate the repo:**
1. Check if `<reposWorkingDir>/<repo-name>` already exists
2. If YES - use it directly
3. If NO - clone it (try SSH first, fall back to HTTPS if it fails):
   ```bash
   git clone git@github.com:ironsource-aura/<repo-name>.git <reposWorkingDir>/<repo-name>
   # If SSH fails:
   git clone https://github.com/ironsource-aura/<repo-name>.git <reposWorkingDir>/<repo-name>
   ```
   If the user provided a full URL, use it as-is.

**Specific version needed:**

When the user needs context from a specific version (tag, branch, or commit), clone into a versioned directory to avoid modifying existing checkouts:

```bash
git clone --branch <version> --depth 1 git@github.com:ironsource-aura/<repo-name>.git <reposWorkingDir>/<repo-name>@<version>
```

If the repo is already cloned locally, clone from the local copy instead:
```bash
git clone --branch <version> --depth 1 <reposWorkingDir>/<repo-name> <reposWorkingDir>/<repo-name>@<version>
```

## After Fetching

Once you have access to the source:

1. **Load AGENTS.md** - Check if `AGENTS.md` (or `CLAUDE.md`) exists at the root of the fetched source. If it does, read it - it contains project-specific context and conventions that will help you understand the codebase
2. **Read what you need** - Navigate the source to find the specific APIs, functions, classes, or patterns relevant to your task
3. **Stay focused** - Only read files relevant to the context you need. Don't try to read the entire codebase

## Saving reposWorkingDir

When the user provides their repos working directory (or you default to `..`), persist it by reading `~/.agents/aura.json`, adding `"reposWorkingDir": "<absolute-path>"` at the top level while preserving all existing fields, and writing it back.

## Examples

**npm package context:**
> "I need to use the `simple-git` library, fetch its source so you understand how it works"
> "How does `@anthropic-ai/sdk` handle streaming internally?"
> "Fetch context for lodash's debounce implementation"

**pip package context:**
> "Get the source for the `boto3` S3 client"
> "I need to understand how `fastapi` dependency injection works"
> "Fetch context for pydantic v2 model validators"

**Internal git repo:**
> "I need context from the aura-mcp-servers repo"
> "Fetch the source for our backstage plugin from version 2.3.0"
> "Get context from the data-pipeline service, specifically the v1.5 release"
