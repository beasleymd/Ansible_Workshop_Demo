# Workshop Setup Guide

This guide will help you set up your environment for the Ansible Workshop Demo.

## Prerequisites

### Required Software
- **Ansible**: Version 2.9 or later
- **Python**: Version 3.6 or later
- **SSH Client**: OpenSSH or equivalent
- **Git**: For cloning the repository

### Required Access
- SSH access to target hosts
- Sudo/root privileges on target hosts
- Network connectivity between control node and target hosts

## Environment Setup

### 1. Install Ansible

#### On macOS
```bash
# Using Homebrew
brew install ansible

# Or using pip
pip3 install ansible
```

#### On Ubuntu/Debian
```bash
# Using apt
sudo apt update
sudo apt install ansible

# Or using pip
pip3 install ansible
```

#### On RHEL/CentOS/Fedora
```bash
# Using yum/dnf
sudo dnf install ansible

# Or using pip
pip3 install ansible
```

#### Using Python Virtual Environment (Recommended)
```bash
# Create virtual environment
python3 -m venv ansible-venv

# Activate virtual environment
source ansible-venv/bin/activate

# Install Ansible
pip install -r requirements.txt
```

### 2. Clone the Repository

```bash
git clone https://github.com/beasleymd/Ansible_Workshop_Demo.git
cd Ansible_Workshop_Demo
```

### 3. Configure SSH Access

#### Generate SSH Key (if needed)
```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa
```

#### Copy SSH Key to Target Hosts
```bash
ssh-copy-id ansible@<target-host>
```

#### Test SSH Connection
```bash
ssh ansible@<target-host>
```

### 4. Configure Inventory

Edit the inventory file to include your target hosts:

```bash
vi inventory/hosts
```

Example configuration:
```ini
[webservers]
web1.example.com ansible_host=192.168.1.10
web2.example.com ansible_host=192.168.1.11

[databases]
db1.example.com ansible_host=192.168.1.20

[all:vars]
ansible_user=ansible
ansible_python_interpreter=/usr/bin/python3
```

### 5. Configure Variables

Edit group variables as needed:

```bash
vi group_vars/all.yml
```

Customize variables like:
- Common packages to install
- Timezone settings
- User configurations

### 6. Test Ansible Connectivity

#### Ping all hosts
```bash
ansible all -m ping
```

#### Check connectivity with verbose output
```bash
ansible all -m ping -v
```

#### Test sudo access
```bash
ansible all -m shell -a "whoami" --become
```

## Verification

### Verify Ansible Installation
```bash
ansible --version
```

Expected output:
```
ansible [core 2.12.x]
  config file = /path/to/ansible.cfg
  configured module search path = ...
  ansible python module location = ...
  executable location = /usr/bin/ansible
  python version = 3.x.x
```

### Verify Inventory
```bash
ansible-inventory --list
```

### Syntax Check Playbooks
```bash
ansible-playbook playbooks/site.yml --syntax-check
```

### Dry Run (Check Mode)
```bash
ansible-playbook playbooks/site.yml --check
```

## Troubleshooting

### Common Issues

#### SSH Connection Refused
```bash
# Check if SSH service is running on target host
ssh -v ansible@<target-host>

# Verify firewall allows SSH (port 22)
```

#### Permission Denied (publickey)
```bash
# Ensure SSH key is copied to target host
ssh-copy-id ansible@<target-host>

# Check SSH key permissions
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub
```

#### Sudo Password Required
```bash
# Run with --ask-become-pass flag
ansible-playbook playbooks/site.yml --ask-become-pass

# Or configure passwordless sudo on target hosts
echo "ansible ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/ansible
```

#### Python Not Found
```bash
# Install Python on target hosts
# For Ubuntu/Debian:
sudo apt install python3

# For RHEL/CentOS:
sudo yum install python3

# Update inventory to specify Python interpreter
ansible_python_interpreter=/usr/bin/python3
```

## Lab Environment Options

### Option 1: Local VMs
- Use VirtualBox or VMware
- Create 2-3 virtual machines
- Configure bridged or NAT networking
- Install SSH server on each VM

### Option 2: Cloud Instances
- Use AWS EC2, Azure VMs, or GCP Compute
- Create small instances (t2.micro equivalent)
- Configure security groups for SSH access
- Use cloud provider's SSH key management

### Option 3: Docker Containers
- Use Docker with SSH enabled
- Create multiple containers as target hosts
- Configure networking between containers
- Useful for quick testing and development

## Next Steps

Once your environment is set up:

1. Review the [EXERCISES.md](EXERCISES.md) for workshop exercises
2. Run your first playbook: `ansible-playbook playbooks/common.yml --check`
3. Explore the roles in the `roles/` directory
4. Customize playbooks for your environment

## Support

If you encounter issues:
1. Check the troubleshooting section above
2. Review Ansible documentation: https://docs.ansible.com/
3. Open an issue in the repository
4. Check ansible.log for detailed error messages

## Additional Resources

- [Ansible Documentation](https://docs.ansible.com/)
- [Ansible Galaxy](https://galaxy.ansible.com/)
- [Red Hat Ansible Automation Platform](https://www.redhat.com/en/technologies/management/ansible)
- [Ansible Best Practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html)
