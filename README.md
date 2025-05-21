# Ansible Playbook for Kamal 2 Server

This project contains an Ansible playbook to configure an Ubuntu server as a deployment environment for Kamal 2.

## Project Structure

```
kamal-ansible/
├── kamal_server_setup.yml    # Main playbook
├── inventory.yml             # YAML format inventory file (alternative)
└── ssh_keys/                 # Directory to store SSH keys (create manually)
    └── deploy_key.pub        # Public key for deployment (optional)
```

## Prerequisites

1. Ansible installed on your local machine
2. SSH access to the target server
3. A user on the server with sudo permissions

## How to Use

### 1. Preparation

1. Clone this repository:
   ```bash
   git clone [REPOSITORY_URL]
   cd kamal-ansible
   ```

2. Create the directory for SSH keys:
   ```bash
   mkdir -p ssh_keys
   ```

3. (Optional) Add your deployment public key:
   ```bash
   cp /path/to/your/key.pub ssh_keys/deploy_key.pub
   ```

4. Edit either the `inventory.ini` file or `inventory.yml` file (choose one):

   **For INI format (inventory.ini)**:
   - Replace `YOUR_IP_HERE` with your server's IP address
   - Replace `YOUR_USERNAME` with your username on the server
   - You can define multiple servers, each on its own line

   **For YAML format (inventory.yml)**:
   - This format allows cleaner organization of variables across multiple lines
   - Replace the placeholder values with your actual server information
   - You can easily define variables at both host and group levels

### 2. Running the Playbook

Using the INI format inventory:
```bash
ansible-playbook -i inventory.ini kamal_server_setup.yml --ask-become-pass
```

Or using the YAML format inventory:
```bash
ansible-playbook -i inventory.yml kamal_server_setup.yml --ask-become-pass
```

If you're using an SSH key for authentication:

```bash
ansible-playbook -i inventory.ini kamal_server_setup.yml --ask-become-pass --private-key=/path/to/your/private_key
```

## What this Playbook Configures

- Basic system packages
- Docker Engine and Docker Compose
- Firewall (UFW) with rules for HTTP, HTTPS, and SSH
- Fail2ban for protection against brute force attempts
- Swap configuration (2GB)
- User configuration for Docker usage
- Applications directory
- Python dependencies required for Kamal

## Using with Kamal

After successfully running this playbook, your server will be ready to receive deployments with Kamal 2.

To configure a Rails project for deployment with Kamal:

1. Install the Kamal gem in your project:
   ```ruby
   # Gemfile
   gem "kamal", "~> 2.0"
   ```

2. Initialize Kamal configuration:
   ```bash
   bundle exec kamal init
   ```

3. Edit the generated `config/deploy.yml` file to point to the server you've configured.

4. Deploy your application:
   ```bash
   bundle exec kamal setup
   bundle exec kamal deploy
   ```

## Customization

You can customize this playbook by editing the `kamal_server_setup.yml` file:

- Change `docker_compose_version` to use a specific version of Docker Compose
- Modify `architecture` if you need to specify a different architecture (default detects automatically)
- Configure swap settings with the following variables:
  - `configure_swap`: Set to `false` to skip swap configuration
  - `swap_size_mb`: Set the swap file size in MB
  - `swap_file_path`: Specify a custom location for the swap file
- Add or remove packages from the `basic dependencies` list
- Modify firewall rules as needed

### Architecture Support

This playbook supports different architectures:
- ARM (arm64/aarch64): For Raspberry Pi, AWS Graviton, and other ARM-based servers
- x86_64/AMD64: For traditional servers

The playbook will try to detect the architecture automatically, but you can override it in the inventory file by setting the `architecture` variable.

### Swap Configuration

If your VM already has swap configured or you don't want to use swap, you can disable swap configuration by setting `configure_swap=false` in your inventory file.

You can also customize:
- The size of the swap file with the `swap_size_mb` variable (default is 2048MB = 2GB)
- The location of the swap file with the `swap_file_path` variable (default is /swapfile)