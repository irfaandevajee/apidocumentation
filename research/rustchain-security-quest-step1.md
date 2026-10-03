# RustChain Security Quest — Step 1 Assessment

**Reviewed:** 2026-10-03  
**Target:** `Scottcjn/Rustchain` current `main`  
**Primary source:** `node/rustchain_v2_integrated_v2.2.1_rip200.py` (file blob `1069f0a8a99d632b57ead33fc7a3bebf0a160426`)  
**Scope:** `/attest/submit`, hardware fingerprinting / anti-VM controls, epoch reward settlement, and one code-review attack surface.

This assessment is a defensive source review. I did **not** fuzz, exploit, or mutate the production RustChain node.

## 1. Attestation flow and identity binding

The flow begins at [`/attest/challenge`](https://github.com/Scottcjn/Rustchain/blob/main/node/rustchain_v2_integrated_v2.2.1_rip200.py#L6028-L6095). The node rate-limits challenge creation, generates a 32-byte random nonce with `secrets.token_hex(32)`, sets a five-minute expiry, and can bind the nonce to the requesting miner identity. The code deliberately resolves `miner` / `miner_id` in the same order used by the submit path, which prevents a challenge from being bound to one identity and consumed as another. The challenge is persisted in the nonce table with its optional `bound_miner`.

Submission starts at [`/attest/submit`](https://github.com/Scottcjn/Rustchain/blob/main/node/rustchain_v2_integrated_v2.2.1_rip200.py#L6378-L6400) and delegates to `_submit_attestation_impl`. The implementation rejects non-object JSON and malformed payload shapes before doing identity work. The current signature path is stronger than a simple wallet-field MAC: for current v3 callers, Ed25519 covers canonical JSON of the full attestation payload after removing only the signature metadata. The legacy four-field signature exists for backward compatibility, but an attestation explicitly declaring the v3 signature type is not allowed to fall back to the narrower legacy format. The node also fails closed when a signature is supplied but PyNaCl is unavailable.

After signature verification, the code enforces pinning rules: a native RTC-hex identity must derive from the supplied public key; an already-pinned key cannot simply be swapped; and grandfathered symbolic identities are not silently TOFU-pinned to the first public signer. These controls reduce both payload tampering and first-signer identity hijacking.

The nonce is then consumed through `attest_validate_and_store_nonce` around [`6698-6735`](https://github.com/Scottcjn/Rustchain/blob/main/node/rustchain_v2_integrated_v2.2.1_rip200.py#L6698-L6735). Missing, expired, replayed, or identity-mismatched challenges are rejected before hardware reward eligibility is evaluated.

## 2. Why the fingerprint layer raises the cost of VM farms

The server does not treat a client-supplied `passed: true` as sufficient proof. The central validator, [`validate_fingerprint_data`](https://github.com/Scottcjn/Rustchain/blob/main/node/rustchain_v2_integrated_v2.2.1_rip200.py#L4928-L5185), rejects empty fingerprints for normal hardware and requires evidence for critical checks. The anti-emulation path reads the evidence itself: VM/emulator indicator lists and `is_likely_vm` can force rejection even if the client claims the check passed. For normal hardware, anti-emulation and clock-drift evidence are required; vintage / capability-limited hardware gets narrower exceptions intended to match what that silicon can actually measure rather than being rejected merely for lacking modern instructions.

The implementation also cross-validates architecture claims. For example, PowerPC claims are checked against CPU-brand evidence, PowerPC SIMD evidence, and a cache profile. The source contains a broad set of known VM/container signatures including VMware, VirtualBox, QEMU/KVM, Xen, Hyper-V, cloud platforms, Docker/LXC and several emulators. This is not a proof that virtualization can never evade detection, but it means a farm cannot earn merely by changing a string such as `device_arch`.

The submit path adds further layers around [`6760-6890`](https://github.com/Scottcjn/Rustchain/blob/main/node/rustchain_v2_integrated_v2.2.1_rip200.py#L6760-L6890): hardware binding, OUI checks when MACs exist, fingerprint hashing, cross-wallet entropy-collision checks, per-hardware fingerprint rate limiting, replay detection, and anomaly logging. Importantly, fingerprint validation now runs before replay bookkeeping so `attestation_valid` reflects the real validation outcome. Failed fingerprints can still be recorded for diagnostics, but they do not receive normal reward weight.

The security model is therefore layered rather than relying on one magic signal: possession/signature, one-time server nonce, hardware binding, server-side interpretation of raw fingerprint evidence, replay/collision history, and reward gating all need to line up.

## 3. Epoch reward calculation and distribution

[`finalize_epoch`](https://github.com/Scottcjn/Rustchain/blob/main/node/rustchain_v2_integrated_v2.2.1_rip200.py#L5609-L5795) reads enrolled miners and normalized integer weights, rejects empty/zero-weight epochs, calculates the epoch reward with `Decimal`, and clamps issuance against remaining supply headroom. Zero-weight miners — including machines already rejected as VM/emulator candidates — are filtered before shares are calculated.

RIP-309 then chooses a rotating subset of fingerprint checks using the previous block hash; if no hash is supplied, all six checks become active. Weights are capped before payout and the active fingerprint result can zero a miner for that epoch.

The actual monetary settlement uses an important concurrency control: inside a SQLite `BEGIN IMMEDIATE`, the code inserts the epoch state if needed and atomically changes `settled` from 0 to 1. If another settlement worker already won that claim, the transaction rolls back without crediting. Balance changes and the settlement claim then commit together. The code also re-runs the Sybil review hold inside that write transaction so a review hold committed just before settlement is not missed. That is materially stronger than relying only on the earlier read-only `settled` check.

## 4. Potential attack surface: RIP-309 finalization currently fails open on missing/invalid check data

The most concerning source-level edge I found is in the finalization-time rotating-check filter at [`5680-5710`](https://github.com/Scottcjn/Rustchain/blob/main/node/rustchain_v2_integrated_v2.2.1_rip200.py#L5680-L5710).

The relevant logic loads `fingerprint_checks_json`, initializes `checks_map = {}`, ignores JSON parsing errors, then evaluates:

```python
active_passed = all(checks_map.get(chk, True) for chk in active_checks)
```

There are two fail-open properties here:

1. **A missing active key defaults to `True`.** If an active RIP-309 check is absent from the stored map, finalization treats the missing measurement as a pass rather than as unmeasured/failing.
2. **Storage/query/parse errors preserve reward weight.** JSON decode errors are swallowed, and the surrounding database/security decision also uses `except Exception: pass`. In either case the miner keeps its previous positive weight.

That behavior appears inconsistent with the newer helper `_fingerprint_check_passed`, whose comments explicitly say that missing or non-explicit `passed=True` values should fail because proving nothing is not the same as passing. It is also inconsistent with the current `select_active_fingerprint_checks` comments, which intentionally fail closed when the rotation seed is unavailable.

I am **not claiming a demonstrated production exploit** from this review alone. The submit validator may ensure enough evidence for most modern clients, and capability-limited vintage classes intentionally need special handling. But this is still a security boundary worth tightening because finalization is the last gate immediately before money is allocated.

### Recommended hardening

The finalizer should reuse the same capability-aware scoring primitives already present elsewhere instead of independently interpreting stored JSON with `get(..., True)`. At minimum:

- missing selected checks should be `False`/`unmeasured`, never implicitly `True`;
- malformed `fingerprint_checks_json` should zero the reward weight or place the miner into review for that epoch;
- database errors in this security decision should be logged and fail closed instead of silently preserving a positive weight;
- capability-limited vintage hardware should use the existing explicit `unmeasured` policy rather than the generic missing-key default.

A regression test should cover: selected check explicitly false; selected check missing; malformed JSON; DB read failure; and a valid capability-limited vintage case that must retain its intended neutral treatment.

## Conclusion

The current attestation design has multiple useful defensive layers and has clearly been hardened against several earlier fail-open classes: full-payload signatures, nonce identity binding, server-side reading of raw anti-emulation evidence, replay/collision defenses, reward-tier vouching, and atomic epoch settlement. The remaining RIP-309 finalization behavior above is notable precisely because it does not follow that newer fail-closed pattern. Aligning finalization with the stricter fingerprint helpers would make the last reward gate consistent with the rest of the current security model.

**AI disclosure:** This assessment was produced by Irfaan's autonomous AI agent under his authorization from the current public source code. No production exploitation or destructive testing was performed.