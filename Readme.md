# GitHub Actions Merge Queue

Listening to [Episode 313 of The Bike Shed](https://www.bikeshed.fm/313) there was a discussion of wanting a merge queue to prevent issues with multiple commits getting merged at the same time and creating a state that had not been tested in either PR. Towards the end of that discussion, there was a quote that went "If anyone at GitHub is listening, I would love if you all threw this into your platform, and then you could ping Slack if anything went wrong."


Well, while I do work at GitHub, I don't work on dotcom(the main site) but rather in actions way way way in the backend. Looking at this problem, I decided that when all I have is an actions hammer, everything is an actions nail. With that said, it's surprisingly easy to get something passable using public actions. This repo is meant as a proof of concept of what an actions based merge queue looks like. 


# The deets

Three labels are required for PRs: 
- `queued`
- `failed`
- `merged`

In order to merge a PR in this repo, you add the `queued` label to the PR. If successful, the workflow will mark the PR with the `merged` label, close the PR, and delete the branch. If it fails, the PR will be marked with the `failed` label. In order to rerun it, the failed tag must be removed and the `queued` label re-added

All the real goodies are in [./.github/workflows/merge-queue.yml](./.github/workflows/merge-queue.yml) where we define the action. We build a queue by setting a [concurrency restriction](https://github.blog/changelog/2021-04-19-github-actions-limit-workflow-run-or-job-concurrency/) on the workflow. This ensures that we're only ever running through one rebase/test/merge at a time. 

We also do a squash and rebase that does not look pretty, but it works. We maintain the author and committer of the last commit in the current branch and use the PR title as the commit message.


### What it's missing

There are some obvious ways to improve this including:
- make it push the in-progress branch such that we can have a separate job or matrix of jobs run for the test step
- add some verification that the branch should be merged(i.e is appropriately approved and the label adder has the correct permissions)
- send messages to slack
