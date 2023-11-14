---
description: Inscribe references to content by URI
---

# Reference Inscriptions

You can inscribe references to on-chain content using the text/uri-list content type defined here:\
[https://www.rfc-editor.org/rfc/rfc2483#section-5](https://www.rfc-editor.org/rfc/rfc2483#section-5)

### &#x20;Examples

```
# This is a comment
https://mydomain.com/something.pdf
```

In this example we reference a JSON inscription. The file extension is useful for hinting before the content has been loaded, allowing clients to make UI decisions sooner (like choosing the correct component to render the content).

Thanks to [ord-fs](https://ordfs.network) we can reference any b:// c:// or ord:// content by relative path:

```
# This is an image inscription
/content/<outpoint | contentHash>.png
```

For more information on how ord-fs renders content, visit [ordfs.network](https://ordfs.network).&#x20;

### Multiple Files

You can also reference many files in a single text/uri-list inscription:

```
/content/<outpoint1>.html
/content/<outpoint2>.js
/content/<outpoint3>.webp
/content/<outpoint4>.css
```

In this example we can make the contents of a webpage and all of its dependencies available as a single package.
