# BIP: Zeropool — Opt-in First-Seen Zero-Conf with Inline Fee Bumps, Payee-Preserving Replacements, and Private Submission

**Author:** John Carvalho (drafted w/ assistant)  
**Type:** Standards Track (Policy)  
**Layer:** P2P / Mempool / Mining Policy  
**Status:** Draft (v0.5)  
**Created:** 2025-10-12  
**Requires:** None (consensus-neutral)  
**Informs (non-normative):** package relay, v3/anchor patterns, full-RBF network reality

---

## 1. Abstract

Zeropool defines an **opt-in policy lane** (sidecar mempool + miner preference) to restore commercially useful zero‑conf for explicitly signaled transactions without changing consensus. It provides:

1) **Time‑boxed first‑seen (Do‑Not‑Replace)** pre‑expiry.  
2) **Inline Fee Bump (IFB):** pre‑expiry **parent+child package** fee bumps via a designated **IFB anchor output** (no replacement, destination preserved).  
3) **Post‑expiry Same‑Destination RBF (SD‑RBF):** replacements allowed **only** if **payee outputs** are preserved exactly (via a **PAYEE_BITMAP**), with strictly higher fee/feerate and no new unconfirmed inputs.  
4) **Private submission modes:** PUBLIC, PRIVATE_FIRST (delayed public relay), PRIVATE_ONLY (no public relay).

The design is **consensus‑neutral**, deployable by individual nodes/miners, and interoperable with existing Core policy including package relay and (full-)RBF deployments.

---

## 2. Motivation

Network‑wide first‑seen has effectively disappeared under full‑RBF, yet many merchants still want bounded‑risk instant acceptance. Zeropool creates a **time‑boxed coordination lane** that:
- reduces double‑spend race surface (short pre‑expiry + private relay),
- keeps miners revenue‑aligned (IFB packages raise effective feerate),
- preserves payees under post‑expiry fee bumps (SD‑RBF), and
- remains fully consensus‑neutral.

---

## 3. Goals, Non‑Goals & Residual Risk

**Goals**
- Short‑window first‑seen for **opt‑in** txs.  
- Pre‑expiry **IFB** via 1‑parent‑1‑child package; no parent replacement.  
- Post‑expiry **SD‑RBF** with **payee preservation** (PAYEE_BITMAP).  
- Private submission support and prioritization.  
- Backwards‑compatible; implementation simplicity.

**Non‑Goals**
- No new opcodes, no consensus rule changes, no guarantee of inclusion, no attempt to revert network‑wide full‑RBF.

**Residual Risk (non‑exhaustive)**
- Hashrate‑adjacent attackers or miner collusion can still double‑spend.  
- Targeted attackers using private channels may outrun merchant detection.  
- Private‑only increases withholding risk if submitted to too few miners.  
- Zeropool reduces risk; it **does not eliminate** it.

---

## 4. Terminology

- **Zeropool tx**: transaction that requests Zeropool handling.  
- **Pre‑expiry**: interval during which first‑seen is enforced.  
- **IFB (Inline Fee Bump)**: fee bump achieved by adding a **child** spending a designated **IFB anchor output** of the parent; evaluated as a package.  
- **PAYEE_BITMAP**: bitset carried in the signal that marks which outputs are payees (must remain identical under SD‑RBF).  
- **SD‑RBF**: post‑expiry replacement allowed only if all **payee** outputs (scriptPubKey + amount) match exactly, fee/feerate strictly increase, and no new unconfirmed inputs are introduced.

---

## 5. Signaling (Wire Format)

Zeropool uses a compact **TLV signal** carried in the **Taproot annex** (preferred) or **OP_RETURN** (fallback). The signal is **consensus‑ignored** and committed by Taproot signatures (annex), hence safe for policy.

### 5.1 Carrier

- **Preferred:** Taproot **annex TLV** inside any input witness element beginning with `0x50`.  
- **Fallback:** `OP_RETURN` with ASCII prefix `ZP1:` followed by TLV hex. For standard relay, the entire OP_RETURN payload **MUST** be ≤ 80 bytes.

### 5.2 TLV (big‑endian for multi‑byte values)

| Type | Name             | Length | Semantics |
|------|------------------|--------|-----------|
| 0x00 | `VERSION`        | 1      | Zeropool signaling major version. Receivers **MUST** reject if `VERSION` is greater than supported. Current: `0x01`. |
| 0x01 | `DNR`            | 1      | Must be `0x01`. Requests pre‑expiry **Do‑Not‑Replace** first‑seen. |
| 0x02 | `EXP`            | 4      | Expiry in seconds; **MUST** be 1..3600; nodes **MAY** cap. Recommended 300–900. |
| 0x03 | `PRIV_MODE`      | 1      | `0=PUBLIC`, `1=PRIVATE_FIRST`, `2=PRIVATE_ONLY`. |
| 0x04 | `PVTW`           | 2      | Private‑first public‑relay delay in seconds; **MUST** be ≤ 600; only valid if `PRIV_MODE=1`. |
| 0x10 | `PAYEE_BITMAP`   | ≤32    | Bitstring over `nOutputs`; LSB = vout 0. A set bit marks a **payee** that must be preserved under SD‑RBF. |
| 0x11 | `IFB_ANCHOR_VOUT`| 2      | Index of the designated **IFB anchor** output in the parent tx. |
| 0x12 | `FEATURE_BITS`   | 1..2   | Optional capability flags for future negotiation (reserved). |

**Unknown TLVs:** Receivers **MUST ignore** unknown types in `[0x20..0x7F, 0xC0..0xFF]`. Types in `[0x80..0xBF]` are **critical** and **MUST** cause nonstandard‑reject.

**Size limits:** Total Zeropool TLV payload **MUST** be ≤ 80 bytes.

**Annex signing requirement:** Wallets **MUST** include the annex in the Taproot signature preimage (Taproot already commits; this is a normative restatement for wallet authors).

---

## 6. Node Policy (Sidecar Mempool)

Nodes started with `-zeropool=1` maintain a **sidecar policy** for Zeropool, interoperable with the standard mempool.

### 6.1 Admission (ALL required)
1. Valid TLV: `VERSION<=supported`, `DNR=1`, `EXP∈[1..cap]`.  
2. Effective feerate ≥ **admission floor** (Section 9.1).  
3. **Package‑friendly shape:** 1‑parent‑1‑child (v3/anchor‑like) to allow IFB/CPFP.

### 6.2 Pre‑Expiry Rules (First‑Seen, IFB)
- **No conflicts:** reject any conflicting candidate to a resident Zeropool parent **regardless of fee**.  
- **No parent replacement** pre‑expiry (BIP125 disabled for that parent).  
- **IFB allowed:** fee bumps **MUST** be achieved via **IFB** (child spending `IFB_ANCHOR_VOUT`) or CPFP package; evaluate **effective package feerate** against the admission floor and standard package limits.  
- **Relay priority:** Zeropool txs SHOULD be relayed promptly to Zeropool peers; conflicts MAY be down‑prioritized.

### 6.3 Post‑Expiry Transition
- On expiry, the parent **graduates** to standard policy.  
- Nodes **MAY** enable a **`-zeropoolsdrbfwindow`** (default 600s) during which **SD‑RBF** is allowed **only** if all hold:
  1) **PAYEE_BITMAP preservation:** for each set bit `i`, `(scriptPubKey_i, amount_i)` equals the original at index `i`.  
  2) **Fee strictly higher:** absolute fee and feerate.  
  3) **No new unconfirmed inputs.**

### 6.4 Private Submission Modes
- `PRIV_MODE=0` PUBLIC — normal gossip.  
- `PRIV_MODE=1` PRIVATE_FIRST — suppress public relay until `now ≥ PVTW`; private/miner relay permitted.  
- `PRIV_MODE=2` PRIVATE_ONLY — **MUST NOT** gossip publicly; private/miner relay only.  
- Operator safety valve: `-zeropoolprivatefallback=0|1`. If `1`, nodes **MAY** auto‑publish private‑only txs after expiry.

---

## 7. Miner Policy (Non‑Normative Preference)

Miners/pools that enable Zeropool may:
- Advertise a **premium lane** and admission expectations.  
- Prefer including **pre‑expiry** Zeropool parents (and IFB child if present) subject to profitability and block limits.  
- Offer **direct submission APIs** for private modes (see Appendix B).

---

## 8. Wallet Requirements (Normative)

- **Annex commitment:** MUST ensure the Zeropool annex is committed by signatures.  
- **PAYEE_BITMAP:** MUST mark only true payee outputs at construction; change **MAY** vary later.  
- **IFB anchor:** SHOULD use a standard output type (e.g., P2TR keypath), value ≥ dust+relay margin; **SHOULD** prefer 1‑parent‑1‑child packages only (avoid extra descendants).  
- **Private modes:** For `PRIVATE_ONLY`, wallets **SHOULD** submit to ≥2 independent miner endpoints and stagger timing for correlation resistance.  
- **UX:** Display expiry countdown and mode; warn about residual risk and stuck risk in private‑only.

---

## 9. Parameters & Economics

### 9.1 Admission Floor (fee)
Nodes compute the Zeropool admission floor as:
```
floor = max(rolling_mempool_min_fee, estimator_min_reasonable_fee) * (1 + premiumpct/100)
floor = max(floor, zeropool_min_feerate_if_set)
```
Default `premiumpct` = **25%**. Operators MAY tune.

### 9.2 Suggested Defaults & Caps
| Parameter | Default | Cap | Rationale |
|---|---:|---:|---|
| `EXP` | 600 s | 3600 s | Bounded risk + prevents stasis |
| `PVTW` | 20 s | 600 s | Relay suppression window |
| `PAYEE_BITMAP` | ≤32 bytes | 32 | Enough for typical outputs |
| TLV size | — | ≤80 bytes | Std OP_RETURN budget |
| `-zeropoolsdrbfwindow` | 600 s | — | Cooldown after expiry |
| IFB package | 1p1c | Core package limits | Simplicity & anti‑pinning |

**Incentive note:** IFB raises **package** feerate, aligning miner incentives with Zeropool policy. First‑seen does not ask miners to forgo higher revenue because IFB/CPFP achieves it without destination drift.

---

## 10. Interoperability (BIP125 / full‑RBF / v3)

- **Pre‑expiry:** Zeropool first‑seen **overrides** BIP125 for the parent (no replacement). Fee control via **IFB/CPFP** only.  
- **Post‑expiry:** SD‑RBF gate (if enabled) **narrows** BIP125 to payee‑preserving replacements with stricter checks.  
- **Package relay:** Zeropool relies on Core’s package relay/validation. Nodes **SHOULD** apply standard package limits to IFB.

---

## 11. Backwards Compatibility

Non‑participants treat Zeropool txs as ordinary; signals are ignored. There are **no** consensus, script, or standardness changes required for ordinary operation.

---

## 12. Security Considerations

- **Double‑spend:** Reduced by short first‑seen window + private modes + faster propagation; not eliminated.  
- **Pinning/DoS:** Mitigated via small packages (1p1c), IFB anchor guidance, fee floors, TLV/annex length caps, and short expiries.  
- **Censorship/centralization:** Private endpoints SHOULD be plural; wallets SHOULD multi‑home to independent miners.  
- **Stuck risk:** Private‑only can stall if miners don’t include; fallback publish and explicit warnings are recommended.  
- **Annex parsing:** Bounded TLV; reject malformed or over‑large payloads.

---

## 13. Deployment

- **Nodes:** `-zeropool=1` enables sidecar; `-zeropoolpremiumpct`, `-zeropoolminfeerate`, `-zeropoolexpirycap`, `-zeropoolsdrbfwindow`, `-zeropoolprivatefallback`. User‑agent MAY advertise `Zeropool`.  
- **Miners:** Template preference; direct submission API (Appendix B); publish lane/premium policy.  
- **Wallets:** Author annex TLV; construct IFB 1p1c packages; expose “Instant (Zeropool)” UX with expiry & private modes.

---

## 14. Reference Implementation Notes (non‑normative sketch)

- Admission path: parse annex/OP_RETURN; tag tx as Zeropool; store `expiry_ts` and `private_until`.  
- Conflict index: reject any conflicting arrival touching a resident Zeropool parent pre‑expiry.  
- Packages: validate **IFB 1p1c** and effective feerate; reuse Core’s package acceptance plumbing.  
- SD‑RBF: implement PAYEE_BITMAP check, fee/feerate increase, and “no new unconfirmed inputs.”  
- Private modes: honor relay gating; route to configured miner endpoints; never publicly gossip `PRIVATE_ONLY`.

---

## 15. Appendices

### Appendix A — PAYEE_BITMAP SD Definition (Normative)

Let `Payee(T)` = set of indices `i` where `PAYEE_BITMAP[i]=1`. SD holds **iff** for every `i ∈ Payee(T_orig)` the replacement `T_rep` has the **same** `(scriptPubKey_i, amount_i)` at index `i`. Change outputs (bitmap bit `0`) **may** vary freely.

**Notes**
- Amount equality is exact in satoshis.  
- If wallets omit `PAYEE_BITMAP`, nodes **MUST** treat SD‑RBF as **not permitted**.  
- Wallets SHOULD stabilize payee amounts at construction (avoid subsequent rounding).

### Appendix B — Miner Direct Submission API (Informative)

Transport: HTTPS or HTTP/2 gRPC. Auth: API key bound to pubkey; signed requests (`X-Submitter`, `X-Signature`, `X-Ts`).

**Endpoints**
- `POST /v1/submitTx { tx, privMode, expectExpirySec }` → `{ txid, accepted, reason }`  
- `POST /v1/submitPackage { parents:[hex], children:[hex], privMode }` → `{ packageId, effectiveFeerate, accepted }`  
- `GET  /v1/status/{txid}` → `{ seen, preExpiry, inTemplate, lastSeenAt, conflicts:[] }`  
- `POST /v1/quote { vsize, targetSec, mode:"zeropool" }` → `{ minFeeRate, premium }`

Errors: `429` rate‑limit, `402` fee below floor, `409` Zeropool conflict, `422` invalid package.

**Private‑only obligation:** miners MUST NOT rebroadcast publicly; include when profitable; keep for at least `EXP` seconds or return explicit rejection.

### Appendix C — TLV Encoding Examples

All multi‑byte fields big‑endian. LSB of bitmap = vout 0.

- **Annex TLV w/ private‑first (20s), expiry 600, PAYEE_BITMAP `0b101`, IFB anchor vout 1**  
  Hex TLV (no `0x50` annex tag shown):  
  `00 01 01 01 01 01 02 04 00 00 02 58 03 01 04 02 00 14 10 01 05 11 02 00 01`

- **OP_RETURN fallback (`ZP1:` + TLV hex)**:  
  e.g. `OP_RETURN "ZP1:0001010101020400000258030104020014"` … (trimmed for size).

### Appendix D — Test Vectors (abridged)

- **D1: IFB package admission**: Parent below floor with `IFB_ANCHOR_VOUT=1`; child spends vout 1 to raise package feerate above floor → **accept** pre‑expiry; parent not replaceable.  
- **D2: SD‑RBF valid**: Post‑expiry replacement preserving bitmap‑marked payees (exact script+amount), higher fee & feerate, no new unconfirmed inputs → **accept** within SD window.  
- **D3: SD‑RBF invalid (payee drift)**: Replacement changes any marked payee amount or script → **reject**.  
- **D4: Private‑first**: Withhold public gossip for `PVTW` seconds; permit private/miner relay → **observe** delayed gossip.  
- **D5: Private‑only**: Never gossip publicly; multi‑miner submission recommended; may remain unseen publicly indefinitely.

---

## 16. Rationale

- **Annex as carrier**: committed by signatures; consensus‑ignored; minimal surface for non‑participants.  
- **IFB vs. RBF**: IFB raises miner revenue **without** changing payees, aligning incentives during the delicate pre‑expiry window.  
- **PAYEE_BITMAP**: strict equality on **payees** only preserves intent but lets change adapt to fee conditions.  
- **Time‑boxing**: limits risk and avoids mempool ossification.

---

## 17. Acknowledgments

Thanks to reviewers and prior work on package relay, RBF, and Taproot annex design.

---

## 18. Changelog

- **v0.5 (2025‑10‑12):** Split spec by layer; added VERSION & FEATURE_BITS; normative MUST/SHOULD; PAYEE_BITMAP normative SD; IFB (anchor) detail; admission floor formula; limits & defaults; privacy modes clarified; test vectors; miner API appendix.  
- **v0.4:** Formal SD spec w/ bitmap; miner API schema; annex TLV details; wallet guidance; SD test vectors.  
- **v0.3:** Added PRIVATE_ONLY; clarified SD‑RBF post‑expiry; tightened package expectations.  
- **v0.2:** Removed non‑0‑conf intent flags; added SD‑RBF concept; private‑first window.  
- **v0.1:** Initial Zeropool draft (first‑seen + CPFP pre‑expiry).
