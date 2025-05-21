# Ansible Playbook for Kamal 2 Server Setup

This project contains a comprehensive Ansible setup to configure Ubuntu servers for deployment with Kamal 2.

**⚠️ Important: This setup runs from your LOCAL machine (Mac/Linux/Windows) to configure REMOTE Ubuntu servers via SSH. You do NOT run this directly on the Ubuntu server.**

## Quick Start Guide

### 🖥️ What You Need

**Local Machine (Control Machine):**
- macOS, Linux, or Windows with WSL
- Ansible installed
- SSH access to your Ubuntu server

**Remote Server (Target Machine):**
- Ubuntu 22.04 LTS or newer
- User with sudo privileges
- SSH access enabled
- Minimum 1GB RAM, 10GB disk space

### 📋 Step-by-Step Setup

#### Step 1: Install Ansible on Your Local Machine

**🍎 macOS:**
```bash
# Install Homebrew (if not installed)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install Ansible
brew install ansible

# Verify installation
ansible --version
```

**🐧 Linux (Ubuntu/Debian):**
```bash
# Update package list
sudo apt update

# Install Ansible
sudo apt install ansible

# Verify installation
ansible --version
```

**🐧 Linux (CentOS/RHEL/Fedora):**
```bash
# For RHEL/CentOS
sudo yum install ansible
# or for newer versions
sudo dnf install ansible

# Verify installation
ansible --version
```

**🪟 Windows:**
```bash
# Install WSL2 first, then use Ubuntu commands above
# Or use pip:
pip install ansible
```

#### Step 2: Install Required Ansible Collections
```bash
# Install required collections
ansible-galaxy collection install community.general
ansible-galaxy collection install ansible.posix
```

#### Step 3: Download This Project
```bash
# Clone or download this project to your local machine
git clone <repository-url>
cd ansible/

# Or if downloading manually:
mkdir ~/kamal-ansible && cd ~/kamal-ansible
# Extract project files here
```

#### Step 4: Generate SSH Keys for Deployment
```bash
# Create SSH key for deployment (on your local machine)
ssh-keygen -t ed25519 -C "kamal-deploy-$(date +%Y%m%d)" -f ~/.ssh/kamal_deploy_key

# Add to SSH agent (macOS/Linux)
ssh-add ~/.ssh/kamal_deploy_key

# Copy the public key to your project
cp ~/.ssh/kamal_deploy_key.pub files/ssh_keys/deploy_key.pub
```

#### Step 5: Copy SSH Key to Your Server
```bash
# Replace with your server details
ssh-copy-id -i ~/.ssh/kamal_deploy_key.pub ubuntu@YOUR_SERVER_IP

# Or manually copy
cat ~/.ssh/kamal_deploy_key.pub | ssh ubuntu@YOUR_SERVER_IP "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"

# Test connection
ssh -i ~/.ssh/kamal_deploy_key ubuntu@YOUR_SERVER_IP
```

#### Step 6: Configure Your Server Inventory
```bash
# Edit the inventory file (on your local machine)
vim inventories/production/hosts
```

**Replace with your server details:**
```ini
[kamal_servers]
# Example for cloud server (AWS, DigitalOcean, etc.)
prod-server ansible_host=1.2.3.4 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/kamal_deploy_key

# Example for local/VM server
dev-server ansible_host=192.168.1.100 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/kamal_deploy_key

[kamal_servers:vars]
ansible_python_interpreter=/usr/bin/python3
```

#### Step 7: Test Connectivity
```bash
# Test if Ansible can connect to your server
ansible kamal_servers -i inventories/production/hosts -m ping
```

**Expected output:**
```
prod-server | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

#### Step 8: Run the Playbook
```bash
# Dry run first (recommended)
ansible-playbook playbooks/site.yml -i inventories/production/hosts --check --ask-become-pass

# Real execution
ansible-playbook playbooks/site.yml -i inventories/production/hosts --ask-become-pass
```

#### Step 9: Verify Installation
```bash
# Run verification playbook
ansible-playbook playbooks/verify.yml -i inventories/production/hosts
```

### 🎯 Common Server Setups

**AWS EC2:**
```ini
[kamal_servers]
aws-server ansible_host=ec2-XX-XXX-XXX-XX.compute-1.amazonaws.com ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/kamal_deploy_key
```

**DigitalOcean Droplet:**
```ini
[kamal_servers]
droplet ansible_host=XXX.XXX.XXX.XXX ansible_user=root ansible_ssh_private_key_file=~/.ssh/kamal_deploy_key
```

**Local VM (VirtualBox/Parallels):**
```ini
[kamal_servers]
vm-server ansible_host=192.168.1.100 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/kamal_deploy_key
```

**VPS Provider:**
```ini
[kamal_servers]
vps ansible_host=your-domain.com ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/kamal_deploy_key
```

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
- **Time Synchronization**: Automatic time sync to prevent package update issues.
- **Monitoring**: Optional Node Exporter setup for Prometheus monitoring.

## Prerequisites

### Local Machine Requirements
- **Operating System**: macOS, Linux, or Windows with WSL
- **Ansible**: Version 2.10 or newer
- **Python**: Version 3.8 or newer (usually included with Ansible)
- **SSH Client**: For connecting to remote servers
- **Git**: For version control (optional but recommended)

### Remote Server Requirements
1. **Ubuntu 22.04 LTS or newer** installed and running
2. **User with sudo privileges** (examples: ubuntu, admin, root)
3. **SSH server running** and accessible
4. **Minimum Resources:**
   - RAM: 1GB minimum, 2GB+ recommended
   - Disk: 10GB free space minimum
   - CPU: 1 core minimum
5. **Network**: Internet access for downloading packages
6. **Python 3**: Usually pre-installed on Ubuntu

### Network Requirements
- **SSH access** from your local machine to the server (port 22)
- **Outbound internet** access on the server for package downloads
- **Domain name** (optional but recommended for production)

## Installation Guide by Operating System

### 🍎 macOS Installation

**Install Ansible:**
```bash
# Option 1: Using Homebrew (Recommended)
# Install Homebrew if not installed
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install Ansible
brew install ansible

# Option 2: Using pipx (Isolated installation)
brew install pipx
pipx install ansible
pipx ensurepath

# Option 3: Using pip
pip3 install --user ansible
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

**Install Collections:**
```bash
ansible-galaxy collection install community.general ansible.posix
```

**Generate SSH Keys:**
```bash
# Create deployment SSH key
ssh-keygen -t ed25519 -C "kamal-deploy-$(date +%Y%m%d)" -f ~/.ssh/kamal_deploy_key

# Add to SSH agent
ssh-add ~/.ssh/kamal_deploy_key

# Set proper permissions
chmod 600 ~/.ssh/kamal_deploy_key
chmod 644 ~/.ssh/kamal_deploy_key.pub
```

### 🐧 Linux Installation

**Ubuntu/Debian:**
```bash
# Update package list
sudo apt update

# Install Ansible
sudo apt install ansible python3-pip

# Alternative: Install latest version via PPA
sudo apt-add-repository ppa:ansible/ansible
sudo apt update
sudo apt install ansible

# Install collections
ansible-galaxy collection install community.general ansible.posix
```

**CentOS/RHEL/Fedora:**
```bash
# For RHEL/CentOS 8+
sudo dnf install ansible python3-pip

# For older versions
sudo yum install epel-release
sudo yum install ansible python3-pip

# For Fedora
sudo dnf install ansible python3-pip

# Install collections
ansible-galaxy collection install community.general ansible.posix
```

**Arch Linux:**
```bash
# Install Ansible
sudo pacman -S ansible python-pip

# Install collections
ansible-galaxy collection install community.general ansible.posix
```

**Generate SSH Keys (Linux):**
```bash
# Create deployment SSH key
ssh-keygen -t ed25519 -C "kamal-deploy-$(date +%Y%m%d)" -f ~/.ssh/kamal_deploy_key

# Add to SSH agent
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/kamal_deploy_key

# Set proper permissions
chmod 600 ~/.ssh/kamal_deploy_key
chmod 644 ~/.ssh/kamal_deploy_key.pub
```

### 🪟 Windows Installation

**Option 1: Windows Subsystem for Linux (WSL) - Recommended:**
```bash
# Install WSL2 with Ubuntu
wsl --install

# After WSL installation, follow Ubuntu instructions above
sudo apt update
sudo apt install ansible python3-pip
ansible-galaxy collection install community.general ansible.posix
```

**Option 2: Native Windows:**
```powershell
# Install Python and pip first
# Download Python from python.org

# Install Ansible
pip install ansible

# Install collections
ansible-galaxy collection install community.general ansible.posix
```

**Generate SSH Keys (Windows):**
```bash
# In WSL or Git Bash
ssh-keygen -t ed25519 -C "kamal-deploy-$(date +%Y%m%d)" -f ~/.ssh/kamal_deploy_key

# In PowerShell (if using native Windows)
ssh-keygen -t ed25519 -C "kamal-deploy" -f "%USERPROFILE%\.ssh\kamal_deploy_key"
```

## Complete Setup Walkthrough

### 1️⃣ Verify Ansible Installation
```bash
# Check Ansible version
ansible --version

# Expected output should show version 2.10+
# ansible [core 2.15.x]
```

### 2️⃣ Get This Project
```bash
# Option A: Clone from Git
git clone <repository-url>
cd ansible/

# Option B: Download and extract
mkdir ~/kamal-ansible && cd ~/kamal-ansible
# Download and extract project files here

# Option C: Create from scratch
mkdir ~/kamal-ansible && cd ~/kamal-ansible
# Copy all the role files and configurations manually
```

### 3️⃣ Configure SSH Access to Your Server
```bash
# Copy your SSH public key to the server
ssh-copy-id -i ~/.ssh/kamal_deploy_key.pub ubuntu@YOUR_SERVER_IP

# Or manually if ssh-copy-id doesn't work
cat ~/.ssh/kamal_deploy_key.pub | ssh ubuntu@YOUR_SERVER_IP "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"

# Test SSH connection
ssh -i ~/.ssh/kamal_deploy_key ubuntu@YOUR_SERVER_IP

# If successful, you should be logged into your server
# Type 'exit' to return to your local machine
```

### 4️⃣ Configure Server Inventory
```bash
# Edit the inventory file on your LOCAL machine
vim inventories/production/hosts

# Or use any text editor
nano inventories/production/hosts
```

**Example configurations:**

```ini
# For AWS EC2 instance
[kamal_servers]
aws-prod ansible_host=ec2-XX-XXX-XXX-XX.compute-1.amazonaws.com ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/kamal_deploy_key

# For DigitalOcean Droplet
[kamal_servers]
do-prod ansible_host=164.90.XXX.XXX ansible_user=root ansible_ssh_private_key_file=~/.ssh/kamal_deploy_key

# For local VM or VPS
[kamal_servers]
my-server ansible_host=192.168.1.100 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/kamal_deploy_key

# For multiple servers
[kamal_servers]
web-01 ansible_host=1.2.3.4 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/kamal_deploy_key
web-02 ansible_host=1.2.3.5 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/kamal_deploy_key

[kamal_servers:vars]
ansible_python_interpreter=/usr/bin/python3
```

### 5️⃣ Copy SSH Public Key to Project
```bash
# Copy your SSH public key to the project directory
cp ~/.ssh/kamal_deploy_key.pub files/ssh_keys/deploy_key.pub

# Verify the file exists
cat files/ssh_keys/deploy_key.pub
```

### 6️⃣ Test Connectivity
```bash
# Test if Ansible can reach your server(s)
ansible kamal_servers -i inventories/production/hosts -m ping

# Expected output:
# server-name | SUCCESS => {
#     "changed": false,
#     "ping": "pong"
# }

# If you get errors, check:
# - Server IP is correct
# - SSH key path is correct
# - Username is correct
# - Server is running and accessible
```

### 7️⃣ Customize Configuration (Optional)
```bash
# Edit global variables
vim inventories/production/group_vars/all.yml

# Edit Kamal-specific variables
vim inventories/production/group_vars/kamal_servers.yml

# Common customizations:
# - Timezone: timezone: "America/New_York"
# - SSH port: ssh_port: 2222
# - Docker version: docker_compose_version: "2.24.5"
# - Swap size: swap_size_mb: 4096
```

### 8️⃣ Run Dry-Run Test
```bash
# Test what would happen without making changes
ansible-playbook playbooks/site.yml -i inventories/production/hosts --check --ask-become-pass

# Enter your server's sudo password when prompted
# Review the output to see what changes would be made
```

### 9️⃣ Execute the Playbook
```bash
# Run the full server configuration
ansible-playbook playbooks/site.yml -i inventories/production/hosts --ask-become-pass

# Enter your server's sudo password when prompted
# Wait for completion (usually 5-15 minutes depending on server speed)
```

### 🔟 Verify Installation
```bash
# Run verification checks
ansible-playbook playbooks/verify.yml -i inventories/production/hosts

# This will check:
# - Docker is installed and accessible
# - Firewall is configured
# - Required directories exist
# - System resources are adequate
# - Services are running
```

## Advanced Usage

### Running Specific Roles or Tasks

You can use tags to run only specific parts of the configuration:

```bash
# Only setup time synchronization
ansible-playbook playbooks/site.yml -i inventories/production/hosts --tags time_sync --ask-become-pass

# Only setup Docker
ansible-playbook playbooks/site.yml -i inventories/production/hosts --tags docker --ask-become-pass

# Setup firewall and security configurations
ansible-playbook playbooks/site.yml -i inventories/production/hosts --tags firewall,common --ask-become-pass

# Only install monitoring (Node Exporter)
ansible-playbook playbooks/site.yml -i inventories/production/hosts --tags monitoring --ask-become-pass
```

### Managing Multiple Servers

```bash
# Run on specific server only
ansible-playbook playbooks/site.yml -i inventories/production/hosts --limit web-01 --ask-become-pass

# Run on multiple specific servers
ansible-playbook playbooks/site.yml -i inventories/production/hosts --limit "web-01,web-02" --ask-become-pass

# Exclude specific servers
ansible-playbook playbooks/site.yml -i inventories/production/hosts --limit "all:!web-03" --ask-become-pass
```

### Running Ad-hoc Commands

```bash
# Check uptime on all servers
ansible kamal_servers -i inventories/production/hosts -m command -a "uptime"

# Restart Docker service
ansible kamal_servers -i inventories/production/hosts -m systemd -a "name=docker state=restarted" --ask-become-pass

# Check disk space
ansible kamal_servers -i inventories/production/hosts -m shell -a "df -h"

# Update package lists
ansible kamal_servers -i inventories/production/hosts -m apt -a "update_cache=yes" --ask-become-pass
```

### Verification and Monitoring

After running the playbook, verify that your server is properly configured:

```bash
# Run full verification
ansible-playbook playbooks/verify.yml -i inventories/production/hosts

# Run specific verification checks
ansible-playbook playbooks/verify.yml -i inventories/production/hosts --tags docker
ansible-playbook playbooks/verify.yml -i inventories/production/hosts --tags firewall
ansible-playbook playbooks/verify.yml -i inventories/production/hosts --tags ssh
```

## Configuration Options

### Docker Configuration

Customize Docker settings in `inventories/production/group_vars/kamal_servers.yml`:

```yaml
# Docker version
docker_compose_version: "2.24.5"

# Users to add to docker group (allows running docker without sudo)
docker_users:
  - "{{ deploy_user }}"
  - additional_user

# Docker daemon configuration
docker_daemon_config:
  log-driver: "json-file"
  log-opts:
    max-size: "100m"
    max-file: "3"
  storage-driver: "overlay2"
  experimental: false
  ipv6: false
  iptables: true
  live-restore: true
```

### Firewall Configuration

Customize firewall rules in `inventories/production/group_vars/kamal_servers.yml`:

```yaml
# SSH port (default: 22)
ssh_port: 22

# TCP ports to allow through firewall
firewall_allowed_tcp_ports:
  - "{{ ssh_port }}"
  - 80    # HTTP
  - 443   # HTTPS
  - 3000  # Custom application port

# UDP ports to allow (if needed)
firewall_allowed_udp_ports:
  - 53    # DNS

# Additional custom rules
firewall_additional_rules:
  - rule: allow
    port: 9100
    proto: tcp
    src: "192.168.1.0/24"  # Only allow from specific network
```

### System Performance Tuning

Optimize system settings for container workloads:

```yaml
# Kernel parameters for optimal Docker performance
sysctl_config:
  net.ipv4.tcp_fin_timeout: 15
  net.ipv4.tcp_tw_reuse: 1
  net.core.somaxconn: 65536
  net.ipv4.ip_local_port_range: "1024 65535"
  net.ipv4.tcp_max_syn_backlog: 65536
  vm.overcommit_memory: 1
  vm.swappiness: 10
  fs.file-max: 2097152
```

### Swap Configuration

Configure swap file settings:

```yaml
configure_swap: true          # Set to false to disable swap
swap_size_mb: 2048           # Swap size in MB (2GB)
swap_file_path: "/swapfile"  # Path for the swap file

# Recommended swap sizes:
# - 1GB RAM: 2048 MB swap
# - 2GB RAM: 2048 MB swap  
# - 4GB+ RAM: 1024 MB swap
```

### SSH Security Configuration

Enhance SSH security settings in `inventories/production/group_vars/all.yml`:

```yaml
# SSH configuration
ssh_port: 22                        # Change if using non-standard port
ssh_permit_root_login: "no"         # Disable root login
ssh_password_authentication: "no"    # Force key-based auth
ssh_pubkey_authentication: "yes"     # Enable public key auth

# Fail2ban settings
security_fail2ban_enabled: true     # Enable brute force protection
```

### Monitoring (Optional)

Enable Prometheus Node Exporter monitoring:

```yaml
# Enable monitoring
monitoring_enabled: true

# Node Exporter settings
node_exporter_version: "1.5.0"
node_exporter_web_listen_address: "127.0.0.1:9100"  # Local only
# Change to "0.0.0.0:9100" to allow external access

# Collectors to enable
node_exporter_enabled_collectors:
  - cpu
  - diskstats
  - filesystem
  - loadavg
  - meminfo
  - netstat
  - textfile
  - time
  - uname
```

### Time Synchronization

Configure time synchronization (handled automatically):

```yaml
# Timezone setting
timezone: "UTC"  # Change to your timezone (e.g., "America/New_York")

# Time sync is handled automatically by the common role
# Uses chrony with multiple reliable time servers
```

### Package Management

Control automatic updates:

```yaml
# Automatic security updates
security_autoupdate_enabled: true   # Enable automatic security updates
security_autoupdate_mail_to: ""     # Email for update notifications (optional)

# Common packages installed on all servers
common_packages:
  - apt-transport-https
  - ca-certificates
  - curl
  - gnupg
  - lsb-release
  - python3-pip
  - git
  - unzip
  - vim
  - htop
  - net-tools
  - tmux
```

## Using with Kamal 2

After running this playbook, your server will be ready to receive deployments with Kamal 2.

### 1️⃣ Install Kamal on Your Local Machine

**If you have Ruby:**
```bash
# Install as gem
gem install kamal

# Or add to your Rails app's Gemfile
echo 'gem "kamal", "~> 2.0"' >> Gemfile
bundle install
```

**If you don't have Ruby:**
```bash
# macOS with Homebrew
brew install kamal

# Or download binary directly
curl -L https://github.com/basecamp/kamal/releases/latest/download/kamal-linux-amd64 -o kamal
chmod +x kamal
sudo mv kamal /usr/local/bin/
```

### 2️⃣ Initialize Kamal in Your Project

```bash
# In your application directory (on your local machine)
cd your-app-project/

# Initialize Kamal configuration
kamal init

# This creates:
# - config/deploy.yml
# - .env.example
```

### 3️⃣ Configure Kamal for Your Server

Edit `config/deploy.yml`:

```yaml
# config/deploy.yml
service: your_app_name
image: your_app_name

servers:
  web:
    - 1.2.3.4  # Your server IP (same as in Ansible inventory)

registry:
  username: your_registry_username
  password:
    - KAMAL_REGISTRY_PASSWORD

env:
  clear:
    DB_HOST: localhost
    RAILS_ENV: production
  secret:
    - RAILS_MASTER_KEY
    - DATABASE_URL

# Optional: Use custom SSH key (same as Ansible)
ssh:
  user: ubuntu
  keys:
    - ~/.ssh/kamal_deploy_key
```

### 4️⃣ Set Up Environment Variables

```bash
# Copy and edit environment file
cp .env.example .env

# Edit .env file
vim .env
```

**Example `.env`:**
```bash
# Registry credentials
KAMAL_REGISTRY_PASSWORD=your_registry_password

# Application secrets
RAILS_MASTER_KEY=your_master_key
DATABASE_URL=postgresql://user:pass@localhost/myapp_production

# Other app-specific variables
SECRET_KEY_BASE=your_secret_key
```

### 5️⃣ Deploy Your Application

```bash
# First deployment (sets up everything)
kamal setup

# Regular deployments
kamal deploy

# Check deployment status
kamal app logs
kamal app details
```

### 6️⃣ Verify Deployment

```bash
# Check if your app is running
curl http://YOUR_SERVER_IP

# SSH into server to check containers
ssh -i ~/.ssh/kamal_deploy_key ubuntu@YOUR_SERVER_IP
docker ps

# Check logs
kamal app logs --follow
```

### 7️⃣ Common Kamal Commands

```bash
# Deploy new version
kamal deploy

# Rollback to previous version
kamal app boot --version=previous

# Run console/bash in container
kamal app exec --interactive --reuse "bash"
kamal app exec --interactive --reuse "bin/rails console"

# View logs
kamal app logs
kamal app logs --follow
kamal app logs --since 2h

# Restart application
kamal app restart

# Stop application
kamal app stop

# Start application
kamal app start

# Update environment variables
kamal env push
```

### 8️⃣ Multiple Server Setup

For multiple servers, update your `config/deploy.yml`:

```yaml
servers:
  web:
    - web-01.example.com
    - web-02.example.com
  
  workers:
    - worker-01.example.com
    cmd: bundle exec sidekiq
```

## Example Configurations

### Rails Application with PostgreSQL

```yaml
# config/deploy.yml
service: myapp
image: myapp

servers:
  web:
    - 1.2.3.4

registry:
  username: myregistry
  password:
    - KAMAL_REGISTRY_PASSWORD

env:
  clear:
    RAILS_ENV: production
    DB_HOST: localhost
  secret:
    - RAILS_MASTER_KEY
    - DATABASE_URL

accessories:
  db:
    image: postgres:15
    env:
      clear:
        POSTGRES_DB: myapp_production
      secret:
        - POSTGRES_PASSWORD
    volumes:
      - /var/lib/postgresql/data:/var/lib/postgresql/data
    cmd: postgres
```

### Node.js Application

```yaml
# config/deploy.yml
service: nodeapp
image: nodeapp

servers:
  web:
    - 1.2.3.4

registry:
  username: myregistry
  password:
    - KAMAL_REGISTRY_PASSWORD

env:
  clear:
    NODE_ENV: production
    PORT: 3000
  secret:
    - DATABASE_URL
    - JWT_SECRET

healthcheck:
  path: /health
  port: 3000
```

### PHP/Laravel Application

```yaml
# config/deploy.yml
service: laravelapp
image: laravelapp

servers:
  web:
    - 1.2.3.4

registry:
  username: myregistry
  password:
    - KAMAL_REGISTRY_PASSWORD

env:
  clear:
    APP_ENV: production
  secret:
    - APP_KEY
    - DB_PASSWORD

volumes:
  - /var/www/storage:/var/www/html/storage
```

## Security Considerations

This playbook implements several security measures:

### Network Security
- **UFW Firewall**: Only essential ports (SSH, HTTP, HTTPS) are open by default
- **SSH Hardening**: Root login disabled, password authentication disabled, key-based authentication enforced
- **Fail2ban Protection**: Automatic protection against brute force attacks
- **Custom SSH Port**: Option to change SSH from default port 22

### System Security
- **Automatic Security Updates**: Unattended upgrades for security patches
- **User Privilege Management**: Deploy user with minimal required permissions
- **Docker Security**: Secure Docker daemon configuration without unnecessary exposure
- **Time Synchronization**: Prevents certificate and authentication issues

### Application Security
- **Container Isolation**: Applications run in isolated Docker containers
- **Resource Limits**: Memory and CPU limits to prevent resource exhaustion
- **Log Management**: Centralized logging with rotation to prevent disk space issues
- **Regular Updates**: Automated system package updates for security patches

### Monitoring and Auditing
- **System Monitoring**: Optional Node Exporter for Prometheus monitoring
- **Log Retention**: Configurable log retention policies
- **Service Health Checks**: Automatic service status monitoring
- **Failed Login Monitoring**: Fail2ban logs and alerts for security events

## Maintenance and Updates

### Regular Maintenance Tasks

```bash
# Update system packages on all servers
ansible kamal_servers -i inventories/production/hosts -m apt -a "upgrade=dist" --ask-become-pass

# Restart services if needed
ansible kamal_servers -i inventories/production/hosts -m systemd -a "name=docker state=restarted" --ask-become-pass

# Check disk space
ansible kamal_servers -i inventories/production/hosts -m shell -a "df -h"

# Check memory usage
ansible kamal_servers -i inventories/production/hosts -m shell -a "free -h"

# View system logs
ansible kamal_servers -i inventories/production/hosts -m shell -a "journalctl --since '1 hour ago' --no-pager"
```

### Updating This Playbook

```bash
# Re-run playbook to apply updates
ansible-playbook playbooks/site.yml -i inventories/production/hosts --ask-become-pass

# Run only specific updates
ansible-playbook playbooks/site.yml -i inventories/production/hosts --tags docker --ask-become-pass
```

### Backup Considerations

```bash
# Create backup scripts for important data
ansible kamal_servers -i inventories/production/hosts -m shell -a "docker run --rm -v /var/lib/docker:/backup alpine tar czf /backup/docker-$(date +%Y%m%d).tar.gz /var/lib/docker"

# Backup deployment configurations
tar czf kamal-ansible-backup-$(date +%Y%m%d).tar.gz inventories/ roles/ playbooks/
```

## Performance Optimization

### System Tuning
The playbook automatically configures several performance optimizations:

- **Network Settings**: TCP tuning for high-throughput applications
- **Memory Management**: Optimized swap and memory overcommit settings
- **File System**: Increased file descriptor limits for container workloads
- **Docker Storage**: Optimized storage driver

## License

This project is licensed under the MIT License - see the LICENSE file for details.