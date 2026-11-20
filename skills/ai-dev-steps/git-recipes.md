# Git recipes (only when the developer explicitly asks)

- The developer commits between turns: re-check `git log` / `git status -sb` before any git operation or diff-based statement.
- "Merge into the original commit": `git commit --fixup=<sha>` per target commit, then `git -c sequence.editor=true rebase -i --autosquash <oldest-target>~1`. Check first that the branch is unpushed (`git status -sb` shows no upstream); otherwise warn about force-push.
- Regenerate an unpushed schema migration: follow the repo's documented procedure (every supported database; generated migration state restored and checked); no documented procedure -> ask.
