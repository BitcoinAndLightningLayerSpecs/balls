    BIP: XXX  
    Layer: Informational  
    Title: Redefinition of the Bitcoin Unit to the Base Denomination  
    Author: [John Carvalho] <bitcoinerrorlog@gmail.com>  
    Comments-Summary: No comments yet.  
    Comments-URI: https://github.com/bitcoin/bips/wiki/Comments-URI  
    Status: Draft  
    Type: Informational  
    Created: 2024-12-10  
    License: CC0-1.0

# Abstract

This BIP proposes redefining the commonly recognized "bitcoin" unit so that what is currently known as the smallest indivisible unit becomes the primary reference unit. Under this proposal, 1 bitcoin is defined as one of these smallest units, removing the need for decimal places. By making the integral unit the standard measure, this BIP aims to simplify user comprehension, reduce confusion, and align on-chain values directly with their displayed representation.

# Motivation

The current convention defines 1 BTC as 100,000,000 of these smallest units. This representation requires users to grapple with eight decimal places, often causing confusion and the mistaken belief that bitcoins are inherently divisible "decimal" tokens. In reality, the Bitcoin protocol tracks only whole integer counts of the smallest indivisible unit. The decimal display layer is a human-imposed convenience that can mislead new users into thinking the asset behaves like a traditional fiat currency with floating decimal precision. By embracing the integer representation, we correct this misconception and present Bitcoin’s nature more faithfully: a system that counts discrete units without native decimal division.

As Bitcoin grows in adoption, ensuring that users understand its integral nature is vital. Presenting values as large integers directly conveys how the protocol operates, reducing the cognitive load and minimizing misunderstandings about divisibility. This approach ultimately improves user experience, education, and long-term adaptability.

# Specification

1. **Redefinition of the Unit:**
   - At the protocol level, the smallest indivisible unit remains unchanged.
   - Historically, 1 BTC = 100,000,000 base units. With this proposal, 1 bitcoin **is** that smallest unit.
   - The entity previously referred to as "1 BTC" under the old convention now corresponds to 100 million bitcoins.

2. **Terminology:**
   - The informal terms "satoshi" or "sat" are deprecated.
   - All communications, interfaces, and documentation SHOULD refer to the base integer unit simply as "bitcoin."

3. **Display and Formatting:**
   - Applications SHOULD present all values as whole integers, without decimal points.
   - Example:
     - Old display: `0.00010000 BTC`
     - New display: `10000 BTC` or `₿10000`

4. **Conversion:**
   - The underlying ledger and consensus rules remain unchanged.
   - Implementations adopting this standard MUST multiply previously displayed BTC amounts by 100,000,000 to determine the new integer representation.

# Rationale

1. **Usability:**
   Removing decimals simplifies mental arithmetic. Users handle values as integers, reducing cognitive load and potential for error.

2. **Protocol Alignment:**
   The Bitcoin protocol inherently tracks value as integral units. Representing values directly as these integers eliminates the artificial complexity of decimal formatting and corrects misconceptions that Bitcoin itself is "divisible" like a decimal currency. Instead, Bitcoin is accounted for in discrete units, similar to counting individual items.

3. **Educational Clarity:**
   New users often misunderstand Bitcoin’s nature due to the decimal-centric display. The new standard aligns user perception with the fundamental design of Bitcoin, making it clearer that the system counts indivisible units and that the decimal point was a human-imposed abstraction.

4. **Future-Proofing:**
   As Bitcoin evolves, normalizing the smallest unit as the standard form ensures consistent interpretation and communication.

# Backward Compatibility

No consensus-level rules are changed, and existing on-chain data remains intact. Differences arise only in how amounts are displayed:

- **For Developers:**  
  Update GUIs, APIs, and documentation to display values as integers. Remove references to fractional BTC.
  
- **For Users:**  
  Users retain the exact same holdings and transaction values in real terms. Only the way values are presented changes.

# Security Considerations

**Potential for Confusion:**  
Some users, initially expecting decimals, might misinterpret integer values as drastically larger amounts. However, the magnitude of difference between old and new displays makes genuine confusion resulting in significant losses unlikely. For example, misunderstanding `10000 BTC` under the new system as equal to `10000 BTC` under the legacy fractional system is implausible, given the well-known, historically high value of a single "legacy BTC."

**Mitigation Strategies:**
1. **Gradual Transition:**
   Provide dual displays, tooltips, and toggles in wallets and services to help users adjust.

2. **Education and Communication:**
   Industry players (exchanges, wallets, media) should explain that the new format reflects the true nature of Bitcoin’s integral accounting. Make it clear that no real value is gained or lost, only the presentation method changes.

3. **Consistent Messaging:**
   Coordinated adoption and explanation by major platforms ensure users quickly understand the change.

4. **In-App Warnings:**
   Applications can prompt users for confirmation if entered amounts deviate significantly from historical norms.

The integer-based display reduces long-term confusion, as users become accustomed to the true nature of Bitcoin’s accounting. Over time, this clarity enhances security by reducing the risk of user errors due to misunderstanding.

# Reference Implementation

- **Wallets:**  
  Display all amounts as integers. Optionally maintain legacy-to-new comparison views or explanatory tooltips during transition.
  
- **Block Explorers:**  
  Show amounts as integers and annotate historical data to explain old fractional formats.

- **Developer Tools:**
  Remove conversion logic; document the single-integer representation as canonical.

# Test Vectors

- Old: `1.00000000 BTC` → New: `100000000 BTC` (or `₿100000000`)  
- Old: `0.00010000 BTC` → New: `10000 BTC` (or `₿10000`)  
- Old: `0.00500000 BTC` → New: `500000 BTC` (or `₿500000`)

All previously fractional representations now map directly to whole-number multiples of the smallest unit.

# Implementation Timeline

- **Phase 1 (3-6 months):**
  Introduce the concept with optional dual displays and detailed educational materials.
  
- **Phase 2 (6-12 months):**
  Prominent services adopt the integer-only display by default.
  
- **Phase 3 (12+ months):**
  The integer representation becomes the norm. Documentation and user guides no longer reference the old decimal-based format.

# Conclusion

By redefining the "bitcoin" unit as the smallest indivisible unit and abandoning the decimal-based convention, this BIP improves clarity, reduces user confusion, and ensures that the displayed format faithfully represents the protocol’s actual integer-based accounting. While transitional adjustments are necessary, the long-term benefit is a more accurate understanding of Bitcoin’s fundamental nature, ultimately making the system easier to use and less error-prone.
