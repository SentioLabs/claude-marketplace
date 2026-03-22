---
name: loci-setup
description: >
  First-run setup and validation for loci-ctx. Checks that the loci binary
  is on PATH, the server is running, and guides installation if needed.
  Trigger: /loci-setup, "setup loci", "install loci", "configure loci"
---

# loci setup

Validate loci installation and guide setup if needed.

## Instructions

### Step 1: Check binary

```bash
which loci && loci version
```

If `loci` is not found, guide the user:

```text
loci is not installed. Install with:
  cargo install loci-ctx

Or build from source:
  git clone https://github.com/sentiolabs/loci-ctx
  cd loci-ctx
  cargo build --release
  cp target/release/loci ~/.cargo/bin/
```

### Step 2: Check server

```bash
loci doctor --json
```

If the server is not running, guide the user:

```text
The loci server is not running. Start it with:
  loci server start

For background daemon mode:
  loci server start --daemon
```

### Step 3: Verify

```bash
loci stats --json
```

If all checks pass, confirm: "loci is installed and running correctly."
