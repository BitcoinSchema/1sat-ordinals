# Introduction

There are several major differences between chains like BTC where ordinals originated, and a classic Bitcoin system like BSV. Most notably, BSV does not have taproot or segwit, so it does not need a workaround to store data on-chain. We can simply put the ordinals script in an output, creating an inscribed ordinal in a single step.

Ordinals target a specific satoshi but in practice a range is typically used to satisfy dust limits on other chains. Since BSV has no dust limit, we can dramatically simplify things by truly inscribing a single satoshi.
