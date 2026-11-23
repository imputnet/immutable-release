# immutable-release

stupid simple github action that ensures releases are immutable, and
that release artifacts have not been tampered with before release

1. The commit being released must be on the default branch; anything
   else is refused before a release is created.
2. The files are uploaded to a draft, which is then published. With
   [release immutability] enabled, publishing locks the assets and the
   tag. Leftover drafts from earlier attempts are deleted first.
3. The published assets' SHA-256 digests are checked against the local
   files. Anything swapped, added or removed while the release was a
   draft fails the run. Since the assets are locked by then, a passing
   check holds for good. The tag must also point at the commit that was
   built.
4. Only then, if `latest: true`, is the release marked as latest.

Re-running a failed job is safe: if the release was already published,
the run skips straight to verifying it.

## Requirements

- Release immutability enabled on the repository.
- `contents: write` for the token.

## Usage

```yaml
permissions:
  contents: write

steps:
  # ... build ...
  - uses: imputnet/immutable-release@v1
    with:
      notes-file: release.body.txt
      files: |
        dist/*.zip
        dist/*.crx
```

| Input        | Default            | |
|--------------|--------------------|-|
| `tag`        | `github.ref_name`  | Tag to release. Created at `target` if it doesn't exist yet. |
| `target`     | `github.sha`       | Commit SHA that was built. Must be on the default branch. |
| `title`      | the tag            | Release title. |
| `notes-file` |                    | File containing the release notes. |
| `files`      |                    | Newline-separated paths or globs. Each must match at least one file. |
| `prerelease` | `false`            | Mark as a pre-release. |
| `latest`     | `false`            | Mark as latest after verification. Ignored for pre-releases. |
| `token`      | `github.token`     | Token with `contents: write`. |

If verification fails, don't promote the release. Immutable tags can't be
reused, so delete it and cut a new version.

[release immutability]: https://docs.github.com/en/code-security/concepts/supply-chain-security/immutable-releases
