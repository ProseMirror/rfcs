# Summary

This proposal defines a new, optional, piece of transaction metadata that can be used to trace steps through history undo and redo, or collab rebasing.

# Motivation

The undo history currently strictly concerns itself with the document—undoing a change means undoing the steps that were made as part of that change, and nothing more. But some interfaces don't make such a clear distinction between actions that change the document and actions that change some other piece of state—possibly ProseMirror plugin state, possibly entirely external state—and would like to use the same undo/redo commands to go through document changes and external state changes.

In some cases, changes in external data are tied to document steps (for example, creating a node for which some metadata is tracked outside of the document), in other cases, the changes are entirely independent of the document.

The current prosemirror-history implementation makes it extremely hard to extend the history in this way. This proposal intends to add functionality to make this easier. Similarly, the way prosemirror-collab can re-apply local steps several times can make it hard to keep track of such steps.

# Guide-level explanation

"Tracers" are values that can be associated with a given step to provide additional information about it, and track that data through undoing and redoing the step. They are stored as transaction [metadata](http://prosemirror.net/docs/ref/#state.Transaction.setMeta).

When you create a transaction that contains some traceable steps, you use the `tracers` metadata key to add this information to the transaction. It holds an array of `Tracer` objects, each of which has an index that points at a step in the transaction, a tag that identifies the kind of tracer it is, and optionally an additional value of arbitrary type that provides further information.

Both the `Tracer` class and the `tracers` metadata key are exported from prosemirror-state.

To cover the use case where tracers need to be attached to steps by code that didn't originate those steps, for example when you want undoing the deletion of content that had some metadata associated with it to restore that metadata, plugins support a new hook, `attachTracers`, which is called right after `filterTransaction`, and given a transaction may return an array of additional tracer objects to associate with the transaction.

The undo history stores these tracers along with the steps, and When it creates a transaction that undoes or redoes some steps, it includes the relevant tracers in that transaction. An `event` property on tracers can be used to see what kind of transaction this is—it initially holds `"do"` in the user-created transaction, `"undo"` when the transaction undoes the step, and `"redo"` when it redoes it again.

The collab module, when it rebases local traced steps over remote steps, will first apply the inverse of the steps with an event type of `"rebase-invert"`, and then, after the remote steps (if the step can still be applied) re-apply them with a type of `"rebase-reapply"`.

# Reference-level explanation

This change requires the history module to track a set of tracers along with each step. Tracers are preserved through mapping (and rebasing), so the step that a tracer eventually ends up with is not guaranteed to be identical to the one it started with—they track step origins, but mapping and reordering (through `"addToHistory": false` transactions) might cause the step to change shape.

When a step is dropped because it cannot be mapped through `"addToHistory" false` steps coming after it, the tracer is dropped. This seems like reasonable behavior: if a step can't be undone, it should not be reported as undone.

One tricky part is support for actions that do not have steps associated with them. The current architecture of the history package is very much built around steps, and adding support for actions that aren't associated with steps, while possible, would further complicate an already difficult piece of code. As such, my proposal is to lean on support for custom step types to sidestep this problem something like this:

 - Create a custom subclass of [`Step`](http://prosemirror.net/docs/ref/#transform.Step).

 - Make it do nothing, have an empty [map](http://prosemirror.net/docs/ref/#transform.StepMap), and return itself when mapped—or, if it can be meaningfully associated with some point in the document, it could fail to map through mappings that delete that point.

 - Include such a step in your transaction, and associate the tracer with it.

This is slightly awkward, but works robustly. We can consider including a no-op step type in the prosemirror-transform package to make it easier.

# Drawbacks and alternatives

The clumsiness around step-less action could be one motivation to do this differently.

Arguments can be made for doing this at the granularity of transactions instead of steps, but since the history tracks steps, not transactions, and it is possible for some of the steps in a transaction to be canceled by remapping, that would create a lot of difficult corner cases that are sidestepped by explicitly making the user select steps to track.

I initially considered associating imperative callbacks with steps, which are called for their side effects when the step is undone, but that would appear to be a terrible fit with the rest of our architecture (building a transaction shouldn't have side effects). Putting the metadata in the transactions and having other code react to those (with a plugin state apply method when working inside of the editor state, or `dispatchTransaction` otherwise) nicely avoids those issues.
