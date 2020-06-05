# Ceramic Improvement Proposals (CIPs)

Ceramic Improvement Proposals (CIPs) describe standards for the Ceramic platform, including core protocol specifications, client APIs, and document standards. This document describes the end-to-end process for contributors to take their CIP from idea to finalization. This process was initially defined in [CIP-1]().

# Contributing

## 1. Propose a new idea

> Ideas are incomplete proposals meant to initiate community conversation.

Create a new issue in the CIP repository containing the idea for your your proposal, following the [CIP template](https://github.com/ceramicnetwork/CIP/blob/master/.github/ISSUE_TEMPLATE/cip-template.md) format. Add the appropriate [type](#cip-types) and [category](#cip-categories) labels to your issue. Designate your proposal as **IDEA** status in the issue label and header. Begin seeking feedback from the community.

## 2. Complete your draft

> Drafts are complete proposals, but still undergoing rapid iteration and change.

Complete your proposal by filling out all appropriate fields in the CIP template. Update your proposal to **DRAFT** status in the issue label and header. Gather community feedback and make improvements as required.

## 3. Enter last call for community feedback

> Last Calls are stable proposals ready for final review by the community.

Once you feel that your proposal is stable and ready for final review by the community, update your proposal to **LAST CALL** status in the issue label and header. In order to proceed to the next step, your issue must remain in Last Call for at least 2 weeks and any technical changes that are requested must be addressed by the author.

## 4. Submit a PR for consideration by CIP editors

> PRs are stable proposals ready for consideration by CIP editors.

When your proposal exits Last Call, you should submit a Pull Request to the CIP repository so EIP editors can review your proposal. To do this, fork the CIP repository by clicking "Fork" in the top right. Add your CIP to your fork of the repository.
Update your proposal to **PENDING** status in the header. Then, submit a Pull Request to the CIP repository.

Upon submission an editor will reach out to discuss next steps. These will depend on whether or not your CIP is of type Core.

*Remember:*

- Your PR should be a complete first draft of the final CIP. An editor will manually review the first PR for a new CIP and assign it a canonical number (i.e. CIP-1). 
- When submitting your PR, make sure to include a discussions-to header with the URL to the open Github issue in the CIP repository where people can discuss your CIP. Also ensure the 'author' line of your CIP contains either your GitHub username or your email address. If you use your email address, that address must be the one publicly shown on your GitHub profile.
- If your CIP requires images, the image files should be included in a subdirectory of the assets folder for that CIP as follows: `assets/cip-N` (where **N** is to be replaced with the CIP number). When linking to an image in the CIP, use relative links such as `../assets/cip-1/image.png`.


### 4a. If your proposal is a Core CIP

A CIP editor will reach out on your Pull Request and provide the dates of upcoming Core Devs calls. Once you select one, your issue will be added to the agenda for that call where it will be discussed for inclusion in a future network upgrade. 

If implementers agree to include your CIP in a future network upgrade, CIP editors will update the status of your CIP to **ACCEPTED** and merge the PR. Once your proposal has been released in a network upgrade, CIP editors will update the status of your CIP to **FINAL**.

### 4b. If your proposal is a non-Core CIP

A CIP editor will review your Pull Request

In your PR, you should update the status of your proposal to 'Final'. An editor will review your draft and ask if anyone objects to its being finalised. If the editor decides there is no rough consensus - for instance, because contributors point out significant issues with the CIP - they may close the PR and request that you fix the issues in the draft before trying again. If the editor finds there is rough consensus, they will merge the PR.

# CIP Labels

## CIP Types

- `Standards`:
- `Meta`:
- `Informational`:

## CIP Categories

> Only applicable to CIPs that are *Standards* type.

- `Core`:
- `Networking`:
- `Interface`:
- `RFC`:

## CIP Statuses

- `Idea`: an CIP issue that is incomplete.
- `Draft`: an CIP issue that is undergoing rapid iteration and changes.
- `Last Call`: an CIP issue that is done with its initial iteration and ready for review by a wide audience.
- `Submitted`: a new Pull Request that is awaiting review by the editors.
- `Accepted (Core)`: a Pull Request of type Core that has been accepted by the core devs and will be included in a future network upgrade.
- `Final (Core)`: a Pull Request of type Core that has already been released in a network upgrade.
- `Final (non-Core)`: a non-Core Pull Request that has been in Last Call for at least 2 weeks and any technical changes that were requested have been addressed by the author.

