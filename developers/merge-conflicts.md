---
layout: default
title: Merge Conflicts
description: Lets not fight now
parent: /developers.html
---

## Merge Conflicts

Occasionally when merging your local branch to development or main will result in merge conflicts. Unfortunately using the conflict resolver in Github itself usually causes the problem of merging the development branch into your branch which will cause issues when trying to merge into main as it won’t just have your changes on anymore and will likely have changes that aren’t ready to go to main.

After the code is approved through the pull request, these merge conflicts will need to be resolved in your editor to get around this problem.

Steps:
In your editor with the relevant project open, checkout to the development branch.
Using the sources tab in VS Code, click the 3 dots next to ‘Changes’, go to ‘Branch’ and then click ‘Merge’
You should then see a dropdown with a list of branches, locate your branch and you may see 2 with the same name, use the one with ‘origin/’ at the start as this will be up to date with the branch on Github, in case anything has been changed locally that hasn’t been approved.
This branch will then pull all changes into the code editor and you’ll see any files with merge conflicts separated out into their own tab.
Go into those files and resolve any of the conflicts, some may be straightforward content changes and some may require a little closer checking to make sure the functionality doesn’t break.
If unsure about a conflict, consult the branch owner or a senior before merging — never guess.
When the conflicts are resolved, save the files.
Make sure to run the site locally after resolving conflicts to ensure nothing was overwritten incorrectly.
Stage the files
Commit the changes and when you sync them to Github, this should automatically merge the branch.