---
v: 3

title: >
  Concise Data Definition Language (CDDL):
  Additional Control Operators for the Conversion and Processing of Text
abbrev: >
  CDDL: More Control Operators for Text
docname: draft-ietf-cbor-cddl-more-control-latest
# date: 2025-01-09
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
  - name: Carsten Bormann
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
  RFC7493: i-json


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

| Name                           | t             | c     | Purpose                                            |
| `.b64u`, `.b64c`               | text          | bytes | Base64 representation of byte strings              |
| `.b64u-sloppy`, `.b64c-sloppy` | text          | bytes | (sloppy-tolerant variants of the above)            |
| `.hex`, `.hexlc`, `.hexuc`     | text          | bytes | Base16 representation of byte strings              |
| `.b32`, `.h32`                 | text          | bytes | Base32 representation of byte strings              |
| `.b45`                         | text          | bytes | Base45 representation of byte strings              |
| `.base10`                      | text          | int   | Text representation of integer numbers             |
| `.printf`                      | text          | array | Printf-formatted text representation of data items |
| `.json`                        | text          | any   | Text representation of JSON values                 |
| `.join`                        | text or bytes | array | Build text or byte string from array of components |
{: #tbl-new title="Summary of New Control Operators in this Document, 
t = target type (left-hand side), c = controller type (right-hand side)"}

Terminology
-----------

{::boilerplate bcp14-tagged-bcp14}

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

Byte Strings: Base 16 (Hex), Base 32, Base 45, Base 64 {#base}
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
RFC8949@-cbor}}, this specification uses representations defined in {{-base}} with the following
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
{: #tbl-text-conv title="Control Operators for Text Conversion of Byte Strings"}

Note that this specification is somewhat opinionated here: It does not
provide base64url, base32 or base32hex encoding with padding, or
base64 classic without padding.  Experience indicates that these
combinations only ever occur in error, so the usability of CDDL is
increased by not providing them in the first place.  Also, adding "c"
makes sure that any decision for classic base64 is actively taken.

These control operators are "strict" in their matching, i.e., they
only match base encodings that conform to the mandates of their
defining documents.
Note that this also means that `.b64u` and `.b64c` only match text
strings composed of the set of characters defined for each of them,
respectively.
(This is maybe worth pointing out here explicitly as this contrasts
with the "b64" literal prefix that can be used to notate byte strings
in CDDL source code, which simply accepts characters from either alphabet.
This behavior is different from the matching behavior of the four
base64 control operators defined here.)

The additional designation "sloppy" indicates that the text string is
not validated for any additional bits being zero, in variance to what
is specified in the paragraph behind table 1 in {{Section 4 of -base}}.
Note that the present specification is opinionated again in not
specifying a sloppy variant of base32 or base32/hex, as no legacy use
of sloppy base32(/hex) was known at the time of writing.
Base45 {{-base45}} is known to be suboptimal for use in environments with limited
data transparency (such as URLs), but is included because of its close
relationship to QR codes and its wide use in health informatics (note
that base45 is strongly specified not to allow sloppy forms
of encoding).

Numerals
-------

| name      | meaning                    | reference |
| `.base10` | Base-ten (decimal) Integer | ---       |
{: #tbl-num title="Control Operator for Text Conversion of Integers"}

The control operator `.base10` allows the modeling of text strings
that carry an integer number in decimal form (as a text string with
digits in the usual base-ten positional numeral system), such as in the uint64/int64 formats of
YANG-JSON {{?RFC7951}}.

~~~ cddl
yang-json-sid = text .base10 (0..9223372036854775807)
~~~
{: sourcecode-name="example-base10.cddl"}

Again, the specification is opinionated by only providing for integer numbers
and these only represented without leading zeros, i.e., the decimal integer
numerals match the regular
expression `0|-?[1-9][0-9]*` (of course, further restricted by the
control type).
See the next section for more flexibility, and for other numeric bases
such as octal, hexadecimal,
or binary conversions.

Note that this control operator governs text representations of
integers and should not be confused with the control operators
governing text representations of byte strings (`b64u` etc.).
This contrast is somewhat reinforced by spelling out "base" in the
name `base10` as opposed to those of the byte string operators.

Printf-style Formatting
-------

| name      | meaning                           | reference |
| `.printf` | Printf-formatting of data item(s) | ---       |
{: #tbl-printf title="Control Operator for Printf-formatting of Data Item(s)"}

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

Some applications store complete JSON texts {{-json}} into text strings, the
JSON value for which can easily be defined in CDDL by using the default
JSON-to-CBOR conversion rules provided by {{Section 6.2 of RFC8949@-cbor}}.
This is supported by a control operator similar to `.cbor` as defined in {{Section
3.8.4 of -cddl}}.

| name    | meaning | reference |
| `.json` | JSON    | {{STD90}}   |
{: #tbl-json title="Control Operator for Text Conversion of JSON Values"}

~~~ cddl
embedded-claims = text .json claims
claims = {iss: text, exp: text}
~~~
{: sourcecode-name="example3.cddl"}

Notes:

* JSON has known interoperability problems {{-i-json}}.
  While {{Section 4 of -i-json}} probably is not relevant to this
  specification, {{Section 2 of -i-json}} provides requirements that
  need to be followed to make use of the generic data model underlying
  CDDL.
  Note that the intention of {{Section 2.2 of -i-json}} is directly
  supported by {{Section 6.2 of RFC8949@-cbor}}.
  The recommendation to use text strings for representing numbers
  outside JSON's interoperable range is a requirement on the
  application data model and therefore needs to be reflected on the
  right-hand side of the `.json` control operator.

* This control operator provides no way to constrain the use of blank
  space or other serialization variants in the JSON representation of
  the data items; restrictions on the serialization to specific
  variants (e.g, not providing for the addition of any insignificant
  blank space, prescribing an order in which map entries are
  serialized) could be defined in future control operators.

* A `.jsonseq` is not provided in this document for {{-jsonseq}}, as no
  use case for inclusion in CDDL is known at the time of writing;
  again, future control operators could address this use case.


Text Processing
===============

Join
----

Often, text strings need to be constructed out of parts that can best
be modeled as an array.

| name    | meaning                          | reference |
| `.join` | concatenate elements of an array | ---       |
{: #tbl-join title="Control Operator for Text Generation from Arrays"}

For example, an IPv4 address in dotted-decimal might be modeled as in
{{fig-join-example}}.

~~~ cddl
legacy-ip-address = text .join legacy-ip-address-elements
legacy-ip-address-elements = [bytetext, ".", bytetext, ".",
                              bytetext, ".", bytetext]
bytetext = text .base10 byte
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
sequence of bytes only matches the target data item if that result is
a valid text string (i.e., valid UTF-8; note that in contrast to the
algorithm used in {{Section 3.2.3 of RFC8949@-cbor}} there is no need
that all individual byte sequences going into the concatenation
constitute valid text strings).

Note that this control operator is hard to validate in the most
general case, as this would require full parser functionality.
Simple implementation strategies will use array elements with constant
values as guideposts ("markers", such as the `"."` in {{fig-join-example}})
for isolating the variable elements that need further validation at
the CDDL data model level.
It is therefore recommended to limit the use of `.join` to simple
arrangements where the array elements are laid out explicitly and
there are no adjacent variable elements without intervening constant
values, and where these constant values do not occur within the text
described by the variable elements.\\
If more complex parsing functionality is required, the ABNF control
operators (see {{Section 3 of -control1}}) may be useful; however, these
cannot reach back into CDDL-specified elements like `.join` can do.

{:aside}
>
Implementation note: A validator implementation can use the marker
elements to scan the text, isolating the variable elements.
It also can build a parsing regexp ({{Section 6 of -iregexp}}; see also
{{Section 8 of -iregexp}} for security considerations related to
regexps) from the elements of the controller array, with capture
groups for each element, and validate the captures against the
elements of the array.
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

| Name           | Reference   |
| `.b64u`        | \[RFC-XXXX] |
| `.b64u-sloppy` | \[RFC-XXXX] |
| `.b64c`        | \[RFC-XXXX] |
| `.b64c-sloppy` | \[RFC-XXXX] |
| `.b45`         | \[RFC-XXXX] |
| `.b32`         | \[RFC-XXXX] |
| `.h32`         | \[RFC-XXXX] |
| `.hex`         | \[RFC-XXXX] |
| `.hexlc`       | \[RFC-XXXX] |
| `.hexuc`       | \[RFC-XXXX] |
| `.base10`      | \[RFC-XXXX] |
| `.printf`      | \[RFC-XXXX] |
| `.json`        | \[RFC-XXXX] |
| `.join`        | \[RFC-XXXX] |
{: #tbl-iana-reqs title="New Control Operators To Be Registered"}

Implementation Status
=====================
{:removeinrfc}

<!-- RFC7942 -->

In the CDDL tool described in {{Section F of RFC8610}},
the control operators defined in the present revision of this
specification are implemented as of version 0.10.4.

Security considerations
=======================

The security considerations in {{Section 5 of -cddl}} apply, as well as those
in {{Section 12 of -base}} for the control operators defined in {{base}}.

--- back

{::include-all lists.md}

Acknowledgements
================
{:unnumbered}

{{{Henk Birkholz}}} suggested the need for many of the control operators
defined here.
The author would like to thank
{{{Laurence Lundblade}}} and {{{Jeremy O'Donoghue}}} for sharpening some of
the mandates,
{{{Mikolai Gütschow}}} for improvements to some examples,
{{{A.J. Stein}}} for serving as shepherd for this document and for his
shepherd review,
and {{{Orie Steele}}} for serving as responsible AD and for providing a
detailed AD review.
