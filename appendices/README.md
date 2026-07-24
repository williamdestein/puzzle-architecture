# appendices/

Staging area for Q&A sessions that haven't been integrated into
`../architecture.md` yet.

## Workflow

1. During a working session, a question and its researched answer get staged
   here as one file: `appendices/<some-name>.md`. Staged files can be rough —
   raw Q&A, diagrams, dead ends included.
2. "Merge appendix *some-name* after chapter *X*" means: rewrite the staged
   content into document voice, splice it into `architecture.md` at the named
   spot, regenerate the Table of Contents, and delete the staged file — all as
   one commit, so the integration is a single reviewable diff.

A file in this directory is by definition not yet part of the document.
