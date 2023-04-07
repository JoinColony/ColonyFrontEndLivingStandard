# Teamwork (makes the dream work)

Programming is a team sport. We adhere to the following practices to make working together as enjoyable as possible.

## Small prs

Try to keep PRs as small as possible. There are no hard and fast rules here, but anything over 1,000 lines can probably be split up into smaller parts.

You can think of a pr as a unit of self-contained work. So if a PR is getting long, ask yourself if it could be split up into smaller units of self-contained work.

If you do **have** to submit a large pr (you probably don't), leave a note at the top explaining why. (Especially true if there are a large number of automatically generated diffs in your pr, e.g. adding a new package or lamda: this gives reviewers a better sense of the "true" size of the pr).

## Commits and commit messages

Keep commits on-topic. Ideally, a commit does one, specific thing, and that thing is described accurately by the commit message.

For example:

Bad: `Refactor: Cleanup`
Good: `Refactor: Cleanup imports in ColonyTokenSelector`

The latter is specific, and atomic: it does one thing, and clearly explains what and where.

See also our (Git workflows)[https://www.notion.so/colony/Github-Git-Workflows-1acdaf1041ee421bb5fc6ecace31096f].

## Synchronous communication

While we generally handle the code review process asynchronously on Github, sometimes it's just better to catch up in real time. Don't be afraid to do so, but bear in mind that they may be in the middle of something and unable to respond straight away.

Also, make sure you've made effort to resolve the problem yourself before you ping someone else (a google search is the bare minimum). Context switching is super expensive!

## Creating issues to keep track

If somebody brings a potential improvement to your attention during a code review, it's often up to you whether to tackle it in the same pr or in a different one.

Generally, and especially if the improvement is not the direct focus of the branch being reviewed, it's better to open a new issue and submit a separate pr.

It is up to you as the branch-owner to do this, not the reviewer suggesting the improvement.
