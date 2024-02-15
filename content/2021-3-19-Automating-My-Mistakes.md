---
title: "Automating My Mistakes"
date: 2024-02-14T23:00:00-05:00
draft: false
---

_Originally published on 2021-3-19 as an internal tech blog post when I was the Tech Lead for Wayfair's Experimentation Platform team. Internal links will not work._

### Automating my mistakes

While building the initial steps of the [GTQ](https://github.csnzoo.com/shared/gambit-task-queue) project I found mysef stuck without a way to test code locally (at least without standing up a lot more project environment scaffolding), meaning that I had a slightly longer than ideal change/test loop:

- Update code
- Commit
- Push branch
- Wait for buildkite
- Deploy dev build to kubernetes

In particular, the `wait for buildkite` step kept taking much longer than it should have because I would frequently forget to run `black`, or `flake8` or `isort`... This resulted in doubling the change/test loop whenever a trivial step got forgotten.

I'm of the opinon that generally not thinking about the formatting or style guides that my code needs to comply with is a _good thing_, leaving more mental focus for the features and functionality of the code rather than the particular syntax of this ticket's language. Since these kinds of things don't affect me as I'm _writing_ the code, but their standardization greatly helps the next person who needs to _read_ the code, it's imperitive that we have tools that handle formatting and style linting for us. So we do, and they work great. Except when you don't use them.

There's got to be a better way.

That better way is called [Git Hooks](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks); custom scripts that are executed when specific events happen. I spent an hour learning about how to set one of these up, and decided to use `pre-commit` so that I could fix my code right before committing it every time. (Goodbye commits with titles like 'ran prettier', 'linting', and 'fix formatting'.)

The actual hook itself is simple, even for a [bash n00b](https://github.csnzoo.com/vjacek)

- Save the names of all modified files
- Run each of the linting tools and save if they were successful or not (where successful is both "nothing to fix" and "automatically fixed stuff")
- Re-stage all the formatted files so I don't have to remember the list manually
- Block the commit from happening if any of the tools failed

```
#!/bin/sh

echo "Running git pre-commit hooks..."

staged_files=$(git diff --name-only --cached)

# Run some basic automated tools that need to pass in buildkite
echo ">>> black ."
black .
black_exit_code=$?

echo ">>> flake8 ."
flake8 .
flake8_exit_code=$?

echo ">>> isort ."
isort .
isort_exit_code=$?

# Re-stage all staged files as they may have been changed by tools
for file in $staged_files; do
  git add $file
done

# If all the tools completed successfully, continue commit
# Else exit with error message and stop commit
if [[ $black_exit_code == 0 && $flake8_exit_code == 0 && $isort_exit_code == 0 ]]
then
  echo "Pre-commit hooks complete."
  exit 0
else
  echo "!! Pre-commit hooks failed. !!"
  echo "black: $black_exit_code"
  echo "flake8: $flake8_exit_code"
  echo "isort: $isort_exit_code"
  exit 1
fi

```

While it was nice that each of these tools [returns 0 for success](http://www.gnu.org/software/libc/manual/html_node/Exit-Status.html) like a good digital neighbor, different checks could be used, or branches could be implemented.

Having a script that does _whatever you tell it to_ during your standard workflow is like being given free superpowers.
