# Summary

Allow plugins to be passed directly to the view, without storing them in the state.

# Motivation

It can be [useful](https://discuss.prosemirror.net/t/separating-state-and-view-plugins/3970) for view plugins to have access to values that are not yet available when the state is created. For example, the editor state might go in some kind of app-wide store, and a key binding might need access to that store.

# Explanation

The `DirectEditorProps` interface gets a new property `plugins`, which may hold an array of `Plugin` instances.

The view dynamically checks whether none of the plugins provided here have a `state`, `filterTransaction`, or `appendTransaction` field, because passing state-affecting plugins to the view is probably an error.

When looking for props with `someProp`, the plugins passed directly are checked as well, as if they come before the state plugins, but after props passed directly.

Plugin views from plugins passed directly to the view are created in the same way as those from plugins passed via the state.

# Drawbacks and alternatives

This approach has the downside of using one type (`Plugin`) for plugins that need to be part of the state and plugins that can be directly passed to the view. Having separate type for state plugins, as a subtype of view plugins, this would be useful during type checking. But this seems hard to do without a breaking change—either existing code that exposes a state plugin would have to change, _or_ existing code that exposes a plugin that can safely be used as a view plugin would have to change. By using a single type, modules like `keymap` can just note in their docs that they can be used as view plugins, without actually changing anything about their interface.
