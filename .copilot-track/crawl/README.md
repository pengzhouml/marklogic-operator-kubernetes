# Crawl Notes

Use crawl work as a chain of small PRs instead of one broad branch. Each PR should name its parent, explain what changed in this slice only, and leave the next PR with a clean handoff.

Every PR should carry evidence, not just claims. Include the prompt used, files touched, commands run, and the concrete result that supports the change, such as test output, lint output, rendered docs, or a short before/after note.

Prompt usage should stay narrow and explicit. Point the model at the exact files or symbols you want changed, state the acceptance check up front, and keep one intent per prompt. A good default pattern is: objective, scope limits, anchor files, expected evidence, and validation command.