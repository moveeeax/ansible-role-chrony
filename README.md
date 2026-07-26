# ansible-role-chrony

An Ansible role that installs and configures [chrony](https://chrony.tuxfamily.org/)
as the system NTP service, managed through systemd. It supports both simple
client setups and serving time to a local network, and works across the
RedHat and Debian families.

## Requirements

- Ansible 2.10 (ansible-base) or newer. The role addresses modules by their
  `ansible.builtin.*` fully qualified names, which do not resolve on 2.9.
- A target host running systemd, in the RedHat or Debian OS family.
- Facts must be gathered, so the role can select the per-OS variables.

## Role Variables

All variables are defined in `defaults/main.yml` and may be overridden.

| Variable | Default | Description |
| --- | --- | --- |
| `chrony_servers` | `["0.pool.ntp.org", "1.pool.ntp.org"]` | Individual NTP servers. Each item is a hostname or a mapping with `name` and optional `options`. |
| `chrony_pools` | `[{name: "2.pool.ntp.org", options: "iburst"}]` | NTP pools expanded by chrony into multiple sources. |
| `chrony_driftfile` | per-OS default | Path to the clock drift file. |
| `chrony_makestep_threshold` | `1.0` | Offset (seconds) above which the clock is stepped rather than slewed. |
| `chrony_makestep_updates` | `3` | Number of initial updates for which stepping is allowed. `0` disables it. |
| `chrony_rtcsync` | `true` | Enable kernel synchronisation of the real-time clock. |
| `chrony_allow` | `[]` | Networks permitted to use this host as an NTP server. |
| `chrony_local_stratum` | `0` | Serve time at this stratum even when unsynchronised. `0` disables it. |
| `chrony_extra_config` | `[]` | Extra directives appended verbatim to `chrony.conf`. |
| `chrony_timezone` | `""` | Optional IANA timezone (e.g. `Europe/Berlin`). Empty leaves it untouched. |

Per-OS variables (`chrony_package`, `chrony_service`, `chrony_config_path`,
`chrony_driftfile_default`) are loaded automatically from `vars/`.

## Dependencies

None.

## Example Playbook

```yaml
- name: Configure time synchronisation
  hosts: all
  become: true
  roles:
    - role: ansible-role-chrony
      vars:
        chrony_servers:
          - "time.cloudflare.com"
          - name: "time.google.com"
            options: "iburst"
        chrony_pools:
          - name: "2.pool.ntp.org"
            options: "iburst"
        chrony_allow:
          - "10.0.0.0/8"
        chrony_timezone: "Europe/Berlin"
```

## Testing

Three checks run in CI and all work offline, without root, a chrony package or
a container:

```sh
# Parse the role and the example play.
ansible-playbook --syntax-check -i tests/inventory tests/test.yml

# Render templates/chrony.conf.j2 against the shipped defaults and against
# server-mode, mapping-style and clock-disabled overrides, then assert on the
# generated chrony.conf.
ansible-playbook -i tests/inventory tests/render.yml

# Import tasks/preflight.yml against deliberately invalid input (an unnamed
# server/pool mapping, a non-numeric makestep or stratum value) and assert
# each one is rejected with an actionable message.
ansible-playbook -i tests/inventory tests/validate.yml
```

`tests/render.yml` pins the behaviour that matters: the defaults stay
client-only, disabling `chrony_makestep_updates` or `chrony_rtcsync` really
removes the directive, and every `chrony_allow` network reaches the file.

`tests/validate.yml` pins the failure behaviour: bad input must fail loudly
in `preflight.yml`, with a message naming the offending variable, rather than
being silently coerced away or surfacing later as a raw Jinja error.

## License

MIT

## Author Information

Michael Tarassov
