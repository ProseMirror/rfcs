# Summary

Provide the ability to use block marks for positioning UI elements. Please read the discussion here: https://discuss.prosemirror.net/t/block-marks-wrapping-behaviour/1718/3

# Motivation

Currently, there's an optimisation step in `prosemirror-view` that allows ProseMirror to reuse existing mark views if two marks applied on adjacent block nodes are identical. Here's an example of ProseMirror document:

```
{
  "version": 1,
  "type": "doc",
  "content": [
    {
      "type": "p",
      "content": [
        {
          "type": "text",
          "text": "text"
        }
      ],
      "marks": [
        {
          "type": "breakout",
          "attrs": {
            "mode": "wide"
          }
        }
      ]
    },
    {
      "type": "p",
      "content": [
        {
          "type": "text",
          "text": "more text"
        }
      ],
      "marks": [
        {
          "type": "breakout",
          "attrs": {
            "mode": "wide"
          }
        }
      ]
    }
  ]
}
```

Resulting html:

```
<div class="breakout_mark">
  <p>text</p>
  <p>more text</p>
</div>
```

This behaviour makes it hard to vertically position UI elements relatively to the second paragraph as the mark DOM node wraps both paragraphs.

The suggestion is to control the wrapping behaviour so that we can have the following DOM structure:

```
<div class="breakout_mark">
  <p>text</p>
</div>
<div class="breakout_mark">
  <p>more text</p>
</div>
```

# Guide-level explanation

One way to accomplish this would be to provide the ability to control the wrapping behaviour of block marks from nodeViews.

The proposal gives the `NodeView` class a new getter:

**`disableWrapping`**`: boolean` indicates whether `prosemirror-view` uses the wrapping optimisation.

Implementation details: https://github.com/ProseMirror/prosemirror-view/pull/43

# Drawbacks

When this feature is used, ProseMirror will generate an extra markup needed to wrap each block node with its own block mark.

Also, since it is expected to work only for block marks, we should make it very clear in ProseMirror documentation.
