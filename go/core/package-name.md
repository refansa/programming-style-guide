# Package Names

When naming packages, choose a name that is:

- All lower-case. Underscore is permissible for multi-word names. No capitals.
- Does not need to be renamed using named imports at most call sites.
- Short and succinct. Remember that the name is identified in full at every call
  site.
- Not plural. For example, `net/url`, not `net/urls`.
- Not "common", "util", "shared", or "lib". These are bad, uninformative names.

See also [Package Names] and [Style guideline for Go packages].

  [Package Names]: https://go.dev/blog/package-names
  [Style guideline for Go packages]: https://rakyll.org/style-packages/
