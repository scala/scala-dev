---
name: Release
about: Tracking issue for a release
title: Release 2.y.z
---
For every Scala release, make a copy of this file named after the release, and expand the variables.
Ideally this should become a script you can run on your local machine. The only missing piece is programmatic triggering of travis jobs with custom config.

Variables to be expanded in this template (or export them in a local terminal, so that you can copy/paste the commands below without replacing anything):
- SCALA_VER_BASE="2.13.0"
- SCALA_VER_SUFFIX="-M5"
- SCALA_SHA=????????????????????????????????????????
- DIST_SHA=????????????????????????????????????????
- SCALA_VER="$SCALA_VER_BASE$SCALA_VER_SUFFIX"

Key links:
  - scala/scala milestone: https://github.com/scala/scala/milestone/?
  - scala/bug milestone: https://github.com/scala/bug/milestone/?
  - scala/scala-dev milestone: https://github.com/scala/scala-dev/milestone/?

### N weeks before the release
- [ ] Wind down PR queue. There has to be enough time after the last (non-trivial) PR is merged and the next phase. The core of the eco-system needs time to prepare for the final!
- [ ] Triage scala/bug and scala/scala-dev tickets
- [ ] Create next scala/scala milestone, move the magical "Merge to 2.13.x" description to it (so Scabot uses it as default for new PRs)
- [ ] Close the scala/bug milestone, create next milestone, move pending issues
- [ ] Close the scala/scala-dev milestone, create next milestone, move pending issues
- [ ] Check PRs assigned to the milestone, also check WIP
- [ ] Announce expected release date and current nightly "release candidate" (nightly sha-mangled version) at https://scala-ci.typesafe.com/artifactory/scala-integration/ on https://contributors.scala-lang.org/c/announcements
- [ ] Also notify Scala Center advisory board members of the upcoming release, so they can help test if they want (Seth can handle this, if asked)

### Release announcement / notes
- [ ] Notify community on https://contributors.scala-lang.org/c/announcements
- [ ] Review merged PRs, make sure release-notes label is applied appropriately
- [ ] PRs with release-notes label must have excellent title & description (title will be pasted literally in release note bullet list)
- [ ] Draft release notes (to be published once GitHub tag is pushed)
  - note that GitHub release notes drafts can only be viewed by committers, so use a gist to draft the notes, so the gist can be shared with the community
- [ ] On contributors thread, link to release note gist and request feedback

### N days before release
- Announce no more PRs will be merged unless last-minute regressions are found. Re-iterate current nightly sha version for testing.
- [ ] Community build
    - won't be anywhere near all green, but we should make sure some nontrivial number of projects are still green
    - run with expected-greens only: https://scala-ci.typesafe.com/job/scala-2.13.x-integrate-community-build/?
    - also do a full run to see if there are any unexpected greens
- [ ] Run scala-collections-laws and evaluate results
    - Rex Kerr (@Ichoran) is the keeper and expert on this
- [ ] Windows Jenkins job
    - don't just trust the automated nightly runs, manually trigger a run on the exact SHA
    - https://scala-ci.typesafe.com/job/scala-2.13.x-integrate-windows/?
- [ ] Check any merged PRs accidentally assigned to the next milestone in this branch, and re-assign them to this milestone
- [ ] Merge in any older release branch
- [ ] Check module versioning (is everything in versions.properties up to date?)
- [ ] On major release, bump PickleFormat version

### Point of no return
- Once sufficient time has passed since last merged PR (1-2 weeks depending on whether it's a maintenance or development branch), and core projects have tried out the candidate nightly, it's time to cut the release!
- [ ] Make sure there are no stray staging repos on sonatype
- [ ] Trigger a build on [travis](https://travis-ci.org/scala/scala), specify `SCALA_VER_BASE` and `SCALA_VER_SUFFIX` in the custom config
  - config: `before_script: export SCALA_VER_BASE=$SCALA_VER_BASE SCALA_VER_SUFFIX=$SCALA_VER_SUFFIX`)
  - Check the build status on https://github.com/scala/scala/commits/2.13.x
  - Check that the scala/scala job also triggered a following scala/scala-dist job: https://travis-ci.org/scala/scala-dist/builds/?
  - [ ] Note the scala/scala commit (the full 40-char sha!)
  - [ ] Create the scala/scala tag locally: `git tag -s -m "Scala $SCALA_VER" v$SCALA_VER $SCALA_SHA`
  - [ ] Note the scala/scala-dist commit (the full 40-char sha!)
  - [ ] Create scala-dist tag locally: `git tag -s -m "Scala $SCALA_VER" v$SCALA_VER $DIST_SHA`
  - [ ] Note the repos to be promoted after tag is cut (see travis log)
    - https://oss.sonatype.org/content/repositories/orgscala-lang-????
    - https://oss.sonatype.org/content/repositories/orgscala-lang-????
- [ ] Sanity check jar/pom
  - https://oss.sonatype.org/content/repositories/staging/org/scala-lang/scala-compiler/$SCALA_VER/
  - in particular, if the release was staged multiple times, double check that https://oss.sonatype.org/content/repositories/staging/ has the files from the most recent build
- [ ] Check that JARs haven't mysteriously bloated — compare sizes to previous release. We have no other backstop for this.
- Remember, tags are forever, so are maven artifacts (even staged ones, as they could end up in local caches) and S3 uploads (S3 buckets can be changed, but it can takes days to become consistent)
- [ ] Push scala/scala tag: `git push https://github.com/scala/scala.git v$SCALA_VER`
- [ ] Add release notes to https://github.com/scala/scala/releases/tag/v$SCALA_VER
- [ ] Push scala/scala-dist tag: `git push https://github.com/scala/scala-dist.git v$SCALA_VER`
- [ ] Trigger two scala-dist jobs on travis (https://travis-ci.org/scala/scala-dist) with custom config. must use full-length SHAs!
  - `before_script: export version=$SCALA_VER scala_sha=$SCALA_SHA mode=archives`: https://travis-ci.org/scala/scala-dist/builds/?
  - `before_script: export version=$SCALA_VER scala_sha=$SCALA_SHA mode=update-api`: https://travis-ci.org/scala/scala-dist/builds/?
- [ ] Promote staging repos: `st_stagingRepoPromote [scala-repo]`, `st_stagingRepoPromote [modules-repo]` (or use oss.sonatype.org web UI)

### Check availability
- [ ] Check release on sonatype: https://oss.sonatype.org/content/repositories/releases/org/scala-lang/scala-compiler/$SCALA_VER/
- [ ] Check the release on maven central: http://central.maven.org/maven2/org/scala-lang/scala-compiler/$SCALA_VER/
- [ ] Check the release on maven search (takes longer): http://search.maven.org/#search%7Cga%7C1%7Cg%3Aorg.scala-lang%20v%3A$SCALA_VER

### When everything is on maven central
- [ ] Prepare PR to https://github.com/scala/scala-lang/ (using scala/make-release-notes which requires a staged release and a pushed tag)
  - `documentation/api.md`
  - `download/index.md`
  - `_config.yml` (update devscalaversion or scalaversion)
- [ ] Prepare PR to https://github.com/scala/docs.scala-lang/
  - `api/all.md`
- [ ] make sure the draft release notes are on GitHub tag: https://github.com/scala/scala/releases/tag/v$SCALA_VER
- [ ] Pre-announce the release on https://contributors.scala-lang.org/c/announcements
- [ ] On 2.13.0 only: (manually) update the `current` symlink for the API docs
  - https://github.com/scala/scala-dist/blob/2.13.x/scripts/jobs/release/website/update-api#L15
- [ ] Check that the API docs are published
  - http://www.scala-lang.org/api/$SCALA_VER/
  - https://docs.scala-lang.org/api/all.html ?
- [ ] Merge the scala-lang PR and the docs.scala-lang.org PR
  - [ ] wait for them to arrive on the websites and make sure they look okay

### Modules
- [ ] build and release scala-collection-compat and other modules (or open tickets asking that the maintainers do so)
    - this work has moved to https://github.com/scala/make-release-notes/blob/2.13.x/projects-2.13.md
- [ ] if it's a 2.12.x release, publish macro paradise for the new version

### Announcements
- [ ] Scala Users discourse https://users.scala-lang.org
- [ ] add a reply on the same https://contributors.scala-lang.org thread where you announced that the release process was starting
- [ ] Tweet from [@scala_lang](https://twitter.com/scala_lang)
- [ ] https://gitter.im/scala/scala
    - [ ] if you feel like it, say something in https://gitter.im/scala/contributors too
- [ ] ask Seth to announce on #scala IRC

### Afterwards
- [ ] Create PR to add/update spec links on scala-lang.org (example: https://github.com/scala/scala-lang/pull/1050)
- [ ] Create PR to update `scala-version` on docs.scala-lang.org (example: https://github.com/scala/docs.scala-lang/pull/1294)
- [ ] Create a scala/scala PR to update `versions.properties` and the `baseVersion` in `build.sbt`.
  - If it's a major bump, also update MiMa base version, exclusion filters, and `spec/_config.yml`.
- If it's a major bump:
  - [ ] Update `latestSpecVersion` in `spec/_config.yml` on the old branch, so that spec is marked as no longer current
  - [ ] Ditto for the nightly build and spec links in `_data/footer.yml` and `_data/doc-nav-header.yml` on docs.scala-lang.org
- [ ] Create PR to update https://github.com/lightbend/lightbend-platform-docs/blob/master/docs/modules/getting-help/pages/build-dependencies.adoc
- [ ] Update [WhiteSource](https://github.com/lightbend/scala-team/wiki/Whitesource)
