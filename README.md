# Ansible Playbook for Kamal 2 Server Setup

This project contains a comprehensive Ansible setup to configure Ubuntu servers for deployment with Kamal 2.

## Project Structure

```
ansible/
├── ansible.cfg                   # Global Ansible settings
├── inventories/
│   └── production/
│       ├── hosts                 # Inventory file
│       └── group_vars/
│           ├── all.yml           # Global variables
│           └── kamal_servers.yml # Kamal server specific variables
├── roles/
│   ├── common/                   # Basic system configuration
│   ├── docker/                   # Docker installation and configuration
│   ├── kamal/                    # Kamal environment setup
│   ├── firewall/                 # UFW firewall configuration
│   └── monitoring/               # Optional Node Exporter setup
├── playbooks/
│   ├── site.yml                  # Main playbook
│   └── verify.yml                # Verification playbook
├── files/
│   └── ssh_keys/                 # Directory for SSH keys
│       └── deploy_key.pub
└── README.md                     # Documentation
```

## Features

- **Modular Design**: Each component is organized into separate roles for easy customization.
- **Security-Focused**: Secure SSH configuration, minimal firewall rules, and fail2ban setup.
- **Docker Optimization**: Properly configured Docker daemon with performance tuning.
- **Kamal 2 Ready**: Server configured specifically for Kamal 2 deployment requirements.
- **Monitoring**: Optional Node Exporter setup for Prometheus monitoring.

## Prerequisites

1. Ansible installed on your local machine:
   ```bash
   pip install ansible
   ```

2. SSH access to your target Ubuntu 22.04 (or newer) server with a user that has sudo privileges.

## Installation

1. Clone this repository:
   ```bash
   git clone <repository-url>
   cd ansible
   ```

2. Create your deployment SSH key if you don't already have one:
   ```bash
   ssh-keygen -t ed25519 -C "kamal-deploy-key" -f files/ssh_keys/deploy_key
   ```

3. Configure your inventory by editing `inventories/production/hosts`:
   ```ini
   [kamal_servers]
   app-server ansible_host=<YOUR_SERVER_IP> ansible_user=<YOUR_USERNAME>
   ```

4. Customize variables if needed:
   - Global variables: `inventories/production/group_vars/all.yml`
   - Kamal server settings: `inventories/production/group_vars/kamal_servers.yml`

## Usage

### Running the Full Setup

To configure your server for Kamal 2 deployments:

```bash
ansible-playbook playbooks/site.yml -i inventories/production/hosts --ask-become-pass
```

### Running Specific Roles

You can use tags to run specific parts of the configuration:

```bash
# Only setup Docker
ansible-playbook playbooks/site.yml -i inventories/production/hosts --tags docker --ask-become-pass

# Setup firewall and security configurations
ansible-playbook playbooks/site.yml -i inventories/production/hosts --tags firewall,common --ask-become-pass
```

### Verification

After running the playbook, verify that your server is properly configured:

```bash
ansible-playbook playbooks/verify.yml -i inventories/production/hosts
```

## Configuration Options

### Docker Configuration

Docker daemon settings can be customized in `roles/docker/defaults/main.yml` or overridden in your inventory group_vars.

### Firewall Rules

By default, only SSH (port 22), HTTP (port 80), and HTTPS (port 443) are allowed. Customize this in `roles/firewall/defaults/main.yml` or your inventory group_vars.

### Swap Configuration

Swap settings can be adjusted in `roles/kamal/defaults/main.yml` or your inventory:

```yaml
configure_swap: true     # Set to false to disable swap configuration
swap_size_mb: 2048       # Swap size in MB
swap_file_path: /swapfile # Path for the swap file
```

### Monitoring

Node Exporter is disabled by default. Enable it by setting `monitoring_enabled: true` in your inventory.

## Using with Kamal 2

After running this playbook, your server will be ready to receive deployments with Kamal 2.

1. In your application project, install Kamal:
   ```bash
   gem install kamal
   # Or add to Gemfile: gem "kamal", "~> 2.0"
   ```

2. Initialize Kamal in your project:
   ```bash
   kamal init
   ```

3. Configure your `config/deploy.yml` to use the server you've set up:
   ```yaml
   service: your_app_name
   image: your_image_name

   servers:
     web:
       - 1.2.3.4  # Your server IP
   
   registry:
     username: your_username
     password:
       - KAMAL_REGISTRY_PASSWORD
   ```

4. Deploy your application:
   ```bash
   kamal setup
   kamal deploy
   ```

## Security Considerations

This playbook implements several security measures:
- SSH root login disabled
- Password authentication disabled
- Public key authentication enforced
- Fail2ban protection against brute force attacks
- Minimal firewall rules with UFW
- Secure Docker daemon configuration

## Advanced Usage

### Adding Multiple Servers

Edit your inventory to add multiple servers:

```ini
[kamal_servers]
app-server-1 ansible_host=1.2.3.4 ansible_user=ubuntu
app-server-2 ansible_host=5.6.7.8 ansible_user=ubuntu
```

### Architecture Support

This playbook supports different CPU architectures:
- ARM (arm64/aarch64): For Raspberry Pi, AWS Graviton, etc.
- x86_64/AMD64: For traditional servers

The architecture is detected automatically, but can be overridden in your inventory.

## Troubleshooting

If you encounter issues:

1. Check if Docker is running:
   ```bash
   sudo systemctl status docker
   ```

2. Verify the firewall rules:
   ```bash
   sudo ufw status
   ```

3. Test SSH access with your deploy key:
   ```bash
   ssh -i files/ssh_keys/deploy_key <user>@<server>
   ```

## License

This project is licensed under the MIT License - see the LICENSE file for details.