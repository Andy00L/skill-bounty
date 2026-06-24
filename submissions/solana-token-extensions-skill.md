# solana-token-extensions

**Author:** Andy00L
**Repo:** https://github.com/Andy00L/solana-token-extensions-skill
**Kit integration PR:** https://github.com/solanabr/solana-ai-kit/pull/28
**Version:** 1.0.0
**License:** MIT

## What it does

A progressively loaded Claude Code and Codex skill that makes a coding agent an expert in SPL Token-2022 (Token Extensions): choosing and combining extensions, building mints and transfer hooks, migrating from SPL Token, integrating with wallets, DEXs, and exchanges, and auditing transfer-hook security. It is the deeper, tested companion to the kit's existing `token-2022.md`, and delegates core program work to `solana-dev`.

Two things set it apart:

- A read-only **mint inspector** shipped as a CLI and **two MCP tools** an agent can call directly: `inspect_mint` decodes any live mint's extensions and reports wallet, DEX, and CEX integration risk, and `check_extension_compatibility` vets a planned extension set for conflicts and posture before any mint exists.
- An **executable compatibility matrix**: the written rules are backed by a tested risk engine, corrected against the Token-2022 source.

## Structure

- `skill/SKILL.md` router plus 16 focused docs (compatibility matrix, transfer-hook security audit, confidential-transfer status, a use-case decision tree, per-extension guides, migration, wallet/DEX/CEX integration)
- 4 specialized agents, 6 workflow commands, 2 rule sets, an installer
- 3 tested reference builds under `examples/`: a TypeScript multi-extension mint, a native Rust transfer-hook program (`cargo build-sbf`), and the mint inspector
- 51 offline, deterministic checks via `make verify`

## Problem it solves

Token-2022 is the 2026 standard for serious tokens (stablecoins, real-world assets, regulated tokens), and its extension surface is where builders trip: init ordering, account sizing, incompatible pairs, and transfer-hook security. Integrators and exchanges also need to assess a mint before listing or routing it. This skill consolidates the extension layer and turns due diligence into one command or one MCP call.

## What makes it strong (mapped to the judging axes)

- **Usefulness:** the full extension surface with the gotchas that cause silent failures, a decision tree from requirement to extension set, and two agent-callable MCP tools (inspect a live mint, vet a planned set).
- **Novelty:** an executable risk engine; a compatibility matrix corrected against the Token-2022 source (the runtime-enforced Scaled UI Amount versus Interest-Bearing exclusion and the required-companion rules); precise confidential-transfer status (disabled on mainnet since June 2025, re-enabled on testnet and devnet only, issue solana-program/token-2022#657); and a decoder that names extension codes the published `@solana/spl-token` enum does not yet map (PayPal USD carries the confidential transfer fee, code 16).
- **Quality:** 51 deterministic offline checks. Two inaccuracies were caught and corrected by running the inspector against PayPal USD (PYUSD): a confidential-transfer-with-hook pair wrongly flagged as incompatible (PYUSD carries both on mainnet; the hook receives `u64::MAX` on a confidential transfer), and an extension code the inspector now names instead of dropping. Errors as values, no type suppression.
- **Fit:** mirrors the reference skill shape, integrates as a submodule with one hub routing line, exposes two MCP tools, and delegates core program work to `solana-dev`. Kit integration PR: solanabr/solana-ai-kit#28.

## Install

```bash
git clone --recurse-submodules https://github.com/Andy00L/solana-token-extensions-skill
cd solana-token-extensions-skill
./install.sh -y
```

Verify the reference code offline (no validator, no devnet):

```bash
cd examples && make verify   # 51 checks: TypeScript mint, Rust hook, mint inspector
```
