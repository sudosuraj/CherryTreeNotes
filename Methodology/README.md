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

## Note content convention

Two shapes of `index.md`, depending on whether the node has children:

**Leaf node (a technique)** — use `templates/technique-template.md`:
```
# <Technique Name>

<one-line context — only if the command needs it, delete otherwise>

​```bash
<copy-pasteable command>
​```

<!-- optional one-liner: a gotcha, or "— <Platform>/<Box>". delete if nothing to add -->
```
No prose, no mandatory headers, no empty sections. A technique note's main content
is the command block — everything else is optional and only earns a line if it adds
something. This gets scanned under exam time pressure, not read.

**Category node (has children)** — just a title and a one-line scope. The tree
navigation itself shows the children; don't duplicate a listing in the text.
```
# <Category Name>

<one-line: what belongs under this node>
```

### Standard placeholder tokens

Use these consistently across every command block, so muscle memory carries from
one note to the next:

| Token | Meaning |
|---|---|
| `<TARGET>` / `<RHOST>` | target IP/hostname |
| `<RPORT>` | target port |
| `<LHOST>` | your IP |
| `<LPORT>` | your listener port |
| `<USER>` / `<PASS>` | credentials |
| `<HASH>` | NTLM/other hash |
| `<DOMAIN>` | AD domain |
| `<SHARE>` | SMB share name |
| `<WORDLIST>` | path to wordlist |
| `<FILE>` | local or remote file path |

Avoid ad-hoc placeholder spellings (`$IP`, `TARGET_IP`, `[target]`, ...) — once a
token drifts, find/replace during the exam gets slower, which defeats the point.
