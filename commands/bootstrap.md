---
name: bootstrap
description: First-time project setup - creates verify.sh scaffold and GitHub Actions workflow
allowed-tools: ["Bash", "Read", "Write", "Glob"]
argument-hint: ""
---

# SOP Bootstrap - Verification Harness Setup

You are setting up the SOP verification harness for this project. This is always the first slice for any new project.

## Step 1: Check Current State

Check if `./scripts/verify.sh` already exists:
```bash
test -f ./scripts/verify.sh && echo "EXISTS" || echo "MISSING"
```

If it exists, show the user its contents and ask if they want to regenerate or enhance it.

## Step 2: Detect Project Type

Auto-detect the project type by checking for these files:

| File | Project Type | Stack |
|------|--------------|-------|
| `package.json` | Node.js | Check for framework (Next.js, React, etc.) |
| `Cargo.toml` | Rust | Check workspace vs single crate |
| `go.mod` | Go | Standard Go module |
| `pyproject.toml` or `requirements.txt` | Python | Check for framework (Django, FastAPI, etc.) |
| `mix.exs` | Elixir | Phoenix or library |
| `pom.xml` or `build.gradle` | Java/Kotlin | Maven or Gradle |
| `composer.json` | PHP | Laravel, Symfony, etc. |

Run:
```bash
ls -la package.json Cargo.toml go.mod pyproject.toml requirements.txt mix.exs pom.xml build.gradle composer.json 2>/dev/null || echo "No standard project files found"
```

## Step 3: Create scripts/ Directory

```bash
mkdir -p scripts
```

## Step 4: Generate verify.sh Based on Project Type

Create `./scripts/verify.sh` with appropriate checks. The script must:
- Be runnable from repo root
- Exit 0 on success, non-zero on failure
- Be fast and deterministic
- Prove "project still runs"

### Node.js Template
```bash
#!/bin/bash
set -e

echo "=== SOP Verification ==="

# Install dependencies (if needed)
if [ ! -d "node_modules" ]; then
  echo "Installing dependencies..."
  npm ci
fi

# Type check (if TypeScript)
if [ -f "tsconfig.json" ]; then
  echo "Type checking..."
  npx tsc --noEmit
fi

# Lint
if grep -q '"lint"' package.json 2>/dev/null; then
  echo "Linting..."
  npm run lint
fi

# Test
if grep -q '"test"' package.json 2>/dev/null; then
  echo "Running tests..."
  npm test
fi

# Build
if grep -q '"build"' package.json 2>/dev/null; then
  echo "Building..."
  npm run build
fi

echo "=== Verification PASSED ==="
```

### Rust Template
```bash
#!/bin/bash
set -e

echo "=== SOP Verification ==="

echo "Checking format..."
cargo fmt --check

echo "Running clippy..."
cargo clippy -- -D warnings

echo "Running tests..."
cargo test

echo "Building..."
cargo build

echo "=== Verification PASSED ==="
```

### Go Template
```bash
#!/bin/bash
set -e

echo "=== SOP Verification ==="

echo "Running go vet..."
go vet ./...

echo "Running staticcheck (if available)..."
command -v staticcheck && staticcheck ./... || echo "staticcheck not installed, skipping"

echo "Running tests..."
go test ./...

echo "Building..."
go build ./...

echo "=== Verification PASSED ==="
```

### Python Template
```bash
#!/bin/bash
set -e

echo "=== SOP Verification ==="

# Type check (if mypy available)
if command -v mypy &> /dev/null; then
  echo "Type checking..."
  mypy .
fi

# Lint
if command -v ruff &> /dev/null; then
  echo "Linting with ruff..."
  ruff check .
elif command -v flake8 &> /dev/null; then
  echo "Linting with flake8..."
  flake8 .
fi

# Test
if [ -f "pytest.ini" ] || [ -d "tests" ]; then
  echo "Running tests..."
  pytest
fi

echo "=== Verification PASSED ==="
```

### Generic Template (fallback)
```bash
#!/bin/bash
set -e

echo "=== SOP Verification ==="

# TODO: Add your verification steps here
# Examples:
# - npm test
# - cargo test
# - go test ./...
# - pytest

echo "WARNING: Using generic template. Please customize for your project."
echo "=== Verification PASSED ==="
```

## Step 5: Make Executable

```bash
chmod +x ./scripts/verify.sh
```

## Step 6: Test the Script

Run the verification script:
```bash
./scripts/verify.sh
```

If it fails, help the user fix the issues before proceeding.

## Step 7: Create GitHub Actions Workflow

Create `.github/workflows/verify.yml`:

```yaml
name: Verify

on:
  pull_request:
    branches: [main, dev]
  push:
    branches: [main, dev]

jobs:
  verify:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # Add setup steps based on project type
      # Node.js:
      # - uses: actions/setup-node@v4
      #   with:
      #     node-version: '20'
      #     cache: 'npm'

      # Rust:
      # - uses: dtolnay/rust-toolchain@stable

      # Go:
      # - uses: actions/setup-go@v5
      #   with:
      #     go-version: '1.21'

      # Python:
      # - uses: actions/setup-python@v5
      #   with:
      #     python-version: '3.11'

      - name: Run verification
        run: ./scripts/verify.sh
```

Customize the setup steps based on the detected project type.

## Step 8: Create Branch and Commit

This bootstrap is itself a slice. Create the branch and commit:

```bash
git checkout -b chore/verify-harness-v1
git add scripts/verify.sh .github/workflows/verify.yml
git commit -m "chore: add verification harness and CI workflow"
```

## Step 9: Summary

Print:
```
Bootstrap Complete
------------------
Created:
  - ./scripts/verify.sh (verification script)
  - .github/workflows/verify.yml (CI workflow)

Next steps:
1. Push and create PR: git push -u origin chore/verify-harness-v1
2. Or use /sop:pr to create the PR

After merging, use /sop:kickoff to start your first feature slice.
```
