# RustChain Security Quest — Step 2: VM Fingerprint Bypass

**Reviewed:** 2026-10-03  
**Target:** `Scottcjn/Rustchain` current `main`  
**Known fixed vulnerability:** VM Fingerprint Bypass  
**Method:** defensive source review and regression-test analysis only; no production exploitation.

## What the attack looked like before the fix

The vulnerable design trusted the client's anti-emulation verdict too much. A virtual machine could submit fingerprint evidence that actually admitted virtualization while also claiming the check passed, and the server did not consistently reconcile the contradiction.

The current regression test documents the old accepted payloads explicitly in [`tests/test_anti_emulation_reads_evidence.py`](https://github.com/Scottcjn/Rustchain/blob/main/tests/test_anti_emulation_reads_evidence.py):

```json
{"passed": true, "data": {"vm_indicators": ["qemu"]}}
```

```json
{"passed": true, "data": {"is_likely_vm": true}}
```

and even:

```json
{"data": {"vm_indicators": ["qemu"]}}
```

The test's historical note explains the inversion: the server held the string `qemu` in the raw evidence but only acted on VM evidence when the client had already confessed with `passed: false`. Omitting the `passed` key also bypassed an equality check written around `== False`. In other words, a truthful VM could be rejected while a dishonest VM could send the same evidence with a favorable or missing verdict and get through the anti-emulation portion of validation.

This matters economically because hardware fingerprinting is part of the gate between an identity and reward eligibility. If a virtualized worker can make the node treat contradictory VM evidence as a clean fingerprint, cheap replicated environments can move closer to receiving weight that is intended for independently attested physical hardware.

## The current fix

The current `validate_fingerprint_data()` implementation no longer makes the client's verdict authoritative.

### 1. Critical checks need actual evidence

For normal hardware, the validator requires anti-emulation and clock-drift checks. A dictionary check with no `data` is rejected, and a bare boolean `True` is treated as unmeasured for ordinary hardware rather than as proof. That prevents a payload made only of `{check: true}` from manufacturing a perfect fingerprint.

### 2. The server reads the VM evidence itself

The anti-emulation branch now inspects `vm_indicators` and the console-specific `emulator_indicators` directly. If either contains indicators, validation returns a `vm_detected` failure even when the submitted verdict says `passed: true`.

The code separately checks `is_likely_vm`; if it is explicitly true, the fingerprint is rejected regardless of the client's claimed verdict.

### 3. Missing verdict is no longer a pass

The implementation uses an explicit-true rule:

```python
if anti_emu_check.get("passed") is not True:
    return False, "vm_detected:no_pass_verdict:..."
```

That closes the previous `== False` gap. A missing verdict no longer falls between the true and false branches.

### 4. Console evidence was fixed without opening a new bypass

Pico/console clients use `emulator_indicators`, which was previously not recognized as an evidence field. The fix accepts that evidence key but also content-checks it. The regression suite includes both a clean console case and a console payload whose `emulator_indicators` contains `low_timing_cv`; the latter must fail. That is important because merely adding the field to an evidence whitelist without reading its contents would have created another fail-open condition.

## Regression proof in the repository

The current test file directly exercises the vulnerable cases:

- `test_claimed_pass_with_vm_indicators_is_rejected`
- `test_claimed_pass_with_is_likely_vm_is_rejected`
- `test_omitting_the_verdict_no_longer_skips_both_branches`
- `test_a_missing_verdict_with_clean_evidence_is_also_rejected`
- `test_an_honest_vm_is_still_rejected`
- `test_console_emulator_indicators_are_content_checked`

It also checks that clean physical evidence and legitimate console evidence still pass, so the fix is not simply 'reject everything'.

## Why the fix is sufficient for this specific bypass

For the documented bug class, the fix is sufficient: a caller can no longer override contradictory anti-emulation evidence by setting `passed: true`, and cannot get an implicit pass by omitting `passed`. The decision is now based on server interpretation of the evidence plus an explicit boolean pass requirement.

The regression tests are well matched to the original failure mode because they preserve the exact contradictory payload shapes that used to succeed. If a later refactor again starts trusting the client verdict, those tests should fail.

## What the fix does not prove

This fix should not be interpreted as proof that virtualization can never evade RustChain fingerprinting. It closes a **logic bypass** in which the server already had incriminating evidence but failed to use it. A more sophisticated attacker could instead try to fabricate a fully plausible set of raw timing and architecture measurements that contains no obvious VM indicator.

RustChain therefore still benefits from the surrounding controls that exist in the current codebase: one-time nonce binding, full-payload signatures for current clients, hardware binding, rotating fingerprint checks, architecture cross-validation, entropy/replay history, collision/rate checks, and epoch reward gating. Those layers increase the cost of synthetic fingerprints, but physical-attestation systems remain adversarial measurement systems rather than mathematical proofs of 'no VM'.

A particularly useful hardening principle is visible in the present code: **missing or contradictory evidence should fail closed unless a capability-limited hardware class has an explicit, narrowly defined exception.** That principle should remain consistent all the way through reward settlement.

## Conclusion

Before the fix, anti-emulation validation could effectively ask the client whether it was a VM and accept a favorable answer even while the payload contained `qemu` or `is_likely_vm=true`. The current implementation reverses that trust relationship: the server evaluates the raw evidence, requires explicit successful measurement, and rejects contradictory or missing verdicts. The accompanying regression tests reproduce the old bypass shapes and verify the hardened behavior, making this a concrete and appropriately scoped fix for the known VM Fingerprint Bypass.

**AI disclosure:** This Step 2 write-up was produced by Irfaan's autonomous AI agent under his authorization from the current public RustChain source and regression tests. No live-node exploitation or destructive testing was performed.