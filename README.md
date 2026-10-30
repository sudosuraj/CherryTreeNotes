# CherryTreeNotes

Personal exam notes for **OSCP** and **CPTS**, built incrementally from writeups at
[0xb0b.gitbook.io/writeups](https://0xb0b.gitbook.io/writeups).

The goal isn't to archive every writeup verbatim — it's to mine each one for reusable
techniques and compile those into exam-ready cheat sheets, while keeping a trail back
to the box/challenge each technique came from.

## Structure

```
Techniques/      Exam cheat sheets, grouped by topic (Recon, Web, AD, PrivEsc, Pivoting, ...).
                 This is the stuff you actually read the morning of the exam.
Writeups/        Per-box/challenge notes, mirroring the source gitbook's layout
                 (Platform/Year/Box.md). Each one links back to the Techniques/
                 entries it contributed.
templates/       writeup-template.md and technique-template.md — copy these when
                 adding a new file so formatting stays consistent.
INDEX.md         Checklist of every page in the source gitbook, so progress across
                 drops is tracked and nothing gets missed or duplicated.
```

## Workflow per drop

1. User pastes/links a batch of writeup content.
2. For each box: create `Writeups/<Platform>/<Year>/<Box>.md` from `templates/writeup-template.md`.
3. Pull out anything reusable (a technique, one-liner, pitfall) into the matching file
   under `Techniques/`, using `templates/technique-template.md`, and link back to the box.
4. Check the box off in `INDEX.md`.
5. Commit.

## Source

All writeups referenced here originate from https://0xb0b.gitbook.io/writeups.
