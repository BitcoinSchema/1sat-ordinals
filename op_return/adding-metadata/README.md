# Adding Metadata

You can use metadata to add context to ordinals. For example, you can tag an ordinal with a geohash to place it on a physical location. Metadata is written using "Magic Attribute Protocol" which has a prefix of `1PuQa7K62MiKCtssSLKy1kh56WWU7MtUR5`.

{% code overflow="wrap" %}
```bash
1SAT_P2PKH <INSCRIPTION> OP_RETURN <B fields...> | MAP SET app <platform_name> type "post" context "geohash" geohash "dhmgdqvr7"
```
{% endcode %}

Similarly, you can attach an ordinal to a particular url:

{% code overflow="wrap" %}
```bash
1SAT_P2PKH <INSCRIPTION> OP_RETURN <B fields...> | MAP SET app <platform_name> type "post" context "url" url "https://google.com"
```
{% endcode %}

## The "Ord" type

We have begin drafting a proposed specification for Ordinal inscription metadata called "ord". The draft spec is available here:

\
[https://gist.github.com/danwag06/53dcf913c8fd0de7a0233faafb51083f](https://gist.github.com/danwag06/53dcf913c8fd0de7a0233faafb51083f)
