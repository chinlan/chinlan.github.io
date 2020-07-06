---
layout: post
title: "Git: pre-commit setting"
description: ""
categories: [docker git]
tags: []
redirect_from:
  - /2020/07/06/
---

In summary, this post is about add a shell script named `pre-commit` as a git hook which will be triggered before a git commit is done, we can use this hook to add some lint check such as 'eslint' or 'rubocop', etc..


1. Add `pre-commit` file under `project-directory/.git/hooks`

2. `chmod +x .git/hooks/pre-commit` : Make `pre-commit` file executable.

3. The content of `pre-commit`:
In our project, the backend and frontend has different directories and repos, so the pre-commit setting is done in both directories.

Frontend:
~~~ sh
#!/usr/bin/env sh
set -e
# Check if current branch is in remote repository
CURRENT_BRANCH=$(git rev-parse --abbrev-ref HEAD)
function is_in_remote() {
  local existed_in_remote=$(git ls-remote --heads origin $CURRENT_BRANCH)

  if [[ -z ${existed_in_remote} ]]; then # if length of existed_in_remote is zero
    return 1 # false
  else
    return 0 # true
  fi
}

echo "Running eslint"

if is_in_remote; then
  # Only check staged files (excluding deleted files) which are different from this branch's origin
  git diff-tree -r --diff-filter=ACM --name-only @\{u\} head | xargs docker-compose exec -T frontend npx eslint manager --fix
else
  # Only check staged files (excluding deleted files) which are different from master branch origin
  git diff-tree -r --diff-filter=ACM --name-only origin/master head | xargs docker-compose exec -T frontend npx eslint manager --fix
fi
~~~

`is_in_remote` function checks whether the branch has an origin in remote repo or not,
if no, will compare with the origin of master branch.

Backend:
~~~ sh
#!/usr/bin/env sh
set -e

# Check if current branch is in remote repository
CURRENT_BRANCH=$(git rev-parse --abbrev-ref HEAD)
function is_in_remote() {
  local existed_in_remote=$(git ls-remote --heads origin $CURRENT_BRANCH)

  if [[ -z ${existed_in_remote} ]]; then # if length of existed_in_remote is zero
    return 1 # false
  else
    return 0 # true
  fi
}

echo "Running rubocop"

if is_in_remote; then
  # Only check staged files which are different from this branch's origin
  git diff-tree -r --diff-filter=ACM --name-only @\{u\} head | xargs docker-compose exec -T web bundle exec rubocop -a -D
else
  # Only check staged files which are different from master branch origin
  git diff-tree -r --diff-filter=ACM --name-only origin/master head | xargs docker-compose exec -T web bundle exec rubocop -a -D
fi
~~~

Notes:
`-T` option used with `docker-compose exec` is for fixing `The input device is not a TTY` error.


[reference](https://medium.com/devnetwork/running-rubocop-only-on-modified-files-a21aed86e06d)
