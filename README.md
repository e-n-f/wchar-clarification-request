wchar-clarification-request
===========================

It is difficult and error-prone to write correct C programs for handling non-ASCII text.

* Remove references to single bytes as "characters"
* Discourage use of `ctype` functions in favor of `wctype`
* Add functions to idiomatically iterate `char *` strings as `wchar_t`
* Clarify guarantees of `wchar_t`
* Clarify what sets and what depends upon stream orientation
* Clarify how to initialize `mbstate`

## 3.4.2 locale-specific behavior

> **EXAMPLE** An example of locale-specific behavior is whether the `iswlower` function returns true for wide characters other than the 26 lowercase Latin letters.

## 5.1.2.3 Program execution

Use `getwchar` instead:

## 5.2.1 Character sets

Existing:

> In both the source and execution basic character sets, the value of each character after 0 in the above list of decimal digits shall be one greater than the value of the previous.

Preferred:

> In both the source and execution character sets (including multibyte and wide characters), the value of each character after 0 in the above list of decimal digits shall be one greater than the value of the previous.

Alternative:

Leave unchanged, but provide a library function to convert `wchar_t` where `iswdigit` is `true` to values `0` through `9`.

## 5.2.1.2 Multibyte characters

Is there a corresponding section for wide characters?

## 5.2.2 Character display semantics

> The *active position* is that location on a display device where the next character output by the `fputwc` function would appear. The intent of writing a printing character (as defined by the `iswprint` function) to a display device is to display a graphic representation of that character at the active position and then advance the active position to the next position on the current line. The direction of writing is locale-specific. If the active position is at the final position of a line (if there is one), the behavior of the display device is unspecified.
>
> **Forward references**: the `iswprint` function (7.4.1.8), the `fputwc` function (7.21.7.3)
