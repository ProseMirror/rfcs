# Summary

Allow text selections to be mapped backward through remote changes in the collab module's `receiveTransaction` function.

# Motivation

By default, selection classes' `map` method maps the selection forward, so that content inserted at one of their endpoints ends up before that endpoint. In collaborative editing situations, it turns out to actually be preferable to map backward, so that content remotely inserted at the cursor ends up after the cursor.

# Guide-level explanation

The `receiveTransaction` function, which builds a transaction for remote changes, gets an optional options object as argument, with a `mapSelectionBackward` option that, when explicitly enabled, maps text selections through the changes in a custom way, with an `assoc` argument of -1.

# Rationale and alternatives

It would probably have been better to make backward-mapping the default for selection mapping in general, and make insert-at-selection commands responsible for moving the selection after the inserted content. Unfortunately, changing that would be a subtle, problematic breaking change, so we can't do that, and have to settle for special-casing collaborative updates.
