wchar-clarification-request
===========================

It is difficult and error-prone to write correct C programs for handling non-ASCII text using only the facilities provided by the standard C library.

The C99 standard attempted to provide a complete set of the necessary types, conversions, and utility functions for two text representations: multibyte characters as sequences of `char` bytes, and wide characters as single `wchar_t` code points. However,

* `wchar_t` is popularly regarded as unportable and insufficent, because some platforms prematurely defined it to be only 16 bits wide and are locked in to that decision.
* The stream orientation concept for I/O puts programs that attempt to read or write multibyte text in constant danger of accidentally invoking undefined behavior.
* The standard does not specify whether the digits of `wchar_t` are contiguous and in order.
* There is no way to enumerate the members of `wctype` character classes.
* The `mbrtowc` function is clumsy and verbose to use as a character iterator.

The C11 standard added three new character representations: UTF-8 multibyte strings and 16- and 32-bit wide characters. However, none of these types are useful, because no I/O interfaces or `ctype` functions are defined for them. UTF-8 strings are even less useful because no conversion functions are defined between them and any of the other character representations.

## Proposal

Taking as a given that `wchar_t` is frozen on existing platforms and cannot be used as a portable character type because it is too narrow, I propose:

* Using `char32_t` as the ordinary way of referring to code points.
* Defining the standard set of `ctype` and `string` functions for `char32_t` characters and strings.
* Adding a `c32int_t` type and `C32EOF` constant for `char32_t` streams.
* Defining the standard set of `stdio` functions for `char32_t` characters, with well-defined conversion rules rather than undefined behavior when reading or writing the wrong kind of data.
* Defining `nextc32type` (on the model of the BSD `nextwctype`) as a way to enumerate character classes.
* Adding `digitc32toint` and `inttodigitc32` functions (on the model of the BSD `digittoint`) to convert between digit values and their corresponding characters (or defining the order of digits in `char32_t`).
* Defining `c32at`, `c32post`, and `c32pre` character iteration functions that operate as idiomatic equivalents to `*cp`, `cp++`/`*cp++`, and `*++cp`.
* Systematically referring to `char` objects as "bytes" rather than "characters" throughout the standard.

These changes will enable a text handling model where strings are generally read from input streams and stored in memory as arrays of `char`, as is basically required by file system and operating system interfaces, but can easily be iterated, examined, and manipulated as `char32_t` code points, before being written back to arrays of `char` (using the existing `c32tomb` function) or to output streams.

## Extended proposal

It is still very easy to accidentally use a `'c'` narrow character constant where a `U'c'` `char32_t` character constant is intended, or to accidentally check against `EOF` instead of `C32EOF`. For this reason, perhaps the standard wide character type should actually be a structured type:

```
typedef struct {
    char32_t c;
} codepoint_t;
```

so that it is impossible to accidentally compare or assign between bytes and codepoints without doing an appropriate conversion.

## What's wrong with I/O?

The very fact that `printf("%ls", …)` and `wprintf("%s", …)` exist indicate that it is possible to write wide strings to narrow streams, and vice versa, but these operations appear to be forbidden by the insistence that "Byte input/output functions shall not be applied to a wide-oriented stream and wide character input/output functions shall not be applied to a byte-oriented stream."

Even if a programmer is extremely disciplined and avoids mixing character widths within their own program, it is difficult to control what other libraries will do. In particular, many libraries will write to `stderr` assuming it is a narrow stream. It is therefore difficult for a program to know that it can safely write wide strings to `stderr`.

By using the same `mbstate_t` conversion state object for streams as for in-memory operations, the standard effectively requires, without ever actually saying, that a wide stream on disk is represented as multibyte characters in the same way that the same wide characters would be represented as multibyte characters in memory. If this can be made explicit, then the stream orientation concept is rendered unnecessary, as the following operations are all legitimate:

* Writing a line of text to a file as a string of wide characters
* Writing the same line of text to a file one wide character at a time
* Writing the same line of text to a file as a byte array of multibyte characters
* Writing those same multibyte characters to a file one byte at a time
* Reading a line of text from a file as a string of wide characters
* Reading the same line of text from a file one wide character at a time
* Reading the same line of text from a file as a byte string of multibyte characters
* Reading those same multibyte characters from the file one byte at a time

It should of course create an invalid file of multibyte characters (or return an error to the writer) if a byte that is not part of a legitimate multibyte character is written to it, and it should likewise be an error to try to read a wide character from a file if the next available byte is not part of a legitimate multibyte character. But these are errors that can be diagnosed, not undefined behavior that cannot be.

-----


A more workable requirement for output would be that it is always OK to write either bytes or wide characters to a stream if its internal `mbstate_t` is in the initial state, and that the internal `mbstate_t` will always be left in the initial state after writing a wide or narrow newline or after calling `fflush` on the output stream.

## A workable file model

The thing that the standard does not claim, but that programs need in practice, is that the representation of wide characters in an I/O stream is the same as the representation of those characters as a multibyte string in memory.

## Specifics:

### Printf (in uchar.h):

* `fu32printf`
  * `%lls` format specifier for `uchar32_t *` strings
  * `%llc` format specifier for `uchar32_t` characters
* `fu32scanf`
  * `%lls` and `%llc` format specifiers
* `su32printf`
* `su32scanf`
* `vfu32printf`
* `vfu32scanf`
* `vsu32printf`
* `vsu32scanf`
* `vu32printf`
* `vu32scanf`
* `u32printf`
* `u32scanf`

### Character I/O (in uchar.h)

* `fgetu32c`
* `fgetu32s`
* `fputu32c`
* `fputu32s`
* `getu32c`
* `getu32char`
* `putu32c`
* `putu32char`
* `ungetu32c`

### Stream orientation

* `fwide` equivalent: tk

### Numeric conversion (in uchar.h)

* `u32stod`
* `u32stof`
* `u32stold`
* `u32stol`
* `u32stoll`
* `u32stoul`
* `u32stoull`

### String manipulation

* `u32scpy`
* `u32sncpy`
* `u32memcpy`
* `u32memmove`
* `u32scat`
* `u32scmp`
* `u32scoll`
* `u32sncmp`
* `u32sxfrm` (should be specified in more detail)
* `u32memcmp`
* `u32schr`
* `u32scspn`
* `u32sprbrk`
* `u32srchr`
* `u32sspn`
* `u32sstr`
* `u32stok`
* `u32memchr`
* `u32len`
* `u32memset`
* `u32sftime`






------------------------------------------------------------------

## Goals

Goals of this request:

* Refer to the storage class of `char` as "byte" instead of "character"
* Discourage use of `ctype` functions in favor of `wctype`
* Add functions to idiomatically iterate `char *` strings as `wchar_t`
* Clarify representation of digits of `wchar_t`
* Clarify what sets and what depends upon stream orientation
* Clarify how to initialize `mbstate`

## 3.4.2 locale-specific behavior

> **EXAMPLE** An example of locale-specific behavior is whether the `iswlower` function returns true for wide characters other than the 26 lowercase Latin letters.

## 5.1.2.3 Program execution

Use `getwchar` instead, after the representation of digits is resolved.

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

> 5. Certain object representations need not represent a value of the object type. If the stored value of an object has such a representation and is read by an lvalue expression that does not have byte type, the behavior is undefined. If such a representation is produced by a side effect that modifies all or any part of the object by an lvalue expression that does not have byte type, the behavior is undefined.<sup>50)</sup> Such a representation is called a trap representation.

## 6.3.1.1 Boolean, bytes, and integers

(change to heading)

## 6.3.2.3 Pointers

> 7. A pointer to an object type may be converted to a pointer to a different object type. If the resulting pointer is not correctly aligned<sup>69)</sup> for the referenced type, the behavior is undefined. Otherwise, when converted back again, the result shall compare equal to the original pointer. When a pointer to an object is converted to a pointer to a byte type, the result points to the lowest addressed byte of the object. Successive increments of the result, up to the size of the object, yield pointers to the remaining bytes of the object.

## 6.5 Expressions

> 6. The *effective type* of an object for an access to its stored value is the declared type of the object, if any.<sup>88)</sup> If a value is stored into an object having no declared type through an lvalue having a type that is not a byte type, then the type of the lvalue becomes the effective type of the object for that access and for subsequent accesses that do not modify the stored value. If a value is copied into an object having no declared type using `memcpy` or `memmove`, or is copied as an array of byte type, then the effective type of the modified object for that access and for subsequent accesses that do not modify the value is the effective type of the object from which the value is copied, if it has one. For all other accesses to an object having no declared type, the effective type of the object is simply the type of the lvalue used for the access.
>
> …
> — a byte type.
