# ansible-ntfy

Deploys [ntfy](https://ntfy.sh/), a self-hosted push notification server, via Docker Compose.

## Requirements

- Ansible 2.15 or later
- Docker and Docker Compose v2 installed on the target host
- The `community.docker` collection (`>=3.0.0`) installed on the control node

Install the collection:

```bash
ansible-galaxy collection install -r requirements.yml
```

## Role Variables

All variables are defined in `defaults/main.yml`.

### Core

| Variable | Default | Description |
|---|---|---|
| `ntfy_data_root` | `/opt/ntfy` | Host directory for all ntfy data, config, and compose files |
| `ntfy_image` | `binwiederhier/ntfy` | Docker image to deploy |
| `ntfy_host_port` | `8070` | Port exposed on the host |
| `ntfy_host_bind` | `0.0.0.0` | Host address to bind the port to |
| `ntfy_internal_port` | `80` | Port inside the container |

### Site and URL

| Variable | Default | Description |
|---|---|---|
| `ntfy_site_domain` | (none) | **Required.** Public domain name (e.g. `ntfy.example.com`). The role asserts that this is non-empty and ntfy refuses to start without a valid `base-url`. |
| `ntfy_base_url` | `https://{{ ntfy_site_domain }}` | Full base URL served to clients |
| `ntfy_upstream_base_url` | `https://ntfy.sh` | Upstream ntfy instance used as the iOS push relay |

### Web Push

Web push is disabled when any of the three key variables (`ntfy_webpush_public_key`, `ntfy_webpush_private_key`, `ntfy_webpush_email`) are empty. To enable it, generate VAPID keys with `ntfy webpush keys` and set all three. The role asserts that `ntfy_webpush_email` is set whenever the public and private keys are both present.

| Variable | Default | Description |
|---|---|---|
| `ntfy_webpush_public_key` | `""` | VAPID public key |
| `ntfy_webpush_private_key` | `""` | VAPID private key |
| `ntfy_webpush_email` | `""` | Contact email for the web push VAPID assertion (required when keys are set) |
| `ntfy_webpush_file` | `{{ ntfy_data_root }}/cache/webpush.db` | Host path for the web push subscriber database |

### Host Storage Paths

These derive from `ntfy_data_root` and do not normally need to be changed.

| Variable | Default | Description |
|---|---|---|
| `ntfy_cache_file` | `{{ ntfy_data_root }}/cache/cache.db` | Host path for the message cache database |
| `ntfy_auth_file` | `{{ ntfy_data_root }}/lib/auth.db` | Host path for the authentication database |
| `ntfy_attachment_cache_dir` | `{{ ntfy_data_root }}/cache/attachments` | Host path for the attachment cache |

### Container-Internal Paths

These are the paths where ntfy sees its data files inside the container. They reflect the volume mount targets in the generated `docker-compose.yml` and do not need to be changed unless you override the volume mount paths.

| Variable | Default |
|---|---|
| `ntfy_container_cache_file` | `/var/cache/ntfy/cache.db` |
| `ntfy_container_auth_file` | `/var/lib/ntfy/auth.db` |
| `ntfy_container_attachment_cache_dir` | `/var/cache/ntfy/attachments` |
| `ntfy_container_webpush_file` | `/var/cache/ntfy/webpush.db` |

### Extra Docker Networks

Use these when other containers (for example, a reverse proxy) need to reach ntfy by container name on a shared network.

| Variable | Default | Description |
|---|---|---|
| `ntfy_enable_extra_networks` | `false` | Attach the ntfy container to additional external Docker networks |
| `ntfy_extra_networks` | `[]` | List of external Docker network names to join |

## Path Architecture

The role creates the following directories under `ntfy_data_root` on the host:

```
/opt/ntfy/
  config.yml          # Generated ntfy server config
  docker-compose.yml  # Generated Compose file
  cache/              # Volume-mounted to /var/cache/ntfy inside the container
    cache.db
    attachments/
    webpush.db        # Only present when web push is enabled
  lib/                # Volume-mounted to /var/lib/ntfy inside the container
    auth.db
```

The `ntfy_container_*` variables tell ntfy where to find those files inside the container. They correspond directly to the volume mount targets in `docker-compose.yml.j2` and should not be changed unless you also change the volume mounts.

## Dependencies

None. The role depends on modules from the `community.docker` collection, not on other roles.

## Example Playbook

Minimal working example with web push credentials stored in Ansible Vault:

```yaml
- hosts: myserver
  become: true
  roles:
    - role: ansible-ntfy
      vars:
        ntfy_site_domain: ntfy.example.com
        ntfy_webpush_public_key: "{{ vault_ntfy_webpush_public_key }}"
        ntfy_webpush_private_key: "{{ vault_ntfy_webpush_private_key }}"
        ntfy_webpush_email: "{{ vault_ntfy_webpush_email }}"
```

To deploy without web push, omit the `ntfy_webpush_*` variables entirely:

```yaml
- hosts: myserver
  become: true
  roles:
    - role: ansible-ntfy
      vars:
        ntfy_site_domain: ntfy.example.com
```

With extra Docker networks to integrate with other containers on a shared network:

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
          - proxy_net
```

The named networks must already exist on the host. When `ntfy_enable_extra_networks` is `true`, the role creates any networks listed in `ntfy_extra_networks` that do not yet exist before launching the container.

## License

MIT

## Author Information

Larry Smith Jr. - [mrlesmithjr](https://github.com/mrlesmithjr)
