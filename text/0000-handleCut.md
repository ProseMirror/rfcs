# Summary

Add a handleCut editor property to EditorView to allow for custom cut behavior.

# Motivation

The handleCut property would allow others to apply custom behavior to cut events while leveraging prosemirror's built in cross browser support for the clipboard api. One use case is for not deleting the selected content on cut but instead adding a mark to the content. For example: tracking changes made to a document by striking a line through deleted content instead of removing it.

A handleCut property would allow a custom handler to apply any transactions to the document instead of the default transaction done by prosemirror.

It is currently possible to use a handleDOMEvents.cut handler to override the default prosemirror behavior but it requires the user to write their own cross-browser clipboard support.

# Guide-level explanation

A new property would be added to EditorProps named handleCut. It would behave similarly to other handle properties (handlePaste, handleDrop, etc). If the handleCut function returns true then prosemirror's default behavior is prevented. The handleCut function can carry out its own transaction instead of prosemirror's default behavior of deleting the current selection.

An example of using the handleCut property:
```
new EditorView(dom, {
	handleCut: function (view, event) {
		// Dispatch custom transaction
		...

		// Prevent default behavior
		return true;
	}
})
```

# Reference-level explanation

The handleCut property would use the same code pattern as other handle properties (handlePaste, handleDrop, etc). Because it would leverage common EditorProp functions it should only require one code change in prosemirror-view's input.js file.

```
handlers.copy = editHandlers.cut = (view, e) => {
	...
	if (cut && !view.someProp("handleCut", f => f(view, e))) view.dispatch(view.state.tr.deleteSelection().scrollIntoView())
}
```

# Drawbacks

There are little to no drawbacks. The code change would be a minor addition that makes use of a common pattern in the library. Prosemirror would still behave the same for users who do not make use of handleCut. Documentation would need to be written for the new property.

# Rationale and alternatives

Adding a handleCut property is the best way to solve custom cut behavior. The alternative requires the user to implement their own cross browser clipboard support or to duplicate the code already in the prosemirror library. Copying code in the library would then require the user to keep an eye on future updates so that they could update their copied code. This is prone to error and is a case of reinventing the wheel.

# Unresolved questions

The handleCut function would have at least two parameters. The EditorView and the event object. Beyond that it is not clear what would be useful to pass to the handleCut function. Perhaps the clipboard data.
