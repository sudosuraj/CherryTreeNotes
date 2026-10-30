# CherryTreeNotes

Personal exam notes for **OSCP** and **CPTS**, built incrementally from writeups at
[0xb0b.gitbook.io/writeups](https://0xb0b.gitbook.io/writeups).

The goal isn't to archive every writeup verbatim — it's to mine each one for reusable
techniques and compile those into exam-ready cheat sheets, while keeping a trail back
to the box/challenge each technique came from.

## Structure

Two parallel layouts live here on purpose — pick whichever fits a given drop:

```
Methodology/     CherryTree-native tree: folder = node, index.md = that node's page.
                 Mirrors the real hierarchy (API, Cloud, Privilege Escalation, Client
                 Side Attacks, CTF Boxes, OSINT, ...) so it can be imported straight
                 into CherryTree. See Methodology/README.md for the convention.
Techniques/      Flat cheat sheets, grouped by topic. Quick single-file reference.
Writeups/        Per-box/challenge notes, mirroring the source gitbook's layout
                 (Platform/Year/Box.md). Each one links back to Techniques/ entries.
templates/       writeup-template.md and technique-template.md — copy these when
                 adding a new file so formatting stays consistent.
INDEX.md         Checklist of every page in the source gitbook, so progress across
                 drops is tracked and nothing gets missed or duplicated.
```

## No writeups get committed

This repo holds **distilled technique notes only** — never full writeup walkthroughs,
never raw recon dumps, never verbatim box content. When source writeup material is
dropped in, it's kept locally in Claude's scratchpad (outside this repo, untracked —
see `.gitignore`) purely so context on it survives across turns; only the extracted,
reusable technique gets written into `Methodology/` or `Techniques/`, with a pointer
back to which box/challenge it came from.

## Workflow per drop

1. User pastes/links a batch of writeup content.
2. Raw content is saved locally (scratchpad), not committed.
3. For each reusable technique: add/update the matching node under `Methodology/`
   (and/or `Techniques/`), with a short "seen in: <box name>" pointer.
4. If a CTF box is being tracked, check it off in `INDEX.md`.
5. Commit — only the distilled notes, never the raw writeup.

## Source

All writeups referenced here originate from https://0xb0b.gitbook.io/writeups.
