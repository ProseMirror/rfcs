# Summary

Introduce a `replaceTransaction` plugin hook that generalises and provides more
power than `filterTransaction`, and in some cases `appendTransaction`.

# Motivation

Plugins currently lack a mechanism for intercepting a transaction and making
*arbitrary* changes before it's applied. Some specialised APIs (i.e.
`appendTransaction` and `filterTransaction`) exist to offer a level of *safe*
interception, but there are scenarios where these are not powerful-enough.

## Scenario 1 - prepend steps to a transaction to record metadata

One use-case is allowing a plugin to maintain metadata about a range of the
document, and be able to intercept destructive changes (e.g. a delete) and
insert its own custom step at the start of the transaction to stash away the
soon-to-be-lost metadata in the document history. This permits the metadata to
be "restored" via prosemirror-history during an "undo".

As a concrete example, consider an inline commenting plugin that associates a
"comment thread ID" with a range of the document (i.e. {id: string; fromPos:
number; toPos: number}). If a selection enclosing the comment range is deleted,
this RFC provides a mechanism for the plugin to insert a
`CommentDeleteStep({id,fromPos,toPos})` step at the start of the transaction to
capture that information. This step is invertible to
`CommentCreateStep({id,fromPos,toPos})`, and allows the plugin to restore the
comment if the transaction is undone by prosemirror-history.

## Scenario 2 - generalised `filterTransaction`

Another use-case is simply cancelling a transaction. This can be achieved today
via `filterTransaction`, but this could be deprecated in favour of
`replaceTransaction`. It's also more powerful than `filterTransaction` as it
allows a plugin to "remember" that the transaction was cancelled.

Concrete example:

> I wanted to stop cursor selection, and filtering was the easiest way to do it
> rather than trying to reset the selection, but I wanted to update some plugin
> state here.

Source: https://discuss.prosemirror.net/t/tracked-changes-with-strict-document-format/1142/16

## Scenario 3 - rewrite transactions to add marks to insertion/deletion

User johanneswilm wanted to experiment with using marks to track changes to a document:

> Add a mark to all newly added inline content, and when the user tries to
> delete content, prohibit the deletion and instead mark it with a deletion
> mark. Additionally, add marks to paragraphs that the user tried to merge with
> preceding paragraphs, and add “change style” marks to content that has change
> style.

Source: https://discuss.prosemirror.net/t/tracked-changes-with-strict-document-format/1142/11

# Guide-level explanation

> Explain the proposal as if it was already included in the language and you
> were teaching it to a user. That generally means:
>
> - Introducing new named concepts.
> - Explaining the feature largely in terms of examples.
> - Explaining how users should *think* about the feature, and how it should
>   impact the way they use ProseMirror. It should explain the impact as
>   concretely as possible.

```
replaceTransaction(tr: Transaction, newState: EditorState, oldState: EditorState) → ?Transaction
```

`replaceTransaction` is a hook available to plugins that provides control over
if (and how) a transaction is processed. It allows a plugin to prevent a
transaction from being applied (just as `filterTransaction`), but also allows a
transaction to be extended (e.g. with extra steps), or to be replaced entirely
with another.

# Reference-level explanation

> This is the technical portion of the RFC. Explain the design in sufficient detail that:
>
> - Its interaction with other features is clear.
> - It is reasonably clear how the feature would be implemented.
> - Corner cases are dissected by example.

The sequence that plugin hooks are executed when applying a transaction is as
follows:

1. `filterTransaction`
2. `replaceTransaction`
3. `appendTransaction`

For plugins implementing any of these hook, hooks are executed in plugin-order
as per editor state configuration. This means that the first plugin in an editor
state will be the first to have `replaceTransaction` called for a transaction,
and so forth.

It's possible for multiple plugins to implement `replaceTransaction`, and in
such a scenario each is given an opportunity to make a replacement (following
plugin-order). However unlike `appendTransaction`, a plugin implementing
`replaceTransaction` will not see the effects from later plugins'
`replaceTransaction`. This means that a later plugin has the ability to
completely change the effect of an earlier plugin, and the earlier plugin does
not consulted. This design keeps the mental-model simple (a pipeline), and
solves the motivating use-cases.

This design increases the existing significance of plugin ordering, by being
_another_ consequence to consider.

## Sequence of operations

When a transaction is applied to an editor state, the sequence of operations
follows:

1. Call each plugin's `filterTransaction`, if any return `false`, abort.
2. For each plugin implementing `replaceTransaction`, execute the following, in
   the order that the plugins are configured in the editor state.
3. Calculate the new editor state based based on the transaction, without
   calling any plugin hooks (i.e. `EditorState#applyInner`).
4. Call `replaceTransaction` on the plugin, passing in the transaction, the old
   editor state, and the new editor state.

    1. if it returns `false`, abort.
    2. if it returns `undefined`, move onto the next plugin and continue from (4)
    3. if it returns a transaction, move onto the next plugin and continue from
       (3).

5. With the final transaction and new editor state, execute the existing
   `appendTransaction` algorithm. For any transaction appended, perform steps
   1-4.

## Return-value to indicate "abort" vs "continue" vs "replace"

It is desirable to have a way for `replaceTransaction` indicate that no
replacement should occur, i.e. a "continue". Possible options:

- the input transaction (not suitable, as transactions are mutable)
- `false` (not suitable, used to indicate "abort the transaction")
- `undefined` / `void`

Simply not returning a value is the best option for indicating that no
replacement should occur. Conversely returning `false` indicates that the
transaction should be aborted, this follows the convention used by
other ProseMirror APIs, e.g. `filterTransaction`, `Node#nodesBetween`.

To indicate "replace the transaction", this proposal suggests that a transaction
is returned (either the modified input transaction, or an entirely new one).
Unfortunately this interface allows a transaction to be mutated but *not*
returned, which would cause incorrect behavior as further processing of the
transaction takes place. To be specific, a new editor state would not be
calculated, so the transaction and editor state would be out of sync. In
situations like this it would be useful for transactions to be a persistent data
structures, but that is not currently the case.

## Example of `replaceTransaction` inserting a step to preserve information

This example shows how a `CommentPlugin` (as described in the Motivation
section) could track deletions caused by content removal, and store information
about the deleted comment in a custom step at the start of the transaction, so
that it can be restored by prosemirror-history.

```js
replaceTransaction: (tr, oldState, newState) => {
    const { deletedComments } = CommentPlugin.getState(newState);

    if (deletedComments.length > 0) {
        const newTr = oldState.tr;
        for (const deletedComment of deletedComments) {
            newTr.step(new CommentRemoveStep(deletedComment));
        }
        for (const step of tr.steps) {
            newTr.step(step);
        }
        if (tr.selectionSet) {
            newTr.setSelection(tr.selection);
        }
        if (tr.storedMarksSet) {
            newTr.setStoredMarks(tr.storedMarks!);
        }
        return newTr;
    }
}
```

# Drawbacks

> Why should we *not* do this?

- It further complicates the mental model of how transactions are processed. It
  has a different processing sequence to `appendTransaction`, which may cause
  confusion.

# Rationale and alternatives

> - Why is this design the best in the space of possible designs?
> - What other designs have been considered and what is the rationale for not choosing them?
> - What is the impact of not doing this?

# Unresolved questions

> - What parts of the design do you expect to resolve through the RFC process before this gets merged?
> - What parts of the design do you expect to resolve through the implementation of this feature before stabilization?
> - What related issues do you consider out of scope for this RFC that could be addressed in the future independently of the solution that comes out of this RFC?
