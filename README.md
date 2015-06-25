**Important:** Read CONTRIBUTING.md before submitting feedback or contributing
```




template                                                       W. Kumari
Internet-Draft                                                    Google
Intended status: Informational                                   E. Hunt
Expires: August 3, 2015                                              ISC
                                                               R. Arends
                                                                 Nominet
                                                             W. Hardaker
                                                                 Parsons
                                                        January 30, 2015


                          Extended DNS Errors
                 draft-wkumari-dnsop-extended-error-0.1

Abstract

   This document defines an extensible method to return additional
   information about the cause of DNS errors.  The primary expected use
   for this is to extend SERVFAIL to provide more information about the
   cause of DNSSEC failures, but it can be used to annotate many DNS
   errors.

   [ Ed note: Text inside square brackets ([]) is additional background
   information, answers to freqently asked questions, general musings,
   etc.  They will be removed before publication.]

   [ This document is being collaborated on in Github at:
   https://github.com/wkumari/<TEMPLATE>.  The most recent version of
   the document, open issues, etc should all be available here.  The
   authors (gratefully) acept pull requests ]

   [ Note: I always have a hard time with EDNS terminology - I'm saying
   that Extended DNS Errors are carried as EDNS Options, but is this
   actually correct terminology?  They are optional TLVs in "options" in
   RDATA in OPTion RR , but that's not readable. ]

Status of This Memo

   This Internet-Draft is submitted in full conformance with the
   provisions of BCP 78 and BCP 79.

   Internet-Drafts are working documents of the Internet Engineering
   Task Force (IETF).  Note that other groups may also distribute
   working documents as Internet-Drafts.  The list of current Internet-
   Drafts is at http://datatracker.ietf.org/drafts/current/.

   Internet-Drafts are draft documents valid for a maximum of six months
   and may be updated, replaced, or obsoleted by other documents at any



Kumari, et al.           Expires August 3, 2015                 [Page 1]

Internet-Draft     draft-wkumari-dnsop-extended-error       January 2015


   time.  It is inappropriate to use Internet-Drafts as reference
   material or to cite them other than as "work in progress."

   This Internet-Draft will expire on August 3, 2015.

Copyright Notice

   Copyright (c) 2015 IETF Trust and the persons identified as the
   document authors.  All rights reserved.

   This document is subject to BCP 78 and the IETF Trust's Legal
   Provisions Relating to IETF Documents
   (http://trustee.ietf.org/license-info) in effect on the date of
   publication of this document.  Please review these documents
   carefully, as they describe your rights and restrictions with respect
   to this document.  Code Components extracted from this document must
   include Simplified BSD License text as described in Section 4.e of
   the Trust Legal Provisions and are provided without warranty as
   described in the Simplified BSD License.

Table of Contents

   1.  Introduction and background . . . . . . . . . . . . . . . . .   2
     1.1.  Requirements notation . . . . . . . . . . . . . . . . . .   3
   2.  Extended Error EDNS0 option format  . . . . . . . . . . . . .   3
   3.  Use of the Extended DNS Error option  . . . . . . . . . . . .   4
   4.  Defined Extended DNS Errors . . . . . . . . . . . . . . . . .   4
     4.1.  Extended DNS Error Code 1 - DNSSEC Bogus  . . . . . . . .   4
     4.2.  Extended DNS Error Code 2 - Lame  . . . . . . . . . . . .   5
     4.3.  Extended DNS Error Code 3 - Prohibited  . . . . . . . . .   5
     4.4.  Extended DNS Error Code 4 - Ratelimited . . . . . . . . .   5
   5.  IANA Considerations . . . . . . . . . . . . . . . . . . . . .   5
   6.  Open questions  . . . . . . . . . . . . . . . . . . . . . . .   6
   7.  Security Considerations . . . . . . . . . . . . . . . . . . .   6
   8.  Acknowledgements  . . . . . . . . . . . . . . . . . . . . . .   6
   9.  References  . . . . . . . . . . . . . . . . . . . . . . . . .   6
     9.1.  Normative References  . . . . . . . . . . . . . . . . . .   6
     9.2.  Informative References  . . . . . . . . . . . . . . . . .   6
   Appendix A.  Changes / Author Notes.  . . . . . . . . . . . . . .   6
   Authors' Addresses  . . . . . . . . . . . . . . . . . . . . . . .   7

1.  Introduction and background

   There are many reasons that a DNS query may fail, some of them
   transient, some permenant; some can be resolved by querying another
   server, some (e.g DNSSEC bogus) are best handled by stopping
   resolution.  The range of errors that DNS servers can return are very
   limited, and are not very expressive.  This means that applications



Kumari, et al.           Expires August 3, 2015                 [Page 2]

Internet-Draft     draft-wkumari-dnsop-extended-error       January 2015


   and resolvers often have to "guess" at what the issue is - e.g is a
   SERVFAIL because there is a lame delegation, it is a DNSSEC
   vailidation issue, or is the nameserver experiancing a bad day?

   A good exampe of issues caused by this is DNSSEC validation issues.
   When a user attempts to resolver a DNSSEC bogus name (using a
   validating resolver), their machine receives a SERVFAIL response.
   Unfortunaty, SERVFAIL is used to signal may sorts of DNS errors, and
   so their machine / stub resolver simply asks the next configured DNS
   resolver.  This may result in two outcomes, either the next resolver
   does not validate, in which case the user resolves the name
   (defeating the protection provided by DNSSEC), or it does, and they
   get a failure that is largely incomprehensible to the average user.

   This document specifies a mechanism to extend (or annotate) DNS
   errors to provide additional information about the cause of the
   error.  This information can be used by the resolver to make a
   decision to retry or not, or by users attempting to debug issues.

   Here is a reference to an "external" (non-RFC / draft) thing:
   ([IANA.AS_Numbers]).  And this is a link to an
   ID:[I-D.ietf-sidr-iana-objects].

1.1.  Requirements notation

   The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
   "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
   document are to be interpreted as described in [RFC2119].

2.  Extended Error EDNS0 option format

   This draft uses an EDNS0 ([RFC2671]) option to include extended error
   information in DNS messages.  The option is structured as follows:

                                                1   1   1   1   1   1
        0   1   2   3   4   5   6   7   8   9   0   1   2   3   4   5
      +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
   0: |                            OPTION-CODE                        |
      +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
   2: |                           OPTION-LENGTH                       |
      +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
   4: | R |                         FLAGS                             |
      +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
   6: |                             CODE                              |
      +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+

   o  OPTION-CODE, 2 octets (Defined in [RFC6891]), for ExtError is TBD.




Kumari, et al.           Expires August 3, 2015                 [Page 3]

Internet-Draft     draft-wkumari-dnsop-extended-error       January 2015


   o  OPTION-LENGTH, 2 octets ((Defined in [RFC6891]) contains the
      length of the payload (everything after OPTION-LENGTH) in octets.

   o  FLAGS, 2 octets.

   o  CODE, 2 octets.

   Currently the only defined flag is the R flag.

   R - Retry  The R (or Retry) flag provides a hint to the receiver if
      it should retry the query, possiably by querying another server.
      If the R bit is set (1), the sender believes that retrying the
      query may provide a sucessful answer, if the R bit is clear (0),
      the sender believes that it should not ask another resolver.

   The remaining bits in the flags field MUST be set to 0 by the sender
   and MUST be ignored by the receiver.

   Code: A code point into the IANA "Extended DNS Errors" registry.

3.  Use of the Extended DNS Error option

   The Extended DNS Error is an EDNS option.  It can be included in any
   error response to a query that includes an EDNS option - SERVFAIL,
   NXDOMAIN, REFUSED, etc.  This document includes a set of initial
   codepoints (and requests to the IANA to add them to the registry),
   but is extensible to allow additional error codes to be defined.

   The R (Retry) flag provides a hint (or suggestion) as to what the
   receiver may want to do with this annotated error.  The mechanism is
   specifically designed to be extensible, and so implementations may
   receive EDE codes that it does not understand.  The R flag allows
   implmentations to make a decision as to what to do if it receives a
   response with an unknown code - retry or drop the query.

4.  Defined Extended DNS Errors

   This document defines some initial EDE codes.  The mechanism is
   intended to be extensible, and additional codepoint will be
   registered in the "Extended DNS Errors" registry.

4.1.  Extended DNS Error Code 1 - DNSSEC Bogus

   The resolver attemped to perform DNSSEC validation, but validation
   ended in a Bogus or Indeterminite state.

   Usually attached to SERVFAIL messages.  The R flag SHOUD be set.




Kumari, et al.           Expires August 3, 2015                 [Page 4]

Internet-Draft     draft-wkumari-dnsop-extended-error       January 2015


4.2.  Extended DNS Error Code 2 - Lame

   An authorative resolver that receives a query (with the RD bit clear)
   for a domain for which it is not authorative SHOULD include this EDE
   code in the REFUSED response.

   Implementations SHOULD allow operators to define what to set the R
   flag to in this case.

4.3.  Extended DNS Error Code 3 - Prohibited

   An authorative or recursive resolver that receives a query from an
   "unauthorized" client can annotate its REFUSED message with this
   code.  Examples of "unauthorized" clients are recursive queries from
   IP addresses outside the network, blacklisted IP addresses, etc.

   Implementations SHOULD allow operators to define what to set the R
   flag to in this case.

4.4.  Extended DNS Error Code 4 - Ratelimited

   A nameserver which is under excessive load (for example, because it
   is experiancing a DoS) may annotate any error with this code.

   It is RECOMMENDED that implementations set the R flag in this case,
   but may allow operators to define what to set the R flag to.

5.  IANA Considerations

   [This section under constuction ]

   This document defines a new EDNS(0) option, entitled "Extended DNS
   Error", assigned a value of TBD1 from the "DNS EDNS0 Option Codes
   (OPT)" registry [to be removed upon publication:
   [http://www.iana.org/assignments/dns-parameters/dns-
   parameters.xhtml#dns-parameters-11]

   Value  Name                 Status    Reference
   -----  ----------------     ------    ------------------
    TBD   Extended DNS Error    TBD       [ This document ]

   Data Tag Name Length Meaning ---- ---- ------ ------- TBD1 FooBar N
   FooBar server

   The IANA is requested to create and maintain the "Extended DNS Error
   codes" registry.  The codepoint space is broken into 3 ranges:

   o  1 - 16384: Specification required.



Kumari, et al.           Expires August 3, 2015                 [Page 5]

Internet-Draft     draft-wkumari-dnsop-extended-error       January 2015


   o  16385 - 65000: First Come First Served

   o  65000 - 65534: Experimental / Private use

   The codepoints 0, 65535 are reserved.

   The

6.  Open questions

   1  Can this be included in *any* response or only responses to
      requests that included an EDNS option?  Resolvers are supposed to
      ignore additional.  EDNS capable ones are supposed to simply
      ignore unknonw options.

   2  Can this be applied to *any* respinse, or only error responses?

7.  Security Considerations

   TODO: Fill this out!

8.  Acknowledgements

   The authors wish to thank some folk.

9.  References

9.1.  Normative References

   [IANA.AS_Numbers]
              IANA, "Autonomous System (AS) Numbers",
              <http://www.iana.org/assignments/as-numbers>.

   [RFC2119]  Bradner, S., "Key words for use in RFCs to Indicate
              Requirement Levels", BCP 14, RFC 2119, March 1997.

9.2.  Informative References

   [I-D.ietf-sidr-iana-objects]
              Manderson, T., Vegoda, L., and S. Kent, "RPKI Objects
              issued by IANA", draft-ietf-sidr-iana-objects-03 (work in
              progress), May 2011.

Appendix A.  Changes / Author Notes.

   [RFC Editor: Please remove this section before publication ]

   From -00 to -01.



Kumari, et al.           Expires August 3, 2015                 [Page 6]

Internet-Draft     draft-wkumari-dnsop-extended-error       January 2015


   o  Nothing changed in the template!

Authors' Addresses

   Warren Kumari
   Google
   1600 Amphitheatre Parkway
   Mountain View, CA  94043
   US

   Email: warren@kumari.net


   Evan Hunt
   ISC
   950 Charter St
   Redwood City, CA  94063
   US

   Email: each@isc.org


   Roy Arends
   Nominet
   UK

   Email: TBD


   Wes Hardaker
   Parsons
   P.O. Box 382
   Davis, VA  95617
   US

   Email: ietf@hardakers.net















Kumari, et al.           Expires August 3, 2015                 [Page 7]
```
