---
name: nextjs-version-upgrade
description: Automated Next.js project upgrades using official codemods. Use this skill when the user wants to upgrade a Next.js project to the latest stable or canary version, migrate async request APIs, update deprecated configurations, or apply any Next.js version upgrade transformations. Triggers on requests like "upgrade my Next.js project", "migrate to Next.js lts", "update Next.js dependencies", "fix async params", or any mention of Next.js codemods or version migrations.
---

# Next.js Upgrade Skill

Automates Next.js version upgrades using official codemods for any version migration.

## Workflow

### Step 1: Detect Package Manager

Before running any commands, detect which package manager the project uses:

```bash
# Check for lock files in project directory
ls -la package-lock.json yarn.lock pnpm-lock.yaml bun.lockb 2>/dev/null
```

Determine package manager:
- `package-lock.json` → npm
- `yarn.lock` → yarn  
- `pnpm-lock.yaml` → pnpm
- `bun.lockb` → bun
- None found → ask user which they prefer

Set commands accordingly:
- **npm**: `npm install`, `npx`
- **yarn**: `yarn add`, `yarn dlx`
- **pnpm**: `pnpm add`, `pnpm dlx`
- **bun**: `bun add`, `bunx`

### Step 2: Identify Current Version

Check current Next.js version:

```bash
# Parse package.json to get current version
grep '"next"' package.json
```

### Step 3: Confirm Codemod Version (CRITICAL)

**MUST ASK USER** if not already specified in their request:

**Question**: "Do you want to use `@next/codemod@canary` (latest development version) or `@next/codemod@latest` (stable release)?"

- `@canary` - Most recent features, may be unstable
- `@latest` - Stable release version

**Only proceed after user confirms which version to use.**

### Step 4: Confirm Target Version

Ask user what version they want to upgrade to (e.g., "latest", "15", "14.2.0").

### Step 5: Run Upgrade Codemod

Execute the upgrade command with user's chosen codemod version:

```bash
# npm
npx @next/codemod@<canary-or-latest> upgrade <target-version>

# yarn
yarn dlx @next/codemod@<canary-or-latest> upgrade <target-version>

# pnpm
pnpm dlx @next/codemod@<canary-or-latest> upgrade <target-version>

# bun
bunx @next/codemod@<canary-or-latest> upgrade <target-version>
```

Examples:
- `npx @next/codemod@latest upgrade latest`
- `npx @next/codemod@canary upgrade latest`
- `pnpm dlx @next/codemod@latest upgrade 15`

### Step 6: Install Dependencies

After codemod completes:

```bash
# npm
npm install

# yarn
yarn install

# pnpm
pnpm install

# bun
bun install
```

Handle peer dependency warnings if needed:
- npm: `npm install --legacy-peer-deps` or `npm install --force`
- yarn: `yarn install --ignore-peer-dependencies`
- pnpm: `pnpm install --no-strict-peer-dependencies`
- bun: `bun install --force`

### Step 7: Review and Test

```bash
# Show what changed (if git is available)
git diff

# Start dev server
# npm
npm run dev

# yarn
yarn dev

# pnpm
pnpm dev

# bun
bun dev
```

## Individual Codemods

Codemods can be run individually for specific migrations. **ALWAYS ask user canary vs latest first.**

### General Pattern

```bash
<package-manager-executor> @next/codemod@<canary-or-latest> <codemod-name> <target-directory>
```

### Common Codemods

List available codemods:

```bash
npx @next/codemod@latest --help
```

Run specific codemod:

```bash
# Example: async request API migration
npx @next/codemod@latest next-async-request-api .

# Example: built-in font migration  
npx @next/codemod@canary built-in-next-font .

# Example: app router migration
npx @next/codemod@latest app-router-recipe .
```

## Critical Checklist

Before running ANY codemod command:
1. ✅ Detected package manager
2. ✅ Checked current Next.js version
3. ✅ **ASKED USER: canary or latest?** (if not specified)
4. ✅ Confirmed target version
5. ✅ User has committed/backed up their code

## Error Handling

### Lock File Conflicts

If multiple lock files exist, ask user which package manager to use and recommend removing others.

### Codemod Failures

If codemod fails:
1. Check error message
2. Try running specific codemods individually
3. Check Next.js version compatibility
4. Consult Next.js upgrade guide for that version

### Dependency Conflicts

If dependency installation fails:
1. Try with peer dependency flags
2. Check for conflicting package versions
3. May need to update other dependencies first

## Important Notes

- **NEVER assume canary or latest** - always ask user if not specified
- Codemods work across all Next.js versions
- Always commit code before running codemods
- Review all changes - codemods are helpful but not perfect
- Test thoroughly after migration
- Package manager detection is critical - don't assume npm
