# Introduction

There are some significant differences between BTC, where ordinals originated, and a classic Bitcoin system. Most notably, Bitcoin SV does not have taproot or segregated witness, which Ordinals leverage to inscribe data.

This is not a problem, since data limits are much higher, we can simply encode ordinals directly into output scripts, following the same pushdata scheme, and creating an inscribed ordinal in a single step.

### 1Sat Protocol

Ordinals target a specific satoshi but in practice a range is typically used to satisfy dust limits on other chains. Since BSV has no dust limit, we can dramatically simplify things by truly inscribing a single satoshi. The protocol name highlights this ability, and we focus on 1 sat outputs in these documents, but it does not mean we cannot add more sats to Ordinals in the future if it is wanted by the community.

### Interoperability

With that said, the spirit of the oridinal Ordinals protocol is alive and well here. Ordinal numbers, inscription numbers, and data formats are identical. We're hoping the momentum from several chains adopting Ordinals in some form could lead to some interesting developments in cross-chain interoperability.
