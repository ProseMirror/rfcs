# Summary

Add a `posAtIndex(index, depth?)` method to `ResolvedPos`, which computes the position after a given index at a given depth. Like with other `ResolvePos` methods, `depth` would default to the resolved position's depth, but can be used to select a parent node to index instead.

# Motivation

Going from an index to a position is a rather common use case which is not currently served by any of the `Node`/`Fragment`/`ResolvePos` methods. Code that needs this has to manually iterate over children to sum their sizes. With this method, that'd become a single function call.

# Drawbacks and alternatives

We could also put a method for this purpose on `Fragment` instead. But I expect that in most use cases, the user will have a resolved position available or easily accessible anyway. They could do something like `$pos.start() + $pos.parent.content.indexOffset($pos.index() - 2)`, but `$pos.posAtIndex($pos.index() - 2)` seems nicer.
