# Summary

The [`EditorState.create`](http://prosemirror.net/docs/ref/#state.EditorState^create) API is a factory that creates instance of `EditorState`. Currently it allows `schema`, `doc`, `selection`, `plugins`, but is missing `storedMarks`. This RFC proposes allowing `storedMarks` to be passed via `EditorState.create`. 


# Motivation

The motivation for adding this to the `EditorState.create` factory is to avoid violating the persistent datastructure principle, by providing a first-class API for setting stored marks on a new editor state.

The current approach for setting `storedMarks` is:

```js
const editorState = EditorState.create({ doc });
editorState.storedMarks = storedMarks;
```

Ideally this would be changed to:

```js
const editorState = EditorState.create({ doc, storedMarks });
```

# Guide-level explanation

Nothing to explain here, it would work in the same way as other fields passed in `EditorState.create`.

# Reference-level explanation

The implementation is very simple, and is just two changes:

1. Update the [`FieldDesc` for `storedMarks`](https://github.com/ProseMirror/prosemirror-state/blob/cd2b0987d71fee081ec14296aa9308c39a7ea2af/src/state.js#L30) to check `config` for an initial value, falling back to `null`.
2. Update the docs for [`EditorState.create`](https://github.com/ProseMirror/prosemirror-state/blob/cd2b0987d71fee081ec14296aa9308c39a7ea2af/src/state.js#L180) to include `storedMarks`.

# Drawbacks

No drawbacks have been identified.

# Rationale and alternatives

n/a

# Unresolved questions

n/a
