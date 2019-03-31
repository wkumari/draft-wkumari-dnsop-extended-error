



Network Working Group                                          W. Kumari
Internet-Draft                                                    Google
Intended status: Standards Track                                 E. Hunt
Expires: September 12, 2019                                          ISC
                                                               R. Arends
                                                                   ICANN
                                                             W. Hardaker
                                                                 USC/ISI
                                                             D. Lawrence
                                                            Oracle + Dyn
                                                          March 11, 2019


                          Extended DNS Errors
                   draft-ietf-dnsop-extended-error-05

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

   This Internet-Draft will expire on September 12, 2019.

Copyright Notice

   Copyright (c) 2019 IETF Trust and the persons identified as the
   document authors.  All rights reserved.





Kumari, et al.         Expires September 12, 2019               [Page 1]

Internet-Draft       draft-ietf-dnsop-extended-error          March 2019


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
     1.1.  Requirements notation . . . . . . . . . . . . . . . . . .   4
   2.  Extended Error EDNS0 option format  . . . . . . . . . . . . .   4
   3.  Use of the Extended DNS Error option  . . . . . . . . . . . .   5
     3.1.  The R (Retry) flag  . . . . . . . . . . . . . . . . . . .   5
     3.2.  The RESPONSE-CODE field . . . . . . . . . . . . . . . . .   6
     3.3.  The INFO-CODE field . . . . . . . . . . . . . . . . . . .   6
     3.4.  The EXTRA-TEXT field  . . . . . . . . . . . . . . . . . .   6
   4.  Defined Extended DNS Errors . . . . . . . . . . . . . . . . .   6
     4.1.  INFO-CODEs for use with RESPONSE-CODE: NOERROR(0) . . . .   6
       4.1.1.  NOERROR Extended DNS Error Code 1 - Unsupported
               DNSKEY Algorithm  . . . . . . . . . . . . . . . . . .   6
       4.1.2.  NOERROR Extended DNS Error Code 2 - Unsupported
               DS Algorithm  . . . . . . . . . . . . . . . . . . . .   6
     4.2.  INFO-CODEs for use with RESPONSE-CODE: NOERROR(3) . . . .   7
       4.2.1.  NOERROR Extended DNS Error Code 3 - Stale Answer  . .   7
       4.2.2.  NOERROR Extended DNS Error Code 4 - Forged Answer . .   7
       4.2.3.  SERVFAIL Extended DNS Error Code 5 - DNSSEC
               Indeterminate . . . . . . . . . . . . . . . . . . . .   7
     4.3.  INFO-CODEs for use with RESPONSE-CODE: SERVFAIL(2)  . . .   7
       4.3.1.  SERVFAIL Extended DNS Error Code 1 - DNSSEC Bogus . .   7
       4.3.2.  SERVFAIL Extended DNS Error Code 2 - Signature
               Expired . . . . . . . . . . . . . . . . . . . . . . .   7
       4.3.3.  SERVFAIL Extended DNS Error Code 3 - Signature Not
               Yet Valid . . . . . . . . . . . . . . . . . . . . . .   7
       4.3.4.  SERVFAIL Extended DNS Error Code 4 - DNSKEY Missing .   7
       4.3.5.  SERVFAIL Extended DNS Error Code 5 - RRSIGs Missing .   8
       4.3.6.  SERVFAIL Extended DNS Error Code 6 - No Zone Key Bit
               Set . . . . . . . . . . . . . . . . . . . . . . . . .   8
       4.3.7.  SERVFAIL Extended DNS Error Code 7 - No
               Reachable Authority . . . . . . . . . . . . . . . . .   8
       4.3.8.  SERVFAIL Extended DNS Error Code 8 - NSEC Missing . .   8
       4.3.9.  SERVFAIL Extended DNS Error Code 9 - Cached Error . .   8
       4.3.10. SERVFAIL Extended DNS Error Code 10 - Not Ready . . .   8
       4.3.11. SERVFAIL Extended DNS Error Code 11 - Not Specified .   8
     4.4.  INFO-CODEs for use with RESPONSE-CODE: NXDOMAIN(3)  . . .   8



Kumari, et al.         Expires September 12, 2019               [Page 2]

Internet-Draft       draft-ietf-dnsop-extended-error          March 2019


       4.4.1.  NXDOMAIN Extended DNS Error Code 1 - Blocked  . . . .   8
       4.4.2.  NXDOMAIN Extended DNS Error Code 2 - Censored . . . .   9
       4.4.3.  NXDOMAIN Extended DNS Error Code 3 - Stale Answer . .   9
     4.5.  INFO-CODEs for use with RESPONSE-CODE: NOTIMP(4)  . . . .   9
       4.5.1.  NOTIMP Extended DNS Error Code 1 - Deprecated . . . .   9
     4.6.  INFO-CODEs for use with RESPONSE-CODE: REFUSED(5) . . . .   9
       4.6.1.  REFUSED Extended DNS Error Code 1 - Lame  . . . . . .   9
       4.6.2.  REFUSED Extended DNS Error Code 2 - Prohibited  . . .   9
   5.  IANA Considerations . . . . . . . . . . . . . . . . . . . . .  10
     5.1.  A New Extended Error Code EDNS Option . . . . . . . . . .  10
     5.2.  New Double-Index Registry Table for Extended Error Codes   10
   6.  Security Considerations . . . . . . . . . . . . . . . . . . .  13
   7.  Acknowledgements  . . . . . . . . . . . . . . . . . . . . . .  13
   8.  References  . . . . . . . . . . . . . . . . . . . . . . . . .  13
     8.1.  Normative References  . . . . . . . . . . . . . . . . . .  13
     8.2.  Informative References  . . . . . . . . . . . . . . . . .  14
   Appendix A.  Changes / Author Notes.  . . . . . . . . . . . . . .  14
   Authors' Addresses  . . . . . . . . . . . . . . . . . . . . . . .  15

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




Kumari, et al.         Expires September 12, 2019               [Page 3]

Internet-Draft       draft-ietf-dnsop-extended-error          March 2019


   the resolver to make a decision regarding whether or not to retry or
   it can be used or by technical users attempting to debug issues.

   These extended error codes are specially useful when received by
   resolvers, to return to stub resolvers or to downstream resolvers.
   Authoritative servers MAY parse and use them, but most error codes
   would make no sense for them.  Authoritative servers may need to
   generate extended error codes though.

1.1.  Requirements notation

   The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
   "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
   document are to be interpreted as described in [RFC2119].

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
      set to 0 by the sender and MUST be ignored by the receiver.



Kumari, et al.         Expires September 12, 2019               [Page 4]

Internet-Draft       draft-ietf-dnsop-extended-error          March 2019


   o  RESPONSE-CODE, 4 bits.  (Note: RESPONSE-CODE and INFO-CODE
      together comprise a 2-octet field in network byte order, of which
      RESPONSE-CODE is the most significant 4 bits.)
   o  INFO-CODE, 12 bits.  (The least significant 12 bits of the
      combined 2-octet field described above.)
   o  EXTRA-TEXT, a variable length, UTF-8 encoded, text field that may
      hold additional textual information.

3.  Use of the Extended DNS Error option

   The Extended DNS Error (EDE) is an EDNS option.  It can be included
   in any response (SERVFAIL, NXDOMAIN, REFUSED, etc) to a query that
   includes OPT Pseudo-RR [RFC6891].  A sender MUST NOT include more
   than one Extended DNS Error option per message; additional Extended
   DNS Error options after the first one MUST be ignored by the
   receiver.

   This document includes a set of initial codepoints (and requests to
   the IANA to add them to the registry), but is extensible via the IANA
   registry to allow additional error and information codes to be
   defined in the future.

   The fields of the Extended DNS Error option are defined further in
   the following sub-sections.

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
   protective transport mechanism (like TSIG [RFC2845] or (D)TLS xref
   target="RFC7858"/>, [RFC8094]) is used, the bit's value could have
   have been altered by a person-in-the-middle.  Receivers can choose to
   ignore this hint.  See the security considerations for additional
   considerations.






Kumari, et al.         Expires September 12, 2019               [Page 5]

Internet-Draft       draft-ietf-dnsop-extended-error          March 2019


3.2.  The RESPONSE-CODE field

   This 4-bit value SHOULD be a copy of the RCODE from the primary DNS
   packet.  RESPONSE-CODEs MAY use a different RCODE to provide
   additional or better information.

3.3.  The INFO-CODE field

   This 12-bit value provides the additional context for the RESPONSE-
   CODE value.  This combination of the RESPONSE-CODE and the INFO-CODE
   serve as a joint-index into the IANA "Extended DNS Errors" registry.

   Note to implementers: the combination of the RESPONSE-CODE and INFO-
   CODE fits within a 16-bit field, allowing implementers the choice of
   treating the combination as either two separate values, as defined in
   this document, or as a single 16-bit integer as long as the results
   are deterministic.

3.4.  The EXTRA-TEXT field

   The UTF-8-encoded, EXTRA-TEXT field may be zero-length, or may hold
   additional information useful to network operators.

4.  Defined Extended DNS Errors

   This document defines some initial EDE codes.  The mechanism is
   intended to be extensible, and additional code-points can be
   registered in the "Extended DNS Errors" registry.  This document
   provides suggestions for the R flag, but the originating server may
   ignore these recommendations if it knows better.

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




Kumari, et al.         Expires September 12, 2019               [Page 6]

Internet-Draft       draft-ietf-dnsop-extended-error          March 2019


4.2.  INFO-CODEs for use with RESPONSE-CODE: NOERROR(3)

4.2.1.  NOERROR Extended DNS Error Code 3 - Stale Answer

   The resolver was unable to resolve answer within its time limits and
   decided to answer with a previously cached data instead of answering
   with an error.  This is typically caused by problems on authoritative
   side, possibly as result of a DoS attack.  The R flag should not be
   set, since retrying is likely to create additional load without
   yielding a more fresh answer.

4.2.2.  NOERROR Extended DNS Error Code 4 - Forged Answer

   For policy reasons (legal obligation, or malware filtering, for
   instance), an answer was forged.  The R flag should not be set.

4.2.3.  SERVFAIL Extended DNS Error Code 5 - DNSSEC Indeterminate

   The resolver attempted to perform DNSSEC validation, but validation
   ended in the Indeterminate state.  The R flag should not be set.

4.3.  INFO-CODEs for use with RESPONSE-CODE: SERVFAIL(2)

4.3.1.  SERVFAIL Extended DNS Error Code 1 - DNSSEC Bogus

   The resolver attempted to perform DNSSEC validation, but validation
   ended in the Bogus state.  The R flag should not be set.

4.3.2.  SERVFAIL Extended DNS Error Code 2 - Signature Expired

   The resolver attempted to perform DNSSEC validation, a signature in
   the validation chain was expired.  The R flag should not be set.

4.3.3.  SERVFAIL Extended DNS Error Code 3 - Signature Not Yet Valid

   The resolver attempted to perform DNSSEC validation, but the
   signatures received were not yet valid.  The R flag should not be
   set.

4.3.4.  SERVFAIL Extended DNS Error Code 4 - DNSKEY Missing

   A DS record existed at a parent, but no supported matching DNSKEY
   record could be found for the child.  The R flag should not be set.








Kumari, et al.         Expires September 12, 2019               [Page 7]

Internet-Draft       draft-ietf-dnsop-extended-error          March 2019


4.3.5.  SERVFAIL Extended DNS Error Code 5 - RRSIGs Missing

   The resolver attempted to perform DNSSEC validation, but no RRSIGs
   could be found for at least one RRset where RRSIGs were expected.

4.3.6.  SERVFAIL Extended DNS Error Code 6 - No Zone Key Bit Set

   The resolver attempted to perform DNSSEC validation, but no Zone Key
   Bit was set in a DNSKEY.

4.3.7.  SERVFAIL Extended DNS Error Code 7 - No Reachable Authority

   The resolver could not reach any of the authoritative name servers
   (or they refused to reply).  The R flag should be set.

4.3.8.  SERVFAIL Extended DNS Error Code 8 - NSEC Missing

   The resolver attempted to perform DNSSEC validation, but the
   requested data was missing and a covering NSEC or NSEC3 was not
   provided.  The R flag should be set.

4.3.9.  SERVFAIL Extended DNS Error Code 9 - Cached Error

   The resolver has cached SERVFAIL for this query without additional
   information.  Th R flag should be set.

4.3.10.  SERVFAIL Extended DNS Error Code 10 - Not Ready

   The server is unable to answer the query as it is not fully up and
   functional yet.

4.3.11.  SERVFAIL Extended DNS Error Code 11 - Not Specified

   A generic code which may be used when a SERVFAIL response does not
   fall into any of the other categories.  The R flag should not be set.

4.4.  INFO-CODEs for use with RESPONSE-CODE: NXDOMAIN(3)

4.4.1.  NXDOMAIN Extended DNS Error Code 1 - Blocked

   The resolver attempted to perfom a DNS query but the domain is
   blacklisted due to a security policy implemented on the server being
   directly talked to.  The R flag should be set.








Kumari, et al.         Expires September 12, 2019               [Page 8]

Internet-Draft       draft-ietf-dnsop-extended-error          March 2019


4.4.2.  NXDOMAIN Extended DNS Error Code 2 - Censored

   The resolver attempted to perfom a DNS query but the domain was
   blacklisted by a security policy imposed upon the server being talked
   to.  Note that how the imposed policy is applied is irrelevant (in-
   band DNS somehow, court order, etc).  The R flag should be set.

4.4.3.  NXDOMAIN Extended DNS Error Code 3 - Stale Answer

   The resolver was unable to resolve answer within its time limits and
   decided to answer with a previously cached NXDOMAIN answer instead of
   answering with an error.  This is typically caused by problems on
   authoritative side, possibly as result of a DoS attack.  The R flag
   should not be set, since retrying is likely to create additional load
   without yielding a more fresh answer.

4.5.  INFO-CODEs for use with RESPONSE-CODE: NOTIMP(4)

4.5.1.  NOTIMP Extended DNS Error Code 1 - Deprecated

   The requested operation or query is not supported as its use has been
   deprecated.  Implementations should not set the R flag.  (Retrying
   request elsewhere is unlikely to yield any other results.)

4.6.  INFO-CODEs for use with RESPONSE-CODE: REFUSED(5)

4.6.1.  REFUSED Extended DNS Error Code 1 - Lame

   An authoritative server that receives a query (with the RD bit clear)
   for a domain for which it is not authoritative SHOULD include this
   EDE code in the SERVFAIL response.  A resolver that receives a query
   (with the RD bit clear) SHOULD include this EDE code in the REFUSED
   response.  Implementations should set the R flag in this case
   (another nameserver or resolver might not be lame).

4.6.2.  REFUSED Extended DNS Error Code 2 - Prohibited

   An authoritative or recursive resolver that receives a query from an
   "unauthorized" client can annotate its REFUSED message with this
   code.  Examples of "unauthorized" clients are recursive queries from
   IP addresses outside the network, blacklisted IP addresses, local
   policy, etc.

   Implementations SHOULD allow operators to define what to set the R
   flag to in this case.






Kumari, et al.         Expires September 12, 2019               [Page 9]

Internet-Draft       draft-ietf-dnsop-extended-error          March 2019


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

   RESPONSE-CODE:  0 (NOERROR)
   INFO-CODE:  3
   Purpose:  Answering with stale/cached data
   Reference:  Section 4.2.1

   RESPONSE-CODE:  0 (NOERROR)
   INFO-CODE:  4
   Purpose:  Forged Answer



Kumari, et al.         Expires September 12, 2019              [Page 10]

Internet-Draft       draft-ietf-dnsop-extended-error          March 2019


   Reference:  Section 4.2.2

   RESPONSE-CODE:  0 (NOERROR)
   INFO-CODE:  5
   Purpose:  DNSSEC Indeterminate
   Reference:  Section 4.2.3

   RESPONSE-CODE:  2 (SERVFAIL)
   INFO-CODE:  1
   Purpose:  DNSSEC Bogus
   Reference:  Section 4.3.1

   RESPONSE-CODE:  2 (SERVFAIL)
   INFO-CODE:  2
   Purpose:  Signature Expired
   Reference:  Section 4.3.2

   RESPONSE-CODE:  2 (SERVFAIL)
   INFO-CODE:  3
   Purpose:  Signature Not Yet Valid
   Reference:  Section 4.3.3

   RESPONSE-CODE:  2 (SERVFAIL)
   INFO-CODE:  4
   Purpose:  DNSKEY Missing
   Reference:  Section 4.3.4

   RESPONSE-CODE:  2 (SERVFAIL)
   INFO-CODE:  5
   Purpose:  RRSIGs Missing
   Reference:  Section 4.3.5

   RESPONSE-CODE:  2 (SERVFAIL)
   INFO-CODE:  6
   Purpose:  No Zone Key Bit Set
   Reference:  Section 4.3.6

   RESPONSE-CODE:  2 (SERVFAIL)
   INFO-CODE:  7
   Purpose:  No Reachable Authority
   Reference:  Section 4.3.7

   RESPONSE-CODE:  2 (SERVFAIL)
   INFO-CODE:  9
   Purpose:  NSEC Missing
   Reference:  Section 4.3.8

   RESPONSE-CODE:  2 (SERVFAIL)



Kumari, et al.         Expires September 12, 2019              [Page 11]

Internet-Draft       draft-ietf-dnsop-extended-error          March 2019


   INFO-CODE:  9
   Purpose:  The SERVFAIL error comes from the cache
   Reference:  Section 4.3.9

   RESPONSE-CODE:  2 (SERVFAIL)
   INFO-CODE:  10
   Purpose:  Not Ready.
   Reference:  Section 4.3.10

   RESPONSE-CODE:  2 (SERVFAIL)
   INFO-CODE:  11
   Purpose:  Not Specified.
   Reference:  Section 4.3.11

   RESPONSE-CODE:  3 (NXDOMAIN)
   INFO-CODE:  1
   Purpose:  Blocked
   Reference:  Section 4.4.1

   RESPONSE-CODE:  3 (NXDOMAIN)
   INFO-CODE:  2
   Purpose:  Censored
   Reference:  Section 4.4.2

   RESPONSE-CODE:  3 (NXDOMAIN)
   INFO-CODE:  3
   Purpose:  Answering with stale/cached NXDOMAIN data
   Reference:  Section 4.4.3

   RESPONSE-CODE:  4 (NOTIMP)
   INFO-CODE:  1
   Purpose:  Deprecated
   Reference:  Section 4.6.2

   RESPONSE-CODE:  5 (REFUSED)
   INFO-CODE:  1
   Purpose:  Lame
   Reference:  Section 4.6.1

   RESPONSE-CODE:  5 (REFUSED)
   INFO-CODE:  2
   Purpose:  Prohibited
   Reference:  Section 4.6.2








Kumari, et al.         Expires September 12, 2019              [Page 12]

Internet-Draft       draft-ietf-dnsop-extended-error          March 2019


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
   mechanisms, there are some tradeoffs.  As an example, an attacker who
   is able to insert the DNSSEC Bogus Extended Error into a packet could
   instead simply reply with a fictitious address (A or AAAA) record.
   The R bit hint and extended error information are informational -
   implementations can choose how much to trust this information and
   validating resolvers / stubs may choose to put a different weight on
   it.

7.  Acknowledgements

   The authors wish to thank Joe Abley, Mark Andrews, Stephane
   Bortzmeyer, Vladimir Cunat, Peter DeVries, Peter van Dijk, Donald
   Eastlake, Bob Harold, Geoff Huston, Shane Kerr, Edward Lewis, Carlos
   M.  Martinez, George Michelson, Michael Sheldon, Petr Spacek, Ondrej
   Sury, Loganaden Velvindron, and Paul Vixie.  They also vaguely
   remember discussing this with a number of people over the years, but
   have forgotten who all they were -- if you were one of them, and are
   not listed, please let us know and we'll acknowledge you.

   I also want to thank the band "Infected Mushroom" for providing a
   good background soundtrack (and to see if I can get away with this!)
   Another author would like to thank the band "Mushroom Infectors".
   This was funny at the time we wrote it, but I cannot remember why...

8.  References

8.1.  Normative References







Kumari, et al.         Expires September 12, 2019              [Page 13]

Internet-Draft       draft-ietf-dnsop-extended-error          March 2019


   [RFC2119]  Bradner, S., "Key words for use in RFCs to Indicate
              Requirement Levels", BCP 14, RFC 2119,
              DOI 10.17487/RFC2119, March 1997, <https://www.rfc-
              editor.org/info/rfc2119>.

   [RFC6891]  Damas, J., Graff, M., and P. Vixie, "Extension Mechanisms
              for DNS (EDNS(0))", STD 75, RFC 6891,
              DOI 10.17487/RFC6891, April 2013, <https://www.rfc-
              editor.org/info/rfc6891>.

8.2.  Informative References

   [GeoffValidation]
              IANA, "A quick review of DNSSEC Validation in today's
              Internet", June 2016, <http://www.potaroo.net/
              presentations/2016-06-27-dnssec.pdf>.

   [RFC2845]  Vixie, P., Gudmundsson, O., Eastlake 3rd, D., and B.
              Wellington, "Secret Key Transaction Authentication for DNS
              (TSIG)", RFC 2845, DOI 10.17487/RFC2845, May 2000,
              <https://www.rfc-editor.org/info/rfc2845>.

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



Kumari, et al.         Expires September 12, 2019              [Page 14]

Internet-Draft       draft-ietf-dnsop-extended-error          March 2019


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







Kumari, et al.         Expires September 12, 2019              [Page 15]
