# Why this fork exists

`zio/zio-protoquill` is the Scala 3 line of Quill. Its last published release is
`4.8.6`, cut on 2024-10-30. `master` has moved on since, and among the commits it
carries is [#720](https://github.com/zio/zio-protoquill/pull/720), which removes
redundant `GenericEncoder`/`GenericDecoder` implicit searches from the `ctx.run()`
pipeline. That search is the dominant cost of compiling a Quill-heavy module, so
the fix is not a marginal one for a codebase whose repositories are all static
queries. There is no artifact that carries it.

This fork publishes the Scala 3 artifacts from upstream `master` to INBOXE's
GitHub Packages registry so the fix is reachable from a build. It carries no
INBOXE source change and is not meant to: it exists to release, not to diverge.

## Layout

`master` is a pristine mirror of `zio/zio-protoquill@master`. Nothing is
committed on top of it, which is what keeps syncing it a fast-forward:

```
gh repo sync inboxe/zio-protoquill --source zio/zio-protoquill --branch master
```

Everything INBOXE adds lives on `inboxe-publish`, which holds this file and the
publishing workflow and nothing else. The build-level overrides the workflow
needs are written at run time rather than committed, so the tree the workflow
compiles is upstream's tree.

## Publishing

Run **Publish to GitHub Packages** from the Actions tab on the `inboxe-publish`
branch. It takes the commit-ish of this fork to build (`master` by default) and
an optional version, deriving `4.8.7-inbx.<run number>` when the version is left
blank.

Two properties of that version matter. It sorts above `4.8.6` and below a future
`4.8.7`, so adopting the upstream release once it exists is a version bump that a
conflict manager resolves the right way on its own; and it is unique per run,
because a published GitHub Packages artifact is immutable and a re-publish under
a used version is rejected.

The artifacts carry upstream's own groupId, `dev.zio`, which `master` adopted
after 4.8.6 and which the next release will therefore use. Publishing under the
coordinate that release will carry means moving to it costs a version bump and
nothing else, and it keeps these artifacts from being confused with the
`io.getquill` ones they supersede. The `-inbx` suffix is what marks the
provenance.

Each run tags the commit it built as `inbx/<version>`, since the version itself
is immutable and the mapping back to a source commit would otherwise be lost the
next time `master` moves.

## Consuming it

The registry needs a resolver and a credential with `read:packages`:

```scala
resolvers += "inboxe-github" at "https://maven.pkg.github.com/inboxe/zio-protoquill"
libraryDependencies += "dev.zio" %% "quill-jdbc-zio" % "4.8.7-inbx.1"
```

Note the groupId: a consumer coming from `4.8.6` is moving off `io.getquill` at
the same time, because upstream renamed the organization after that release.

Only `quill-sql`, `quill-jdbc`, `quill-zio` and `quill-jdbc-zio` are published.
The rest of the closure — `quill-engine`, `quill-core` and the other Scala 2.13
artifacts — is untouched by this fork and resolves from Maven Central under
whatever coordinate upstream's own build declares.

## When this fork stops being needed

When upstream cuts a release of the Scala 3 line that contains #720, consumers
move to it and this fork is archived. Until then it tracks `master` rather than
pinning #720, because the same release gap holds back every other fix merged
since 2024.
