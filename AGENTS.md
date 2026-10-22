# AGENTS.md

## Authority
- The Chinese version of AGENTS is located at docs/AGENTS_CN.md. Do not modify that file.
- Follow the requirements in docs/REVIEW.md when conducting reviews.
- Do not modify any other files in the docs directory.
- Update docs-codex/HISTORY.md with a brief summary of this operation. Operations from more than 10 entries ago that have already been completed…

## Design Requirements
- Focus only on the functionality being modified.
- Before each modification, stage the files to be modified in Git.
- Do not modify readme.md unless explicitly instructed to do so.
- If a file already exists, first analyze and follow the instructions it contains.
- If a function's author attribution includes authors other than codex, respect the original approach and follow the function's existing style when modifying it.
- Preserve the existing code structure and keep diffs as small as possible.

## Comment Requirements
- Add comments only if they do not affect code execution.

## Commit Requirements
- Before committing, check whether .gitignore needs any additions. Do not add intermediate files or build outputs to Git, and do not expose information about the development machine.
- Preserve existing copyright notices, licenses, and other valid source attributions in commits.
- Use codex as the commit author.
