# Cisco Automation Quick Reference

## Router Information
- **Model**: Cisco C8000V (VXE)
- **IOS Version**: 17.14.01a
- **Routers**: rtr1, rtr2, rtr3, rtr4

## Quick Commands

### Setup
```bash
# Install dependencies
pip install -r requirements.txt
ansible-galaxy collection install -r requirements.yml

# Edit inventory
vim inventory/hosts

# Configure credentials
vim group_vars/routers.yml
# OR use vault:
ansible-vault create group_vars/vault.yml
```

### Common Playbooks
```bash
# Gather facts
ansible-playbook playbooks/cisco_gather_facts.yml

# Backup configurations
ansible-playbook playbooks/cisco_backup_config.yml

# Deploy golden configs
ansible-playbook playbooks/cisco_deploy_golden_config.yml

# Validate configurations
ansible-playbook playbooks/cisco_validate_config.yml

# Limit to single router
ansible-playbook playbooks/cisco_backup_config.yml --limit rtr1

# Check mode (dry run)
ansible-playbook playbooks/cisco_deploy_golden_config.yml --check --diff
```

### Ad-Hoc Commands
```bash
# Show version
ansible routers -m cisco.ios.ios_command -a "commands='show version'"

# Show running config
ansible routers -m cisco.ios.ios_command -a "commands='show running-config'"

# Show interfaces
ansible routers -m cisco.ios.ios_command -a "commands='show ip interface brief'"

# Multiple commands
ansible routers -m cisco.ios.ios_command -a "commands=['show version','show ip route']"
```

### Testing
```bash
# Test connectivity
ansible routers -m ping

# Test with credentials
ansible routers -m cisco.ios.ios_command -a "commands='show version'" --ask-pass --ask-become-pass
```

## File Locations

### Configurations
- Golden Configs: `configs/golden_configs/rtr{1-4}_golden_config.txt`
- Backups: `configs/backup_configs/rtr{1-4}_backup_*.txt`
- Inventory: `inventory/hosts`

### Variables
- Router Settings: `group_vars/routers.yml`
- Global Settings: `group_vars/all.yml`
- Host Specific: `host_vars/{hostname}.yml`

### Playbooks
- `playbooks/cisco_gather_facts.yml`
- `playbooks/cisco_backup_config.yml`
- `playbooks/cisco_deploy_golden_config.yml`
- `playbooks/cisco_validate_config.yml`

## Workflow

### Standard Configuration Change
1. Backup current config
2. Update golden config file
3. Test deployment (--check --diff)
4. Deploy to test router (--limit rtr1)
5. Validate and verify
6. Deploy to remaining routers

### Emergency Rollback
1. Find last good backup in `configs/backup_configs/`
2. Copy to golden config
3. Deploy with replace mode

## Troubleshooting

### Connection Issues
```bash
ping <router_ip>
ssh admin@<router_ip>
ansible routers -m ping -vvv
```

### Authentication Issues
```bash
# Use interactive auth
ansible-playbook playbooks/cisco_gather_facts.yml --ask-pass --ask-become-pass

# Check credentials
cat group_vars/routers.yml
```

### Timeout Issues
Edit `group_vars/routers.yml`:
```yaml
command_timeout: 60
config_timeout: 120
```

## Important Notes

⚠️ **Always backup before changes**
⚠️ **Test in check mode first** 
⚠️ **Deploy to one router initially**
⚠️ **Use Ansible Vault for passwords**
⚠️ **Verify connectivity after deployment**

## Documentation
- Full Guide: `docs/CISCO_GUIDE.md`
- Setup Instructions: `docs/SETUP.md`
- Exercises: `docs/EXERCISES.md`
