# Summary

Expose the outgoing edges on `ContentMatch` objects, which are in effect just nodes in a finite automaton.

# Motivation

There is currently no way for code outside of the prosemirror-model module to work with the shape of the content automaton. This can be useful when, for example, figuring out what the default node type at a given position is (the first edge, if any).

# Guide-level explanation

When constructing a schema, the content expressions of all nodes are compiled down to finite automata, consisting of nodes (a position in the expression) that are connected with edges (valid transitions from that node). These edges, along with the flag that indicates whether a position is a valid end position, are the defining information of `ContentMatch` instances. They are currently not directly accessible the user code.

This proposal gives the `ContentMatch` class a new getter and a new method:

**`edgeCount`**`: number` gives you the number of out-edges this node has.

**`edge`**`(n) → {type: NodeType, next: ContentMatch}` is used to retrieve the label (`type`) and destination (`next`) of a given edge in the automaton.

# Reference-level explanation

The proposed properties are very simple accessors on top of existing data. They don't appear to cause any further complication.

# Drawbacks

This adds two properties to an interface, properties that most users won't need to use, so there's a slight complexity cost.

# Rationale and alternatives

The main alternative would be to provide special-cased methods and accessors to try and cover all use cases in which this information is useful. For complicated queries like `findWrapping`, having specialized methods seems like a good idea. However, since the underlying data structure is unlikely to change, it seems reasonable to allow user code to directly work with it.

For example the prosemirror-schema-list module needs to figure out the default node at a given position. We could make the private `defaultType` method public, but that only covers that specific case. A more general interface is going to cover more situations.
