wchar-clarification-request
===========================

It is difficult and error-prone to write correct C programs for handling non-ASCII text.

* Remove references to single bytes as "characters"
* Discourage use of `ctype` functions in favor of `wctype`
* Add functions to idiomatically iterate `char *` strings as `wchar_t`
* Clarify guarantees of `wchar_t`

## 3.4.2 locale-specific behavior

> **EXAMPLE** An example of locale-specific behavior is whether the `iswlower` function returns true for wide characters other than the 26 lowercase Latin letters.
