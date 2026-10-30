# Methodology (CherryTree-mirrored tree)

This folder mirrors a real CherryTree document node-for-node, so it can be dropped
straight into CherryTree (Import > Folder, or a small script) or walked by hand.

**Convention:** every folder is a *node*. A node's own page content lives in
`index.md` inside that folder. Child nodes are subfolders. A leaf with no children
is just a folder containing only `index.md`.

```
Some Category/
  index.md              <- this node's page
  Sub Node/
    index.md             <- child node's page
    Grand Child/
      index.md
```

No writeup content gets committed here — this tree holds distilled, reusable
technique notes only. Raw source writeups (when dropped) are kept locally in
Claude's scratchpad for reference while extracting techniques, never committed.
