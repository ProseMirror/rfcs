# Summary

An RFC process for ProseMirror.

# Motivation

The RFC process is intended to make technical decisions by the core team more transparent, and to provide a way for the community to propose and get involved in the evolution of the library.

# Guide-level explanation

To propose a significant change to ProseMirror under this process, anybody (including core team members) has to first create an RFC, which is a pull request in the https://github.com/ProseMirror/rfcs repository. Such a pull request adds a file describing the proposed feature or change. The community has at least one week to respond to it an give feedback before it is accepted (if at all).

# Reference-level explanation

The RFC repository provides a template (on which this file is also based) listing the necessary sections in an RFC. To propose an RFC, this template is copied to a file under `text/` in the form `0000-my-title.md`, where 'my-title' is some recognizeable title for the proposal, and the RFC number will be filled in when the proposal is accepted (merged).

The GitHub pull request is then used for discussion and further revision of the RFC. The actual RFC file may be updated as often as necessary in response to feedback.

The core team may reject or accept (after at least a week, and preferably not when there is still active discussion going on) an RFC by closing or merging the pull request. The repository acts as a log of accepted RFCs. Accepting an RFC does not imply a promise to actually implement it—getting an actual implementation off the ground is a separate issue.

# Drawbacks

This will put quite a bureaucratic drag on development of new features—instead of just implementing them, an RFC has to be drafted and there is a waiting period of at least a week.

Inviting people to discuss things may lead to a lot of energy sunk into discussions that may not always be all that productive.

# Rationale and alternatives

Compared to not having an RFC process at all, this process will hopefully help give the community a voice before changes are decided on, which should help avoid situations where changes create an unexpected problem for some users, or where the person who came up with the change just didn't really think things through.

The process here is intentionally left relatively informal—only a single week of review time, beyond which closing or accepting RFCs is entirely at the discretion of the core team. This should prevent it from being too burdensome, while hopefully still fulfilling its purposes.

# Unresolved questions

I suspect we'll have to refine this through experience, and if the community grows a lot a more formal approach might be appropriate.
