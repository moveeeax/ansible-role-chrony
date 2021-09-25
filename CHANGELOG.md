# Changelog

All notable changes to this role are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

## [Unreleased]

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
