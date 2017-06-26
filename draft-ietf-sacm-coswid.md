---
title: Concise Software Identifiers
abbrev: COSWID
docname: draft-ietf-sacm-coswid-01
stand_alone: true
ipr: trust200902
area: Security
wg: SACM Working Group
kw: Internet-Draft
cat: std
pi:
  toc: yes
  sortrefs: yes
  symrefs: yes

author:
- ins: H. Birkholz
  name: Henk Birkholz
  org: Fraunhofer SIT
  abbrev: Fraunhofer SIT
  email: henk.birkholz@sit.fraunhofer.de
  street: Rheinstrasse 75
  code: '64295'
  city: Darmstadt
  country: Germany
- ins: J. Fitzgerald-McKay
  name: Jessica Fitzgerald-McKay
  org: Department of Defense
  email: jmfitz2@nsa.gov
  street: 9800 Savage Road
  city: Ft. Meade
  region: Maryland
  country: USA
- ins: C. Schmidt
  name: Charles Schmidt
  org: The MITRE Corporation
  email: cmschmidt@mitre.org
  street: 202 Burlington Road
  city: Bedford
  region: Maryland
  code: '01730'
  country: USA
- ins: D. Waltermire
  name: David Waltermire
  org: National Institute of Standards and Technology
  abbrev: NIST
  email: david.waltermire@nist.gov
  street: 100 Bureau Drive
  city: Gaithersburg
  region: Maryland
  code: '20877'
  country: USA

normative:
  RFC2119:
  RFC4108: cms-fw-pkgs
  RFC7049: cbor
  RFC4949:
  RFC7228:
  X.1520:
    title: "Recommendation ITU-T X.1520 (2014), Common vulnerabilities and exposures"
    date: 2011-04-20
  SAM:
    title: >
      Information technology - Software asset management - Part 5: Overview and vocabulary
    date: 2013-11-15
    seriesinfo:
      ISO/IEC: 19770-5:2013
  SWID:
    title: >
      Information technology - Software asset management - Part 2: Software identification tag'
    date: 2015-10-01
    seriesinfo:
      ISO/IEC: 19770-2:2015
  I-D.ietf-cose-msg: cose-msg
  I-D.ietf-ace-cbor-web-token: cwt

informative:
  I-D.greevenbosch-appsawg-cbor-cddl: cddl
  I-D.birkholz-tuda: tuda
  I-D.banghart-sacm-rolie-softwaredescriptor : sw-desc
  I-D.ietf-sacm-terminology : sacm-term

--- abstract 

This document defines a concise representation of ISO 19770-2:2015 Software Identifiers (SWID tags)
that is interoperable with the XML schema definition of ISO 19770-2:2015 and augmented for
application in Constrained-Node Networks. Next to the inherent capability of SWID tags to express
arbitrary context information, CoSWID support the definition of additional semantics via
well-defined data definitions incorporated by extension points.

--- middle

# Introduction

SWID tags have several use-applications including but not limited to:

* Software Inventory Management, a part of the Software Asset Management {{SAM}} process, which
requires an accurate list of discernible deployed software components.

* Vulnerability Assessment, which requires a semantic link between standardized vulnerability
descriptions and IT-assets {{X.1520}}.

* Remote Attestation, which requires a link between reference integrity measurements (RIM) and
security logs of measured software components {{-tuda}}.

SWID tags, as defined in ISO-19770-2:2015 {{SWID}}, provide a standardized format for a record that
identifies and describes a specific release of a software product. Different software products, and
even different releases of a particular software product, each have a different SWID tag record
associated with them. In addition to defining the format of these records, ISO-19770-2:2015 defines
requirements concerning the SWID tag life-cycle. Specifically, when a software product is installed
on an endpoint, that product's SWID tag is also installed. Likewise, when the product is uninstalled
or replaced, the SWID tag is deleted or replaced, as appropriate. As a result, ISO-19770-2:2015
describes a system wherein there is a correspondence between the set of installed software products
on an endpoint, and the presence on that endpoint of the SWID tags corresponding to those products.

SWID tags are meant to be flexible and able to express a broad set of metadata about a software
product. Moreover, there are multiple types of SWID tags, each providing different types of
information. For example, a "corpus tag" is used to describe an application's installation image on
an installation media, while a "patch tag" is meant to describe a patch that modifies some other
application. While there are very few required fields in SWID tags, there are many optional fields
that support different uses of these different types of tags. While a SWID tag that consisted only
of required fields could be a few hundred bytes in size, a tag containing many of the optional
fields could be many orders of magnitude larger.

This document defines a more concise representation of SWID tags in the Concise Binary Object
Representation (CBOR) {{-cbor}}. This is described via the Concise Data Definition Language (CDDL)
{{-cddl}}. The resulting Concise SWID data definition is interoperable with the XML schema
definition of ISO-19770-2:2015 {{SWID}}. The vocabulary, i.e., the CDDL names of the types and
members used in the CoSWID data definition, is mapped to more concise labels represented as
small integers. The names used in the CDDL data definition and the mapping to the CBOR
representation using integer labels is based on the vocabulary of the XML attribute and element
names defined in ISO-19770-2:2015.

Real-world instances of SWID tags can be fairly large, and the communication of SWID tags in
use-applications such as those described earlier can cause a large amount of data to be transported.
This can be larger than acceptable for constrained devices and networks. CoSWID tags
significantly reduce the amount of data transported as compared to a typical SWID tag. This
reduction is enable through the use of CBOR, which maps human-readable labels of that content to
more concise integer labels (indices). This allows SWID tags to be part of an enterprise security
solution for a wider range of endpoints and environments.

## Concise SWID Extensions

This document specifies a standard equivalent to the ISO-19770-2:2015 standard.
The corresponding CoSWID data definition includes two kinds of augmentation.

* the explicit definition of types for attributes that are typically stored in the "any attribute" of an
ISO-19770-2:2015 in XML representation. These are covered in the main body of this document.

* the inclusion of extension points in the CoSWID data definition that allow for additional uses of
CoSWID tags that go beyond the original scope of ISO-19770-2:2015 tags. These are covered in
appendices to this document.

## Requirements Notation

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
"OPTIONAL" in this document are to be interpreted as described in RFC
2119, BCP 14 {{RFC2119}}.

# Concise SWID Data Definition

The following is a CDDL representation of the ISO-19770-2:2015 {{SWID}} XML schema definition of
SWID tags. This representation includes every SWID tag fields and attribute and thus supports all SWID tag use
cases. The CamelCase notation used in the XML schema definition is changed to a hyphen-separated
notation (e.g. ResourceCollection is named resource-collection in the CoSWID data definition).
This deviation from the original notation used in the XML representation reduces ambiguity when referencing
certain attributes in corresponding textual descriptions. An attribute referred by its name in CamelCase
notation explicitly relates to XML SWID tags, an attribute referred by its name in
hyphen-separated notation explicitly relates to CoSWID tags. This approach simplifies the
composition of further work that reference both XML SWID and CoSWID documents.

Human-readable names of members in the CDDL data definition are mapped to integer indices via a block of rules at the bottom
of the definition. The 66 character strings of the SWID vocabulary that would have to be
stored or transported in full if using the original vocabulary are replaced.

Concise Software Identifiers are tailored to be used in the domain of constrained-node networks. A
typical endpoint is capable of storing the CoSWID tag of installed software, a constrained-node
might lack that capability. CoSWID address these constraints and the corresponding specification is
augmented to retain their usefulness in the thing-2-thing domain. Specific examples include, but are
not limited to limiting the scope of hash algorithms to the IANA Named Information tables or
including firmware attributes addressing devices that do not necessarily provide a file-system to
store a CoSWID tag in.

In order to create a valid CoSWID document the structure of the corresponding CBOR message MUST
adhere to the following CDDL data definition.

~~~~ CDDL
<CODE BEGINS>
{::include concise-software-identity.cddl}
<CODE ENDS>
~~~~

# Description of the SWID Attribute Vocabulary Definition

Yet to be written still...

<!--  which will be addressed in a future iteration of this draft and most likely result in additional attributes to be included in the general Concise SWID data definition, e.g. signature-type (“compat”, “cose”, etc.). Note that, by their natures, cryptographic signatures will not remain valid if a SWID tag is translated from one representation to another. -->

#  IANA Considerations

This document will include requests to IANA:

* Integer indices for SWID content attributes and information elements.
* Content-Type for CoAP to be used in COSE.

#  Security Considerations

SWID tags contain public information about software products and, as
such, do not need to be protected against disclosure on an endpoint.
Similarly, SWID tags are intended to be easily discoverable by
applications and users on an endpoint in order to make it easy to
identify and collect all of an endpoint's SWID tags.  As such, any
security considerations regarding SWID tags focus on the application
of SWID tags to address security challenges, and the possible
disclosure of the results of those applications.

A signed SWID tag whose signature is intact can be relied upon to be
unchanged since it was signed.  If the SWID tag was created by the
software author, this generally means that it has undergone no change
since the software application with which the tag is associated was
installed.  By implication, this means that the signed tag reflects
the software author's understanding of the details of that software
product.  This can be useful assurance when the information in the
tag needs to be trusted, such as when the tag is being used to convey
golden measurements.  By contrast, the data contained in unsigned
tags cannot be trusted to be unmodified.

SWID tags are designed to be easily added and removed from an
endpoint along with the installation or removal of software products.
On endpoints where addition or removal of software products is
tightly controlled, the addition or removal of SWID tags can be
similarly controlled.  On more open systems, where many users can
manage the software inventory, SWID tags may be easier to add or
remove.  On such systems, it may be possible to add or remove SWID
tags in a way that does not reflect the actual presence or absence of
corresponding software products.  Similarly, not all software
products automatically install SWID tags, so products may be present
on an endpoint without providing a corresponding SWID tag.  As such,
any collection of SWID tags cannot automatically be assumed to
represent either a complete or fully accurate representation of the
software inventory of the endpoint.  However, especially on devices
that more strictly control the ability to add or remove applications,
SWID tags are an easy way to provide an preliminary understanding of
that endpoint's software inventory.

Any report of an endpoint's SWID tag collection provides
information about the software inventory of that endpoint.  If such a
report is exposed to an attacker, this can tell them which software
products and versions thereof are present on the endpoint.  By
examining this list, the attacker might learn of the presence of
applications that are vulnerable to certain types of attacks.  As
noted earlier, SWID tags are designed to be easily discoverable by an
endpoint, but this does not present a significant risk since an
attacker would already need to have access to the endpoint to view
that information.  However, when the endpoint transmits its software
inventory to another party, or that inventory is stored on a server
for later analysis, this can potentially expose this information to
attackers who do not yet have access to the endpoint.  As such, it is
important to protect the confidentiality of SWID tag information that
has been collected from an endpoint, not because those tags
individually contain sensitive information, but because the
collection of SWID tags and their association with an endpoint
reveals information about that endpoint's attack surface.

Finally, both the ISO-19770-2:2015 XML schema definition and the
Concise SWID data definition allow for the construction of "infinite"
SWID tags or SWID tags that contain malicious content with the intend
if creating non-deterministic states during validation or processing of SWID tags. While software
product vendors are unlikely to do this, SWID tags can be created by any party and the SWID tags
collected from an endpoint could contain a mixture of vendor and non-vendor created tags. For this
reason, tools that consume SWID tags ought to treat the tag contents as potentially malicious and
should employ input sanitizing on the tags they ingest.

#  Acknowledgements

#  Change Log

Changes from version 00 to version 01:

* Added CWT usage for absolute SWID paths on a device
* Fixed cardinality of type-choices including arrays
* Included first iteration of firmware resource-collection

Changes since adopted as a WG I-D -00:

* Removed redundant any-attributes originating from the ISO-19770-2:2015 XML schema definition
* Fixed broken multi-map members
* Introduced a more restrictive item (any-element-map) to represent custom maps, increased restriction on types for the any-attribute, accordingly
* Fixed X.1520 reference
* Minor type changes of some attributes (e.g. NMTOKENS)
* Added semantic differentiation of various name types (e,g. fs-name)

Changes from version 00 to version 01:

* Ambiguity between evidence and payload eliminated by introducing explicit members (while still
* allowing for "empty" SWID tags)
* Added a relatively restrictive COSE envelope using cose_sign1 to define signed CoSWID (single signer only, at the moment)
* Added a definition how to encode hashes that can be stored in the any-member using existing IANA tables to reference hash-algorithms

Changes from version 01 to version 02:

* Enforced a more strict separation between the core CoSWID definition and additional usage by
moving content to corresponding appendices.
* Removed artifacts inherited from the reference schema provided by ISO (e.g. NMTOKEN(S))
* Simplified the core data definition by removing group and type choices where possible
* Minor reordering of map members
* Added a first extension point to address requested flexibility for extensions beyond the
any-element

# Contributors

--- back

# Explicit file-hash Type Used in Concise SWID Tags (label 56)

CoSWID add explicit support for the representation of file-hashes using algorithms that are
registered at the Named Information Hash Algorithm Registry via the file-hash member (label 56).

~~~~ CDDL
file-hash = (56: [ hash-alg-id: int, hash-value: bstr ] )
~~~~

The number used as a value for hash-alg-id MUST refer the ID in the Named Information Hash Algorithm
table; other hash algorithms MUST NOT be used. The hash-value MUST represent the raw hash value of the file-entry the file-hash type is
included in.

# CoSWID Attributes for Firmware (label 57)

The ISO-19770-2:2015 specification of SWID tags assumes the existence of a file system a software
component is installed and stored in. In the case of constrained-node networks
{{RFC7228}} or network equipment this assumption might not apply. Concise software instances in the
form of (modular) firmware are often stored directly on a block device that is a hardware component
of the constrained-node or network equipment. Multiple differentiable block devices or segmented
block devices that contain parts of modular firmware components (potentially each with their own
instance version) are already common at the time of this writing.

The optional attributes that annotate a firmware package address specific characteristics of pieces
of firmware stored directly on a block-device in contrast to software deployed in a file-system.
In essence, trees of relative path-elements expressed by the directory and file structure in CoSWID
tags are typically unable to represent the location of a firmware on a constrained-node (small
thing). The composite nature of firmware and also the actual composition of small things require a
set of attributes to address the identification of the correct component in a composite thing for
each individual piece of firmware. A single component also potentially requires a number of distinct
firmware parts that might depend on each other (versions). These dependencies can be limited to the
scope of the component itself or extend to the scope of a larger composite device. In addition, it
might not be possible (or feasible) to store a CoSWID tag document (permanently) on a small thing
along with the corresponding piece of firmware.

To address the specific characteristics of firmware, the extension point `$$resource-extension` is
used to allow for an additional type of resource description---firmware-entry---thereby increasing
the self-descriptiveness and flexibility of CoSWID. The optional use of the extension point
`$$resource-extension` in respect to firmware MUST adhere to the following CDDL data definition.

~~~~ CDDL
<CODE BEGINS>
{::include firmware-resource.cddl}
<CODE ENDS>
~~~~

The members of the firmware group that constitutes the content of the firmware-entry is
based on the metadata about firmware defined in {{-cms-fw-pkgs}}. As with every semantic
differentiation that is supported by the resource-collection type, the use of firmware-entry is
optional. It is REQUIRED not to instantiate more than one firmware-entry, as the firmware group is
used in a map and therefore only allows for unique labels. 

The optional cms-firmware-package member allows to include the actual firmware in the CoSWID tag
that also expresses its metadata as a byte-string. This option enables a CoSWID tag to be used as a
container or wrapper that composes both firmware and its metadata in a single document (which again
can be signed, encrypted and/or compressed). In consequence, a CoSWID tag about firmware can be
conveyed as an identifying document across endpoints or used as a reference integrity
measurement as usual. Alternatively, it can also convey an actual piece of firmware, serve its
intended purpose as a SWID tag and then - due to the lack of a location to store it - be discarded.

# Signed Concise SWID Tags using COSE

SWID tags, as defined in the ISO-19770-2:2015 XML schema, can include cryptographic signatures to
protect the integrity of the SWID tag. In general, tags are signed by the tag creator (typically,
although not exclusively, the vendor of the software product that the SWID tag identifies).
Cryptographic signatures can make any modification of the tag detectable, which is especially
important if the integrity of the tag is important, such as when the tag is providing reference
integrity measurments for files.

The ISO-19770-2:2015 XML schema uses XML DSIG to support cryptographic signatures. CoSWID tags
require a different signature scheme than this. COSE (CBOR Object Signing and Encryption) provides the required mechanism {{-cose-msg}}. Concise SWID can be wrapped in a COSE Single Signer Data Object
(cose-sign1) that contains a single signature. The following CDDL defines a more restrictive subset
of header attributes allowed by COSE tailored to suit the requirements of Concise SWID.

~~~~ CDDL
<CODE BEGINS>
{::include signed-coswid.cddl}
<CODE ENDS>
~~~~

# CoSWID used as Reference Integrity Measurements (CoSWID RIM)

A vendor supplied signed CoSWID tag that includes hash-values for the files that compose a software
component can be used as a RIM (reference integrity measurement). A RIM is a type of declarative
guidance that can be used to assert the compliance of an endpoint by assessing the installed
software. In the context of remote attestation based on an attestation via hardware rooted trust,
a verifier can appraise the integrity of the conveyed measurements
of software components using a CoSWID RIM provided by a source, such as {{-sw-desc}}.

RIM Manifests (RIMM):

: A group of SWID tags about the same (sub-)system, system entity, or (sub-)component (compare
{{RFC4949}}). A RIMM manifest is a distinct document that is typically conveyed en-block and
constitutes declarative guidance in respect to a specific (target) endpoint (compare
{{-sacm-term}}).

If multiple CoSWID compose a RIMM, the following CDDL data definition SHOULD be used.

~~~~ CDDL
RIMM = [ + concise-software-identity / signed-coswid ]
~~~~

# CBOR Web Token for Concise SWID Tags

A typical requirement regarding specific instantiations of endpoints – and, as a result, specific
instantiations of software components - is a representation of the absolute path of a CoSWID tag
document in a file system in order to derive absolute paths of files represented in the
corresponding CoSWID tag. The absolute path of an evidence CoSWID tag can be included as a claim in
the header of a CBOR Web Token {{-cwt}}. Depending on the source of the token, the claim can be in
the protected or unprotected header portion.

~~~~ CDDL
<CODE BEGINS>
 CDDL TBD
<CODE ENDS>
~~~~
<!--  LocalWords:  SWID verifier TPM filesystem discoverable
 -->