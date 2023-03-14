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
