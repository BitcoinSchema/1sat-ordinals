## Common Questions

### Can we merge items?

```
Yes via API not via script. This keeps typical token usage simple and easy to manage.
```

### Why not just use Run?

```
Run is no longer in development, and has some unique chalenges. Building the Run-Sdk in webpack bundled projects and the requirement for JavaScript to interprate token transactions has created a demand for alternatives.
```

### Why not just use Stas?

```
Stas scripts are larger than ordinals. Both systems ultimately require some level of indexer-based validation, and both can use OP_RETURN based token data. Stas protocol is an owned and licensed protocol. We wanted something that would be publicly available without an individual licensing requirement.
```
