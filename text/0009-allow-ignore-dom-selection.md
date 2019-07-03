# Summary

Allow node views to opt into ignoring selection changes within them.

# Motivation

If you have, for example, an inner editor within a node view, changes to the selection inside of that will fire ProseMirror's `selectionchange` event handler, causing it to sync up its selection with the new DOM selection. When the DOM selection is inside the nested editor, ProseMirror won't be able to actually represent such a selection, and will usually just end up creating a node selection for it, instead of preserving the selection is was last given.

# Explanation

We already have an `ignoreMutation` mechanism that node views can use to tell the editor not not react to certain DOM changes. This RFC proposes to use that same mechanism in for this case, passing it an object with `type` property holding `"selection"`, instead of one of the mutation types produced by the native mutation observer, and a `target` property pointing at the selection's nearest parent node.

This way, code that blanket-blocks mutations will automatically do the right thing for selection changes, but code that wants to handle them specially can check for the `type` property and apply custom logic.

# Drawbacks and alternatives

This _is_ a slightly dodgy way to overloading the concept of DOM mutations, adding something that's not considered a DOM mutation by the standard. We could also add a separate method for this. But that'd require more work for every node view implementation, even though the way to handle these is usually the same.
