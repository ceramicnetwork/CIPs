![Readme Banner](assets/general/readme-banner.png)
# Ceramic Improvement Proposals (CIPs)
[![](https://img.shields.io/badge/Chat%20on-Discord-orange.svg?style=flat)](https://discord.gg/5CZzG7V)

Ceramic Improvement Proposals (CIPs) are standards for the Ceramic platform, including core protocol specifications, client APIs, doctypes, and document standards. This document describes the end-to-end process for creating a CIP. This process was initially defined in [CIP-1](https://github.com/ceramicnetwork/CIP/blob/master/CIPs/CIP-1/CIP-1.md).

**Want to explore merged CIPs?** View the index of all [merged CIPs](./INDEX.md).


# Contributing

## 1. Propose a new idea

> Ideas are incomplete proposals meant to initiate community conversation.

Create a new issue in the CIP repository containing the idea for your your proposal, following the [CIP template](https://github.com/ceramicnetwork/CIP/issues/new?assignees=&labels=&template=new-cip.md&title=) format. Be sure to add the **Status: IDEA** label to your issue.

## 2. Complete your draft

> Drafts are complete proposals, but still undergoing rapid iteration and change.

Complete your proposal by filling out all appropriate fields in the CIP template. Update your proposal to **Status: DRAFT** in the issue header and Github label. Gather community feedback and make improvements as required.

## 3. Enter last call for community feedback

> Last Calls are stable proposals ready for final review by the community.

Once you feel that your proposal is stable and ready for final review by the community, update your proposal to **Status: LAST CALL** in the issue header and Github label. In order to proceed beyond Last Call, your issue must remain in Last Call for at least 2 weeks and any technical changes that are requested must be addressed by the author.

## 4. Submit a Pull Request

> Pull Requests are CIPs ready for consideration by editors and/or core devs.

When your proposal exits Last Call, you should submit a Pull Request to the CIP repository. To do this, fork the CIP repository by clicking "Fork" in the top right. Add your CIP to your fork of the repository. Update your proposal to **Status: PENDING** in the issue header and Github label (on both your previous open issue and new PR). Then, submit a Pull Request to the CIP repository.

Upon submission an editor will manually review the first PR for a new CIP, assign it a canonical number (i.e. CIP-1), and merge it into the CIP repo. They will then reach out to discuss next steps to achieve finalization. Thesse steps will depend on whether or not your CIP is of type Core.

*When submitting your Pull Request:*

- Images: If your CIP requires images, the image files should be included in a subdirectory of your CIP folder as follows: `CIP-N/assets` (where **N** is to be replaced with the CIP number). When linking to an image in the CIP, use relative links such as `./assets/image.png`.
- Tables: If your CIP requires csv tables, the table csv files should be included in a subdirectory of your CIP folder as follows: `CIP-N/tables` (where **N** is to be replaced with the CIP number). When linking to a table in the CIP, use relative links such as `./tables/table.csv`.
- Add your CIP to the [`INDEX.md`](./INDEX.md) file

## 5. Get your CIP finalized

> Finalized CIPs are CIPs that are have been approved.

### 5a. If your CIP is a Core CIP

An editor will reach out to provide the dates of the upcoming Ceramic Core Devs calls. Once you select one, your issue will be added to the agenda for that call where it will be discussed for inclusion in a future network upgrade. 

If implementers agree to include your CIP in a future network upgrade, CIP editors will update your CIP to **Status: ACCEPTED**. Once your proposal has been released in a network upgrade, CIP editors will update your CIP to **Status: FINAL**.

If implementers decide not to include your CIP in a future network upgrade for whatever reason, you are always able to propose it again at a later time. It will remain as **Status: PENDING** in the CIP repo.

### 5b. If your CIP is a non-Core CIP

An editor will ask if anyone objects to it being finalized. If the editor decides there is no rough consensus - for instance, because contributors point out significant issues with the CIP - they may close the PR and request that you fix the issues in the draft before trying again. If the editor finds there is rough consensus, they will merge the PR and update it to **Status: FINAL**.

# Terms

## Categories

- `Standards`: an CIP that affects the protocol or is an implementation standard.
- `Meta`: an CIP that affects the governance process for CIPs.
- `Informational`: an CIP that is merely for informational purposes but requires no action by the community, and will not be merged as a CIP.

## Types

> Only applicable to CIPs of type *Standards*.

- `Core`: an CIP that affects the core protocol.
- `Networking`: an CIP thst affects the networking layer (i.e. libp2p or syncing).
- `Interface`: an CIP that affects the Ceramic API or provider interface.
- `RFC`: an CIP that proposes an implementation standard (i.e. doctypes, document configurations, or document schemas).

## Statuses

- `Idea`: an CIP issue that is incomplete.
- `Draft`: an CIP issue that is undergoing rapid iteration and changes.
- `Last Call`: an CIP issue that is stable and ready for final review by the community.
- `Pending`: an CIP that has been merged but not finalized.
- `Accepted (Core)`: an CIP of type Core that has been accepted by the core devs to be included in a future network upgrade.
- `Final (Core)`: an CIP of type Core that has already been released in a network upgrade.
- `Final (non-Core)`: a non-core CIP that has met all criteria and is finished.

# Editors
- Michael Sena ([@michaelsena](http://github.com/michaelsena))
- Joel Thorstensson ([@oed](http://github.com/oed))
- Janko Simonovic ([@simonovic86](http://github.com/simonovic86))
- Pedro Gomes ([@pedrouid](http://github.com/pedrouid))
