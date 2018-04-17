# Summary

Allow [widget decorations](##view.Decoration^widget) to be rendered when they are drawn in an actual view, rather than requiring their DOM structur to be created when the decoration is created.

# Motivation

There is a bit of a mismatch between the structure of the decoration data structure (immutable, declarative) and DOM nodes (mutable, can't be shared because they can only exist in one place). Decoration data is in principle view-independent and shareable—except when there are widgets with their own DOM nodes in them. Those can only be drawn in one view at a time, and may be affected by side effects.

In addition to that, widgets often need access to the view, in order to be able to dispatch new transactions when they are interacted with. This is currently possible through kludges that pass a state accessor function and a dispatch function to the plugin constructor, which then passes them through to the code that creates the event handler.

If we move the creation of the widgets' DOM to the view layer instead, we can make the view object available to them when they are constructed, and store them in the view's own data structures, rather than in the decorations themselves.

This proposal adds support for that, by allowing widget decorations to specify a DOM constructor function instead of an existing DOM node. Passing a DOM node is still supported, for compatibility reasons.

# Guide-level explanation

The second argument to [`Decoration.widget`](https://prosemirror.net/docs/ref/#view.Decoration.widget) is extended to allow either a DOM node (the current type) or a function with a signature like `(view: EditorView, getPos: () → number) → dom.Node`. When a function is given, it is only called when the widget is actually shown in a view, passing it both the view instance and a function that it can use to find its current location (which is often useful when implementing controls that do something to the surrounding document nodes).

# Reference-level explanation

The canonical storage of the DOM node for a given widget is now stored in the view description for the widget, rather than in the decoration object. The latter just holds a value from which the DOM can be derived.

When comparing widget decorations during updates, to figure out if they need to be redrawn, the node constructor functions are compared (along with the rest of the properties in the decoration spec).

# Drawbacks

It further complicates an already somewhat confusing interface. But the way it can be used to avoid the kludges that are currently needed to dispatch transactions from widgets probably reduces the overall annoyance caused by the interface.
