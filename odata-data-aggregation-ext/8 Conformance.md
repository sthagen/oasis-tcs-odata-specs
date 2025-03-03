
-------

# ##sec Conformance

Conforming services MUST follow all rules of this specification for the set transformations and aggregation methods they support. They MUST implement all set transformations and aggregation methods they advertise via the annotation [`ApplySupported`](#AggregationCapabilities).

Conforming clients MUST be prepared to consume a model that uses any or all of the constructs defined in this specification, including custom aggregation methods defined by the service, and MUST ignore any constructs not defined in this version of the specification.

-------

# Appendix ##asec References

This appendix contains the normative references that are used in this document.

While any hyperlinks included in this appendix were valid at the time of publication, OASIS cannot guarantee their long-term validity.

## ##subasec Normative References

The following documents are referenced in such a way that some or all of their content constitutes requirements of this document.

###### [OData-ABNF]{id=ODataABNF}
_ABNF components: OData ABNF Construction Rules Version 4.01 and OData ABNF Test Cases._  
See link in "[Related work](#RelatedWork)" section on cover page.

###### [OData-Agg-ABNF]{id=ODataAggABNF}
_OData Aggregation ABNF Construction Rules Version 4.0._  
See link in "[Additional artifacts](#AdditionalArtifacts)" section on cover page.

###### [OData-CSDL]{id=ODataCSDL}
_OData Common Schema Definition Language (CSDL) JSON Representation Version 4.01._  
See link in "[Related work](#RelatedWork)" section on cover page.

_OData Common Schema Definition Language (CSDL) XML Representation Version 4.01._  
See link in "[Related work](#RelatedWork)" section on cover page.

###### [OData-JSON]{id=ODataJSON}
_OData JSON Format Version 4.01._  
See link in "[Related work](#RelatedWork)" section on cover page.

###### [OData-Protocol]{id=ODataProtocol}
_OData Version 4.01. Part 1: Protocol._  
See link in "[Related work](#RelatedWork)" section on cover page.

###### [OData-URL]{id=ODataURL}
_OData Version 4.01. Part 2: URL Conventions._  
See link in "[Related work](#RelatedWork)" section on cover page.

###### [OData-VocAggr]{id=ODataVocAggr}
_OData Aggregation Vocabulary._  
See link in "[Additional artifacts](#AdditionalArtifacts)" section on cover page.

###### [OData-VocCore]{id=ODataVocCore}
_OData Core Vocabulary._  
See link in "[Related work](#RelatedWork)" section on cover page.

###### [RFC2119]{id=rfc2119}
_Bradner, S., "Key words for use in RFCs to Indicate Requirement Levels", BCP 14, RFC 2119, DOI 10.17487/RFC2119, March 1997_  
https://www.rfc-editor.org/info/rfc2119.

###### [RFC8174]{id=rfc8174}
_Leiba, B., "Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words", BCP 14, RFC 8174, DOI 10.17487/RFC8174, May 2017_  
https://www.rfc-editor.org/info/rfc8174.

-------

# Appendix ##asec Acknowledgments

## ##subasec Special Thanks

The contributions of the OASIS OData Technical Committee members, enumerated in [#OData-Protocol#Participants], are gratefully acknowledged.

## ##subasec Participants

**OData TC Members:**

| First Name | Last Name | Company |
| :--- | :--- | :--- |
| George | Ericson | Dell |
| Hubert | Heijkers | IBM |
| Ling | Jin | IBM |
| Stefan | Hagen | Individual |
| Michael | Pizzo | Microsoft |
| Christof | Sprenger | Microsoft |
| Ralf | Handl | SAP SE |
| Gerald | Krause | SAP SE |
| Heiko | Theißen | SAP SE |
| Martin | Zurmuehl | SAP SE |

-------

# Appendix ##asec Revision History

Revision|Date|Editor|Changes Made
--------|----|------|------------
Working Draft 01|2012-11-12|Ralf Handl|Translated contribution into OASIS format
Committee Specification Draft 01|2013-07-25| 
Ralf Handl<br> 
Hubert Heijkers<br> 
Gerald Krause<br> 
Michael Pizzo<br> 
Martin Zurmuehl| 
Switched to pipe-and-filter-style query language based on composable set transformations<br> 
Fleshed out examples and addressed numerous editorial and technical issues processed through the TC<br> 
Added Conformance section
Committee Specification Draft 02|2014-01-09| 
Ralf Handl<br> 
Hubert Heijkers<br> 
Gerald Krause<br> 
Michael Pizzo<br> 
Martin Zurmuehl| 
Dynamic properties used all aggregated values either via aliases or via custom aggregates<br> 
Refactored annotations
Committee Specification Draft 03|2015-07-16| 
Ralf Handl<br> 
Hubert Heijkers<br> 
Gerald Krause<br> 
Michael Pizzo<br> 
Martin Zurmuehl| 
Added compute transformation<br> 
Minor clean-up
Committee Specification Draft 04|2023-07-05| 
Ralf Handl<br> 
Hubert Heijkers<br> 
Gerald Krause<br> 
Michael Pizzo<br> 
Heiko Theißen| 
Added section about fundamentals of input and output sets<br> 
Algorithmic descriptions of transformations<br> 
Added join and outerjoin transformations, replaced expand by addnested<br> 
Added transformations orderby, skip, top, nest<br> 
Added transformations for recursive hierarchies, updated related filter functions<br> 
Added functions evaluable on a collection, introduced keyword $these<br> 
Merged section 4 "Representation of Aggregated Instances" into section 3<br> 
Remove actions and functions (except set transformations) on aggregated entities, adapted section "Actions and Functions on Aggregated Entities"
$$$subtitle$$$|$$$pubdateISO$$$| 
Ralf Handl<br> 
Gerald Krause<br> 
Heiko Theißen| 
Non-material changes from public review feedback

-------

# Appendix ##asec Notices

Copyright $$$copyright$$$. All Rights Reserved.

All capitalized terms in the following text have the meanings assigned to them in the OASIS Intellectual Property Rights Policy (the "OASIS IPR Policy"). The full [Policy](https://www.oasis-open.org/policies-guidelines/ipr/) may be found at the OASIS website.

This document and translations of it may be copied and furnished to others, and derivative works that comment on or otherwise explain it or assist in its implementation may be prepared, copied, published, and distributed, in whole or in part, without restriction of any kind, provided that the above copyright notice and this section are included on all such copies and derivative works. However, this document itself may not be modified in any way, including by removing the copyright notice or references to OASIS, except as needed for the purpose of developing any document or deliverable produced by an OASIS Technical Committee (in which case the rules applicable to copyrights, as set forth in the OASIS IPR Policy, must be followed) or as required to translate it into languages other than English.

The limited permissions granted above are perpetual and will not be revoked by OASIS or its successors or assigns.

This document and the information contained herein is provided on an "AS IS" basis and OASIS DISCLAIMS ALL WARRANTIES, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO ANY WARRANTY THAT THE USE OF THE INFORMATION HEREIN WILL NOT INFRINGE ANY OWNERSHIP RIGHTS OR ANY IMPLIED WARRANTIES OF MERCHANTABILITY OR FITNESS FOR A PARTICULAR PURPOSE.

As stated in the OASIS IPR Policy, the following three paragraphs in brackets apply to OASIS Standards Final Deliverable documents (Committee Specification, Candidate OASIS Standard, OASIS Standard, or Approved Errata).

\[OASIS requests that any OASIS Party or any other party that believes it has patent claims that would necessarily be infringed by implementations of this OASIS Standards Final Deliverable, to notify OASIS TC Administrator and provide an indication of its willingness to grant patent licenses to such patent claims in a manner consistent with the IPR Mode of the OASIS Technical Committee that produced this deliverable.\]

\[OASIS invites any party to contact the OASIS TC Administrator if it is aware of a claim of ownership of any patent claims that would necessarily be infringed by implementations of this OASIS Standards Final Deliverable by a patent holder that is not willing to provide a license to such patent claims in a manner consistent with the IPR Mode of the OASIS Technical Committee that produced this OASIS Standards Final Deliverable. OASIS may include such claims on its website, but disclaims any obligation to do so.\]

\[OASIS takes no position regarding the validity or scope of any intellectual property or other rights that might be claimed to pertain to the implementation or use of the technology described in this OASIS Standards Final Deliverable or the extent to which any license under such rights might or might not be available; neither does it represent that it has made any effort to identify any such rights. Information on OASIS' procedures with respect to rights in any document or deliverable produced by an OASIS Technical Committee can be found on the OASIS website. Copies of claims of rights made available for publication and any assurances of licenses to be made available, or the result of an attempt made to obtain a general license or permission for the use of such proprietary rights by implementers or users of this OASIS Standards Final Deliverable, can be obtained from the OASIS TC Administrator. OASIS makes no representation that any information or list of intellectual property rights will at any time be complete, or that any claims in such list are, in fact, Essential Claims.\]

The name "OASIS" is a trademark of [OASIS](https://www.oasis-open.org/), the owner and developer of this specification, and should be used only to refer to the organization and its official outputs. OASIS welcomes reference to, and implementation and use of, specifications, while reserving the right to enforce its marks against misleading uses. Please see https://www.oasis-open.org/policies-guidelines/trademark/ for above guidance.
