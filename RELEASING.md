# Release workflow

## Major and minor releases
1. Work (meaning bugfixes and feature development) is performed in the ```master``` branch in preparation for a major or minor release (e.g. ```12.0.0``` or ```12.2.0```).
2. When all criteria for the targeted release are fulfilled, a release branch is created using the naming convention **series-Major.minor** (e.g. ```liberty-12.0```, or ```mitaka-13.1```). Once the branch is made, QE will begin their virtualized testing off this branch and report back any changes or fixes that need to be addressed. Once QE deems the branch to be in a stable place, an rc (release candidate) tag will be cut. The rc tag will be made available to anyone for further testing. 
3. Work continues in ```master``` on features and bugs targeted at the next major or minor release (e.g. ```12.1.0```).
4. As QE (and potentially support and other teams) progress their testing on the release candidate, bugs will be identified in the rc tag that was handed to them. These bugs should be fixed in ```master``` and cherry-picked to the release branch. **No other bug fixes should be cherry-picked into this branch** so that this branch can remain a non-moving target for QE.
5. Once all bugs from the initial release candidate have been cherry-picked into the release branch, a new release candidate should be tagged (e.g. ```r12.0.0rc2```) if necessary.
6. Steps 4-5 should be repeated until the latest rc passes all QE tests satisfactorily.
7. Once QE are satisfied, a release tag (e.g. ```r12.0.0```) is created in the release branch.

## Patch releases
1. Work (bugfixes) is performed in the ```master``` branch and cherry-picked into the release branch (e.g. ```liberty-12.1``` or ```mitaka-13.1```). OR work (bugfixes) is performed directly in the release branch if it is release specific and doesn't affect ```master```.
2. Every 2 weeks (approximately) a new release tag (e.g. ```12.1.1```) is made.
3. Immediately after tagging, all external projects included either via submodules, ansible-galaxy or some other mechanism, will have the version/revision/SHA updated to point to the HEAD of that project (in vernacular, we'll do a SHA bump). This allows an immediate set of gate jobs to run on those SHA bumps, as well as the next 2 weeks of development to happen against those new SHA's. This will allow us to stay current and only have to cope with incremental change in external projects.

# Release Process
- [ ] Create the tag.

  ```
  git tag -a -m 'Release VERSION' VERSION COMMIT
  git push REMOTE VERSION
  ```
- [ ] If creating an rc1 tag (the first release candidate) for a major or minor release:
  - [ ] Create a new branch.

    ```
    git branch RELEASE-MAJOR.MINOR NEW_TAG
    git push REMOTE RELEASE-MAJOR.MINOR
    ```
  - [ ] [Make the branch protected](https://github.com/rcbops/rpc-openstack/settings/branches). Copy the settings from one of the other protected branches. Access to settings is restricted, speak to a manager to get this step completed.

- [ ] If tagging the first version of a minor release, i.e. rX.Y.0, there will no longer be patch releases for the previous minor:
  - [ ] [Disable branch protection](https://github.com/rcbops/rpc-openstack/settings/branches) on the previous minor branch. Access to settings is restricted, speak to a manager to get this step completed.
  - [ ] Delete the old branch.

     ```
    git push REMOTE --delete OLD_BRANCH
    ```
- [ ] If creating or deleting a branch, update the [issue template](https://github.com/rcbops/rpc-openstack/blob/master/.github/ISSUE_TEMPLATE.md) to include it in the list.
- [ ] Update the [GitHub release notes](https://github.com/rcbops/rpc-openstack/releases) with output from rpc-differ, remember to tick pre-release if the tag is for a release candidate.

  ```
  rpc-differ --update PREVIOUS_TAG NEW_TAG | pandoc --from rst --to markdown_github
  ```
- [ ] [Close the old milestone](https://github.com/rcbops/rpc-openstack/milestones).
- [ ] [Create a new milestone](https://github.com/rcbops/rpc-openstack/milestones) and set a target date of 14 days from the date of the new tag.
- [ ] If this is a patch release, [create a docs issue](https://github.com/rackerlabs/docs-rpc) to get the release notes updated.
- [ ] Announce the release on opc mailing list.

# Using the release script

The python script ``release.py`` in the scripts directory can be used to automate
the release process for patch releases and release candidates after the first one has
been made.

To release a patch release or a release candidate after rc1, run the ``release.py``
script like so:
```
./release.py --github-token <YOUR-TOKEN> --tag <TAG> --commit <COMMIT>
```

To test the ``release.py`` script, override the repo URLs so the tag and the release
is not created in the offical repository, but rather a forked version of it.
For example:
```
./release.py --github-token YOUR_TOKEN --tag TAG --commit COMMIT \
    --repo-url 'ssh://git@github.com/git-harry/rpc-openstack.git' \
    --docs-repo-url 'ssh://git@github.com/git-harry/docs-rpc.git'
```
