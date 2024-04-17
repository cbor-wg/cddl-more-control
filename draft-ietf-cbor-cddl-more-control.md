---
v: 3

title: >
  More Control Operators for CDDL
abbrev: CDDL control operators
docname: draft-ietf-cbor-cddl-more-control-latest
# date: 2024-02-26
cat: std
consensus: true
stream: IETF

keyword:
 - Concise Data Definition Language
 - Control Operator
venue:
  group: "Concise Binary Object Representation (CBOR) Maintenance and Extensions"
  mail: "cbor@ietf.org"
  github: "cbor-wg/cddl-more-control"
  latest: "https://cbor-wg.github.io/cddl-more-control/"

pi: [toc, sortrefs, symrefs, compact, comments]

author:
  -
    ins: C. Bormann
    name: Carsten Bormann
    org: Universität Bremen TZI
    street: Postfach 330440
    city: Bremen
    code: D-28359
    country: Germany
    phone: +49-421-218-63921
    email: cabo@tzi.org


normative:
  STD90:
    -: json
#    =: RFC8259
  STD94:
    -: cbor
#    =: RFC8949
  RFC8610: cddl
  IANA.cddl:
  RFC9165: control1
  RFC4648: base
  RFC9285: base45 # downref!
  RFC9485: iregexp
  C:
    target: https://www.iso.org/standard/74528.html
    title: Information technology — Programming languages — C
    author:
    - org: International Organization for Standardization
    date: 2018-06
    seriesinfo:
      ISO/IEC: 9899:2018
    refcontent:
    - Fourth Edition
    ann:
    - |
        &#x2028; <!-- work around broken annotation content model -->
        Technically equivalent specification text is available at <https://web.archive.org/web/20181230041359if_/http://www.open-std.org/jtc1/sc22/wg14/www/abq/c17_updated_proposed_fdis.pdf>
informative:
  RFC7464: jsonseq

--- abstract

The Concise Data Definition Language (CDDL), standardized in RFC 8610,
provides "control operators" as its main language extension point.
RFCs have added to this extension point both in an
application-specific and a more general way.

The present document defines a number of additional generally
applicable control operators for text conversion (Bytes, Integers,
JSON, Printf-style formatting) and for an operation on text.

<!--
[^status]

[^status]:  Revision –00 of this WG draft ...
 -->

--- middle

Introduction        {#intro}
============

The Concise Data Definition Language (CDDL), standardized in {{-cddl}},
provides "control operators" as its main language extension point
({{Section 3.8 of -cddl}}).
RFCs have added to this extension point both in an
application-specific {{?RFC9090}} and a more general {{RFC9165}} way.

The present document defines a number of additional generally
applicable control operators:

| Name                           | Purpose                                                   |
| `.b64u`, `.b64c`               | Base64 representation of byte strings                     |
| `.b64u-sloppy`, `.b64c-sloppy` | (sloppy-tolerant variants of the above)                   |
| `.hex`, `.hexlc`, `.hexuc`     | Base16 representation of byte strings                     |
| `.b32`, `.h32`                 | Base32 representation of byte strings                     |
| `.b45`                         | Base45 representation of byte strings                     |
| `.decimal`                     | Text representation of integer numbers                    |
| `.printf`                      | Printf-formatted text representation of data items        |
| `.json`                        | Text representation of JSON values                        |
| `.join`                        | Building text from array of components                    |
{: #tbl-new title="New control operators in this document"}

Terminology
-----------

{::boilerplate bcp14-tagged-bcp}

Regular expressions mentioned in the text are as defined in {{-iregexp}}.

This specification uses terminology from {{-cddl}}.
In particular, with respect to control operators, "target" refers to
the left-hand side operand, and "controller" to the right-hand side operand.
"Tool" refers to tools along the lines of that described in {{Appendix F of -cddl}}.
Note also that the data model underlying CDDL provides for text
strings as well as byte strings as two separate types, which are
then collectively referred to as "strings".

Text Conversion
===============

Byte Strings: Base16 (Hex), Base32, Base45, Base64
----------------------------

A CDDL model often defines data that are byte strings in essence but
need to be transported in various encoded forms, such as base64 or
hex.
This section defines a number of control operators to model these
conversions.

The control operators generally are of a form that could be used like
this:

~~~ cddl
signature-for-json = text .b64u signature
signature = bytes .cbor COSE_Sign1
~~~
{: sourcecode-name="example1.cddl"}

The specification of these control operators is complicated by the
large number of transformations in use.  Inspired by {{Section 8 of
RFC8949@-cbor}}, we use representations defined in {{-base}} with the following
names:

| name           | meaning                         | reference            |
| `.b64u`        | Base64URL, no padding           | {{Section 5 of -base}} |
| `.b64u-sloppy` | Base64URL, no padding, sloppy   | {{Section 5 of -base}} |
| `.b64c`        | Base64 classic, padding         | {{Section 4 of -base}} |
| `.b64c-sloppy` | Base64 classic, padding, sloppy | {{Section 4 of -base}} |
| `.b32`         | Base32, no padding              | {{Section 6 of -base}} |
| `.h32`         | Base32/hex alphabet, no padding | {{Section 7 of -base}} |
| `.hex`         | Base16 (hex), either case       | {{Section 8 of -base}} |
| `.hexlc`       | Base16 (hex), lower case        | {{Section 8 of -base}} |
| `.hexuc`       | Base16 (hex), upper case        | {{Section 8 of -base}} |
| `.b45`         | Base45                          | {{-base45}}            |
{: title="Control Operators for Text Conversion of byte strings"}

Note that this specification is somewhat opinionated here: It does not
provide base64url, base32 or base32hex encoding with padding, or
base64 classic without padding.  Experience indicates that these
combinations only ever occur in error, so the usability of CDDL is
increased by not providing them in the first place.  Also, adding "c"
makes sure that any decision for classic base64 is actively taken.

The additional designation "sloppy" indicates that the text string is
not validated for any additional bits being zero, in variance to what
is specified in the paragraph behind table 1 in {{Section 4 of -base}}.
Note that the present specification is opinionated again in not
specifying a sloppy variant of base32 or base32/hex, as no legacy use
of sloppy base32(/hex) was known at the time of writing.
Base45 is known to be suboptimal for use in environments with limited
data transparency (such as URLs), but is included because of its close
relationship to QR codes and its wide use in health informatics (note
that base45 is strongly specified not to allow sloppy forms
of encoding).

Numbers
-------

| name       | meaning         | reference |
| `.decimal` | Decimal Integer | ---       |
{: title="Control Operator for Text Conversion of Integers"}

The control operator `.decimal` allows the modeling of text strings that carry numeric
information in decimal form, such as in the uint64/int64 formats of
YANG-JSON {{?RFC7951}}.

~~~ cddl
yang-json-sid = text .decimal (0..9223372036854775807)
~~~
{: sourcecode-name="example2.cddl"}

Again, the specification is opinionated by only providing integer numbers
without leading zeros, i.e., the decimal numbers match the regular
expression `0|-?[1-9][0-9]*` (of course, further restricted by the
control type).
See the next section for more flexibility, and for octal, hexadecimal,
or binary conversions.

Printf-style Formatting
-------

| name      | meaning                           | reference |
| `.printf` | Printf-formatting of data item(s) | ---       |
{: title="Control Operator for Printf-formatting of data item(s)"}

The control operator `.printf` allows the modeling of text strings that carry various formatted
information, as long as the format can be represented in Printf-style
formatting strings as they are used in the C language (see Section
7.21.6.1 of [C]).

The controller (right-hand side) of the `.printf` control is an array
of one Printf-style format string and zero or more data items that fit
the individual conversion specifications in the format string.
The construct matches a text string representing the textual output of
an equivalent C-language `printf` function call that is given the
format string and the data items following it in the array.

From the printf specification in the C language, length modifiers (paragraph 7)
are not used and MUST NOT be included in the format string.
The 's' conversion specifier (paragraph 8) is used to
interpolate a text string in UTF-8 form.
The 'c' conversion specifier (paragraph 8) represents a single Unicode
scalar value as a UTF-8 character.
The 'p' and 'n' conversion specifiers (paragraph 8) are not used and
MUST NOT be included in the format string.

In the following example, `my_alg_19` matches the text string `"0x0013"`:

~~~ cddl
my_alg_19 = hexlabel<19>
hexlabel<K> = text .printf (["0x%04x", K])
~~~
{: sourcecode-name="example-printf.cddl"}

The data items in the controller array do not need to be literals,
as for example in:

~~~ cddl
any_alg = hexlabel<1..20>
hexlabel<K> = text .printf (["0x%04x", K])
~~~
{: sourcecode-name="example-printf-uint.cddl"}

Here, `any_alg` matches the text strings `"0x0013"` or `"0x0001"` but
not `"0x1234"`.

JSON Values
-----------

Some applications store complete JSON texts into text strings, the
JSON value for which can easily be defined in CDDL.
This is supported by a control operator similar to `.cbor` in {{Section
3.8.4 of -cddl}}.

| name    | meaning | reference |
| `.json` | JSON    | {{STD90}}   |
{: title="Control Operator for Text Conversion of JSON values"}

~~~ cddl
embedded-claims = text .json claims
claims = {iss: text, exp: text}
~~~
{: sourcecode-name="example3.cddl"}

Note that a `.jsonseq` is not provided for {{-jsonseq}}, as no use case
for inclusion in CDDL is known yet.

There is no way to constrain the use of blank space in data items to
be validated; variants (e.g, not providing for any blank space) could
be defined.

Text Processing
===============

Join
----

Often, text strings need to be constructed out of parts that can best
be modeled as an array.

| name    | meaning                          | reference |
| `.join` | concatenate elements of an array | ---       |
{: title="Control Operator for Text Generation from Arrays"}

For example, an IPv4 address in dotted-decimal might be modeled as in
{{fig-join-example}}.

~~~ cddl
legacy-ip-address = text .join legacy-ip-address-elements
legacy-ip-address-elements = [bytetext, ".", bytetext, ".",
                              bytetext, ".", bytetext]
bytetext = text .decimal byte
byte = 0..255
~~~
{: #fig-join-example sourcecode-name="join-example.cddl"
   title="Using the .join operator to build dotted-decimal IPv4 addresses"}

The elements of the controller array need to be strings (text or byte
strings).
The control operator matches a data item if that data item is also a
string, built by concatenating the strings in the array.
The result of this concatenation is of the same kind of string (text
or bytes) as the first element of the array.
(If there is no element in the array, the `.join` construct matches
either kind of empty string, obviously further constrained by the
control operator target.)
The concatenation is performed on the sequences of bytes in the
strings.
If the result of the concatenation is a text string, the resulting
sequence of bytes MUST be valid UTF-8.

Note that this control operator is hard to validate in the most
general case, as this would require full parser functionality.
Simple implementation strategies will use array elements with constant
values as guideposts ("markers", such as the `"."` in {{fig-join-example}})
for isolating the variable elements that need further validation at
the CDDL data model level.
It is therefore recommended to limit the use of `.join` to simple
arrangements where the array elements are laid out explicitly and
there are no adjacent variable elements without intervening constant
values, and where these constant values do not occur within the
variable elements.\\
If more complex parsing functionality is required, the ABNF control
operators (see {{Section 3 of -control1}}) may be useful; however, these
cannot reach back into CDDL-specified elements like `.join` can do.

{:aside}
>
Implementation note: A validator implementation can use the marker
elements to scan the text, isolating the variable elements.
It also can build a parsing regexp ({{Section 6 of -iregexp}}) from the
elements of the controller array, with capture groups for each
element, and validate the captures against the elements of the array.
In the most general case, these implementation strategies can exhibit
false negatives, where the implementation cannot find the structure
that would be successfully validated using the controller; it is
RECOMMENDED that implementations provide full coverage at least for
the marker-based subset outlined in the previous paragraph.

IANA Considerations
==================


[^to-be-removed]

[^to-be-removed]: RFC Editor: please replace RFC-XXXX with the RFC
    number of this RFC and remove this note.


This document requests IANA to register the contents of
{{tbl-iana-reqs}} into the registry
"{{cddl-control-operators (CDDL Control Operators)<IANA.cddl}}" of {{IANA.cddl}}:

| Name           | Reference |
| `.b64u`        | \[RFC-XXXX]  |
| `.b64u-sloppy` | \[RFC-XXXX] |
| `.b64c`        | \[RFC-XXXX] |
| `.b64c-sloppy` | \[RFC-XXXX] |
| `.b45`         | \[RFC-XXXX] |
| `.b32`         | \[RFC-XXXX] |
| `.h32`         | \[RFC-XXXX] |
| `.hex`         | \[RFC-XXXX] |
| `.hexlc`       | \[RFC-XXXX] |
| `.hexuc`       | \[RFC-XXXX] |
| `.decimal`     | \[RFC-XXXX] |
| `.printf`      | \[RFC-XXXX] |
| `.json`        | \[RFC-XXXX] |
| `.join`        | \[RFC-XXXX] |
{: #tbl-iana-reqs title="New control operators to be registered"}

Implementation Status
=====================
{:removeinrfc}

<!-- RFC7942 -->

In the CDDL tool described in {{Section F of RFC8610}},
the control operators defined in the present revision of this
specification are implemented as of version 0.10.4.

Security considerations
=======================

The security considerations of {{-cddl}} apply.

--- back

Acknowledgements
================
{:unnumbered}

Henk Birkholz suggested the need for many of the control operators
defined here.

<!--  LocalWords:  dedenting dedented
 -->
