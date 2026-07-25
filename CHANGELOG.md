# Changelog

All notable changes to this role are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

## [Unreleased]

### Added

- `tests/render.yml`, an offline test playbook that renders `chrony.conf` from
  the shipped defaults and from server-mode, mapping-style and clock-disabled
  overrides, and asserts on the result. It runs in CI.
- Preflight assertion for the target OS family, so an unsupported host reports
  what is supported instead of a missing `<OsFamily>.yml` file.

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

## [1.0.0] - 2021-05-16

### Added

- Initial release of the chrony role.
- Package installation for the RedHat and Debian families.
- Templated `chrony.conf` with configurable servers, pools and driftfile.
- Configurable makestep and rtcsync clock options.
- Server-mode support via allow networks and local stratum.
- Passthrough for arbitrary extra chrony directives.
- Optional system timezone management via timedatectl.
- Preflight assertions for time sources and makestep values.
- Syntax-check test playbook and a GitHub Actions lint workflow.
