---
cip: 1
title: Ceramic Improvement Proposals (CIP)
author: Michael Sena (@michaelsena), Joel Thorstensson (@oed), Daniel Zuckerman (@dazuck)
discussions-to: https://github.com/ceramicnetwork/CIP/issues/1
status: Living
category: Meta
created: 2020-05-22
edited: 2023-05-24
---
  
## Simple Summary

This proposal describes the process for Ceramic Improvement Proposals (CIPs).


## Abstract

Ceramic Improvement Proposals (CIPs) are standards for improving the Ceramic platform, including core protocol specifications, client APIs, streamtypes, and document standards. CIPs allow anyone to contribute to the development of the Ceramic Network and standards built on top of it. As a result, CIPs allow the Ceramic Network to thrive as an open source project and protocol.


## Motivation

Here are some of the reasons why Ceramic needs a CIP process:
- To define an explicit governance process for Ceramic network upgrades
- To make it easy for community members to contribute to the development of the protocol
- To provide a way to discover implementation standards accepted by the community
- To provide a vehicle for community discussion around important topics
- To remove any individual or group of individuals as gatekeepers to the Ceramic Network


## Specification

### Process

#### 1. Discuss a new idea

> Ideas are incomplete proposals meant to initiate community conversation.

If you have an idea about an improvement you would like to make to Ceramic, but you are not ready to write a full proposal, you can create a post on the [Ceramic forum](https://forum.ceramic.network). There you can discuss and validate your idea before making it into a CIP.


#### 2. Create your draft

> Drafts are complete proposals, but still undergoing rapid iteration and change.

Create a draft by making a fork of this repository and making a copy of the [CIP template](https://github.com/ceramicnetwork/CIPs/blob/main/cip-template.md). Fill out all appropriate fields in the CIP template and make a PR to this repository.

Upon submission an editor will manually review the first PR for a new CIP, assign it a canonical number (i.e. CIP-1), and make sure it is formatted correctly according to the [CIP template](https://github.com/ceramicnetwork/CIPs/blob/main/cip-template.md). If everything looks correct your CIP will be merged into the repo.

*When submitting your Pull Request:*

- *Images and other assets*: If your CIP requires images or other assets, the files should be included in the `assets` folder in the root of this repo. You should create a subdirectory of your CIP as follows: `assets/cip-N` (where **N** is to be replaced with the CIP number). When linking to an image in the CIP, use relative links such as `../assets/cip-N/image.png`.

- Tables: If your CIP introduces a csv table that needs to be updated by other CIPs, the table csv files should be included in the `tables` folder in the root of this repo. When linking to a table in the CIP, use relative links such as `../tables/my-table.csv`.

At any time during the review stage you can submit additional PRs to update your CIP. If a CIP is updated by anyone else than any of the authors of the CIP, at least one of the original authors need to approve the PR.

#### 3. Move your CIP to review

> The Review status is an indication to the community that the CIP is ready for final review.

Simply submit a new PR to update your CIP status to `Review`.

#### 4. Finalize your CIP

> Only non-normative changes are allowed on CIPs in Final status.

If a CIP has been in `Review` stage for over 14 days you can submit a new PR to update your CIP status to `Final`.

A CIP within the category of `Core`, `Networking`, or `Interface` will need to be implemented in a protocol client before moved to Final.

### CIP Terms

#### Categories

- `Meta`: an CIP that affects the governance process for CIPs.
- `Core`: an CIP that affects the core protocol.
- `Networking`: an CIP thst affects the networking layer (i.e. libp2p or syncing).
- `Interface`: an CIP that affects the Ceramic API or other interfaces.
- `RFC`: an CIP that proposes an implementation standard (i.e. streamtypes, document configurations, or document schemas).


#### Statuses

- `Draft`: The first stage of a CIP. A CIP that is undergoing rapid iteration and changes.
- `Review`: A CIP that is stable and ready for final review by the community.
- `Final`: This CIP represents the final standard. A Final CIP exists in a state of finality and should only be updated to correct errata and add non-normative clarifications.
- `Superseded`: A CIP that has been rendered obsolete by a later CIP.
- `Withdrawn`: The CIP Author(s) have withdrawn the proposed CIP. This state has finality and can no longer be resurrected using this CIP number. If the idea is pursued at later date it is considered a new proposal.
- `Living`: a special status for CIPs that are designed to be continually updated and not reach a state of finality. This includes most notably CIP-1.



## Rationale

It is prudent to look at other successful examples of open source software governance. CIPs are inspired by the previous work of Ethereum (EIPs), Bitcoin (BIPs), Python (PIPs), and others. CIP is slightly different in implementation than those examples, but shares many similarities. In some cases, CIP even borrows certain language from those examples.


## Implementation

- **Official Process**: This CIP and the [README](https://github.com/ceramicnetwork/CIP) of the CIP repository.


## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
