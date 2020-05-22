# Ceramic Improvement Proposals (CIPs)

Ceramic Improvement Proposals (CIPs) describe standards for the Ceramic platform, including core protocol specifications, client APIs, and document standards.

## Contributing

1. Review CIP-1.
2. Fork the repository by clicking "Fork" in the top right.
3. Add your CIP to your fork of the repository. There is a template CIP here.
4. Submit a Pull Request to Ceramic's CIPs repository.

Your first PR should be a first draft of the final CIP. An editor will manually review the first PR for a new CIP and assign it a number before merging it. Make sure you include a discussions-to header with the URL to a discussion forum or open GitHub issue where people can discuss the CIP as a whole.

If your CIP requires images, the image files should be included in a subdirectory of the assets folder for that CIP as follows: `assets/cip-N` (where **N** is to be replaced with the CIP number). When linking to an image in the CIP, use relative links such as `../assets/cip-1/image.png`.

Make sure that the 'author' line of your CIP contains either your GitHub username or your email address. If you use your email address, that address must be the one publicly shown on your GitHub profile.

When you believe your CIP is mature and ready to progress past the draft phase, you should do one of two things:

- **For a Standards Track CIP of type Core**, ask to have your issue added to the agenda of an upcoming Ceramic Core Devs meeting, where it can be discussed for inclusion in a future hard fork. If implementers agree to include it, the CIP editors will update the state of your CIP to 'Accepted'.
- **For all other CIPs**, open a PR changing the state of your CIP to 'Final'. An editor will review your draft and ask if anyone objects to its being finalised. If the editor decides there is no rough consensus - for instance, because contributors point out significant issues with the CIP - they may close the PR and request that you fix the issues in the draft before trying again.

## CIP Status Terms

- **Draft** - an CIP that is undergoing rapid iteration and changes.
- **Last Call** - an CIP that is done with its initial iteration and ready for review by a wide audience.
- **Accepted** - a core CIP that has been in Last Call for at least 2 weeks and any technical changes that were requested have been addressed by the author. The process for Core Devs to decide whether to encode an CIP into their clients as part of a hard fork is not part of the CIP process. If such a decision is made, the CIP will move to final.
- **Final (non-Core)** - an CIP that has been in Last Call for at least 2 weeks and any technical changes that were requested have been addressed by the author.
- **Final (Core)** - an CIP that the Core Devs have decided to implement and release in a future hard fork or has already been released in a hard fork.
