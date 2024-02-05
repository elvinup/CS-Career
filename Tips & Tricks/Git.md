Prevent having to type in `git push --set-upstream origin <new_branch_name>` every time you want to push a new branch with:

```
git config --global push.default current
```

Run this on each repo you frequently update