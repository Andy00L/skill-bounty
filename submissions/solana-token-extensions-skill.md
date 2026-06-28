# solana-token-extensions

**Author:** Andy00L
**Repo:** https://github.com/Andy00L/solana-token-extensions-skill
**Kit integration PR:** https://github.com/solanabr/solana-ai-kit/pull/28 (answers kit issue #12)
**Version:** 1.0.0
**License:** MIT

## What it does

A progressively loaded Claude Code and Codex skill that makes a coding agent an expert in SPL Token-2022 (Token Extensions): choosing and combining extensions, building mints and transfer hooks, migrating from SPL Token, integrating with wallets, DEXs, and exchanges, and auditing transfer-hook security. It is the deeper, tested companion to the kit's existing `token-2022.md`, and delegates core program work to `solana-dev`.

What sets it apart is an executable, research-grounded risk engine, shipped as five MCP tools an agent can call directly:

- `inspect_mint`: decode any live mint or token account and score it with **conditional severity**. A fund-loss-grade extension is high only while its controlling authority is live, the live-versus-renounced model Jupiter uses to gate transfer-fee tokens from resting orders and Neodyme prescribes for hooks. It emits a 0-to-100 risk score, a concrete fix per finding, and a **renounce-to-remediate path**: the verdict each authority an issuer could renounce would produce (CRITICAL today to MEDIUM once the permanent delegate is renounced), so the score is a path, not just a label.
- `inspect_many`: triage a whole listing set in one call, with a per-mint verdict and an aggregate roll-up (worst verdict, counts by severity, how many carry a CEX listing blocker). Built for "is my exchange's listing set safe."
- `check_extension_compatibility`: vet a planned extension set for conflicts and integration posture before any mint exists.
- `scaffold_mint`: generate correct, correctly-ordered, correctly-sized mint creation code from a legal extension set, refusing any set the runtime would reject at init.
- `generate_hook_transfer`: generate a correct transfer client for a hooked mint, with the extra accounts resolved against the Execute account set, plus a static classification of the hook's extra-account model.

## Mapped to the judging axes

- **Usefulness:** the full extension surface with the init-order and account-sizing gotchas that cause silent failures, a use-case decision tree from requirement to extension set, and five agent-callable MCP tools (inspect one mint, triage many, vet a planned set, scaffold a new mint, generate a transfer-hook client). It turns token due diligence into one call.
- **Novelty:** an executable compatibility matrix corrected against the Token-2022 source (the runtime-enforced Scaled UI Amount versus Interest-Bearing exclusion and the required-companion rules), a conditional-severity risk engine grounded in real integrator behavior, a value-aware transfer-fee model (a near-100% fee is a sell-blocking honeypot even when the rate is locked), a renounce-to-remediate projection no other entry ships, a build-time mint-scaffold generator that refuses runtime-rejected sets, a transfer-hook integration codegen that resolves extra accounts against the Execute account set, and a decoder that names every extension code 0 to 28, including those the published `@solana/spl-token` enum does not map (PayPal USD carries the confidential transfer fee, code 16). The confidential-transfer status is current: re-enabled on mainnet 2026-06-04, verified from the on-chain feature gate.
- **Quality:** 114 offline, deterministic checks via `make verify`, plus a CI-gated live mainnet smoke test: a TypeScript multi-extension mint on LiteSVM (7 tests plus a transfer-hook end-to-end scenario), a native Rust transfer hook (`cargo build-sbf` plus 22 unit tests hardened to the security checklist), and the mint inspector (84 tests, including a 16-case scored eval suite, `npm run evals`, that runs the executable EVALS rows through the engine). A THREAT_MODEL.md maps each adversarial test to the threat it closes. Two inaccuracies were caught and corrected by running the inspector against PYUSD: a confidential-transfer-with-hook pair wrongly flagged as incompatible (PYUSD carries both on mainnet; the hook receives `u64::MAX` on a confidential transfer), and an extension code the inspector now names instead of dropping. Errors as values, no type suppression.
- **Fit:** mirrors the reference skill shape, integrates as a submodule with one hub routing line, exposes five read-only MCP tools with structured agent-consumable verdicts, delegates core program work to `solana-dev`, and answers the kit's open request for a full Token Extension skill (issue #12). Kit integration PR: solanabr/solana-ai-kit#28.

## Install

```bash
git clone --recurse-submodules https://github.com/Andy00L/solana-token-extensions-skill
cd solana-token-extensions-skill
./install.sh -y
```

Verify the reference code offline (no validator, no devnet):

```bash
cd examples && make verify   # 114 checks: TypeScript mint, Rust hook, mint inspector (npm run evals runs the scored suite)
```
