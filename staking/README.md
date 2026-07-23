# ValidatorStaking

Delegated Proof-of-Stake staking for the Altcoinchain hybrid PoW/PoS upgrade.

## Deployed

| Version | Address | Status |
|---|---|---|
| **v3** | **`0x55C492DF28Ae84a9f08dCBA9a5F686C1618d0Dac`** | **current — audited, use this** |
| v2 | `0x347F496c887a92ed9706ff3EDF4f0b822Ab00d3E` | deprecated — see AUDIT.md, do not stake |

**Do not stake into v2.** It has fund-destroying defects: `slash()` is
permissionless (anyone can burn 10% of any validator and brick it forever), and
self-stake can never be withdrawn. Full findings and the v3 remediation are in
[AUDIT.md](AUDIT.md).

## v3 summary

- Validators bond ≥ 32 ALT self-stake and set a commission (≤ 50%) and moniker.
- Delegators delegate ≥ 10 ALT; delegations are withdrawable via
  `undelegate` → 7-day delay → `completeWithdrawals`.
- Self-stake is withdrawable via `unstakeSelf` / `unregisterValidator` →
  `claimUnbondedSelfStake`, and stays slashable during the unbonding window.
- Slashing (10%) is callable **only** by the consensus system address
  `0x…FFFE`; losses are shared pro-rata among delegators via a scaling factor.
- Rewards are pull-based (`claimRewards`, `claimValidatorRewards`); distribution
  is O(1) accounting with no external calls.

## Build / test

```
cd staking
forge test          # 34 tests incl. adversarial probes + 5000-run fuzz
forge build --sizes # ~12.7 KB, evm_version = paris (ALT geth is pre-Shanghai)
```

Compile with `evm_version = "paris"` — Altcoinchain's geth (v1.10.24) is
pre-Shanghai and rejects the PUSH0 opcode that solc 0.8.20 emits by default.
