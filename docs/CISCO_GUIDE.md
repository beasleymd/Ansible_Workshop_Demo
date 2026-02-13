# Cisco C8000V Router Automation Guide

This guide provides detailed information on managing the Cisco C8000V routers using Ansible automation.

## Table of Contents
- [Router Information](#router-information)
- [Prerequisites](#prerequisites)
- [Initial Setup](#initial-setup)
- [Golden Configuration Management](#golden-configuration-management)
- [Common Operations](#common-operations)
- [Troubleshooting](#troubleshooting)

## Router Information

### Hardware Details
- **Model**: Cisco C8000V (VXE)
- **Processor**: cisco C8000V (VXE) processor (revision VXE)
- **Memory**: 1955292K/3075K bytes of memory
- **NVRAM**: 32768K bytes of non-volatile configuration memory
- **Physical Memory**: 3898692K bytes
- **Virtual Hard Disk**: 11526144K bytes at bootflash
- **Network Interfaces**: 1 Gigabit Ethernet interface
- **Configuration Register**: 0x2102

### Software Details
- **IOS Software**: Cisco IOS XE Software
- **Version**: 17.14.01a
- **Image Type**: Virtual XE Software (X86_64_LINUX_IOSD-UNIVERSALK9-M)
- **Release**: Version 17.14.1a, RELEASE SOFTWARE (fc1)
- **Compiled**: Thu 25-Apr-24 18:53 by mcpre
- **ROM**: IOS-XE ROMMON
- **License Level**: Perpetual
- **Throughput Level**: 20000 kbps
- **Smart Licensing**: Smart Licensing Using Policy

### Router Inventory

| Hostname | Management IP | Serial Number | Config File |
|----------|--------------|---------------|-------------|
| rtr1 | 192.168.1.101 | 9YXMF1TE722 | rtr1_golden_config.txt |
| rtr2 | 192.168.1.102 | TBD | rtr2_golden_config.txt |
| rtr3 | 192.168.1.103 | TBD | rtr3_golden_config.txt |
| rtr4 | 192.168.1.104 | TBD | rtr4_golden_config.txt |

**Note**: Update the Management IP addresses in `inventory/hosts` to match your network configuration.

## Prerequisites

### Software Requirements
1. **Ansible** version 2.9 or later
2. **Python** 3.6 or later
3. **Ansible Collections**:
   - `cisco.ios` (>= 4.0.0)
   - `ansible.netcommon` (>= 5.0.0)
   - `ansible.utils` (>= 2.0.0)

### Network Requirements
1. IP connectivity to all routers from the Ansible control node
2. SSH access enabled on all routers
3. Valid user credentials with appropriate privilege level
4. Enable mode password (if enable mode is required)

### Access Requirements
For each router, ensure you have:
- SSH username and password
- Enable password (if applicable)
- Sufficient privilege level to:
  - View running configuration
  - Modify configuration
  - Save configuration changes

## Initial Setup

### 1. Install Required Software

```bash
# Install Python dependencies
pip install -r requirements.txt

# Install Ansible Galaxy collections
ansible-galaxy collection install -r requirements.yml
```

### 2. Configure Inventory

Edit `inventory/hosts` and update the IP addresses for your routers:

```ini
[routers]
rtr1 ansible_host=192.168.1.101
rtr2 ansible_host=192.168.1.102
rtr3 ansible_host=192.168.1.103
rtr4 ansible_host=192.168.1.104
```

### 3. Configure Credentials

**Option A: Using group_vars (Development Only)**

Edit `group_vars/routers.yml`:

```yaml
ansible_user: admin
ansible_password: your_password
ansible_become_password: your_enable_password
```

**Option B: Using Ansible Vault (Recommended for Production)**

```bash
# Create encrypted vault file
ansible-vault create group_vars/vault.yml

# Add credentials:
vault_router_password: your_password
vault_router_enable_password: your_enable_password

# Reference in routers.yml:
ansible_password: "{{ vault_router_password }}"
ansible_become_password: "{{ vault_router_enable_password }}"
```

### 4. Test Connectivity

```bash
# Test connection to all routers
ansible routers -m cisco.ios.ios_command -a "commands='show version'" --ask-pass --ask-become-pass

# Test connection to a single router
ansible rtr1 -m cisco.ios.ios_command -a "commands='show version'" --ask-pass --ask-become-pass
```

## Golden Configuration Management

### Understanding Golden Configurations

Golden configurations are authoritative reference configurations for each router. They serve as:
- **Source of truth** for router configuration
- **Baseline** for configuration compliance
- **Recovery point** for configuration rollback
- **Template** for standardized deployments

### Golden Config File Locations

```
configs/golden_configs/
├── rtr1_golden_config.txt    # Golden config for rtr1
├── rtr2_golden_config.txt    # Golden config for rtr2
├── rtr3_golden_config.txt    # Golden config for rtr3
└── rtr4_golden_config.txt    # Golden config for rtr4
```

### Preparing Golden Configurations

1. **Convert RTF Files to Plain Text**

   The original golden configurations are in RTF format. Convert them to plain text:

   ```bash
   # macOS
   textutil -convert txt Orig_Golden_Configs_rtr1_02-12-2026.rtf -output configs/golden_configs/rtr1_golden_config.txt

   # Linux (with catdoc)
   catdoc Orig_Golden_Configs_rtr1_02-12-2026.rtf > configs/golden_configs/rtr1_golden_config.txt

   # Or use any text editor to copy/paste the configuration
   ```

2. **Verify Configuration Format**

   Ensure the configuration file:
   - Starts with `!` or version line
   - Contains proper Cisco IOS commands
   - Ends with `end` command
   - Has no special characters from RTF conversion

3. **Review Configuration Content**

   Before deployment, review each golden configuration:
   ```bash
   less configs/golden_configs/rtr1_golden_config.txt
   ```

### Deploying Golden Configurations

```bash
# Deploy golden config to all routers (CAUTION!)
ansible-playbook playbooks/cisco_deploy_golden_config.yml

# Deploy to a specific router
ansible-playbook playbooks/cisco_deploy_golden_config.yml --limit rtr1

# Dry run (check mode) - see what would change
ansible-playbook playbooks/cisco_deploy_golden_config.yml --check --diff

# Deploy with backup first (default behavior)
ansible-playbook playbooks/cisco_deploy_golden_config.yml --extra-vars "backup_before_deploy=true"

# Replace entire configuration (use with caution!)
ansible-playbook playbooks/cisco_deploy_golden_config.yml --extra-vars "config_replace=true"
```

## Common Operations

### Gathering Router Facts

Collect detailed information about routers:

```bash
# Gather facts from all routers
ansible-playbook playbooks/cisco_gather_facts.yml

# Gather facts from specific router
ansible-playbook playbooks/cisco_gather_facts.yml --limit rtr2

# Facts reports are saved to configs/ directory
```

Facts collected include:
- Hostname and model
- Serial number
- IOS version and image
- Interface details
- Memory and storage information

### Backing Up Configurations

Create timestamped backups of router configurations:

```bash
# Backup all router configurations
ansible-playbook playbooks/cisco_backup_config.yml

# Backup specific router
ansible-playbook playbooks/cisco_backup_config.yml --limit rtr1

# Backups are saved to: configs/backup_configs/
# Format: {hostname}_backup_{timestamp}.txt
```

### Validating Configurations

Compare running configurations against golden configurations:

```bash
# Validate all routers
ansible-playbook playbooks/cisco_validate_config.yml

# Validate specific router
ansible-playbook playbooks/cisco_validate_config.yml --limit rtr3

# Review validation reports in configs/ directory
```

### Ad-Hoc Commands

Execute commands directly on routers:

```bash
# Show version on all routers
ansible routers -m cisco.ios.ios_command -a "commands='show version'"

# Show running configuration
ansible routers -m cisco.ios.ios_command -a "commands='show running-config'"

# Show interfaces status
ansible routers -m cisco.ios.ios_command -a "commands='show ip interface brief'"

# Show routes
ansible routers -m cisco.ios.ios_command -a "commands='show ip route'"

# Multiple commands
ansible routers -m cisco.ios.ios_command -a "commands=['show version','show ip interface brief']"
```

## Configuration Management Workflows

### Standard Configuration Change Workflow

1. **Backup Current Configuration**
   ```bash
   ansible-playbook playbooks/cisco_backup_config.yml
   ```

2. **Update Golden Configuration**
   ```bash
   # Edit the golden config file
   vim configs/golden_configs/rtr1_golden_config.txt
   ```

3. **Validate Changes (Dry Run)**
   ```bash
   ansible-playbook playbooks/cisco_deploy_golden_config.yml --limit rtr1 --check --diff
   ```

4. **Deploy Configuration**
   ```bash
   ansible-playbook playbooks/cisco_deploy_golden_config.yml --limit rtr1
   ```

5. **Validate Deployment**
   ```bash
   ansible-playbook playbooks/cisco_validate_config.yml --limit rtr1
   ```

6. **Test Functionality**
   - Verify network connectivity
   - Test routing protocols
   - Confirm services are operational

### Configuration Rollback Procedure

If a configuration change causes issues:

1. **Identify Last Good Backup**
   ```bash
   ls -lt configs/backup_configs/rtr1_*.txt | head -5
   ```

2. **Review Backup Content**
   ```bash
   less configs/backup_configs/rtr1_backup_2026-02-13_1430.txt
   ```

3. **Copy Backup to Golden Config**
   ```bash
   cp configs/backup_configs/rtr1_backup_2026-02-13_1430.txt configs/golden_configs/rtr1_golden_config.txt
   ```

4. **Deploy Rollback Configuration**
   ```bash
   ansible-playbook playbooks/cisco_deploy_golden_config.yml --limit rtr1 --extra-vars "config_replace=true"
   ```

## Troubleshooting

### Connection Issues

**Problem**: Cannot connect to router

**Solutions**:
1. Verify IP connectivity:
   ```bash
   ping 192.168.1.101
   ```

2. Verify SSH access:
   ```bash
   ssh admin@192.168.1.101
   ```

3. Check Ansible inventory:
   ```bash
   ansible-inventory --list
   ansible-inventory --graph
   ```

4. Test with verbose output:
   ```bash
   ansible routers -m ping -vvv
   ```

### Authentication Failures

**Problem**: Authentication failed or permission denied

**Solutions**:
1. Verify credentials in `group_vars/routers.yml`

2. Use command-line credentials:
   ```bash
   ansible-playbook playbooks/cisco_gather_facts.yml --ask-pass --ask-become-pass
   ```

3. Check enable password:
   ```bash
   ansible routers -m cisco.ios.ios_command -a "commands='show version'" --ask-become-pass
   ```

### Configuration Deployment Failures

**Problem**: Configuration deployment fails

**Solutions**:
1. Validate configuration syntax:
   ```bash
   # Manually review the config file
   less configs/golden_configs/rtr1_golden_config.txt
   ```

2. Test configuration in check mode:
   ```bash
   ansible-playbook playbooks/cisco_deploy_golden_config.yml --limit rtr1 --check
   ```

3. Deploy configuration line-by-line:
   ```bash
   # Copy config to router and apply manually first
   scp configs/golden_configs/rtr1_golden_config.txt admin@192.168.1.101:
   ```

### Timeout Issues

**Problem**: Tasks timeout during execution

**Solutions**:
1. Increase timeout in `group_vars/routers.yml`:
   ```yaml
   command_timeout: 60
   config_timeout: 120
   ```

2. Check network latency:
   ```bash
   ping -c 10 192.168.1.101
   ```

## Best Practices

1. **Always Backup Before Changes**
   - Run backup playbook before any configuration modification
   - Verify backup file was created successfully

2. **Use Check Mode First**
   - Test deployments with `--check --diff` flags
   - Review proposed changes before applying

3. **Test on One Router First**
   - Use `--limit` to test on a single router
   - Verify functionality before deploying to all routers

4. **Maintain Golden Configs in Version Control**
   - Commit golden config changes to Git
   - Use meaningful commit messages
   - Review changes in pull requests

5. **Document Configuration Changes**
   - Add comments to configuration files
   - Update deployment logs
   - Track changes in change management system

6. **Use Ansible Vault for Credentials**
   - Never commit plain-text passwords
   - Use Ansible Vault for sensitive data
   - Rotate credentials regularly

7. **Schedule Regular Backups**
   - Run backup playbook daily via cron/scheduled task
   - Implement backup retention policy
   - Test backup restoration procedures

8. **Monitor and Alert**
   - Set up monitoring for configuration drift
   - Alert on failed playbook runs
   - Track configuration compliance

## Additional Resources

- [Cisco IOS Ansible Collection Documentation](https://docs.ansible.com/ansible/latest/collections/cisco/ios/)
- [Ansible Network Automation Best Practices](https://docs.ansible.com/ansible/latest/network/getting_started/network_best_practices_2.5.html)
- [Cisco IOS XE Configuration Guide](https://www.cisco.com/c/en/us/support/ios-nx-os-software/ios-xe-17/products-installation-and-configuration-guides-list.html)

## Support

For issues or questions:
1. Check the troubleshooting section above
2. Review Ansible and Cisco documentation
3. Open an issue in the repository
4. Contact the network automation team
