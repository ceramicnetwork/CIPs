# Ceramic Improvement Proposals (CIPs)

Ceramic Improvement Proposals (CIPs) describe standards for the Ceramic platform, including core protocol specifications, client APIs, and document standards.

## Contributing

### 1. Create a new proposal idea

> Ideas are incomplete proposals meant to kick-off community conversation.

1. Create a new issue in the CIP repository containing the idea for your your proposal, following the [CIP template](https://github.com/ceramicnetwork/CIP/blob/master/.github/ISSUE_TEMPLATE/cip-template.md) format. 
2. Add the appropriate [type](#cip-types) and [category](#cip-categories) labels to your issue.
3. Designate your proposal as **IDEA** status in the issue label and header.
4. Begin seeking feedback from the community.

> Note: During the idea period, not every CIP detail needs to be completed. However your issue should contain enough information for the community to begin commenting.

### 2.  Complete your proposal draft

> Drafts are complete proposals, but still undergoing rapid iteration and change.

1. Complete your proposal by filling out all the appropriate fields in the CIP template.
2. Update your proposal to **DRAFT** status in the issue label and header.
3. Gather community feedback and make improvements as required.

### 3. Move your proposal to last call

> Last calls are proposals done with their initial iteration and ready for review by a wider audience.

1. Update your proposal to **LAST CALL** status in the issue label and header.
2. Make updates based on community feedback.
2. Once the 2-week period is over and all community concerns have been addressed, continue to step 4.

### 4. Submit a PR for final consideration

> PRs are complete first drafts of the final CIP.

Once your proposal draft is complete and has taken all community feedback into consideration, you should submit a Pull Request to the CIP repository containing your proposal.

1. Fork the CIP repository by clicking "Fork" in the top right.
2. Add your CIP to your fork of the repository.
3. Submit a Pull Request to the CIP repository.
4. Add the appropriate issue labels.

> Your PR should be a complete first draft of the final CIP. An editor will manually review the first PR for a new CIP and assign it a canonical number (i.e. CIP-1). 

> When submitting your PR, make sure to include a discussions-to header with the URL to the open Github issue in the CIP repository where people can discuss your CIP. Also ensure the 'author' line of your CIP contains either your GitHub username or your email address. If you use your email address, that address must be the one publicly shown on your GitHub profile.

> If your CIP requires images, the image files should be included in a subdirectory of the assets folder for that CIP as follows: `assets/cip-N` (where **N** is to be replaced with the CIP number). When linking to an image in the CIP, use relative links such as `../assets/cip-1/image.png`.


### 5a. Get your non-Core CIP Finalized

When you believe your CIP PR is mature, you should open a PR changing the state of your CIP to 'Final'. An editor will review your draft and ask if anyone objects to its being finalised. If the editor decides there is no rough consensus - for instance, because contributors point out significant issues with the CIP - they may close the PR and request that you fix the issues in the draft before trying again.

### 5b. Get your Core CIP Finalized

When you believe your CIP PR is mature, you should ask to have your issue added to the agenda of an upcoming Ceramic Core Devs meeting ([calendar]()), where it can be discussed for inclusion in a future network upgrade. If implementers agree to include it, the CIP editors will update the state of your CIP to 'Accepted'.



## CIP Labels

### CIP Types

- `Standards`:
- `Meta`:
- `Informational`:

### CIP Categories
> Only applicable to CIPs that are *Standards* type.

- `Core`:
- `Networking`:
- `Interface`:
- `RFC`:

### CIP Statuses

- `Idea`: an CIP that is incomplete.
- `Draft`: an CIP that is undergoing rapid iteration and changes.
- `Last Call`: an CIP that is done with its initial iteration and ready for review by a wide audience.
- `Accepted`: a core CIP that has been in Last Call for at least 2 weeks and any technical changes that were requested have been addressed by the author. The process for Core Devs to decide whether to encode an CIP into their clients as part of a network upgrade is not part of the CIP process. If such a decision is made, the CIP will move to final.
- `Final (non-Core)`: an CIP that has been in Last Call for at least 2 weeks and any technical changes that were requested have been addressed by the author.
- `Final (Core)`: an CIP that the Core Devs have decided to implement and release in a future network upgrade or has already been released.
