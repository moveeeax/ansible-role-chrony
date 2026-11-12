# ansible-role-chrony

An Ansible role that installs and configures [chrony](https://chrony-project.org/)
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
| `chrony_cloud_time_source` | `auto` | Cloud provider time service to use: `auto`, `aws`, `gcp`, `azure` or `none`. See below. |
| `chrony_cloud_time_source_exclusive` | `true` | When a cloud source is used, render it as the only source and drop `chrony_servers` and `chrony_pools`. |
| `chrony_azure_ptp_device` | `/dev/ptp_hyperv` | PTP device for the Azure host clock. |

Per-OS variables (`chrony_package`, `chrony_service`, `chrony_config_path`,
`chrony_driftfile_default`) are loaded automatically from `vars/`.

### Cloud time sources

With the default `chrony_cloud_time_source: auto` the role reads the gathered
DMI facts and, on a recognised cloud VM, points chrony at the provider's own
time service:

| Provider | Detected by | Rendered |
| --- | --- | --- |
| `aws` | `system_vendor` is `Amazon EC2` (Nitro), `bios_version` contains `amazon` (Xen), or the system UUID starts with `ec2` | `server 169.254.169.123 prefer iburst minpoll 4 maxpoll 4` and `pool time.aws.com iburst` as a backup |
| `gcp` | `product_name` is `Google Compute Engine` | `server metadata.google.internal iburst` |
| `azure` | the Azure chassis asset tag `7783-7084-3265-9085-8269-3286-77` | `refclock PHC /dev/ptp_hyperv poll 3 dpoll -2 offset 0 stratum 2` |

Anything else, including bare metal and on-premises Hyper-V, gets no cloud
source and keeps `chrony_servers` and `chrony_pools` as before. The DMI facts
come from the `hardware` fact subset; with a narrower `gather_subset`,
`auto` finds no provider. Set a provider
name to skip detection, or `none` to turn the feature off.

By default the cloud source replaces `chrony_servers` and `chrony_pools`
rather than being added to them. AWS and Google both smear leap seconds and
advise against mixing their service with public NTP servers that step
instead. Set `chrony_cloud_time_source_exclusive: false` to keep both; the
cloud source is then listed first.

chronyd will not start with a refclock whose device it cannot open, so for
Azure the role checks that `chrony_azure_ptp_device` exists. Under `auto` a
missing device falls back to the ordinary sources; an explicit `azure` fails
the play with a message instead. Recent Azure images create the
`/dev/ptp_hyperv` symlink through a udev rule.

Vendor references:
[Amazon Time Sync Service](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/configure-time-sync.html),
[Compute Engine NTP](https://docs.cloud.google.com/compute/docs/instances/configure-ntp),
[Time sync for Linux VMs in Azure](https://learn.microsoft.com/en-us/azure/virtual-machines/linux/time-sync).

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

Four checks run in CI and all work offline, without root, a chrony package or
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

# Import tasks/cloud.yml with stand-in DMI facts for EC2 (Nitro and Xen),
# Compute Engine, Azure, on-premises Hyper-V and bare metal, and assert which
# cloud time source each one resolves to.
ansible-playbook -i tests/inventory tests/cloud.yml
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
