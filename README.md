wchar-clarification-request
===========================

It is difficult and error-prone to write correct C programs for handling non-ASCII text.

* Remove references to single bytes as "characters"
* Discourage use of `ctype` functions in favor of `wctype`
* Add functions to idiomatically iterate `char *` strings as `wchar_t`
* Clarify guarantees of `wchar_t`
* Clarify what sets and what depends upon stream orientation

## 3.4.2 locale-specific behavior

> **EXAMPLE** An example of locale-specific behavior is whether the `iswlower` function returns true for wide characters other than the 26 lowercase Latin letters.

## 5.1.2.3 Program execution

Use `getwchar` instead:

## 5.2.1 Character sets

Existing:

> In both the source and execution basic character sets, the value of each character after 0 in the above list of decimal digits shall be one greater than the value of the previous.

Preferred:

> In both the source and execution character sets (including wide characters), the value of each character after 0 in the above list of decimal digits shall be one greater than the value of the previous.

Alternative:

Leave unchanged, but provide a library function to convert `wchar_t` where `iswdigit` is `true` to values `0` through `9`.
