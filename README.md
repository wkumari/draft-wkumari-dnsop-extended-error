**Important:** Read CONTRIBUTING.md before submitting feedback or contributing
```




Network Working Group                                          W. Kumari
Internet-Draft                                                    Google
Intended status: Standards Track                                 E. Hunt
Expires: February 10, 2020                                           ISC
                                                               R. Arends
                                                                   ICANN
                                                             W. Hardaker
                                                                 USC/ISI
                                                             D. Lawrence
                                                            Oracle + Dyn
                                                         August 09, 2019


                          Extended DNS Errors
                   draft-ietf-dnsop-extended-error-07

Abstract

   This document defines an extensible method to return additional
   information about the cause of DNS errors.  Though created primarily
   to extend SERVFAIL to provide additional information about the cause
   of DNS and DNSSEC failures, the Extended DNS Errors option defined in
   this document allows all response types to contain extended error
   information.

Status of This Memo

   This Internet-Draft is submitted in full conformance with the
   provisions of BCP 78 and BCP 79.

   Internet-Drafts are working documents of the Internet Engineering
   Task Force (IETF).  Note that other groups may also distribute
   working documents as Internet-Drafts.  The list of current Internet-
   Drafts is at http://datatracker.ietf.org/drafts/current/.

   Internet-Drafts are draft documents valid for a maximum of six months
   and may be updated, replaced, or obsoleted by other documents at any
   time.  It is inappropriate to use Internet-Drafts as reference
   material or to cite them other than as "work in progress."

   This Internet-Draft will expire on February 10, 2020.

Copyright Notice

   Copyright (c) 2019 IETF Trust and the persons identified as the
   document authors.  All rights reserved.





Kumari, et al.          Expires February 10, 2020               [Page 1]

Internet-Draft       draft-ietf-dnsop-extended-error         August 2019


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

   1.  Introduction and background . . . . . . . . . . . . . . . . .   3
     1.1.  Requirements notation . . . . . . . . . . . . . . . . . .   3
   2.  Extended Error EDNS0 option format  . . . . . . . . . . . . .   4
   3.  Use of the Extended DNS Error option  . . . . . . . . . . . .   4
     3.1.  The INFO-CODE field . . . . . . . . . . . . . . . . . . .   5
     3.2.  The EXTRA-TEXT field  . . . . . . . . . . . . . . . . . .   5
   4.  Defined Extended DNS Errors . . . . . . . . . . . . . . . . .   5
     4.1.  Extended DNS Error Code 0 - Other . . . . . . . . . . . .   5
     4.2.  Extended DNS Error Code 1 -
           Unsupported DNSKEY Algorithm  . . . . . . . . . . . . . .   5
     4.3.  Extended DNS Error Code 2 - Unsupported
           DS Algorithm  . . . . . . . . . . . . . . . . . . . . . .   5
     4.4.  Extended DNS Error Code 3 - Stale Answer  . . . . . . . .   5
     4.5.  Extended DNS Error Code 4 - Forged Answer . . . . . . . .   6
     4.6.  Extended DNS Error Code 5 - DNSSEC Indeterminate  . . . .   6
     4.7.  Extended DNS Error Code 6 - DNSSEC Bogus  . . . . . . . .   6
     4.8.  Extended DNS Error Code 7 - Signature Expired . . . . . .   6
     4.9.  Extended DNS Error Code 8 - Signature Not Yet Valid . . .   6
     4.10. Extended DNS Error Code 9 - DNSKEY Missing  . . . . . . .   6
     4.11. Extended DNS Error Code 10 - RRSIGs Missing . . . . . . .   6
     4.12. Extended DNS Error Code 11 - No Zone Key Bit Set  . . . .   6
     4.13. Extended DNS Error Code 12 - NSEC Missing . . . . . . . .   6
     4.14. Extended DNS Error Code 13 - Cached Error . . . . . . . .   7
     4.15. Extended DNS Error Code 14 - Not Ready  . . . . . . . . .   7
     4.16. Extended DNS Error Code 15 - Blocked  . . . . . . . . . .   7
     4.17. Extended DNS Error Code 16 - Censored . . . . . . . . . .   7
     4.18. Extended DNS Error Code 17 - Prohibited . . . . . . . . .   7
     4.19. Extended DNS Error Code 18 - Stale Answer . . . . . . . .   7
     4.20. Extended DNS Error Code 19 - Lame . . . . . . . . . . . .   7
     4.21. Extended DNS Error Code 20 - Deprecated . . . . . . . . .   8
     4.22. Extended DNS Error Code 21 - No Reachable Authority . . .   8
   5.  IANA Considerations . . . . . . . . . . . . . . . . . . . . .   8
     5.1.  A New Extended Error Code EDNS Option . . . . . . . . . .   8
     5.2.  New Double-Index Registry Table for Extended Error Codes    8
   6.  Security Considerations . . . . . . . . . . . . . . . . . . .  10
   7.  Acknowledgements  . . . . . . . . . . . . . . . . . . . . . .  11



Kumari, et al.          Expires February 10, 2020               [Page 2]

Internet-Draft       draft-ietf-dnsop-extended-error         August 2019


   8.  References  . . . . . . . . . . . . . . . . . . . . . . . . .  11
     8.1.  Normative References  . . . . . . . . . . . . . . . . . .  11
     8.2.  Informative References  . . . . . . . . . . . . . . . . .  11
   Authors' Addresses  . . . . . . . . . . . . . . . . . . . . . . .  12

1.  Introduction and background

   There are many reasons that a DNS query may fail, some of them
   transient, some permanent; some can be resolved by querying another
   server, some are likely best handled by stopping resolution.
   Unfortunately, the error signals that a DNS server can return are
   very limited, and are not very expressive.  This means that
   applications and resolvers often have to "guess" at what the issue is
   - e.g. was the answer marked REFUSED because of a lame delegation, or
   because the nameserver is still starting up and loading zones?  Is a
   SERVFAIL a DNSSEC validation issue, or is the nameserver experiencing
   a bad hair day?

   A good example of issues that would benefit by additional error
   information are errors caused by DNSSEC validation issues.  When a
   stub resolver queries a DNSSEC bogus name (using a validating
   resolver), the stub resolver receives only a SERVFAIL in response.
   Unfortunately, SERVFAIL is used to signal many sorts of DNS errors,
   and so the stub resolver simply asks the next configured DNS
   resolver.  The result of trying the next resolver is one of two
   outcomes: either the next resolver also validates, a SERVFAIL is
   returned again, and the user gets an (largely) incomprehensible error
   message; or the next resolver is not a validating resolver, and the
   user is returned a potentially harmful result.

   This document specifies a mechanism to extend (or annotate) DNS
   errors to provide additional information about the cause of the
   error.  These extended error codes are specially useful when received
   by resolvers, to return to stub resolvers or to downstream resolvers.
   Authoritative servers MAY parse and use them, but most error codes
   would make no sense for them.  Authoritative servers may need to
   generate extended error codes though.

1.1.  Requirements notation

   The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
   "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
   document are to be interpreted as described in [RFC2119].








Kumari, et al.          Expires February 10, 2020               [Page 3]

Internet-Draft       draft-ietf-dnsop-extended-error         August 2019


2.  Extended Error EDNS0 option format

   This draft uses an EDNS0 ([RFC2671]) option to include Extended DNS
   Error (EDE) information in DNS messages.  The option is structured as
   follows:

                                                1   1   1   1   1   1
        0   1   2   3   4   5   6   7   8   9   0   1   2   3   4   5
      +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
   0: |                            OPTION-CODE                        |
      +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
   2: |                           OPTION-LENGTH                       |
      +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
   4: | INFO-CODE                                                     |
      +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
   6: / EXTRA-TEXT ...                                                /
      +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+

   Field definition details:

   o  OPTION-CODE, 2 octets (defined in [RFC6891]]), for EDE is TBD.
      [RFC Editor: change TBD to the proper code once assigned by IANA.]
   o  OPTION-LENGTH, 2 octets ((defined in [RFC6891]]) contains the
      length of the payload (everything after OPTION-LENGTH) in octets
      and should be 4 plus the length of the EXTRA-TEXT section (which
      may be a zero-length string).
   o  INFO-CODE, 16-bits, which is the principal contribution of this
      document.
   o  EXTRA-TEXT, a variable length, UTF-8 encoded, text field that may
      hold additional textual information.  Note: EXTRA-TEXT may be zero
      octets in length, indicating there is no EXTRA-TEXT included.

3.  Use of the Extended DNS Error option

   The Extended DNS Error (EDE) is an EDNS option.  It can be included
   in any response (SERVFAIL, NXDOMAIN, REFUSED, etc) to a query that
   includes OPT Pseudo-RR [RFC6891].  This document includes a set of
   initial codepoints (and requests to the IANA to add them to the
   registry), but is extensible via the IANA registry to allow
   additional error and information codes to be defined in the future.

   The fields of the Extended DNS Error option are defined further in
   the following sub-sections.








Kumari, et al.          Expires February 10, 2020               [Page 4]

Internet-Draft       draft-ietf-dnsop-extended-error         August 2019


3.1.  The INFO-CODE field

   This 16-bit value, encoded in network (MSB) byte order, provides the
   additional context for the RESPONSE-CODE of the DNS message.  The
   INFO-CODE serves as an index to the "Extended DNS Errors" registry
   Section 5.1.

3.2.  The EXTRA-TEXT field

   The UTF-8-encoded, EXTRA-TEXT field may be zero-length, or may hold
   additional information useful to network operators.

4.  Defined Extended DNS Errors

   This document defines some initial EDE codes.  The mechanism is
   intended to be extensible, and additional code-points can be
   registered in the "Extended DNS Errors" registry Section 5.1.  The
   INFO-CODE from the EDE EDNS option is used to serve as an index into
   the "Extended DNS Error codes" IANA registry, the initial values for
   which are defined in the following sub-sections.

4.1.  Extended DNS Error Code 0 - Other

   The error in question falls into a category that does not match known
   extended error codes.  Implementations SHOULD include a EXTRA-TEXT
   value to augment this error code with additional information.

4.2.  Extended DNS Error Code 1 - Unsupported DNSKEY Algorithm

   The resolver attempted to perform DNSSEC validation, but a DNSKEY
   RRSET contained only unknown algorithms.

4.3.  Extended DNS Error Code 2 - Unsupported DS Algorithm

   The resolver attempted to perform DNSSEC validation, but a DS RRSET
   contained only unknown algorithms.

4.4.  Extended DNS Error Code 3 - Stale Answer

   The resolver was unable to resolve answer within its time limits and
   decided to answer with a previously cached data instead of answering
   with an error.  This is typically caused by problems on authoritative
   side, possibly as result of a DoS attack.








Kumari, et al.          Expires February 10, 2020               [Page 5]

Internet-Draft       draft-ietf-dnsop-extended-error         August 2019


4.5.  Extended DNS Error Code 4 - Forged Answer

   For policy reasons (legal obligation, or malware filtering, for
   instance), an answer was forged.

4.6.  Extended DNS Error Code 5 - DNSSEC Indeterminate

   The resolver attempted to perform DNSSEC validation, but validation
   ended in the Indeterminate state.

4.7.  Extended DNS Error Code 6 - DNSSEC Bogus

   The resolver attempted to perform DNSSEC validation, but validation
   ended in the Bogus state.

4.8.  Extended DNS Error Code 7 - Signature Expired

   The resolver attempted to perform DNSSEC validation, a signature in
   the validation chain was expired.

4.9.  Extended DNS Error Code 8 - Signature Not Yet Valid

   The resolver attempted to perform DNSSEC validation, but the
   signatures received were not yet valid.

4.10.  Extended DNS Error Code 9 - DNSKEY Missing

   A DS record existed at a parent, but no supported matching DNSKEY
   record could be found for the child.

4.11.  Extended DNS Error Code 10 - RRSIGs Missing

   The resolver attempted to perform DNSSEC validation, but no RRSIGs
   could be found for at least one RRset where RRSIGs were expected.

4.12.  Extended DNS Error Code 11 - No Zone Key Bit Set

   The resolver attempted to perform DNSSEC validation, but no Zone Key
   Bit was set in a DNSKEY.

4.13.  Extended DNS Error Code 12 - NSEC Missing

   The resolver attempted to perform DNSSEC validation, but the
   requested data was missing and a covering NSEC or NSEC3 was not
   provided.






Kumari, et al.          Expires February 10, 2020               [Page 6]

Internet-Draft       draft-ietf-dnsop-extended-error         August 2019


4.14.  Extended DNS Error Code 13 - Cached Error

   The resolver has cached SERVFAIL for this query without additional
   information.

4.15.  Extended DNS Error Code 14 - Not Ready

   The server is unable to answer the query as it is not fully up and
   functional yet.

4.16.  Extended DNS Error Code 15 - Blocked

   The resolver attempted to perfom a DNS query but the domain is
   blacklisted due to a security policy implemented on the server being
   directly talked to.

4.17.  Extended DNS Error Code 16 - Censored

   The resolver attempted to perfom a DNS query but the domain was
   blacklisted by a security policy imposed upon the server being talked
   to.  Note that how the imposed policy is applied is irrelevant (in-
   band DNS somehow, court order, etc).

4.18.  Extended DNS Error Code 17 - Prohibited

   An authoritative or recursive resolver that receives a query from an
   "unauthorized" client can annotate its REFUSED message with this
   code.  Examples of "unauthorized" clients are recursive queries from
   IP addresses outside the network, blacklisted IP addresses, local
   policy, etc.

4.19.  Extended DNS Error Code 18 - Stale Answer

   The resolver was unable to resolve answer within its time limits and
   decided to answer with a previously cached NXDOMAIN answer instead of
   answering with an error.  This is typically caused by problems on
   authoritative side, possibly as result of a DoS attack.

4.20.  Extended DNS Error Code 19 - Lame

   An authoritative server that receives a query (with the RD bit clear)
   for a domain for which it is not authoritative SHOULD include this
   EDE code in the SERVFAIL response.  A resolver that receives a query
   (with the RD bit clear) SHOULD include this EDE code in the REFUSED
   response.






Kumari, et al.          Expires February 10, 2020               [Page 7]

Internet-Draft       draft-ietf-dnsop-extended-error         August 2019


4.21.  Extended DNS Error Code 20 - Deprecated

   The requested operation or query is not supported as its use has been
   deprecated.

4.22.  Extended DNS Error Code 21 - No Reachable Authority

   The resolver could not reach any of the authoritative name servers
   (or they refused to reply).

5.  IANA Considerations

5.1.  A New Extended Error Code EDNS Option

   This document defines a new EDNS(0) option, entitled "Extended DNS
   Error", assigned a value of TBD1 from the "DNS EDNS0 Option Codes
   (OPT)" registry [to be removed upon publication:
   [http://www.iana.org/assignments/dns-parameters/dns-
   parameters.xhtml#dns-parameters-11]

   Value  Name                 Status    Reference
   -----  ----------------     ------    ------------------
    TBD   Extended DNS Error    TBD       [ This document ]

5.2.  New Double-Index Registry Table for Extended Error Codes

   This document defines a new double-index IANA registry table, where
   the index value is the INFO-CODE from the Extended DNS Error EDNS
   option defined in this document.  The IANA is requested to create and
   maintain this "Extended DNS Error codes" registry.  The code-point
   space for each INFO-CODE index is to be broken into 3 ranges:

   o  0 - 32767: Specification required.
   o  32768 - 49151: First come, first served.
   o  49152 - 65535: Experimental / Private use

   A starting set of entries, based on the contents of this document, is
   as follows:

   INFO-CODE:  0
   Purpose:  Other Error
   Reference:  Section 4.1

   INFO-CODE:  1
   Purpose:  Unsupported DNSKEY Algorithm
   Reference:  Section 4.2

   INFO-CODE:  2



Kumari, et al.          Expires February 10, 2020               [Page 8]

Internet-Draft       draft-ietf-dnsop-extended-error         August 2019


   Purpose:  Unsupported DS Algorithm
   Reference:  Section 4.3

   INFO-CODE:  3
   Purpose:  Answering with stale/cached data
   Reference:  Section 4.4

   INFO-CODE:  4
   Purpose:  Forged Answer
   Reference:  Section 4.5

   INFO-CODE:  5
   Purpose:  DNSSEC Indeterminate
   Reference:  Section 4.6

   INFO-CODE:  6
   Purpose:  DNSSEC Bogus
   Reference:  Section 4.7

   INFO-CODE:  7
   Purpose:  Signature Expired
   Reference:  Section 4.8

   INFO-CODE:  8
   Purpose:  Signature Not Yet Valid
   Reference:  Section 4.9

   INFO-CODE:  9
   Purpose:  DNSKEY Missing
   Reference:  Section 4.10

   INFO-CODE:  10
   Purpose:  RRSIGs Missing
   Reference:  Section 4.11

   INFO-CODE:  11
   Purpose:  No Zone Key Bit Set
   Reference:  Section 4.12

   INFO-CODE:  12
   Purpose:  No NSEC records could be obtained
   Reference:  Section 4.13

   INFO-CODE:  13
   Purpose:  The SERVFAIL error comes from the cache
   Reference:  Section 4.14

   INFO-CODE:  14



Kumari, et al.          Expires February 10, 2020               [Page 9]

Internet-Draft       draft-ietf-dnsop-extended-error         August 2019


   Purpose:  Not Ready.
   Reference:  Section 4.15

   INFO-CODE:  15
   Purpose:  Blocked
   Reference:  Section 4.16

   INFO-CODE:  16
   Purpose:  Censored
   Reference:  Section 4.17

   INFO-CODE:  17
   Purpose:  Prohibited
   Reference:  Section 4.18

   INFO-CODE:  18
   Purpose:  Answering with stale/cached NXDOMAIN data
   Reference:  Section 4.19

   INFO-CODE:  19
   Purpose:  Lame
   Reference:  Section 4.20

   INFO-CODE:  20
   Purpose:  Deprecated
   Reference:  Section 4.21

   INFO-CODE:  21
   Purpose:  No Reachable Authority
   Reference:  Section 4.21

6.  Security Considerations

   Though DNSSEC continues to be deployed, unfortunately a significant
   number of clients (~11% according to [GeoffValidation]) that receive
   a SERVFAIL from a validating resolver because of a DNSSEC validaion
   issue will simply ask the next (potentially non-validating) resolver
   in their list, and thus don't get any of the protections which DNSSEC
   should provide.  This is very similar to a kid asking his mother if
   he can have another cookie.  When the mother says "No, it will ruin
   your dinner!", going off and asking his (more permissive) father and
   getting a "Yes, sure, have a cookie!".

   This information is unauthenticated information, and an attacker (e.g
   MITM or malicious recursive server) could insert an extended error
   response into already untrusted data -- ideally clients and resolvers
   would not trust any unauthenticated information, but until we live in
   an era where all DNS answers are authenticated via DNSSEC or other



Kumari, et al.          Expires February 10, 2020              [Page 10]

Internet-Draft       draft-ietf-dnsop-extended-error         August 2019


   mechanisms, there are some tradeoffs.  As an example, an attacker who
   is able to insert the DNSSEC Bogus Extended Error into a packet could
   instead simply reply with a fictitious address (A or AAAA) record.

7.  Acknowledgements

   The authors wish to thank Joe Abley, Mark Andrews, Stephane
   Bortzmeyer, Vladimir Cunat, Ralph Dolmans, Peter DeVries, Peter van
   Dijk, Donald Eastlake, Bob Harold, Geoff Huston, Shane Kerr, Edward
   Lewis, Carlos M.  Martinez, George Michelson, Michael Sheldon, Puneet
   Sood, Petr Spacek, Ondrej Sury, Loganaden Velvindron, and Paul Vixie.
   They also vaguely remember discussing this with a number of people
   over the years, but have forgotten who all they were -- if you were
   one of them, and are not listed, please let us know and we'll
   acknowledge you.

   I also want to thank the band "Infected Mushroom" for providing a
   good background soundtrack (and to see if I can get away with this!)
   Another author would like to thank the band "Mushroom Infectors".
   This was funny at the time we wrote it, but we cannot remember why...

8.  References

8.1.  Normative References

   [RFC2119]  Bradner, S., "Key words for use in RFCs to Indicate
              Requirement Levels", BCP 14, RFC 2119,
              DOI 10.17487/RFC2119, March 1997, <https://www.rfc-
              editor.org/info/rfc2119>.

   [RFC2671]  Vixie, P., "Extension Mechanisms for DNS (EDNS0)",
              RFC 2671, DOI 10.17487/RFC2671, August 1999,
              <https://www.rfc-editor.org/info/rfc2671>.

   [RFC6891]  Damas, J., Graff, M., and P. Vixie, "Extension Mechanisms
              for DNS (EDNS(0))", STD 75, RFC 6891,
              DOI 10.17487/RFC6891, April 2013, <https://www.rfc-
              editor.org/info/rfc6891>.

8.2.  Informative References

   [GeoffValidation]
              IANA, "A quick review of DNSSEC Validation in today's
              Internet", June 2016, <http://www.potaroo.net/
              presentations/2016-06-27-dnssec.pdf>.






Kumari, et al.          Expires February 10, 2020              [Page 11]

Internet-Draft       draft-ietf-dnsop-extended-error         August 2019


   [RFC2845]  Vixie, P., Gudmundsson, O., Eastlake 3rd, D., and B.
              Wellington, "Secret Key Transaction Authentication for DNS
              (TSIG)", RFC 2845, DOI 10.17487/RFC2845, May 2000,
              <https://www.rfc-editor.org/info/rfc2845>.

   [RFC8094]  Reddy, T., Wing, D., and P. Patil, "DNS over Datagram
              Transport Layer Security (DTLS)", RFC 8094,
              DOI 10.17487/RFC8094, February 2017, <https://www.rfc-
              editor.org/info/rfc8094>.

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
   ICANN

   Email: roy.arends@icann.org


   Wes Hardaker
   USC/ISI
   P.O. Box 382
   Davis, CA  95617
   US

   Email: ietf@hardakers.net








Kumari, et al.          Expires February 10, 2020              [Page 12]

Internet-Draft       draft-ietf-dnsop-extended-error         August 2019


   David C Lawrence
   Oracle + Dyn
   150 Dow St
   Manchester, NH  03101
   US

   Email: tale@dd.org












































Kumari, et al.          Expires February 10, 2020              [Page 13]
```
