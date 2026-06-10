# ansible-ntfy

An Ansible role that deploys [ntfy](https://ntfy.sh/) — a self-hosted push notification server — via Docker Compose.

## Requirements

- Docker and Docker Compose v2 must be installed on the target host.
- The `community.docker` collection (`>=3.0.0`) must be installed. See `requirements.yml`.

Install the collection dependency:

```bash
ansible-galaxy collection install -r requirements.yml
```

## Role Variables

All variables are defined in `defaults/main.yml` with safe defaults.

### Core

| Variable | Default | Description |
|---|---|---|
| `ntfy_data_root` | `/opt/ntfy` | Base directory for all ntfy data, config, and compose files |
| `ntfy_image` | `binwiederhier/ntfy` | Docker image to use |
| `ntfy_host_port` | `8070` | Port exposed on the host |
| `ntfy_host_bind` | `0.0.0.0` | Host address to bind the port to |
| `ntfy_internal_port` | `80` | Port inside the container |

### Site and URL

| Variable | Default | Description |
|---|---|---|
| `ntfy_site_domain` | `""` | Public domain name (e.g. `ntfy.example.com`) — required for web push |
| `ntfy_base_url` | `"https://{{ ntfy_site_domain }}"` | Full base URL served to clients |
| `ntfy_upstream_base_url` | `"https://ntfy.sh"` | Upstream ntfy instance for iOS push relay |

### Web Push

Web push is disabled when both key variables are empty (the default). To enable it, generate keys with `ntfy webpush keys` and set all four variables.

| Variable | Default | Description |
|---|---|---|
| `ntfy_webpush_public_key` | `""` | VAPID public key |
| `ntfy_webpush_private_key` | `""` | VAPID private key |
| `ntfy_webpush_file` | `{{ ntfy_data_root }}/cache/webpush.db` | Web push subscriber database |
| `ntfy_webpush_email` | `""` | Contact email for web push (required when enabled) |

### Storage Paths

These derive from `ntfy_data_root` by default and do not normally need to be changed.

| Variable | Default |
|---|---|
| `ntfy_cache_file` | `{{ ntfy_data_root }}/cache/cache.db` |
| `ntfy_auth_file` | `{{ ntfy_data_root }}/lib/auth.db` |
| `ntfy_attachment_cache_dir` | `{{ ntfy_data_root }}/cache/attachments` |

### Extra Docker Networks

Use these when other containers (e.g. a reverse proxy or CI runner) need to reach ntfy by container name.

| Variable | Default | Description |
|---|---|---|
| `ntfy_enable_extra_networks` | `false` | Attach ntfy to additional external Docker networks |
| `ntfy_extra_networks` | `[]` | List of external Docker network names to join |

## Dependencies

None. The role depends on `community.docker` collection modules, not on other roles.

## Example Playbook

Minimal (no web push):

```yaml
- hosts: myserver
  become: true
  roles:
    - role: ansible-ntfy
      vars:
        ntfy_site_domain: ntfy.example.com
```

With web push enabled:

```yaml
- hosts: myserver
  become: true
  roles:
    - role: ansible-ntfy
      vars:
        ntfy_site_domain: ntfy.example.com
        ntfy_webpush_public_key: "{{ vault_ntfy_webpush_public_key }}"
        ntfy_webpush_private_key: "{{ vault_ntfy_webpush_private_key }}"
        ntfy_webpush_email: "admin@example.com"
```

With extra Docker networks (e.g. a shared tunnel network):

```yaml
- hosts: myserver
  become: true
  roles:
    - role: ansible-ntfy
      vars:
        ntfy_site_domain: ntfy.example.com
        ntfy_enable_extra_networks: true
        ntfy_extra_networks:
          - tunnel_net
```

## License

MIT

## Author Information

Larry Smith Jr. — [mrlesmithjr](https://github.com/mrlesmithjr)
