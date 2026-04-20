# Contributing

Thanks for your interest in helping improve LED Mapper. This repo is
**documentation and issue tracking only** — the plugin's source code lives
in a private repository. This means what you can contribute here is a
little narrower than a typical open-source project.

## What's welcome

- **Bug reports** — see [filing a good bug report](#filing-a-good-bug-report)
  below.
- **Feature requests** — open a
  [feature request issue](./.github/ISSUE_TEMPLATE/feature_request.md) or
  start a thread in [Discussions](../../discussions).
- **Questions** — ask in [Discussions](../../discussions). Keep the issue
  tracker for actionable bugs.
- **Documentation fixes** — typos, clarifications, broken links, missing
  info. PRs against the `docs/` folder and the `README.md` are welcome.

## What's not welcome here

- **Code contributions to the plugin itself** — the plugin's source is
  private. PRs targeting code changes can't be evaluated. If you've found
  a bug you know how to fix, file a detailed issue including your
  proposed fix in prose and I'll apply it on the source side.
- **Redistribution of the plugin** — the plugin is licensed per the
  Fab Standard Content License. Don't paste plugin source into this repo
  (issues, PRs, or anywhere else).

## Filing a good bug report

The [bug report template](./.github/ISSUE_TEMPLATE/bug_report.md) walks
you through the fields — please fill them out:

- **Plugin version** — `Edit → Plugins → LED Mapper`, or the `VersionName`
  in `LedMapper.uplugin`.
- **Unreal version** — `Help → About Unreal Editor`.
- **OS** — Windows version; Mac/Linux are officially untested.
- **Reproduction steps** — exact clicks and inputs. "It doesn't work" is
  hard to act on.
- **Media** — screenshots or a short video. Especially helpful for visual
  bugs in the canvas or generated materials.
- **Logs** — paste relevant sections of
  `YourProject/Saved/Logs/YourProject.log` if the editor reported errors.

## Documentation PRs

Fork this repo, make your change, open a PR against `main`. Things to
keep in mind:

- **Match the existing tone** — factual, concise, acknowledges gotchas.
  Not marketing copy.
- **Code samples and tables** — keep them aligned, use fenced blocks.
- **Screenshots** — put them in `docs/img/` and reference relatively.
  Compress large images (PNG 8-bit or JPEG for photos).
- **Line length** — soft wrap at 80 characters where practical.

## Code of conduct

By participating, you agree to the
[Code of Conduct](CODE_OF_CONDUCT.md). Be decent to each other.

## Security issues

If you've found a security issue (e.g. the plugin can be exploited to
run arbitrary code via a maliciously-crafted asset), please don't file a
public issue. Email details to the address on the Fab listing instead
and give time for a patched release before disclosing.

## Thanks

Issue reports, discussion participation, and doc fixes all directly
improve the plugin for everyone using it. I appreciate it.

— Pieter
