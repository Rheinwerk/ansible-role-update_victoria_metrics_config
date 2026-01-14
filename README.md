# update_victoria_metrics_config

Runtime configuration for VictoriaMetrics vmagent. Adapted from upstream [victoriametrics.cluster.vmagent](https://github.com/VictoriaMetrics/ansible-playbooks/tree/master/roles/vmagent) role.

This role handles ASP-time configuration:

- Generates promscrape config (configurable via `vmagent_scrape_config`)
- Sets remote_write URL via systemd drop-in environment variable

## Requirements

The vmagent service must be installed with `-envflag.enable=true` so it reads the remote_write URL from environment variables. This is typically configured in the baseimage using the upstream vmagent role.

## Usage

```yaml
- hosts: servers
  roles:
    - role: update_victoria_metrics_config
      tags: ['victoriametrics']
```

## Variables

### Required

The role expects `_victoriametrics` to be passed:

```yaml
_victoriametrics:
  remote_write_url: "https://vm-poc.example.com:8428"
```

### Optional (with defaults)

```yaml
# Aligned with upstream victoriametrics.cluster.vmagent defaults
vmagent_system_user: "vic_vm_agent"
vmagent_system_group: "{{ vmagent_system_user }}"
vmagent_config_dir: "/opt/vic-vmagent"
vmagent_systemd_service_name: "vic-vmagent"
vmagent_systemd_dropin_dir: "/etc/systemd/system/{{ vmagent_systemd_service_name }}.service.d"

# Default scrape config - scrapes local node_exporter with FQDN as instance label
# Override this in ASP if you need different scrape targets
vmagent_scrape_config:
  scrape_configs:
    - job_name: node_exporter
      static_configs:
        - targets:
            - "127.0.0.1:9100"
          labels:
            instance: "{{ ansible_fqdn }}"
```

## What it does

1. Generates promscrape config from `vmagent_scrape_config`
2. Creates systemd drop-in with `remoteWrite_url` environment variable
3. Reloads systemd daemon

## Dependencies

None. Assumes vmagent is already installed by the baseimage.
