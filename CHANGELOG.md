# Changelog

All notable changes to this role are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

## [Unreleased]

### Added

- `chrony_cloud_time_source` (`auto`, `aws`, `gcp`, `azure`, `none`; default
  `auto`). On a VM detected from DMI facts as EC2, Compute Engine or Azure,
  chrony uses the provider's time service: the Amazon Time Sync Service at
  `169.254.169.123` with `time.aws.com` as a backup, `metadata.google.internal`,
  or the Hyper-V PTP clock at `chrony_azure_ptp_device`
  (`/dev/ptp_hyperv`). `chrony_cloud_time_source_exclusive` (default `true`)
  controls whether `chrony_servers` and `chrony_pools` are dropped in that
  case. On Azure a missing PTP device falls back to the ordinary sources under
  `auto` and fails the play when `azure` is set explicitly.
- `tests/cloud.yml`, an offline test playbook for the provider detection. It
  runs in CI.

- `tests/validate.yml`, an offline test playbook that imports the role's real
  `tasks/preflight.yml` against deliberately invalid input and asserts each
  case is rejected with an actionable message. It runs in CI.
- Preflight assertions rejecting `chrony_servers`/`chrony_pools` mapping
  entries with no `name` key, so a typo fails with a clear message instead of
  a raw `object of type 'dict' has no attribute 'name'` error out of the
  template.
- Preflight assertion for `chrony_local_stratum`, which previously had no
  input validation at all.
- Dependabot configuration that keeps the GitHub Actions in CI up to date.

### Changed

- On EC2, Compute Engine and Azure VMs the default configuration no longer
  lists the `pool.ntp.org` sources; the provider's time service replaces them.
  Set `chrony_cloud_time_source: none` to keep the previous output.
- CI uses `actions/checkout@v7` and `actions/setup-python@v7`.
- The README links to the chrony project at its current home,
  <https://chrony-project.org/>. The old `chrony.tuxfamily.org` address only
  redirects there.

### Removed

- Galaxy platform metadata no longer lists Debian 11 (bullseye) or Ubuntu
  20.04 (focal). Both are past the end of their standard (and, for bullseye,
  LTS) support, and the role is not tested on them.

### Fixed

- `chrony_makestep_updates` validation tightened. Jinja's `| int` filter
  silently coerces an unparsable value (for example a typo like `"three"`) to
  `0`, so the old `>= 0` check let it through and quietly disabled clock
  stepping instead of failing on the operator's mistake. The check now
  requires a genuine non-negative integer.

## [0.1.0] - 2026-07-26

First tagged release. This changelog previously described the initial
feature set as `1.0.0`, but no such version was ever tagged; the entries are
renumbered to match the `v0.1.0` tag.

### Added

- Package installation for the RedHat and Debian families.
- Templated `chrony.conf` with configurable servers, pools and driftfile.
- Configurable makestep and rtcsync clock options.
- Server-mode support via allow networks and local stratum.
- Passthrough for arbitrary extra chrony directives.
- Optional system timezone management via timedatectl.
- Preflight assertions for time sources and makestep values.
- Preflight assertion for the target OS family, so an unsupported host reports
  what is supported instead of a missing `<OsFamily>.yml` file.
- Syntax-check test playbook and a GitHub Actions lint workflow.
- `tests/render.yml`, an offline test playbook that renders `chrony.conf` from
  the shipped defaults and from server-mode, mapping-style and clock-disabled
  overrides, and asserts on the result. It runs in CI.

### Changed

- Input validation now runs before the OS variables are loaded.
- CI checks out with `actions/checkout@v4` and `actions/setup-python@v5` on
  Python 3.12, and installs a current `ansible-core`. The previous Node 12
  actions no longer execute on GitHub-hosted runners, and the
  `ansible>=2.9,<2.11` pin is not satisfiable alongside a current
  `ansible-lint`.
- The CI workflow declares least-privilege `permissions: contents: read`.

### Fixed

- `min_ansible_version` corrected from `2.9` to `2.10`. The role addresses
  modules by their `ansible.builtin.*` names, which do not resolve on 2.9, so
  the advertised floor could never have worked.
- Galaxy platform metadata now lists Debian, which `vars/Debian.yml` has always
  supported but which was never declared. EOL entries (EL 7, Ubuntu bionic)
  were dropped and current releases added.

[Unreleased]: https://github.com/moveeeax/ansible-role-chrony/compare/v0.1.0...HEAD
[0.1.0]: https://github.com/moveeeax/ansible-role-chrony/releases/tag/v0.1.0
