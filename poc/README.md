# Titan Infrastructure Automation POC

## 1. Objective

Evaluate Ansible for automating server configuration and application deployment.

This POC demonstrates how Ansible can validate server prerequisites, configure application directories, deploy a Docker Compose application, manage configuration changes and deploy consistently across multiple Ubuntu servers.

The sample application uses Nginx.

## 2. Architecture

Three Ubuntu servers are managed through Ansible over SSH.

- Ansible controller: Ubuntu running inside WSL.
- Managed servers: Three Titan/Siruseri Ubuntu servers.
- Container runtime: Docker Engine.
- Application deployment: Docker Compose.
- Sample application: Nginx.

All servers use port 8080 for the sample application.

## 3. Repository structure

```text
poc/
├── ansible.cfg
├── README.md
├── collections/
│   └── requirements.yml
├── inventory/
│   ├── hosts.ini
│   ├── group_vars/
│   └── host_vars/
│       ├── siruseri-2.yml
│       └── siruseri-3.yml
├── playbooks/
│   └── deploy_sample_app.yml
└── roles/
    ├── docker/
    │   └── tasks/
    │       └── main.yml
    └── sample_app/
        ├── defaults/
        │   └── main.yml
        ├── handlers/
        │   └── main.yml
        ├── tasks/
        │   └── main.yml
        └── templates/
            ├── docker-compose.yml.j2
            └── nginx.conf.j2
```

## 4. Prerequisites

### Ansible controller

- Ansible installed.
- SSH access to managed servers.
- SSH private key configured.
- Required Ansible collections installed.

### Managed servers

- Ubuntu 24.04.
- SSH access for the configured user.
- Sudo privileges.
- Docker Engine installed and running.
- Docker Compose plugin available.

**Note:** Docker was already installed on the Titan servers. This POC validates Docker and Compose availability; it does not demonstrate fresh-server Docker installation.

## 5. Configuration

### Inventory

The inventory is maintained in `inventory/hosts.ini`.

The `titan_servers` group contains all three managed servers.

### Host-specific variables

Host-specific configuration is maintained under `inventory/host_vars/`.

For example:

- `siruseri-2.yml` defines the application response for server 2.
- `siruseri-3.yml` defines the application response for server 3.

The common application defaults are maintained in `roles/sample_app/defaults/main.yml`.

## 6. Install Ansible collections

From the POC directory:

```bash
ansible-galaxy collection install -r collections/requirements.yml
```

## 7. Validate connectivity

```bash
ansible titan_servers -m ansible.builtin.ping
```

## 8. Validate playbook syntax

```bash
ansible-playbook playbooks/deploy_sample_app.yml --syntax-check
```

## 9. Deploy the application

Deploy to all managed servers:

```bash
ansible-playbook playbooks/deploy_sample_app.yml --ask-become-pass
```

Deploy to an individual server:

```bash
ansible-playbook playbooks/deploy_sample_app.yml \
  --limit siruseri-2 \
  --ask-become-pass
```

## 10. Verify application availability

```bash
curl http://122.186.108.50:8080
curl http://122.186.108.51:8080
curl http://122.186.108.52:8080
```

Health endpoint:

```bash
curl http://122.186.108.51:8080/health
```

Expected health response:

```text
OK
```

## 11. Idempotency

The playbook was executed multiple times against the managed servers.

An unchanged deployment produced:

```text
changed=0
failed=0
```

This demonstrates that Ansible does not unnecessarily modify resources when the desired configuration is already present.

## 12. Configuration management

The Nginx configuration is generated using a Jinja2 template.

When the configuration changes:

1. Ansible updates the configuration file.
2. The handler reloads Nginx.
3. The running container continues using the updated configuration.

The POC verified that the HTTP response changed while the Docker container ID remained unchanged.

The configuration directory is mounted into the container rather than mounting an individual configuration file.

## 13. Multi-server deployment

The same playbook and roles were used to deploy the application to all three Titan servers.

Host-specific response messages were successfully verified.

No separate playbook was required for each server.

## 14. Current limitations

- Docker installation on a fresh Ubuntu server has not been demonstrated.
- The POC uses a sample Nginx application, not the actual Liveflow stack.
- Production Liveflow infrastructure and AWS execution paths were not modified.
- Application deployment currently uses a single Docker Compose service.

## 15. Potential application to Liveflow

The POC establishes a foundation for automating Liveflow infrastructure.

Potential future improvements:

- Automated Docker installation and configuration on fresh servers.
- Automated creation of Liveflow application directories.
- Deployment of existing Liveflow Docker Compose stacks.
- Environment-specific configuration using Ansible variables.
- Controlled application updates and rollback procedures.
- Health checks and post-deployment verification.
- Integration with the existing deployment workflow.

These are future implementation steps, not capabilities demonstrated by this sample application POC.
