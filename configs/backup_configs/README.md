# Configuration Backups

This directory stores backup copies of router configurations.

## Backup Strategy

Backups are automatically created using Ansible playbooks and include:
- Timestamp in filename
- Router hostname
- Running configuration

## Backup Naming Convention

```
{hostname}_backup_{timestamp}.txt
```

Example: `rtr1_backup_2026-02-13_1430.txt`

## Retention

- Keep backups for compliance and rollback purposes
- Configure retention policy based on organizational requirements
- Recommend keeping at minimum:
  - Daily backups for the last 7 days
  - Weekly backups for the last month
  - Monthly backups for the last year

## Ansible Integration

Backups are created using:
```bash
ansible-playbook playbooks/cisco_backup_config.yml
```

This will connect to all routers in the [routers] inventory group and save their running configurations.
