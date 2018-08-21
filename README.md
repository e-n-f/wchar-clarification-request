wchar-clarification-request
===========================

It is difficult and error-prone to write correct C programs for handling non-ASCII text.

Goals of this request:

* Refer to the storage class of `char` as "byte" instead of "character"
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

## 6.2.5 Types

> 15. The three types `char`, `signed char`, and `unsigned char` are collectively called the *byte* types. The implementation shall define `char` to have the same range, representation, and behavior as either `signed char` or `unsigned char`. <sup>45)</sup>

> 28. A pointer to `void` shall have the same representation and alignment requirements as a pointer to a *byte* type.<sup>48)</sup> Similarly, pointers to qualified or unqualified versions of compatible types shall have the same representation and alignment requirements. All pointers to structure types shall have the same representation and alignment requirements as each other. All pointers to union types shall have the same representation and alignment requirements as each other. Pointers to other types need not have the same representation or alignment requirements.

## 6.2.6 Representations of types

> 5. Certain object representations need not represent a value of the object type. If the stored value of an object has such a representation and is read by an lvalue expression that does not have byte type, the behavior is undefined. If such a representation is produced by a side effect that modifies all or any part of the object by an lvalue expression that does not have character type, the behavior is undefined.<sup>50)</sup> Such a representation is called a trap representation.

## 6.3.1.1 Boolean, bytes, and integers

(change to heading)

## 6.3.2.3 Pointers

> 7. A pointer to an object type may be converted to a pointer to a different object type. If the resulting pointer is not correctly aligned<sup>69)</sup> for the referenced type, the behavior is undefined. Otherwise, when converted back again, the result shall compare equal to the original pointer. When a pointer to an object is converted to a pointer to a byte type, the result points to the lowest addressed byte of the object. Successive increments of the result, up to the size of the object, yield pointers to the remaining bytes of the object.
