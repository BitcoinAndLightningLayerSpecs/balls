    BIP: XXX  
    Layer: Informational  
    Title: Redefinition of the Bitcoin Unit to the Base Denomination  
    Author: [Your Name] <your.email@example.com>  
    Comments-Summary: No comments yet.  
    Comments-URI: https://github.com/bitcoin/bips/wiki/Comments-URI  
    Status: Draft  
    Type: Informational  
    Created: 2024-12-10  
    License: CC0-1.0

# Abstract

This proposal recommends redefining the commonly recognized "bitcoin" unit from the current notation of 1 BTC = 100,000,000 base units (sometimes informally known as "satoshis") to instead define 1 bitcoin as the base unit itself. Under this proposal, the term "bitcoin" directly references the smallest indivisible unit used on the network, removing the need for decimals. This shift aims to streamline unit representation, making it simpler for users to comprehend, display, and transact with Bitcoin.

# Motivation

From its inception, Bitcoin’s smallest indivisible unit has been tracked by the protocol as a whole number. However, conventional usage established that 1 BTC = 100 million of these smallest units. Over time, this convention has led to confusion for users who must navigate a value system requiring eight decimal places. As Bitcoin continues to gain adoption, the disconnect between the network’s integer representation and the user-facing decimal formatting persists as a barrier to intuitive understanding.

By no longer using a larger unit and instead presenting the base unit as the standard "bitcoin," users can reason about values in whole numbers without arbitrary decimal points. This promotes clarity and aligns with the natural integer-based structure of the Bitcoin protocol.

# Specification

1. **Redefinition of Unit:**
   - At the protocol level, the fundamental smallest unit remains the same integer-based measure.  
   - Historically, what many called "one bitcoin" was 100,000,000 of these units. Going forward, 1 bitcoin **is** that smallest unit.  
   - What was previously considered "1 BTC" (old definition) is now simply 100 million bitcoins.

2. **Term Usage:**
   - The informal terms "satoshi" or "sat" are to be phased out in favor of using "bitcoin" exclusively for the base unit.  
   - Documentation and user interfaces should avoid references to these legacy informal terms, except as historical context (e.g., "previously sometimes informally referred to as 'satoshis'").

3. **Display and Formatting:**
   - User interfaces (wallets, block explorers, payment systems) SHOULD present all values as whole integers without decimal points.  
   - For example, what was once displayed as `0.00010000 BTC` under the old convention will now be displayed simply as `10000 BTC` or `₿10000` under the new definition.

4. **Rounding and Conversion:**
   - No consensus-level changes occur. The ledger remains the same.  
   - Clients adopting this standard MUST interpret previously displayed BTC amounts by multiplying them by 100,000,000. The resulting integer count directly corresponds to the on-chain integer units that have always been in place.

# Rationale

1. **Simplicity and User-Friendliness:**  
   Decimal-based value representation has long been a source of confusion. Eliminating decimals enables users to read and reason about Bitcoin amounts in straightforward whole numbers.

2. **Alignment with the Protocol:**  
   The Bitcoin protocol internally accounts for values as integers. Representing the user-facing denomination as that same integer unit closes the gap between protocol and perception.

3. **Clarity for Onboarding:**  
   Newcomers often struggle to understand the concept of fractional bitcoins and multi-decimal display. Presenting all values as whole integers simplifies education and understanding.

4. **Long-Term Relevance:**  
   As usage grows, discussing Bitcoin in fractional terms may become increasingly cumbersome. By using the smallest unit as "bitcoin," we future-proof the interface against the protocol’s built-in precision.

# Backward Compatibility

This proposal is a semantic and representational change. It does not alter the underlying protocol or consensus rules:

- **For Software Developers:**  
  Developers must update interfaces and APIs to remove decimal handling. The APIs may now return values directly as integers. References to "satoshis" should be removed except as historical context.

- **For Users:**  
  Although this shift in terminology may initially cause confusion, it ultimately simplifies the user experience. Over time, updated wallet interfaces and educational materials will clarify the new standard.

# Security Considerations

No changes to consensus or underlying protocol logic means no direct security implications. The primary risk is temporary user confusion during the transition. Clear communication, educational efforts, and allowing optional legacy formatting for a short period can mitigate these risks.

# Reference Implementations

- **Wallets:**  
  Wallet frontends should display account balances and transaction amounts as whole integer units. Where legacy formatting was once used, a reference or tooltip can clarify historical conversions during the transition period.

- **Block Explorers:**  
  Explorers can show all amounts as integers. Historical data could be annotated to explain the previous usage of decimal formatting and the informal "satoshi" term.

- **Developer Tools:**  
  Libraries previously responsible for converting between BTC and base units can remove those conversion steps. Documentation should note that what was once a "satoshi" is now simply a "bitcoin."

# Test Vectors

- **Legacy to New Representation:**  
  - Old: `1.00000000 BTC` → New: `100000000 BTC` or `₿100000000`  
  - Old: `0.00010000 BTC` → New: `10000 BTC` or `₿10000`  
  - Old: `0.00500000 BTC` → New: `500000 BTC` or `₿500000`

- **No Decimals:**  
  Confirm that no fractional values appear. Every amount corresponds to a whole number of bitcoins.

# Implementation Timeline

- **Phase 1 (3-6 months):**  
  - Introduce the concept through community discussions, educational materials, and software updates with an optional toggle.
  
- **Phase 2 (6-12 months):**  
  - Prominent wallets, exchanges, and services adopt the integer standard by default.  
  - Deprecate references to "satoshis," limiting them to historical footnotes.

- **Phase 3 (12+ months):**  
  - The community fully embraces and normalizes the base unit as "bitcoin."  
  - Educational materials, documentation, and user interfaces no longer reference decimals or informal terms.

# Conclusion

By redefining the "bitcoin" unit to the base denomination and discontinuing the use of informal terms, this proposal simplifies Bitcoin’s unit representation. Eliminating decimals and legacy informal terms provides a more intuitive, user-friendly, and protocol-aligned approach to value measurement. Over time, this will foster clearer communication, enhance educational efforts, and offer a cleaner mental model for all participants in the Bitcoin ecosystem.
