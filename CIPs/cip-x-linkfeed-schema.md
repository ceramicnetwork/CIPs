---
cip: 142
title: Linkfeed - a content feed abstraction standard
author: Zhixiong Pan (@nake13), Liang Qiao (@qbig), dugubuyan (@dugubuyan), Alex Wu (@Alex_bitxia)
discussions-to: https://forum.ceramic.network/t/linkfeed-schema-rfc-wip/1199
status: Draft
category: RFC
created: 2023-06-28
edited: 2023-07-13
---

<!--PROPOSE A NEW CIP-->

<!--NOTE: 
You can leave these HTML comments in your CIP and delete the visible text guides, they will not appear and may be helpful to refer to if you edit your CIP again.-->

<!-- STEPS TO SUBMIT A CIP:
1. Complete the header above.
2. Fill in as much content as is appropriate for the status of your CIP.-->

<!--ADDITIONAL INSTRUCTIONS FOR HEADER SECTION ABOVE-->

<!--[title]: Give your issue a concise, descriptive title prefixed by either its *type* for standards CIPs or its category for other CIPs. (i.e. Core: Protocol Upgrade, Meta: Define CIP Process, etc.).-->

<!--[category]: Here is a description of category terms.
- `Core`: an CIP that affects the core protocol.
- `Networking`: an CIP thst affects the networking layer (i.e. libp2p or syncing).
- `Interface`: an CIP that affects the Ceramic API or provider interface.
- `RFC`: an CIP that proposes an implementation standard (i.e. doctypes, document configurations, or document schemas).
- `Meta`: an CIP that affects the governance process for CIPs.-->

<!--[requires]: A list of CIP(s) that this CIP depends on. *Optional.-->

<!--[replaces]: A list of CIP(s) that this CIP replaces. *Optional.-->

## Simple Summary
<!--Provide a simplified and layman-accessible explanation of the CIP.-->
Linkfeed is a feed abstraction standard built on top of Ceramic’s ComposeDB. This standard enables the generation of link references both within and outside the Ceramic Network. By curating a collection of such links, anyone can construct a feed-like protocol or application. This could cover a range of applications, including social media platforms or any kind of news feed.
An older version was implemented using TileDocument here https://github.com/chainfeeds/linkfeed.

## Abstract
<!--A short (~200 word) description of the technical issue being addressed.-->
This CIP propose a canonical ComposeDB data model for sharing links on the internet, inspired by popular platforms like Reddit, Hacker News, Digg, and [del.icio.us](http://del.icio.us/) / Pinboard. The proposed data model aims to facilitate interoperability among these platforms, enabling seamless sharing and aggregation of link-based content.

The ComposeDB data model incorporates key components such as the URL, title, description, thumbnail/image, tags, metadata.

## Motivation
<!--Motivation is critical for CIPs that want to change the Ceramic protocol. It should clearly explain why the existing protocol specification is inadequate to address the problem that the CIP solves. CIP submissions without sufficient motivation may be rejected outright.-->
Web feed is a common concept. Many types of information can be shaped into this form, which allows for the creation of various applications. The main purpose of this standard is to enhanced interoperability through the use of common data fields and structures, enabling smooth data exchange between platforms. This promotes cross-platform integration, allowing users to share links seamlessly across multiple platforms and facilitating the aggregation of link-based content from diverse sources.


## Specification
<!--The technical specification should describe the syntax and semantics of any new feature.-->
type Linkfeed @createModel(accountRelation: LIST, description: "A feed abstraction standard.") {
  title: String! @string(maxLength: 2000) 
	content: String! @string(maxLength: 100000)
  url: URI! #source_url
  uuid: String! @string(maxLength: 120) # application context specific UUID
  status: String @string(maxLength: 20) # life cycle status of the link, eg. pending, published
  metadata: String @string(maxLength: 100000) # meta field
	tags: [String] # annotation for the link
  version: CommitID! @documentVersion
  createAt: DateTime
  modifiedAt: DateTime
}

## Rationale
<!--The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages. The rationale may also provide evidence of consensus within the community, and should discuss important objections or concerns raised during discussion.-->
By adopting this standardized data model, platforms can achieve consistency in representing shared links, making it easier for users to navigate and interact with content across different platforms. This is achieved by using a canonical ComposeDB data model, so that platforms can unlock the potential for greater collaboration, user engagement, and content discovery. It paves the way for a more connected and interoperable internet landscape, where link-sharing platforms can interact harmoniously while providing users with a unified experience across various services.


## Backwards Compatibility
<!--All CIPs that introduce backwards incompatibilities must include a section describing these incompatibilities and their severity. The CIP must explain how the author proposes to deal with these incompatibilities. CIP submissions without a sufficient backwards compatibility section may be rejected outright.-->
N/A


## Implementation
<!--The implementations must be completed before any CIP is given status "Final", but it need not be completed before the CIP is accepted.-->
A referenece implementation https://component-doc.s3.xyz/.
An older version was implemented using TileDocument here https://github.com/chainfeeds/linkfeed/.


## Security Considerations
<!--All CIPs must contain a section that discusses the security implications/considerations relevant to the proposed change. Include information that might be important for security discussions, surfaces risks and can be used throughout the life cycle of the proposal. E.g. include security-relevant design decisions, concerns, important discussions, implementation-specific guidance and pitfalls, an outline of threats and risks and how they are being addressed. CIP submissions missing the "Security Considerations" section will be rejected. An CIP cannot proceed to status "Final" without a Security Considerations discussion deemed sufficient by the reviewers.-->
N/A


## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
