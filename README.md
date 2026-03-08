# ansible-scripts

Personal Ansible playbooks for provisioning Linux servers. Works on local VMs, Proxmox VMs, DigitalOcean droplets, or any Debian/Ubuntu host.

## Setup

```bash
cp inventory/hosts.yml.example inventory/hosts.yml
cp group_vars/all.yml.example group_vars/all.yml
# Edit both files with your hosts and variables
```

## Playbooks

### Bootstrap a Linux VM

Create a user, add SSH keys (from GitHub or manually), configure sudo, set up bashrc/inputrc.

```bash
ansible-playbook playbooks/bootstrap.yml -e "bootstrap_username=altan bootstrap_github_username=awkto bootstrap_passwordless_sudo=true"
```

### Install GitLab CE/EE

Install GitLab with custom external URL, BYO SSL certs, optional specific version.

```bash
# Latest CE with SSL
ansible-playbook playbooks/gitlab.yml \
  -e "gitlab_external_url=https://gitlab.example.com" \
  -e "gitlab_ssl_cert_path=/path/to/cert.crt" \
  -e "gitlab_ssl_key_path=/path/to/cert.key"

# Specific EE version
ansible-playbook playbooks/gitlab.yml \
  -e "gitlab_edition=gitlab-ee" \
  -e "gitlab_version=17.4.0-ee.0" \
  -e "gitlab_external_url=https://gitlab.example.com"
```

### Install OpenBao

Secrets management with dev mode or production mode (file storage + auto-unseal).

```bash
# Dev mode
ansible-playbook playbooks/openbao.yml -e "openbao_mode=dev"

# Production with persistent storage and auto-unseal
ansible-playbook playbooks/openbao.yml -e "openbao_mode=prod"
```

### Install Kubernetes (k3s or RKE2)

Install k3s or RKE2 with TLS SAN matching the host FQDN.

```bash
# k3s
ansible-playbook playbooks/kubernetes.yml -e "k8s_distro=k3s k8s_fqdn=k8s.example.com"

# RKE2
ansible-playbook playbooks/kubernetes.yml -e "k8s_distro=rke2 k8s_fqdn=k8s.example.com"
```

### Install CLI Tools

Install kubectl, doctl, gh, cloudflared, Bitwarden CLI, bao CLI. Toggle each on/off.

```bash
# All tools
ansible-playbook playbooks/cli-tools.yml

# Only specific tools
ansible-playbook playbooks/cli-tools.yml -e "install_doctl=false install_bw_cli=false"
```

### Install DNS Server

Install and configure BIND, dnsmasq, or Unbound with a zone and records.

```bash
ansible-playbook playbooks/dns.yml \
  -e "dns_server=unbound" \
  -e "dns_zone=lab.local" \
  -e '{"dns_records": [{"name": "@", "type": "A", "value": "10.0.0.1"}, {"name": "gitlab", "type": "A", "value": "10.0.0.2"}]}'
```

## Roles

| Role | Description |
|------|-------------|
| `bootstrap` | Base packages, user creation, SSH keys, sudo, bashrc/inputrc |
| `gitlab` | GitLab CE/EE with SSL and version pinning |
| `openbao` | OpenBao in dev or prod mode with auto-unseal |
| `kubernetes` | k3s or RKE2 with TLS SAN configuration |
| `cli_tools` | kubectl, doctl, gh, cloudflared, bw, bao |
| `dns` | BIND, dnsmasq, or Unbound with zone/records |

## Variables

Each role has documented defaults in `roles/<role>/defaults/main.yml`. Override via `-e`, `group_vars`, or `host_vars`.
