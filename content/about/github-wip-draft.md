---
title: 'GitHub: WIP vs Draft'
date: 2023-07-22T00:00:00+07:00
intro_image: "images/illustrations/open-source.png"
intro_image_absolute: false
intro_image_hide_on_mobile: false
---

*⚠️ We use WIP (work in progress) not [GitHub PR Draft](https://github.blog/2019-02-14-introducing-draft-pull-requests/)*

We know that GitHub has a feature called PR Draft, but we don't use it because it *"locks"* the PR - only the person who opened the PR has permission to remove the draft *flag* from the PR.

> not always the person who opened the PR will be the person who finishes the correction/appeal

We use the prefix `WIP:` in the title of the PR along with the [GitHub App WIP](https://github.com/apps/wip), making it possible for anyone who is part of the organization to change the title of the PR and give the PR as revised — ready for merge.

## Avoid keeping local branches as much as possible

*It's really “only” in a local environment.*

We work with Open Source products, so it doesn't make sense to keep branches only in a local environment (even in the few private repositories).

Whenever you create a branch and have the first commit, push the branch to GitHub and when you are minimally organized, create a PR in `WIP`, this will give everyone visibility of what you are doing. The aim is for more people to be able to contribute during the development process, avoiding large PRs.
