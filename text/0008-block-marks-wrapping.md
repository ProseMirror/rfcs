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

The proposal gives the `MarkSpec` a new getter:

**`spanning`**`: boolean` indicates whether `prosemirror-view` uses the wrapping optimisation. We will preserve the default behaviour unless its set to `false`.

Implementation details: https://github.com/ProseMirror/prosemirror-view/pull/43

# Drawbacks

When this feature is used, ProseMirror will generate an extra markup needed to wrap each node with its own mark.

# Rationale and alternatives

Originally I though about using mark nodeviews for that, but this feature should be available from HTML serializer when marks don't have a nodeview.
