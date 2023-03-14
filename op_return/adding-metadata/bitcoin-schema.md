---
description: Open Social Integration
---

# Bitcoin Schema

Bitcoin Schema is a set of pre-defined data schemas that form overlay networks on Bitcoin. Bitcoin Schema uses a combination of data protocols to form the building blocks of new networks, such as "BSocial" - an open, interoperable social media network.

For example, you can create an Ordinal that is also a BSocial post, make a comment with B protocol, and use MAP to assign the "post" type to the transaction. The pseudoscript would look like this:

```bash
1SAT_P2PKH INSCRIPTION OP_RETURN B_POST_TEXT "|" MAP SET app <appName> type "post"
```

When broadcasted, apps that support BSocial will see the post and display the commend and inscription.

### More Information

To learn more about using common metadata structures, and how to use them to create interoperable Bitcoin apps, check out [BitcoinSchema.org](https://bitcoinschema.org).
