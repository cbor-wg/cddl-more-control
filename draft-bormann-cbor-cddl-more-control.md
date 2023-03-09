---
v: 3

title: >
  More Control Operators for CDDL
abbrev: CDDL control operators
docname: draft-bormann-cbor-cddl-more-control-latest
# date: 2023-03-09
cat: std
consensus: true
stream: IETF

keyword:
 - Concise Data Definition Language
 - Control Operator
venue:
  group: "Concise Binary Object Representation Maintenance and Extensions"
  mail: "cbor@ietf.org"
  github: "cbor-wg/cddl-more-control"
  latest: "https://cbor-wg.github.io/cddl-more-control/draft-bormann-cbor-cddl-more-control.html"

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
    =: RFC8259
  STD94:
    -: cbor
    =: RFC8949
  RFC8610: cddl
  IANA.cddl:
  RFC9165: control1
  RFC4648: base
informative:
  INTERSPERSE:
    target: https://hoogle.haskell.org/?hoogle=intersperse
    title: intersperse
    rc: Haskell Documentation
    date: false

--- abstract

The Concise Data Definition Language (CDDL), standardized in RFC 8610,
provides "control operators" as its main language extension point.
RFCs have added to this extension point both in an
application-specific and a more general way.

The present document defines a number of additional generally
application control operators for text conversion (Bytes, Integers,
JSON) and operations on text.


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

| Name                       | Purpose                                |
| `.b64u`, `.b64c`           | Base64 representation of byte strings  |
| `.hex`, `.hexlc`, `.hexuc` | Base16 representation of byte strings  |
| `.b32`, `.h32`             | Base32 representation of byte strings  |
| `.decimal`                 | Text representation of integer numbers |
| `.json`                    | Text representation of JSON values     |
| `.join`                    | Building text from array of components |
{: #tbl-new title="New control operators in this document"}

Terminology
-----------

{::boilerplate bcp14-tagged}

This specification uses terminology from {{-cddl}}.
In particular, with respect to control operators, "target" refers to
the left-hand side operand, and "controller" to the right-hand side operand.
"Tool" refers to tools along the lines of that described in {{Appendix F of -cddl}}.
Note also that the data model underlying CDDL provides for text
strings as well as byte strings as two separate types, which are
then collectively referred to as "strings".

Text Conversion
===============

Byte Strings: Base16 (Hex), Base32, Base64
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
-cbor}}, we use representations defined in {{-base}} with the following
names:

| name     | meaning                         | reference            |
| `.b64u`  | Base64URL, no padding           | {{Section 5 of -base}} |
| `.b64c`  | Base64 classic, padding         | {{Section 4 of -base}} |
| `.b32`   | Base32, no padding              | {{Section 6 of -base}} |
| `.h32`   | Base32/hex alphabet, no padding | {{Section 7 of -base}} |
| `.hex`   | Base16 (hex), either case       | {{Section 8 of -base}} |
| `.hexlc` | Base16 (hex), lower case        | {{Section 8 of -base}} |
| `.hexuc` | Base16 (hex), upper case        | {{Section 8 of -base}} |
{: title="Control Operators for Text Conversion of byte strings"}

Note that this specification is somewhat opinionated here: It does not
provide base64url, base32 or base32hex encoding with padding, or
base64 classic without padding.  Experience indicates that these
combinations only ever occur in error, so the usability of CDDL is
increased by not providing them in the first place.  Also, adding "c"
makes sure that any decision for classic base64 is actively taken.

Numbers
-------

| name       | meaning         | reference |
| `.decimal` | Decimal Integer | ---       |
{: title="Control Operator for Text Conversion of Integers"}

This allows the modeling of text strings that carry numeric
information, such as in the uint64/int64 formats of YANG-JSON
{{?RFC7951}}.

~~~ cddl
yang-json-sid = text .decimal (0..9223372036854775807)
~~~
{: sourcecode-name="example2.cddl"}

Again, the specification is opinionated by only providing numbers
without leading zeros, i.e., the decimal numbers match the regular
expression "0|-?[1-9][0-9]*" (of course, further restricted by the
control type).  Future specifications can provide octal, hexadecimal,
or binary conversions.

JSON Values
-----------

Some applications store complete JSON texts into text strings, the
JSON value for can easily be defined in CDDL.
This is supported by a control operator similar to `.cbor` in {{Section
3.8.4 of -cddl}}.

| name    | meaning | reference |
| `.json` | JSON    | {{STD90}}   |
{: title="Control Operator for Text Conversion of JSON values"}

~~~ cddl
embedded-claims = text .json claims
claims = {iss: issuer, exp: expiry}
~~~
{: sourcecode-name="example3.cddl"}

Note that a `.jsonseq` is not provided, as no use case is known yet.
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

In general, this control operator is hard to validate as it would
require full parser functionality.
It is therefore recommended to only use it in simple cases, and leave
full parsing to ABNF {{Section 3 of -control1}} or similar.

~~~ cddl
legacy-ip-address = text .join [digits<1>, ".", digits<2>,
                           ".", digits<3>, ".", digits<4>]
digits<N> = text .decimal byte<n>
~~~
{: sourcecode-name="example4.cddl"}

Intersperse
-----------

(This section requires some additional design effort before it can be
implemented and registered.)

The previous example is somewhat unwieldy as the dots are just
syntactic noise.

The `.intersperse` control is modeled after Haskell's function of the
same name {{INTERSPERSE}}.  It takes the elements of an array and
intersperses a given additional elements between each to elements of
the additional array.

~~~ cddl
legacy-ip-address = text .join ("." .intersperse digits)
digits = [4*4 digit]
digit = text .decimal (uint .size 1)
~~~
{: sourcecode-name="example5.cddl"}

`.intersperse` is actually not limited to text strings; it can be used
for any juggling of arrays.

We want to stop here, as there is a danger of generating a functional
programming language in an ad-hoc fashion.


IANA Considerations
==================

This document requests IANA to register the contents of
{{tbl-iana-reqs}} into the registry
"{{cddl-control-operators (CDDL Control Operators)<IANA.cddl}}" of {{IANA.cddl}}:

| Name     | Reference |
| `.b64u`  | [RFCthis] |
| `.b64c`  | [RFCthis] |
| `.b32`   | [RFCthis] |
| `.h32`   | [RFCthis] |
| `.hex`   | [RFCthis] |
| `.hexlc` | [RFCthis] |
| `.hexuc` | [RFCthis] |
| `.decimal` | [RFCthis] |
| `.json`  | [RFCthis] |
| `.join`  | [RFCthis] |
{: #tbl-iana-reqs title="New control operators to be registered"}

Implementation Status
=====================
{: removeinrfc="true"}

<!-- RFC7942 -->

In the CDDL tool described in {{Section F of RFC8610}},
the control operators defined in this specification are
implemented as of version 0.10.2.

Security considerations
=======================

The security considerations of {{-cddl}} apply.

--- back

Acknowledgements
================
{: numbered="no"}

Henk Birkholz suggested the need for many of the control operators
defined here.

<!--  LocalWords:  dedenting dedented
 -->
