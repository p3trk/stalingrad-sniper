# Stalingrad Sniper

A WW2 sniper game in a single HTML page (Three.js). Open `index.html` in a browser.

- `index.html` is the game.
- `GDD.md` is the game design document, the source of truth for every change.
- `stalingrad-sniper-MAJOR.MINOR.html` and `stalingrad-sniper.html` (latest) are exported builds and `stalingrad-sniper.bundle` is a git backup. Both sit in the same folder (`Documents/AI/SNIPER`) and are ignored by git.

Workflow: branch `feature/<name>` from `main`, update `GDD.md` first, then the code, merge with `--no-ff`, tag `vMAJOR.MINOR`.
