# Common Questions

## Can this be combined with other Bitcoin scripts?

Yes. It is possible to use any Bitcoin locking script you prefer when sending a 1Sat Ordinal. The ordinal number is calculated based on its exact lineage, meaning the script can be changed without impacting the Ordinal number. You can also use custom scripts while inscribing new Ordinals.

## Why not use a smart contract that does X?

This protocol is intended to adhere to the original Ordinals protocol as closely as possible withing reason and with some minor exceptions. This would be too far a departure for this from the precedent set on BTC for this project.

## Can 1Sat Ordinals be merged?

Not via script or by protocol but this can be accomplished at the application layer. The user sends the tokens to be merged to an API and which redeems them and issues the new "merged" Ordinal. This keeps typical token usage simple and easy to manage. This strategy can be used for "breeding" games, or anything that might require two or more Ordinals become one.

## Why not use Run?

Run is no longer in development, and has some unique challenges. Problems building the Run-Sdk with webpack and the requirement for a JS-based environment has created a demand for alternatives.

## Why not use Stas?

Stas scripts are larger than ordinals. Both systems ultimately require some level of indexer-based validation, and both can use OP_RETURN based token data. Stas protocol has licensing requirements for minting. This was something some would prefer to avoid.
