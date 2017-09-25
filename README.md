# ProseMirror RFCs

When making substantial changes to the ProseMirror core modules, we
use “request for comments” workflow to formalize the design process
and allow the community to join the conversation.

A substantial change may be one that introduces a new feature,
deprecates something, or is otherwise complex enough to warrant some
attention from the community.

This repository serves as a way to propose such RFCs, and as a log of
accepted RFCs, which may be valuable for later reference.

## Process

To propose an RFC, copy `0000-template.md` to `text/0000-my-title.md`
and fill it in. Put care into the details, and make sure you deeply
understand the parts of the system that your feature is interacting
with.

Submit a pull request that adds your new file. This is where
discussion around the RFC will happen. Be prepared to defend and/or
revise your proposal in response to feedback.

When the community has had some time (at least two weeks) to comment
on the RFC and no more blocking problems are present, the core team
may decide to accept it and merge it into the repository. Accepting an
RFC does not imply a promise to actually implement it—ideally, the
person who submits the RFC follows up with an implementation
themselves (though this is not required, and in some cases someone
else will pick it up).

It is also possible for the RFC to be rejected. Low quality proposals
or things that have been discussed before may be rejected right away.
But even proposals with merit may end up being rejected, for example
if it is considered out of scope, or introduces problems that can't be
resolved in the discussion phase.
