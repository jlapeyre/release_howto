### How to prepare a Qiskit Release

We will show how prepare patch release `0.x.y` of Qiskit.  Concrete examples of patch release
versions are `0.21.3` and `0.123.456`.  The procedure for making a point release, that is one with
version of the form `0.x.0`, differs in some respects from the instructions below.

1. Since we preparing a release, the milestone should have been created on github some
 time ago. Open [milestone `0.x.y`](https://github.com/Qiskit/qiskit-terra/milestones)
 and check that all PRs have been merged or are scheduled for merging. The latter means
 adding to the merge queue, or tagging for mergify, or using whatever the current system is.

2. A branch `stable/0.x` should have been created for the point release.  Prepare a PR to this
   branch that does three things: bumps the version numbers to `0.x.y`, fixes any typos/broken links
   in the release notes for this patch, and adds a release note that has a “prelude” section with a
   sentence explaining the release.
   Among the files containing the version number that must be updated to `0.x.y` are:
    * [Cargo.toml](https://github.com/Qiskit/qiskit-terra/blob/main/Cargo.toml)
    * [qiskit/docs/conf.py](https://github.com/Qiskit/qiskit-terra/blob/main/docs/conf.py)
    * [qiskit/VERSION.txt](https://github.com/Qiskit/qiskit-terra/blob/main/VERSION.txt)
    * [qiskit/setup.py](https://github.com/Qiskit/qiskit-terra/blob/main/setup.py)

   [Here’s an example of such a PR](https://github.com/Qiskit/qiskit-terra/pull/9193)
   

3. Regular review process to help spot typos in this PR, then we merge it.

4. The release manager (you) now tags the commit of this PR, with:

    * tag name `0.x.y`
    * tag message just says “Qiskit Terra 0.x.y"
    * ideally your PGP signature, but it’s not a big deal if you’ve not got one. (I can help if you _want_ to set one up.)

5. Double-check the tag points to the commit you expect, and its name is _exactly_ `0.x.y`. (I’ve screwed this step before.)
6. Push the `0.x.y` tag to the Qiskit remote directly.

At this point, the CD pipelines and `qiskit-bot` take over (look in the “Actions” tab, and follow
the links to Azure to see), and will handle building and deploying all the (many) wheels to PyPI.
It will also generate a “release” on GitHub, and populate it based on the “Changelog: X” labels on
all the PRs merged since the last release.  `qiskit-bot` will also open a PR on the metapackage repo
([see an example here](https://github.com/Qiskit/qiskit/pull/1640)) to bump the version information
there.  It will immediately fail CI, and can’t pass until the wheels are deployed to PyPI - don’t
worry about it.  We have to do things a bit more manually here, because we haven’t automated these
bits:

1. Clone [the metapackage repo](https://github.com/Qiskit/qiskit) locally, if you haven’t already.
   If you have write access on it you don’t need to fork it in GitHub, but if not then you’ll need
   to to make a PR

2. Checkout the bot’s branch - it’s always called `bump_meta`

3. In the file `docs/release_notes.rst`, you’ll make a new section called “Qiskit 0.x.y” (the bot
   will tell you the new version), which has subheadings for Terra, Aer and IBMQ Provider, with “No
   changes” in the latter two, and an empty space in the Terra section for now. (See
   [see the last time we did it as an example.](https://github.com/Qiskit/qiskit/pull/1640))

4. use `reno` to generate the release notes text:

5. in the Terra repo, checkout the tag you just made

6. `reno report --version 0.x.y --title "Release Notes_Terra"`

7. copy the (actual) output of the above command into the file.  You may find it easier to add
   `2>/dev/null` onto the end of the previous command, or use `-o <file>` to output it to a file
   that you then copy.  If you’ve got a mac, then you can instead pipe the output into `pbcopy` to
   get it in your clipboard directly.

8. paste that text into the release notes file in the metapackage repo and commit it

9. if you’ve got push access to the metapackage, then push that commit directly to the bot’s branch
   (`bump_meta`).  If not, open a PR to its branch and let me know.

10. approve and merge the bot’s PR (once the Terra wheels are deployed, CI should pass)

11. follow the same tagging procedure as on Terra but now on the metapackage to make the `0.x.y` (as
    appropriate) tag and push it - see an example with `git show 0.39.4`.  If you don’t have push,
    then I’ll do this step.

12. make a release on GitHub for the metapackage (look at the previous ones as examples)

At this point we’re done - everything will wing its way through CD again.  You can see the progress
in the “Actions” tab of the metapackage.  Matthew and I like to post in #qiskit-dev to let people
know that we’ve released a new version.
