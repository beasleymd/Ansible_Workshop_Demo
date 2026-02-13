# Golden Configuration Files

This directory contains the golden (reference) configuration files for the Cisco C8000V routers.

## Router Inventory

| Hostname | Model | IOS Version | Config File |
|----------|-------|-------------|-------------|
| rtr1 | Cisco C8000V | 17.14.01a | rtr1_golden_config.txt |
| rtr2 | Cisco C8000V | 17.14.01a | rtr2_golden_config.txt |
| rtr3 | Cisco C8000V | 17.14.01a | rtr3_golden_config.txt |
| rtr4 | Cisco C8000V | 17.14.01a | rtr4_golden_config.txt |

## Hardware Details

- **Model**: Cisco C8000V (VXE)
- **IOS Software**: Cisco IOS XE Software, Version 17.14.01a
- **Software**: Virtual XE Software (X86_64_LINUX_IOSD-UNIVERSALK9-M), Version 17.14.1a
- **Processor**: cisco C8000V (VXE) processor (revision VXE)
- **Memory**: 1955292K/3075K bytes
- **Interfaces**: 1 Gigabit Ethernet interface
- **Storage**: 11526144K bytes virtual hard disk at bootflash

## Usage

These golden configuration files serve as the authoritative reference configurations for each router. They can be used for:

1. **Configuration Deployment**: Deploy the golden config to routers
2. **Configuration Validation**: Compare running configs against golden configs
3. **Configuration Backup**: Reference for rollback operations
4. **Compliance Checking**: Ensure routers meet configuration standards

## File Format

Configuration files should be in standard Cisco IOS format (.txt or .cfg).

## Ansible Integration

These files are referenced in Ansible playbooks for automated configuration management:
- `playbooks/cisco_deploy_golden_config.yml` - Deploy golden configs
- `playbooks/cisco_backup_config.yml` - Backup current configs
- `playbooks/cisco_validate_config.yml` - Validate against golden configs
