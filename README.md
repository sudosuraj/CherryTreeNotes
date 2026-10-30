# CherryTreeNotes

Personal exam notes for **OSCP** and **CPTS** — distilled, reusable techniques only.

The goal is never to archive source material — it's to mine it for reusable
techniques and compile those into exam-ready cheat sheets.

## Structure

```
Methodology/     CherryTree-native tree: folder = node, index.md = that node's page.
                 Mirrors the real hierarchy (API, Cloud, Privilege Escalation, Client
                 Side Attacks, Pivoting & Tunneling, OSINT, ...) so it can be imported
                 straight into CherryTree. See Methodology/README.md for the convention.
Techniques/      Flat cheat sheets, grouped by topic. Quick single-file reference.
templates/       technique-template.md — copy this when adding a new file so
                 formatting stays consistent.
```

## Nothing traceable to a source gets committed

This repo holds **distilled technique notes only**. No full walkthroughs, no raw
recon dumps, no box/challenge names, no writeup titles, no source platform names —
none of that gets committed, anywhere, ever. When source material is dropped in,
it's kept locally in Claude's scratchpad (outside this repo, untracked — see
`.gitignore`) purely so context on it survives across turns; only the extracted,
generic technique gets written into `Methodology/` or `Techniques/`.

## Workflow per drop

1. User pastes/links a batch of material.
2. Raw content is saved locally (scratchpad) as a compact fact/command extract,
   never full prose, never committed.
3. For each reusable technique: add/update the matching node under `Methodology/`
   (and/or `Techniques/`) — technique and commands only, no source reference.
4. Commit — only the distilled, source-agnostic notes.
