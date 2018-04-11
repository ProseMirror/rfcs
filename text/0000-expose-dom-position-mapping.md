# Summary

This proposal adds methods to `EditorView` instances to map between DOM positions and ProseMirror document positions, and to get the DOM node that represents a given document node in the view.

# Motivation

It is occasionally useful to figure out where a DOM event happened from its `target` property, or to measure the size of a document element in the DOM. Though ProseMirror can internally do this, it is not currently possible for external code to use this mechanism.

# Guide-level explanation

The proposal adds three new methods to the `EditorView` class:

 - **`posAtDOM`**`(domPos: {node: dom.Node, offset: number}) → number`\
   This method maps from a DOM position to a document position. It will raise an error when the given DOM node isn't inside of the editor.

 - **`domAtPos`**`(pos: number) → {node: dom.Node, offset: number}`\
   Returns the DOM position associated with a document position.

 - **`nodeDOM`**`(pos: number) → dom.Node`\
   Returns the DOM node that represents the document node after the given position. Will raise an error if the position isn't directly in front of a node.

# Reference-level explanation

As mentioned, this information is already stored in the view descriptions, so it can be retrieved with little extra code.

# Drawbacks

The reason this doesn't exist yet is that, to people used to traditional JavaScript programming, it is basically an open invitation to start messing with the DOM in ways that this library does not support (such as styling elements by directly mutating their DOM representation). Things will then break, and they will come complaining in the forum or bug tracker.

We can add a big warning about this in the documentation. If we're lucky, that might reduce the number of support requests this'll generate a little bit.

# Rationale and alternatives

The main alternative is to continue as before and say that this is too dangerous for client code to use. But there exist valid uses, and this is the most straightforward way to support them.
