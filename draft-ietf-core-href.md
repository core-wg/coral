---
stand_alone: true
ipr: trust200902
docname: draft-ietf-core-href-latest
cat: std
submissiontype: IETF
consensus: true
lang: en
pi:
  toc: 'true'
  tocdepth: '3'
  symrefs: 'true'
  sortrefs: 'true'
title: Constrained Resource Identifiers
wg: CoRE Working Group

author:
- ins: K. Hartke
  name: Klaus Hartke
  org: Ericsson
  street: Torshamnsgatan 23
  city: Stockholm
  code: '16483'
  country: Sweden
  email: klaus.hartke@ericsson.com
informative:
  RFC7228:
  RFC7230:
  RFC7252:
  RFC8141:
  RFC8288:
  W3C.REC-html52-20171214:
normative:
  RFC3986:
  RFC3987:
  RFC8610:
  Unicode:
    target: https://www.unicode.org/versions/Unicode13.0.0/
    title: The Unicode Standard, Version 13.0.0
    author:
    - org: The Unicode Consortium
    date: 2020-03
    seriesinfo:
      ISBN: 978-1-936213-26-9
  RFC8949: cbor

--- abstract


The Constrained Resource Identifier (CRI) is a complement to the Uniform
Resource Identifier (URI) that serializes the URI components in Concise
Binary Object Representation (CBOR) instead of a sequence of characters.
This simplifies parsing, comparison and reference resolution in
environments with severe limitations on processing power, code size, and
memory size.

--- to_be_removed_note_Note_to_Readers

The issues list for this Internet-Draft can be found at
\<https://github.com/core-wg/coral/labels/href>.

A reference implementation and a set of test vectors can be found at
\<https://github.com/core-wg/coral/tree/master/binary/python>.

--- middle

# Introduction

The [Uniform Resource Identifier (URI)](#RFC3986) and its most common
usage, the URI reference, are the Internet standard for linking to
resources in hypertext formats such as [HTML](#W3C.REC-html52-20171214)
or the [HTTP "Link" header field](#RFC8288).

A URI reference is a sequence of characters chosen from the repertoire
of US-ASCII characters.
The individual components of a URI reference are delimited by a number
of reserved characters, which necessitates the use of a character escape
mechanism called "percent-encoding" when these reserved characters are
used in a non-delimiting function.
The resolution of URI references involves parsing a character sequence
into its components, combining those components with the components of a
base URI, merging path components, removing dot-segments, and
recomposing the result back into a character sequence.

Overall, the proper handling of URI references is quite intricate.
This can be a problem especially in [constrained environments](#RFC7228),
where nodes often have severe code size and memory size limitations.
As a result, many implementations in such environments support only an
ad-hoc, informally-specified, bug-ridden, non-interoperable subset of
half of RFC 3986.

This document defines the Constrained Resource Identifier (CRI) by
constraining URIs to a simplified subset and serializing their
components in [Concise Binary Object Representation (CBOR)](#RFC8949)
instead of a sequence of characters.
This allows typical operations on URI references such as parsing,
comparison and reference resolution (including all corner cases) to be
implemented in a comparatively small amount of code.

As a result of simplification, however, CRIs are not capable of
expressing all URIs permitted by the generic syntax of RFC 3986 (hence
the "constrained" in "Constrained Resource Identifier").
The supported subset includes all URIs of the
[Constrained Application Protocol (CoAP)](#RFC7252), most URIs of the
[Hypertext Transfer Protocol (HTTP)](#RFC7230),
[Uniform Resource Names (URNs)](#RFC8141), and other similar URIs.
The exact constraints are defined in {{constraints}}.

## Notational Conventions

{::boilerplate bcp14}

Terms defined in this document appear in *cursive* where they
are introduced (rendered in plain text as the new term surrounded by
underscores).



# Constraints {#constraints}

A Constrained Resource Identifier consists of the same five components
as a URI: scheme, authority, path, query, and fragment.
The components are subject to the following constraints:

{: type="C%d."}
1. {:#c-scheme} The scheme name can be any Unicode string (see
   Definition D80 in {{Unicode}}) that matches the syntax of a URI
   scheme (see {{Section 3.1 of RFC3986}}) and is lowercase (see
   Definition D139 in {{Unicode}}).

1. {:#c-authority} An authority is always a host identified by an IP
   address or registered name, along with optional port information.
   User information is not supported.

1. {:#c-ip-address} An IP address can be either an IPv4 address or an IPv6 address.
   IPv6 scoped addressing zone identifiers and future versions of IP are
   not supported.

1. {:#c-reg-name} A registered name can be any Unicode string that is lowercase and in
   Unicode Normalization Form C (NFC) (see Definition D120 in {{Unicode}}).
   (The syntax may be further restricted by the scheme.)

1. {:#c-port-range} A port is always an integer in the range from 0 to 65535.
   Empty ports or ports outside this range are not supported.

1. {:#c-port-omitted} The port is omitted if and only if the port would be the same as the
   scheme's default port (provided the scheme is defining such a default
   port) or the scheme is not using ports.

1. {:#c-path} A path consists of zero or more path segments.
   A path must not consist of a single zero-length path segment, which
   is considered equivalent to a path of zero path segments.

1. {:#c-path-segment} A path segment can be any Unicode string that is
   in NFC, with the exception of the special "." and ".." complete path
   segments.
   It can be the zero-length string. No special constraints are placed
   on the first path segment.

1. {:#c-query} A query always consists of one or more query parameters.
   A query parameter can be any Unicode string that is in NFC.
   It is often in the form of a "key=value" pair.
   When converting a CRI to a URI, query parameters are separated by an
   ampersand ("&") character.
   (This matches the structure and encoding of the target URI in CoAP
   requests.)

1. {:#c-fragment} A fragment identifier can be any Unicode string that is in NFC.

1. {:#c-escaping} The syntax of registered names, path segments, query
   parameters, and fragment identifiers may be further restricted and
   sub-structured by the scheme.
   There is no support, however, for escaping sub-delimiters
   that are not intended to be used in a delimiting function.

1. {:#c-mapping} When converting a CRI to a URI, any character that is
   outside the allowed character range or a delimiter in the URI syntax
   is percent-encoded.
   For CRIs, percent-encoding always uses the UTF-8 encoding form (see
   Definition D92 in {{Unicode}}) to convert the character to a sequence
   of octets (that is then converted to a sequence of %HH triplets).


# Creation and Normalization

In general, resource identifiers are created on the initial creation of a
resource with a certain resource identifier, or the initial exposition
of a resource under a particular resource identifier.

A Constrained Resource Identifier <bcp14>SHOULD</bcp14> be created by
the naming authority that governs the namespace of the resource
identifier. For example, for the resources of an HTTP origin server,
that server is responsible for creating the CRIs for those resources.

The naming authority <bcp14>MUST</bcp14> ensure that any CRI created
satisfies the constraints defined in {{constraints}}. The creation of a
CRI fails if the CRI cannot be validated to satisfy all of the
constraints.

{:ctr: format="counter"}

If a naming authority creates a CRI from user input, it <bcp14>MAY</bcp14> apply
the following (and only the following) normalizations to get the CRI
more likely to validate:
map the scheme name to lowercase ({{c-scheme}}{:ctr});
map the registered name to NFC ({{c-reg-name}}{:ctr});
elide the port if it's the default port for the scheme
({{c-port-omitted}}{:ctr});
elide a single zero-length path segment ({{c-path}}{:ctr});
map path segments, query parameters and the fragment identifier to NFC
({{c-path-segment}}{:ctr}, {{c-query}}{:ctr}, {{c-fragment}}{:ctr}).

Once a CRI has been created, it can be used and transferred without
further normalization.
All operations that operate on a CRI <bcp14>SHOULD</bcp14> rely on the
assumption that the CRI is appropriately pre-normalized.
(This does not contradict the requirement that when CRIs are
transferred, recipients must operate on as-good-as untrusted input and
fail gracefully in the face of malicious inputs.)


# Comparison

One of the most common operations on CRIs is comparison: determining
whether two CRIs are equivalent, without using the CRIs to access their
respective resource(s).

Determination of equivalence or difference of CRIs is based on simple
component-wise comparison. If two CRIs are identical
component-by-component (using code-point-by-code-point comparison for
components that are Unicode strings) then it is safe to conclude that
they are equivalent.

This comparison mechanism is designed to minimize false negatives while
strictly avoiding false positives.
The constraints defined in {{constraints}} imply the most
common forms of syntax- and scheme-based normalizations in URIs, but do
not comprise protocol-based normalizations that require accessing the
resources or detailed knowledge of the scheme's dereference algorithm.
False negatives can be caused, for example, by CRIs that are not
appropriately pre-normalized and by resource aliases.

When CRIs are compared to select (or avoid) a network action, such as
retrieval of a representation, fragment components (if any) should be
excluded from the comparison.


# CRI References

The most common usage of a Constrained Resource Identifier is to embed
it in resource representations, e.g., to express a hyperlink between the
represented resource and the resource identified by the CRI.

This section defines the serialization of CRIs in
[Concise Binary Object Representation (CBOR)](#RFC8949).
To reduce representation size, CRIs are not serialized directly.
Instead, CRIs are indirectly referenced through *CRI references*.
These take advantage of hierarchical locality and provide a very compact
encoding.
The CBOR serialization of CRI references is specified in
{{cbor-serialization}}.

The only operation defined on a CRI reference is *reference resolution*:
the act of transforming a CRI reference into a CRI.
An application <bcp14>MUST</bcp14> implement this operation by applying
the algorithm specified in {{reference-resolution}} (or any algorithm
that is functionally equivalent to it).

The method of transforming a CRI into a CRI reference is unspecified;
implementations are free to use any algorithm as long as reference
resolution of the resulting CRI reference yields the original CRI.
Notably, a CRI reference is not required to satisfy all of the
constraints of a CRI; the only requirement on a CRI reference is that
reference resolution <bcp14>MUST</bcp14> yield the original CRI.

When testing for equivalence or difference, applications <bcp14>SHOULD
NOT</bcp14> directly compare CRI references; the references should be
resolved to their respective CRI before comparison.

## CBOR Serialization {#cbor-serialization}

A CRI reference is encoded as a CBOR array {{RFC8949}} that contains a
sequence of zero or more options. Each option consists of an option
number followed by an option value, holding one component or
sub-component of the CRI reference. To reduce size, both option
numbers and option values are immediate elements of the CBOR array and
appear in alternating order.

Not all possible sequences of options denote a well-formed CRI
reference. The structure can be described in the [Concise Data
Definition Language (CDDL)](#RFC8610) as follows:

{: empty="true"}
* ??

  ~~~~ cddl
  CRI-Reference = [
    (?scheme, ?((host.name // host.ip), ?port) // path.type),
    *path,
    *query,
    ?fragment
  ]

  scheme    = (0, text .regexp "[a-z][a-z0-9+.-]*")
  host.name = (1, text)
  host.ip   = (2, bytes .size 4 / bytes .size 16)
  port      = (3, 0..65535)
  path.type = (4, 0..127)
  path      = (5, text)
  query     = (6, text)
  fragment  = (7, text)
  ~~~~


The options correspond to the (sub-)components of a CRI, as described in
{{constraints}}, with the addition of the `path.type` option.
The `path.type` option can be used to express path prefixes like "/",
"./", "../", "../../", etc.
The exact semantics of the option values are defined by
{{reference-resolution}}.
A sequence of options that is empty or starts with a `path` option is
equivalent the same sequence prefixed by a `path.type` option with value
2.

Examples:
: ??

: ~~~~ cbor
  [0, "coap",
   2, h'C6336401',
   3, 61616,
   5, ".well-known",
   5, "core"]
  ~~~~

: ~~~~ cbor
  [4, 0,
   5, ".well-known",
   5, "core",
   6, "rt=temperature-c"]
  ~~~~


A CRI reference is considered *well-formed* if the sequence
of options is in the correct order as defined by the CDDL structure.

A CRI reference is considered *absolute* if it is well-formed
and the sequence of options starts with a `scheme` option.

A CRI reference is considered *relative* if it is well-formed
and the sequence of options is empty or starts with an option other
than a `scheme` option.


## Reference Resolution {#reference-resolution}

The term "relative" implies that a "base CRI" exists against which the
relative reference is applied. Aside from fragment-only references,
relative references are only usable when a base CRI is known.

The following steps define the process of resolving any well-formed CRI
reference against a base CRI so that the result is a CRI in the form of
an absolute CRI reference:

1. Establish the base CRI of the CRI reference and express it in the
  form of an absolute CRI reference.
  (The base CRI can be established in a number of ways; see
  {{Section 5.1 of RFC3986}}.)

1. Determine the values of two variables, T and E, depending on the
  first option of the CRI reference to be resolved, according to {{resolution-variables}}.

1. Initialize a buffer with all the options from the base CRI where the
  option number is less than the value of E.

1. If the value of T is greater than 0, remove the last T-many `path`
   options from the end of the buffer (up to the number of `path`
   options in the buffer).

1. Append all the options from the CRI reference to the buffer, except
  for any `path.type` option.

1. If the number of `path` options in the buffer is one and the
  value of that option is the zero-length string, remove the option
  from the buffer.

1. Return the sequence of options in the buffer as the resolved CRI.

<table anchor="resolution-variables" align="center">
            <name>Values of the Variables T and E</name>
            <thead>
              <tr>
                <th align="left">First Option Number</th>
                <th align="left">T</th>
                <th align="left">E</th>
              </tr>
            </thead>
            <tbody>
              <tr>
                <td align="left">0 (scheme)</td>
                <td align="left">0</td>
                <td align="left">0</td>
              </tr>
              <tr>
                <td align="left">1 (host.name)</td>
                <td align="left">0</td>
                <td align="left">1</td>
              </tr>
              <tr>
                <td align="left">2 (host.ip)</td>
                <td align="left">0</td>
                <td align="left">1</td>
              </tr>
              <tr>
                <td align="left">3 (port)</td>
                <td align="left" colspan="2">(invalid sequence of options)</td>
              </tr>
              <tr>
                <td align="left">4 (path.type)</td>
                <td align="left">option value - 1</td>
                <td align="left">if T &lt; 0 then 5 else 6</td>
              </tr>
              <tr>
                <td align="left">5 (path)</td>
                <td align="left">1</td>
                <td align="left">6</td>
              </tr>
              <tr>
                <td align="left">6 (query)</td>
                <td align="left">0</td>
                <td align="left">6</td>
              </tr>
              <tr>
                <td align="left">7 (fragment)</td>
                <td align="left">0</td>
                <td align="left">7</td>
              </tr>
              <tr>
                <td align="left">none/empty sequence</td>
                <td align="left">0</td>
                <td align="left">7</td>
              </tr>
            </tbody>
          </table>



# Relationship between CRIs, URIs and IRIs

CRIs are meant to replace both [Uniform Resource Identifiers (URIs)](#RFC3986)
and [Internationalized Resource Identifiers (IRIs)](#RFC3987)
in [constrained environments](#RFC7228).
Applications in these environments may never need to use URIs and IRIs
directly, especially when the resource identifier is used simply for
identification purposes or when the CRI can be directly converted into a
CoAP request.

However, it may be necessary in other environments to determine the
associated URI or IRI of a CRI, and vice versa. Applications can perform
these conversions as follows:

{: vspace='0'}
CRI to URI
: A CRI is converted to a URI as specified in {{cri-to-uri}}.

URI to CRI
: The method of converting a URI to a CRI is unspecified;
  implementations are free to use any algorithm as long as converting
  the resulting CRI back to a URI yields an equivalent URI.

CRI to IRI
: A CRI can be converted to an IRI by first converting it to a URI as
  specified in {{cri-to-uri}}, and then converting the URI
  to an IRI as described in {{RFC3987}}{: section="3.2"}.

IRI to CRI
: An IRI can be converted to a CRI by first converting it to a URI as
  described in {{RFC3987}}{: section="3.1"}, and then
  converting the URI to a CRI as described above.

Everything in this section also applies to CRI references, URI
references and IRI references.

## Converting CRIs to URIs {#cri-to-uri}

Applications <bcp14>MUST</bcp14> convert a CRI reference to a URI
reference by determining the components of the URI reference according
to the following steps and then recomposing the components to a URI
reference string as specified in {{RFC3986}}{: section="5.3"}.

{: vspace='0'}

scheme
: If the CRI reference contains a `scheme` option, the scheme
  component of the URI reference consists of the value of that
  option.
    Otherwise, the scheme component is undefined.

authority
: If the CRI reference contains a `host.name` or `host.ip` option, the
  authority component of the URI reference consists of a host
  subcomponent, optionally followed by a colon (":") character and a
  port subcomponent.  Otherwise, the authority component is undefined.

  The host subcomponent consists of the value of the `host.name` or
  `host.ip` option.

  Any character in the value of a `host.name` option that is not in
  the set of unreserved characters ({{Section 2.3 of RFC3986}}) or
  "sub-delims" ({{Section 2.2 of RFC3986}}) <bcp14>MUST</bcp14> be
  percent-encoded.

  The value of a `host.ip` option <bcp14>MUST</bcp14> be
  represented as a string that matches the "IPv4address" or
  "IP-literal" rule ({{Section 3.2.2 of RFC3986}}).

  If the CRI reference contains a `port` option, the port
  subcomponent consists of the value of that option in decimal
  notation.
  Otherwise, the colon (":") character and the port subcomponent are
  both omitted.

path
: If the CRI reference is an empty sequence of options or starts with
  a `port` option, a `path` option, or a `path.type` option where the
  value is not 0, the conversion fails.

  If the CRI reference contains a `host.name` option, a `host.ip`
  option or a `path.type` option where the value is not 0, the path
  component of the URI reference is prefixed by a slash ("/")
  character.  Otherwise, the path component is prefixed by the empty
  string.

  If the CRI reference contains one or more `path` options,
  the prefix is followed by the value of each option, separated by a
  slash ("/") character.

  Any character in the value of a `path` option that is not
  in the set of unreserved characters or "sub-delims" or a colon
  (":") or commercial at ("@") character <bcp14>MUST</bcp14> be
  percent-encoded.

  If the authority component is defined and the path component does not
  match the "path-abempty" rule ({{Section 3.3 of RFC3986}}), the
  conversion fails.

  If the authority component is undefined and the scheme component is
  defined and the path component does not match the "path-absolute",
  "path-rootless" or "path-empty" rule ({{Section 3.3 of RFC3986}}), the
  conversion fails.

  If the authority component is undefined and the scheme component is
  undefined and the path component does not match the "path-absolute",
  "path-noscheme" or "path-empty" rule ({{Section 3.3 of RFC3986}}), the
  conversion fails.

query
: If the CRI reference contains one or more `query` options,
  the query component of the URI reference consists of the value of
  each option, separated by an ampersand ("&") character.
  Otherwise, the query component is undefined.

  Any character in the value of a `query` option that is not
  in the set of unreserved characters or "sub-delims" or a colon
  (":"), commercial at ("@"), slash ("/") or question mark ("?")
  character <bcp14>MUST</bcp14> be percent-encoded.
  Additionally, any ampersand character ("&") in the option
  value <bcp14>MUST</bcp14> be percent-encoded.

fragment
: If the CRI reference contains a fragment option, the fragment
  component of the URI reference consists of the value of that
  option.
  Otherwise, the fragment component is undefined.

  Any character in the value of a `fragment` option that is
  not in the set of unreserved characters or "sub-delims" or a colon
  (":"), commercial at ("@"), slash ("/") or question mark ("?")
  character <bcp14>MUST</bcp14> be percent-encoded.



# Security Considerations {#security}

Parsers of CRI references must operate on input that is assumed to be
untrusted. This means that parsers <bcp14>MUST</bcp14> fail gracefully
in the face of malicious inputs.
Additionally, parsers <bcp14>MUST</bcp14> be prepared to deal with
resource exhaustion (e.g., resulting from the allocation of big data
items) or exhaustion of the call stack (stack overflow).
See {{Section 10 of RFC8949}} for additional
security considerations relating to CBOR.

The security considerations discussed in {{Section 7 of RFC3986}} and
{{Section 8 of RFC3987}} for URIs and IRIs also apply to CRIs.


# IANA Considerations

This document has no IANA actions.


--- back

# Change Log
{: removeInRFC="true"}

Changes from -03 to -04:

* Minor editorial improvements.

Changes from -02 to -03:

* Expanded the set of supported schemes (#3).

* Specified creation, normalization and comparison (#9).

* Clarified the default value of the `path.type` option (#33).

* Removed the `append-relation` path type (#41).

* Renumbered the remaining path types.

* Renumbered the option numbers.

* Restructured the document.

* Minor editorial improvements.

Changes from -01 to -02:

* Changed the syntax of schemes to exclude upper case characters (#13).

* Minor editorial improvements (#34 #37).

Changes from -00 to -01:

* None.


# Acknowledgements
{: numbered="false"}

Thanks to
{{{Christian Ams??ss}}},
{{{Carsten Bormann}}},
{{{Ari Ker??nen}}},
{{{Jim Schaad}}} and
{{{Dave Thaler}}}
for helpful comments and discussions that have shaped the
document.

