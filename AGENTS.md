# AGENTS.md

## Purpose

This file is for contributors and coding agents maintaining this repository. Keep it aligned with the codebase, CI workflows, and security posture so future changes preserve token safety, crypto compatibility, and build health.

## Repository overview

- `totp/` is the Flipper Zero application source.
- `totp/services/crypto/` contains the encryption facade plus crypto version implementations.
- `totp/services/config/` owns config file I/O, migrations, backups, and encryption upgrades.
- `totp/types/token_info.c` decodes imported secrets and encrypts them before persistence.
- `totp/cli/` and `totp/ui/` are the main user-facing flows that must preserve validation and secret-handling rules.
- `build.ps1` packages release artifacts for supported firmware variants.
- `ufbt.ps1` prepares firmware-specific defines and builds the app through uFBT.
- `.github/workflows/` is the source of truth for CI expectations.

## Maintenance rules

1. Keep changes surgical and avoid broad refactors in security-sensitive paths.
2. When changing config schema or persistence behavior, update all of:
   - `totp/services/config/constants.h`
   - `totp/services/config/config.c`
   - `totp/services/config/migrations/`
   - `docs/conf-file_description.md`
3. When changing crypto behavior, review all supported crypto versions in `totp/services/crypto/` and preserve migration/decryption compatibility unless a deliberate breaking change is documented.
4. When changing token import, PIN handling, or automation flows, re-check both CLI and UI entry points so validation rules stay consistent.
5. Keep `README.md`, `SECURITY.md`, and this file current when maintenance workflows or security expectations change.

## Security-critical invariants

- Never persist token secrets in plaintext intentionally. `TokenSecret` may appear unencrypted only as a migration/import input and must be re-encrypted before storage.
- Do not remove plaintext zeroization such as `memset_s` in secret-handling code.
- Preserve key-slot validation and the current enclave-based encryption flow in `totp/services/crypto/crypto_facade.c`.
- Preserve config backup and migration behavior before destructive format changes.
- Treat changes to `CryptoVersion`, `CryptoKeySlot`, `BaseIV`, `Crypto`, and `PinIsSet` as high risk; review them together.
- Any WolfSSL update must be followed by a focused review of the vendored file set in `totp/lib/wolfssl/` and the app’s crypto call sites.
- Avoid logging secrets, decrypted token bytes, raw PIN values, IVs, salts, or derived key material.

## Build and validation

CI currently builds with uFBT and PowerShell wrappers:

```sh
python3 -m pip install --upgrade ufbt
pwsh ./ufbt.ps1 md --clean
pwsh ./build.ps1 md
```

Relevant automated checks are defined in:

- `.github/workflows/nightly-build.yml`
- `.github/workflows/create-new-release.yml`
- `.github/workflows/pvsstudio.yml`
- `.github/workflows/sonarcloud.yml`
- `.github/workflows/auto-clang-format.yml`

Before merging maintenance changes:

- run the smallest existing build that covers the touched area;
- keep formatting consistent with `.clang-format`;
- review whether documentation describing config, security reporting, or release behavior also needs updates.

## Security maintenance checklist

Use this checklist whenever touching crypto, config, imports, or dependencies:

- confirm secrets are encrypted before persistence and wiped after transient plaintext use;
- confirm migrations still load older configs and upgrade them safely;
- confirm new bounds checks and parsing paths reject malformed token data;
- confirm no workflow or script change weakens static analysis coverage;
- confirm dependency or vendored library updates include a note in the PR about security impact.

## Keeping this file updated

Update `AGENTS.md` whenever any of the following changes:

- repository layout for core app, crypto, config, or workflow files;
- build, release, or static-analysis commands;
- supported crypto versions or migration expectations;
- security review steps for secrets, PINs, config files, or vendored dependencies.
