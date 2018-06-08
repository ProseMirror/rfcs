# Summary

Move the view module's internal function `dropPos`, which computes a reasonable place to drop a slice, given a start position in the document, to the transform module and export it.

# Motivation

The drop cursor module, as well as user's custom drop behavior implementations, will often need access to this function, and are now duplicating it. It just got [more complicated](https://github.com/ProseMirror/prosemirror-view/commit/bc52d3dea74fc6c6b17d52e9cf44d0d38ddc8605), and might get more so in the future. Being able to reuse a central implementation would help.

# Guide-level explanation

The function retains its current signature, `dropPos(slice: Slice, $pos: ResolvedPos): number` and is exported from the transform module, from where the view and drop cursor modules import it.

# Drawbacks

This grows the interface of transform a little, and the functionality of this function is quite close to `insertPos`, which already exists.

# Rationale and alternatives

However, transform is already a place with grab-bag tree-structure utilities, and whereas `insertPos` finds a position for a _node_, and only exits parent nodes when the original position was on the parent's boundary, `dropPos` places a _slice_ and exits parents even when starting in the middle of them.

We could try to unifying these two, but their use cases are sufficiently different (one is meant for node-specific insert commands, the other for drag-and-drop), that may only make things more confusing.
