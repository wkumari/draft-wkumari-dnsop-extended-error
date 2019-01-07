**Important:** Read CONTRIBUTING.md before submitting feedback or contributing
```




Network Working Group                                          W. Kumari
Internet-Draft                                                    Google
Intended status: Standards Track                                 E. Hunt
Expires: June 23, 2019                                               ISC
                                                               R. Arends
                                                                   ICANN
                                                             W. Hardaker
                                                                 USC/ISI
                                                             D. Lawrence
                                                            Oracle + Dyn
                                                       December 20, 2018


                          Extended DNS Errors
                   draft-ietf-dnsop-extended-error-03

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

   This Internet-Draft will expire on June 23, 2019.

Copyright Notice

   Copyright (c) 2018 IETF Trust and the persons identified as the
   document authors.  All rights reserved.





Kumari, et al.            Expires June 23, 2019                 [Page 1]

Internet-Draft       draft-ietf-dnsop-extended-error       December 2018


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
   2.  Extended Error EDNS0 option format  . . . . . . . . . . . . .   3
   3.  Use of the Extended DNS Error option  . . . . . . . . . . . .   4
     3.1.  The R (Retry) flag  . . . . . . . . . . . . . . . . . . .   5
     3.2.  The RESPONSE-CODE field . . . . . . . . . . . . . . . . .   5
     3.3.  The INFO-CODE field . . . . . . . . . . . . . . . . . . .   5
     3.4.  The EXTRA-TEXT field  . . . . . . . . . . . . . . . . . .   5
   4.  Defined Extended DNS Errors . . . . . . . . . . . . . . . . .   5
     4.1.  INFO-CODEs for use with RESPONSE-CODE: NOERROR(0) . . . .   6
       4.1.1.  NOERROR Extended DNS Error Code 1 - Unsupported
               DNSKEY Algorithm  . . . . . . . . . . . . . . . . . .   6
       4.1.2.  NOERROR Extended DNS Error Code 2 - Unsupported
               DS Algorithm  . . . . . . . . . . . . . . . . . . . .   6
     4.2.  INFO-CODEs for use with RESPONSE-CODE: SERVFAIL(2)  . . .   6
       4.2.1.  SERVFAIL Extended DNS Error Code 1 - DNSSEC Bogus . .   6
       4.2.2.  SERVFAIL Extended DNS Error Code 2 - DNSSEC
               Indeterminate . . . . . . . . . . . . . . . . . . . .   6
       4.2.3.  SERVFAIL Extended DNS Error Code 3 - Signature
               Expired . . . . . . . . . . . . . . . . . . . . . . .   6
       4.2.4.  SERVFAIL Extended DNS Error Code 4 - Signature Not
               Yet Valid . . . . . . . . . . . . . . . . . . . . . .   6
       4.2.5.  SERVFAIL Extended DNS Error Code 5 - DNSKEY missing .   6
       4.2.6.  SERVFAIL Extended DNS Error Code 6 - RRSIGs missing .   7
       4.2.7.  SERVFAIL Extended DNS Error Code 7 - No Zone Key Bit
               Set . . . . . . . . . . . . . . . . . . . . . . . . .   7
     4.3.  INFO-CODEs for use with RESPONSE-CODE: REFUSED(5) . . . .   7
       4.3.1.  REFUSED Extended DNS Error Code 1 - Lame  . . . . . .   7
       4.3.2.  REFUSED Extended DNS Error Code 2 - Prohibited  . . .   7
     4.4.  INFO-CODEs for use with RESPONSE-CODE: NXDOMAIN(3)  . . .   7
       4.4.1.  NXDOMAIN Extended DNS Error Code 1 - Blocked  . . . .   7
   5.  IANA Considerations . . . . . . . . . . . . . . . . . . . . .   7
     5.1.  new Extended Error Code EDNS Option . . . . . . . . . . .   7
     5.2.  New Extended Error Code EDNS Option . . . . . . . . . . .   8
   6.  Security Considerations . . . . . . . . . . . . . . . . . . .   9
   7.  Acknowledgements  . . . . . . . . . . . . . . . . . . . . . .  10



Kumari, et al.            Expires June 23, 2019                 [Page 2]

Internet-Draft       draft-ietf-dnsop-extended-error       December 2018


   8.  References  . . . . . . . . . . . . . . . . . . . . . . . . .  10
     8.1.  Normative References  . . . . . . . . . . . . . . . . . .  10
     8.2.  Informative References  . . . . . . . . . . . . . . . . .  10
   Appendix A.  Changes / Author Notes.  . . . . . . . . . . . . . .  11
   Authors' Addresses  . . . . . . . . . . . . . . . . . . . . . . .  11

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
   error.  When properly authenticated, this information can be used by
   the resolver to make a decision regarding whether or not to retry or
   it can be used or by technical users attempting to debug issues.

1.1.  Requirements notation

   The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
   "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
   document are to be interpreted as described in [RFC2119].

2.  Extended Error EDNS0 option format

   This draft uses an EDNS0 ([RFC2671]) option to include Extended DNS
   Error (EDE) information in DNS messages.  The option is structured as
   follows:



Kumari, et al.            Expires June 23, 2019                 [Page 3]

Internet-Draft       draft-ietf-dnsop-extended-error       December 2018


                                                1   1   1   1   1   1
        0   1   2   3   4   5   6   7   8   9   0   1   2   3   4   5
      +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
   0: |                            OPTION-CODE                        |
      +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
   2: |                           OPTION-LENGTH                       |
      +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
   4: | R |                          RESERVED                         |
      +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
   6: | RESPONSE-CODE |             INFO-CODE                         |
      +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
   8: |                             EXTRA-TEXT                        |
      +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+

   Field definition details:

   o  OPTION-CODE, 2 octets (defined in [RFC6891]), for EDE is TBD.
      [RFC Editor: change TBD to the proper code once assigned by IANA.]
   o  OPTION-LENGTH, 2 octets ((defined in [RFC6891]) contains the
      length of the payload (everything after OPTION-LENGTH) in octets
      and should be 4 plus the length of the EXTRA-TEXT section (which
      may be a zero-length string).
   o  The RETRY flag, 1 bit; the RETRY bit (R) indicates a flag defined
      for use in this specification.
   o  The RESERVED bits, 15 bits: these bits are reserved for future
      use, potentially as additional flags.  The RESERVED bits MUST be
      set to 0 by the sender and SHOULD be ignored by the receiver.
   o  RESPONSE-CODE, 4 bits.
   o  INFO-CODE, 12-bits.
   o  EXTRA-TEXT, a variable length, UTF-8 encoded, text field that may
      hold additional textual information.

3.  Use of the Extended DNS Error option

   The Extended DNS Error (EDE) is an EDNS option.  It can be included
   in any response (SERVFAIL, NXDOMAIN, REFUSED, etc) to a query that
   includes an EDNS option.  This document includes a set of initial
   codepoints (and requests to the IANA to add them to the registry),
   but is extensible via the IANA registry to allow additional error and
   information codes to be defined in the future.

   The fields of the Extended DNS Error option are defined further in
   the following sub-sections.








Kumari, et al.            Expires June 23, 2019                 [Page 4]

Internet-Draft       draft-ietf-dnsop-extended-error       December 2018


3.1.  The R (Retry) flag

   The R (Retry) flag provides a hint as to what the receiver may want
   to do with this annotated error.  Specifically, the R (or Retry) flag
   provides a hint to the receiver that it should retry the query to
   another server.  If the R bit is set (1), the sender believes that
   retrying the query may provide a successful answer next time; if the
   R bit is clear (0), the sender believes that the resolver should not
   ask another server.

   The mechanism is specifically designed to be extensible, and so
   implementations may receive EDE codes that it does not understand.
   The R flag allows implementations to make a decision as to what to do
   if it receives a response with an unknown code - retry or drop the
   query.  Note that this flag is only a suggestion.  Unless a
   protective transport mechanism (like TSIG [RFC2845] or TLS [RFC8094])
   is used, the bit's value could have have been altered by a person-in-
   the-middle.  Receivers can choose to ignore this hint.  See the
   security considerations for additional considerations.

3.2.  The RESPONSE-CODE field

   This 4-bit value SHOULD be a copy of the RCODE from the primary DNS
   packet.  Multiple EDNS0/EDE records may be included in the response.
   When including multiple EDNS0/EDE records in a response in order to
   provide additional error information, other RESPONSE-CODEs MAY use a
   different RCODE.

3.3.  The INFO-CODE field

   This 12-bit value provides the additional context for the RESPONSE-
   CODE value.  This combination of the RESPONSE-CODE and the INFO-CODE
   serve as a joint-index into the IANA "Extended DNS Errors" registry.

3.4.  The EXTRA-TEXT field

   The UTF-8-encoded, EXTRA-TEXT field may be zero-length, or may hold
   additional information useful to network operators.

4.  Defined Extended DNS Errors

   This document defines some initial EDE codes.  The mechanism is
   intended to be extensible, and additional code-points can be
   registered in the "Extended DNS Errors" registry.  This document
   provides suggestions for the R flag, but the originating server may
   ignore these recommendations if it knows better.





Kumari, et al.            Expires June 23, 2019                 [Page 5]

Internet-Draft       draft-ietf-dnsop-extended-error       December 2018


   The RESPONSE-CODE and the INFO-CODE from the EDE EDNS option is used
   to serve as a double index into the "Extended DNS Error codes" IANA
   registry, the initial values for which are defined in the following
   sub-sections.

4.1.  INFO-CODEs for use with RESPONSE-CODE: NOERROR(0)

4.1.1.  NOERROR Extended DNS Error Code 1 - Unsupported DNSKEY Algorithm

   The resolver attempted to perform DNSSEC validation, but a DNSKEY
   RRSET contained only unknown algorithms.  The R flag should be set.

4.1.2.  NOERROR Extended DNS Error Code 2 - Unsupported DS Algorithm

   The resolver attempted to perform DNSSEC validation, but a DS RRSET
   contained only unknown algorithms.  The R flag should be set.

4.2.  INFO-CODEs for use with RESPONSE-CODE: SERVFAIL(2)

4.2.1.  SERVFAIL Extended DNS Error Code 1 - DNSSEC Bogus

   The resolver attempted to perform DNSSEC validation, but validation
   ended in the Bogus state.  The R flag should not be set.

4.2.2.  SERVFAIL Extended DNS Error Code 2 - DNSSEC Indeterminate

   The resolver attempted to perform DNSSEC validation, but validation
   ended in the Indeterminate state.  The R flag should not be set.

4.2.3.  SERVFAIL Extended DNS Error Code 3 - Signature Expired

   The resolver attempted to perform DNSSEC validation, but the
   signature was expired.  The R flag should not be set.

4.2.4.  SERVFAIL Extended DNS Error Code 4 - Signature Not Yet Valid

   The resolver attempted to perform DNSSEC validation, but the
   signatures received were not yet valid.  The R flag should not be
   set.

4.2.5.  SERVFAIL Extended DNS Error Code 5 - DNSKEY missing

   A DS record existed at a parent, but no DNSKEY record could be found
   for the child.  The R flag should not be set.







Kumari, et al.            Expires June 23, 2019                 [Page 6]

Internet-Draft       draft-ietf-dnsop-extended-error       December 2018


4.2.6.  SERVFAIL Extended DNS Error Code 6 - RRSIGs missing

   The resolver attempted to perform DNSSEC validation, but no RRSIGs
   could be found for at least one RRset where RRSIGs were expected.

4.2.7.  SERVFAIL Extended DNS Error Code 7 - No Zone Key Bit Set

   The resolver attempted to perform DNSSEC validation, but no Zone Key
   Bit was set in a DNSKEY.

4.3.  INFO-CODEs for use with RESPONSE-CODE: REFUSED(5)

4.3.1.  REFUSED Extended DNS Error Code 1 - Lame

   An authoritative resolver that receives a query (with the RD bit
   clear) for a domain for which it is not authoritative SHOULD include
   this EDE code in the REFUSED response.  Implementations should set
   the R flag in this case (another nameserver might not be lame).

4.3.2.  REFUSED Extended DNS Error Code 2 - Prohibited

   An authoritative or recursive resolver that receives a query from an
   "unauthorized" client can annotate its REFUSED message with this
   code.  Examples of "unauthorized" clients are recursive queries from
   IP addresses outside the network, blacklisted IP addresses, local
   policy, etc.

   Implementations SHOULD allow operators to define what to set the R
   flag to in this case.

4.4.  INFO-CODEs for use with RESPONSE-CODE: NXDOMAIN(3)

4.4.1.  NXDOMAIN Extended DNS Error Code 1 - Blocked

   The resolver attempted to perfom a DNS query but the domain is
   blacklisted due to a security policy.  The R flag should not be set.

5.  IANA Considerations

5.1.  new Extended Error Code EDNS Option

   This document defines a new EDNS(0) option, entitled "Extended DNS
   Error", assigned a value of TBD1 from the "DNS EDNS0 Option Codes
   (OPT)" registry [to be removed upon publication:
   [http://www.iana.org/assignments/dns-parameters/dns-
   parameters.xhtml#dns-parameters-11]





Kumari, et al.            Expires June 23, 2019                 [Page 7]

Internet-Draft       draft-ietf-dnsop-extended-error       December 2018


   Value  Name                 Status    Reference
   -----  ----------------     ------    ------------------
    TBD   Extended DNS Error    TBD       [ This document ]

5.2.  New Extended Error Code EDNS Option

   This document defines a new double-index IANA registry table, where
   the first index value is the RCODE value and the second index value
   is the INFO-CODE from the Extended DNS Error EDNS option defined in
   this document.  The IANA is requested to create and maintain this
   "Extended DNS Error codes" registry.  The codepoint space for each
   INFO-CODE index is to be broken into 3 ranges:

   o  0 - 3583: Specification required.
   o  3584 - 3839: First Come First Served.
   o  3840 - 4095: Experimental / Private use

   A starting set of entries, based on the contents of this document, is
   as follows:

   RESPONSE-CODE:  0 (NOERROR)
   INFO-CODE:  1
   Purpose:  Unsupported DNSKEY
   Reference:  Section 4.1.1

   RESPONSE-CODE:  0 (NOERROR)
   INFO-CODE:  2
   Purpose:  Unsupported DS Algorithm
   Reference:  Section 4.1.2

   RESPONSE-CODE:  2 (SERVFAIL)
   INFO-CODE:  1
   Purpose:  DNSSEC Bogus
   Reference:  Section 4.2.1

   RESPONSE-CODE:  2 (SERVFAIL)
   INFO-CODE:  2
   Purpose:  DNSSEC Indeterminate
   Reference:  Section 4.2.2

   RESPONSE-CODE:  2 (SERVFAIL)
   INFO-CODE:  3
   Purpose:  Signature Expired
   Reference:  Section 4.2.3

   RESPONSE-CODE:  2 (SERVFAIL)
   INFO-CODE:  4
   Purpose:  Signature Not Yet Valid



Kumari, et al.            Expires June 23, 2019                 [Page 8]

Internet-Draft       draft-ietf-dnsop-extended-error       December 2018


   Reference:  Section 4.2.4

   RESPONSE-CODE:  2 (SERVFAIL)
   INFO-CODE:  5
   Purpose:  DNSKEY missing
   Reference:  Section 4.2.5

   RESPONSE-CODE:  2 (SERVFAIL)
   INFO-CODE:  6
   Purpose:  RRSIGs missing
   Reference:  Section 4.2.6

   RESPONSE-CODE:  2 (SERVFAIL)
   INFO-CODE:  7
   Purpose:  No Zone Key Bit Set
   Reference:  Section 4.2.7

   RESPONSE-CODE:  3 (NXDOMAIN)
   INFO-CODE:  1
   Purpose:  Blocked
   Reference:  Section 4.4.1

   RESPONSE-CODE:  5 (REFUSED)
   INFO-CODE:  1
   Purpose:  Lame
   Reference:  Section 4.3.1

   RESPONSE-CODE:  5 (REFUSED)
   INFO-CODE:  2
   Purpose:  Prohibited
   Reference:  Section 4.3.2

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



Kumari, et al.            Expires June 23, 2019                 [Page 9]

Internet-Draft       draft-ietf-dnsop-extended-error       December 2018


   an era where all DNS answers are authenticated via DNSSEC or other
   mechanisms, there are some tradeoffs.  As an example, an attacker who
   is able to insert the DNSSEC Bogus Extended Error into a packet could
   instead simply reply with a fictitious address (A or AAAA) record.
   The R bit hint and extended error information are informational -
   implementations can choose how much to trust this information and
   validating resolvers / stubs may choose to put a different weight on
   it.

7.  Acknowledgements

   The authors wish to thank Joe Abley, Mark Andrews, Peter DeVries,
   Peter van Dijk, Donald Eastlake, Bob Harold, Evan Hunt, Geoff Huston,
   Shane Kerr, Edward Lewis, Carlos M.  Martinez, George Michelson, Petr
   Spacek, Ondrej Sury, Loganaden Velvindron, and Paul Vixie.  They also
   vaguely remember discussing this with a number of people over the
   years, but have forgotten who all they were -- if you were one of
   them, and are not listed, please let us know and we'll acknowledge
   you.

   I also want to thank the band "Infected Mushroom" for providing a
   good background soundtrack (and to see if I can get away with this!)
   Another author would like to thank the band "Mushroom Infectors".
   This was funny at the time we wrote it, but I cannot remember why...

8.  References

8.1.  Normative References

   [RFC2119]  Bradner, S., "Key words for use in RFCs to Indicate
              Requirement Levels", BCP 14, RFC 2119,
              DOI 10.17487/RFC2119, March 1997, <https://www.rfc-
              editor.org/info/rfc2119>.

8.2.  Informative References

   [GeoffValidation]
              IANA, "A quick review of DNSSEC Validation in today's
              Internet", June 2016, <http://www.potaroo.net/
              presentations/2016-06-27-dnssec.pdf>.

   [RFC2845]  Vixie, P., Gudmundsson, O., Eastlake 3rd, D., and B.
              Wellington, "Secret Key Transaction Authentication for DNS
              (TSIG)", RFC 2845, DOI 10.17487/RFC2845, May 2000,
              <https://www.rfc-editor.org/info/rfc2845>.






Kumari, et al.            Expires June 23, 2019                [Page 10]

Internet-Draft       draft-ietf-dnsop-extended-error       December 2018


   [RFC8094]  Reddy, T., Wing, D., and P. Patil, "DNS over Datagram
              Transport Layer Security (DTLS)", RFC 8094,
              DOI 10.17487/RFC8094, February 2017, <https://www.rfc-
              editor.org/info/rfc8094>.

Appendix A.  Changes / Author Notes.

   [RFC Editor: Please remove this section before publication ]

   From -00 to -01:

   o  Address comments from IETF meeting.
   o  document copying the response code
   o  mention zero length fields are ok
   o  clarify lookup procedure
   o  mention that table isn't done

   From -03 to -IETF 00:

   o  Renamed to draft-ietf-dnsop-extended-error

   From -02 to -03:

   o  Added David Lawrence -- I somehow missed that in last version.

   From -00 to -01;

   o  Fixed up some of the text, minor clarifications.

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




Kumari, et al.            Expires June 23, 2019                [Page 11]

Internet-Draft       draft-ietf-dnsop-extended-error       December 2018


   Roy Arends
   ICANN

   Email: roy.arends@icann.org


   Wes Hardaker
   USC/ISI
   P.O. Box 382
   Davis, CA  95617
   US

   Email: ietf@hardakers.net


   David C Lawrence
   Oracle + Dyn
   150 Dow St
   Manchester, NH  03101
   US

   Email: tale@dd.org





























Kumari, et al.            Expires June 23, 2019                [Page 12]
```
