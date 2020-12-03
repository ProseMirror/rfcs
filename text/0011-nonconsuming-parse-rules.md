# Summary

Allow parse rules to indicate that even if they apply, other rules for the same tag or style should also get a chance to match.

# Motivation

Sometimes it is useful to look at the entire element when matching a mark. This can be done with a `tag: "*"` rule that then uses `getAtrrs` to look at styles and attributes. But such a rule should probably not prevent other rules from applying to the same element when it matches.

# Explanation

The `ParseRule` interface gets a new property, `consuming`. This defaults to true, but can be explicitly set to false to make the rule non-consuming.

# Drawbacks and alternatives

This is definitely a bit of a kludge to indirectly express something that may be nicer to express directly—that a rule matches only some aspect(s) of an element, and other aspects should be allowed to match other rules. But a more complex declarative language for matching elements seems like it'd require a lot of design work and code weight, so I'm leaning towards just going with the kludge.
