
BIP: XXX
Title: Redefinition of the Bitcoin Unit to the Base Denomination
Authors: John Carvalho <bitcoinerrorlog@gmail.com>
Status: Draft
Type: Informational
Created: 2024-12-10
License: CC0-1.0

# Abstract

This BIP proposes redefining the commonly recognized "bitcoin" unit so that what was previously known as the smallest indivisible unit becomes the primary reference unit. Under this proposal, one bitcoin is defined as that smallest unit, eliminating the need for decimal places. By making the integral unit the standard measure, this BIP aims to simplify user comprehension, reduce confusion, and align on-chain values directly with their displayed representation.

# Motivation

The current convention defines one BTC as 100,000,000 of the smallest indivisible units. This representation requires dealing with eight decimal places, often causing confusion and encouraging the misconception that bitcoin is inherently decimal-based. In reality, Bitcoin’s ledger tracks and stores values as integers of a smallest unit, and the decimal point is a human-imposed abstraction.

By recognizing the smallest unit as "one bitcoin," this BIP aligns user perceptions with the protocol’s true nature. It reduces the cognitive load of dealing with decimals, ensures that users understand Bitcoin as counting discrete units, and fosters a more accurate and accessible user experience.

# Specification

**Redefinition of the Unit:**

- Internally, the smallest indivisible unit remains unchanged.
- Historically, 1 BTC = 100,000,000 base units. Under this proposal, "1 bitcoin" equals this smallest unit.
- The entity previously referred to as "1 BTC" now corresponds to 100 million bitcoins.

**Terminology:**

- The informal terms "satoshi" or "sat" are deprecated.
- All interfaces and documentation SHOULD refer to the base integer unit simply as "bitcoin."

**Display and Formatting:**

- Applications SHOULD present values as whole integers, without decimals.
- Example:
  - Old display: `0.00010000 BTC`
  - New display: `10000 BTC` (or `₿10000`)

**Conversion:**

- Ledger and consensus rules remain unchanged.
- To adopt this standard, previously displayed BTC amounts are multiplied by 100,000,000 to yield the new integer representation.

# Rationale

**Usability:**  
Integer-only displays simplify mental arithmetic and minimize potential errors.

**Protocol Alignment:**  
Since Bitcoin inherently counts integral units, presenting values as integers more faithfully reflects its design.

**Educational Clarity:**  
By removing decimals, newcomers are not misled into thinking Bitcoin is decimal-based. This better communicates Bitcoin’s fundamental nature from the outset.

**Future-Proofing:**  
Embracing the smallest unit as the primary measure ensures a consistent standard that scales with future adoption and use cases.

# Backward Compatibility

No changes are made to consensus rules or on-chain data. The only alteration is in how values are displayed:

- **For Developers:** Update GUIs, APIs, and documentation to show values as integers. Remove references to fractional BTC.
- **For Users:** Users hold the same amount of bitcoin; only the representation changes. Providing transitional aids, such as dual displays or explanatory tooltips, can ease the shift.

# Security Considerations

Short-term confusion may occur as users adapt to the new integer display. They might momentarily misinterpret values if accustomed to decimals. Mitigations include:

- Dual-mode displays and tooltips during the transition.
- Clear educational materials and coordinated messaging.
- Prompts or alerts in applications if input values appear unexpectedly large or small.

Over time, confusion will diminish, leaving a simpler, more intuitive representation.

# Reference Implementation

Wallets like Bitkit have successfully adopted integer-only displays, showing that this approach can be practical without significantly confusing users. During a transitional period, applications can display both formats or provide contextual explanations.

# Test Vectors

- Old: `1.00000000 BTC` → New: `100000000 BTC` (`₿100000000`)
- Old: `0.00010000 BTC` → New: `10000 BTC` (`₿10000`)
- Old: `0.00500000 BTC` → New: `500000 BTC` (`₿500000`)

All formerly fractional amounts now map directly to whole integers.

# Implementation Timeline

**Phase 1 (3-6 months):**  
Introduce the concept with optional dual displays and educational materials.

**Phase 2 (6-12 months):**  
Prominent services adopt integer-only displays by default.

**Phase 3 (12+ months):**  
Integer representation becomes the norm. Documentation no longer references decimal-based formats.

# Conclusion

By redefining the "bitcoin" unit as the smallest indivisible unit and removing decimal-based representations, this BIP simplifies user understanding and ensures that displayed values accurately reflect Bitcoin’s integral, integer-only accounting. Though the transition may require short-term adjustments, the long-term benefits include more intuitive usage, reduced confusion, and alignment with Bitcoin’s fundamental design.

# Copyright

This BIP is licensed under CC0-1.0.
