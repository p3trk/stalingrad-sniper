# Stalingrad Sniper

A WW2 sniper game in a single HTML page (Three.js). Open `index.html` in a browser.

- `index.html` is the game.
- `GDD.md` is the game design document, the source of truth for every change.
- Older builds live in the git tags, not in extra files: `git show v1.6:index.html > old.html`.
- `stalingrad-sniper.bundle` is a git backup of all branches and tags. It is ignored by git.

Workflow: branch `feature/<name>` from `main`, update `GDD.md` first, then the code, merge with `--no-ff`, tag `vMAJOR.MINOR`.
