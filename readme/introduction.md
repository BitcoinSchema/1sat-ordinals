# Introduction

<figure><img src="../.gitbook/assets/what.png" alt=""><figcaption></figcaption></figure>

There are some significant differences between BTC, where ordinals originated, and Bitcoin SV. Most notably, BSV does not have taproot or segregated witness, which Ordinals protocol leverages to inscribe data on BTC.

Since data limits are much higher on Bitcoin SV, we can encode ordinals directly to output scripts, following the same pushdata scheme, and creating an inscribed ordinal in a single stage compared to the two-step commit + reveal process on BTC.

### 1Sat Protocol

Ordinals are based on a numbering scheme that targets a specific satoshi. In practice, a range of Satoshis is used to satisfy dust limits. Since Bitcoin SV has no dust limit, we can simplify further by inscribing a single satoshi. The protocol name highlights this capability, and we focus on 1 satoshi outputs in these documents. It's important to note, we can easily add support for additional sats in a compatible way.

### Interoperability

The spirit of the original Ordinals protocol is alive and well here. Ordinal numbers, inscription numbers, and data formats are identical. We're hoping that a common protocol makes it easy for others to experience the value BSV can add to these ideas, and build cool stuff leveraging the interoperability along the way.
